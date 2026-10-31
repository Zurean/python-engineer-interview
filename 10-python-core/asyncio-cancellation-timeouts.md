---
type: theory
title: Отмена и таймауты в asyncio
parent: "[[asyncio]]"
topic: asyncio
order: 3
status: review
tags:
  - python
  - asyncio
updated: 2026-10-31
---

# Отмена и таймауты в asyncio

Родительская тема: [[asyncio]].

## Результаты обучения

После этой части ты сможешь:

- объяснить отмену как запрос, а не мгновенное убийство задачи;
- писать cleanup, который сохраняет семантику `CancelledError`;
- различать `asyncio.timeout()` и `wait_for()`;
- использовать deadline вместо цепочки несвязанных локальных таймаутов;
- объяснить границы `shield()` и диагностировать проглоченную отмену.

## Отмена — протокол кооперации

`task.cancel()` помечает задачу для отмены. При следующей возможности event loop внедряет в неё `asyncio.CancelledError`, обычно на ближайшем приостанавливающем `await`. Это запрос: задача может ещё выполнять cleanup, а код без точек приостановки вообще не увидит отмену, пока не отдаст управление.

`CancelledError` наследуется напрямую от `BaseException`, а не от `Exception`. Обычный `except Exception` его не ловит — это полезная защита от случайного проглатывания управляющего сигнала.

```python
import asyncio


async def worker() -> None:
    resource = "connection"
    try:
        while True:
            print("working")
            await asyncio.sleep(0.2)
    finally:
        print("closing", resource)


async def main() -> None:
    task = asyncio.create_task(worker())
    await asyncio.sleep(0.35)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("worker cancelled")


asyncio.run(main())
```

Владелец не только вызывает `cancel()`, но и `await task`: так он дожидается `finally` и наблюдает финальное состояние.

## Почему нельзя проглатывать `CancelledError`

TaskGroup и `asyncio.timeout()` сами используют отмену как внутренний механизм. Если низкоуровневая корутина ловит `CancelledError` и продолжает работу, внешний управляющий контекст считает, что запрос отмены был обработан, а фактическая работа не остановилась.

Антипример:

```python
async def broken_worker() -> None:
    try:
        await asyncio.sleep(60)
    except asyncio.CancelledError:
        print("ignored")  # функция успешно возвращается: отмена потеряна
```

Правильный шаблон, если нужен асинхронный cleanup:

```python
async def worker() -> None:
    try:
        await serve()
    except asyncio.CancelledError:
        await flush_metrics()
        raise
```

Чаще достаточно `try/finally`: исключение автоматически продолжит распространение после cleanup. Подавление отмены — редкий осознанный контракт; документация требует также снять состояние отмены через `uncancel()`. В прикладном коде это почти всегда сигнал пересмотреть дизайн.

## Таймаут — ограниченный бюджет ожидания

Таймаут нужен не только для пользовательского комфорта. Он ограничивает удержание соединений, памяти и слотов конкурентности. Без таймаутов зависшая зависимость постепенно исчерпывает ресурсы сервиса.

### `asyncio.timeout()`: ограничить блок текущей задачи

Python 3.11+ предоставляет асинхронный контекстный менеджер:

```python
import asyncio


async def main() -> None:
    try:
        async with asyncio.timeout(0.1):
            await asyncio.sleep(1)
    except TimeoutError:
        print("deadline exceeded")


asyncio.run(main())
```

Внутри блока текущая задача получает `CancelledError`. Контекстный менеджер перехватывает его и снаружи преобразует в встроенный `TimeoutError`. Поэтому `TimeoutError` ловят **за пределами** блока.

Контексты timeout можно вкладывать. Объект timeout также поддерживает абсолютный deadline через `asyncio.timeout_at()`, проверку `expired()` и перенос срока `reschedule()`.

### `wait_for()`: ограничить один awaitable

```python
result = await asyncio.wait_for(fetch(), timeout=0.5)
```

Если `fetch()` передана как корутина, `wait_for` планирует её как Task. При истечении срока она отменяется, и `wait_for` ждёт завершения отмены. Поэтому реальное время возврата может превысить 0.5 секунды, если cleanup долгий.

Практическое правило:

- `asyncio.timeout()` — бюджет для составной операции или блока с несколькими `await`;
- `wait_for()` — локальная граница вокруг одного awaitable, особенно в коде, который должен поддерживать Python до 3.11.

## Deadline лучше суммы таймаутов

Рассмотрим HTTP-запрос с бюджетом 800 мс: прочитать пользователя, рекомендации и сформировать ответ. Если каждой операции дать таймаут 800 мс, суммарная задержка может стать 1.6 секунды и больше.

Передавай один абсолютный deadline или ограничивай всю операцию общим контекстом:

```python
async def build_response(user_id: int) -> dict[str, object]:
    async with asyncio.timeout(0.8):
        user = await load_user(user_id)
        recommendations = await load_recommendations(user)
        return {"user": user, "recommendations": recommendations}
```

В реальном сервисе таймауты нижнего уровня должны укладываться в оставшийся бюджет и оставлять время на сериализацию ответа, cleanup и сетевую передачу.

## Что на самом деле делает `shield()`

`await asyncio.shield(task)` защищает внутреннюю задачу от **отмены ожидающего её caller**. Сам caller всё равно получает `CancelledError`; защищённая задача продолжает работу. Она может быть отменена другим владельцем или изнутри.

Это не универсальный способ «гарантировать выполнение». Нужно хранить сильную ссылку на Task, позднее получить её результат и определить shutdown-политику. Для обязательной бизнес-операции обычно надёжнее транзакция, outbox или внешняя очередь, чем `shield()` внутри процесса.

Обоснованный случай — короткий локальный cleanup, который должен получить небольшой отдельный бюджет. Даже тогда нужно решить, что произойдёт при завершении процесса.

## Graceful shutdown как применение модели отмены

Упрощённый порядок остановки сервиса:

1. перестать принимать новую работу;
2. уведомить долгоживущие задачи или отменить их;
3. дождаться `finally`: закрытия клиентов, возврата соединений, завершения consumers;
4. установить верхний предел на shutdown;
5. залогировать работу, которая не завершилась в срок.

TaskGroup помогает собрать lifecycle связанных workers в одной области. Framework lifespan обычно является владельцем этой области.

## Самопроверка

1. Почему `cancel()` не гарантирует мгновенную остановку?
2. Зачем после `task.cancel()` делать `await task`?
3. Где нужно ловить `TimeoutError` для `asyncio.timeout()`?
4. Почему `wait_for(..., 0.5)` иногда возвращается позже 0.5 секунды?
5. Почему `shield()` не заменяет очередь для обязательной фоновой работы?

### Разбор кода

Что не так и как исправить?

```python
async def poll() -> None:
    while True:
        try:
            await receive_message()
        except BaseException:
            logger.exception("poll failed")
```

<details>
<summary>Ориентиры ответов</summary>

- `except BaseException` ловит `CancelledError`, цикл продолжает работу и мешает shutdown.
- Лови ожидаемые операционные исключения (`OSError`, ошибку клиента и т. п.) отдельно.
- Если отмену нужно залогировать или выполнить cleanup, лови `CancelledError` отдельно и делай `raise`; либо используй `finally`.

</details>

## Краткое резюме

Отмена — управляющее исключение, доставляемое кооперативно. Владелец запрашивает отмену и ждёт завершения, а задача освобождает ресурсы и не скрывает `CancelledError`. Таймаут — это deadline и бюджет ресурса, а не просто число у сетевого вызова. `shield()` меняет путь отмены, но не создаёт надёжность и не снимает обязанность владеть задачей.

## Ресурсы

- [Task cancellation — Python 3.14](https://docs.python.org/3.14/library/asyncio-task.html#task-cancellation) — контракт `cancel()`, cleanup и ограничения подавления отмены.
- [Timeouts — Python 3.14](https://docs.python.org/3.14/library/asyncio-task.html#timeouts) — `timeout`, `timeout_at` и `wait_for`.
- [Shielding from cancellation — Python 3.14](https://docs.python.org/3.14/library/asyncio-task.html#shielding-from-cancellation) — точные границы защиты и требование хранить ссылку на задачу.

---

Предыдущая часть: [[asyncio-tasks-structured-concurrency]] · Следующая часть: [[asyncio-backend-patterns-practice]].
