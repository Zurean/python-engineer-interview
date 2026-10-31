---
type: theory
title: Задачи и структурная конкурентность в asyncio
parent: "[asyncio](asyncio.md)"
topic: asyncio
order: 2
status: review
tags:
  - python
  - asyncio
updated: 2026-10-31
---

# Задачи и структурная конкурентность в asyncio

Родительская тема: [asyncio](asyncio.md).

## Результаты обучения

После этой части ты сможешь:

- выбирать между обычным `await`, `create_task()`, `gather()` и `TaskGroup`;
- объяснить, кому принадлежит созданная задача и когда заканчивается её жизненный цикл;
- предсказать поведение группы при ошибке дочерней задачи;
- обработать `ExceptionGroup` через `except*`;
- распознать «осиротевшую» фоновую задачу и утечку исключения.

## Главный вопрос: кто владеет задачей

Создание задачи отделяет её жизненный цикл от текущего стека вызовов. Поэтому у каждого `create_task()` должен быть ответ на три вопроса:

1. кто хранит сильную ссылку на задачу;
2. кто дождётся результата или извлечёт исключение;
3. кто отменит её при завершении родительской операции.

Если ответы неочевидны, задача похожа на ручное управление ресурсом без `finally`: happy path работает, а ошибки и shutdown становятся недетерминированными.

**Структурная конкурентность** ограничивает дочерние задачи лексической областью. Родитель не выходит из этой области, пока дети не завершены; ошибка одного ребёнка приводит группу в согласованное состояние. В стандартной библиотеке Python 3.11+ эту модель даёт `TaskGroup`.

## Четыре способа организовать выполнение

### Последовательный `await`

Используй, когда следующая операция зависит от результата предыдущей или конкурентность не даёт пользы.

```python
profile = await load_profile(user_id)
orders = await load_orders(profile.account_id)
```

### `TaskGroup`: предпочтительный владелец связанных задач

```python
import asyncio


async def fetch(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return name.upper()


async def main() -> None:
    async with asyncio.TaskGroup() as group:
        user_task = group.create_task(fetch("user", 0.2))
        orders_task = group.create_task(fetch("orders", 0.1))

    print(user_task.result(), orders_task.result())


asyncio.run(main())
```

Выход из `async with` не произойдёт, пока обе задачи не закончатся. После успешного выхода их результаты доступны синхронно через `.result()`.

Если одна задача падает исключением, отличным от `CancelledError`, группа отменяет незавершённых соседей, дожидается их cleanup и затем поднимает `ExceptionGroup`. Это fail-fast-поведение не означает мгновенного возврата: корректное завершение отменённых задач занимает время.

### `gather`: собрать результаты с другой семантикой ошибок

`await asyncio.gather(a(), b())` сохраняет порядок результатов согласно порядку аргументов. По умолчанию первое возникшее исключение сразу передаётся ожидающему коду, но другие awaitable **не отменяются автоматически** и продолжают работу.

`return_exceptions=True` превращает исключения в элементы списка. Это полезно только когда частичный успех — часть контракта и вызывающий код явно классифицирует каждый результат. Иначе ошибки легко принять за данные.

```python
results = await asyncio.gather(
    fetch_optional("avatar"),
    fetch_optional("recommendations"),
    return_exceptions=True,
)

for result in results:
    if isinstance(result, Exception):
        log_optional_failure(result)
```

`gather` не «хуже» TaskGroup: это другой контракт. TaskGroup подходит, когда дети образуют единую операцию и должны иметь общий lifecycle. `gather(return_exceptions=True)` подходит для независимых операций, где допускается собрать успехи и ошибки.

### Свободный `create_task()`: только с явным владельцем

Для действительно долгоживущих задач храни ссылки и извлекай исключения:

```python
import asyncio
from collections.abc import Coroutine


background_tasks: set[asyncio.Task[None]] = set()


def spawn(coro: Coroutine[object, object, None]) -> None:
    task = asyncio.create_task(coro)
    background_tasks.add(task)
    task.add_done_callback(background_tasks.discard)
```

Даже этот шаблон не определяет политику shutdown и логирования ошибок. В приложении лучше иметь компонент-владелец с `TaskGroup`, lifespan или отдельным supervisor, а не глобальный fire-and-forget.

## Как TaskGroup сообщает несколько ошибок

Несколько дочерних задач могут почти одновременно упасть до обработки отмены. Потерять все ошибки кроме одной — плохая диагностика, поэтому TaskGroup поднимает `ExceptionGroup`. Python 3.11 добавил `except*`, который выбирает из группы все подходящие листовые исключения.

```python
import asyncio


async def fail(error: Exception) -> None:
    await asyncio.sleep(0)
    raise error


async def main() -> None:
    try:
        async with asyncio.TaskGroup() as group:
            group.create_task(fail(ValueError("bad payload")))
            group.create_task(fail(LookupError("missing record")))
    except* ValueError as group:
        print("validation:", len(group.exceptions))
    except* LookupError as group:
        print("lookup:", len(group.exceptions))


asyncio.run(main())
```

Обрабатывать ошибку в самой дочерней операции обычно удобнее: там ещё есть контекст запроса, URL и идентификаторы. `except*` на границе группы нужен для общей политики — логирования, преобразования класса ошибок или метрик.

## Типичные ошибки

### «Создал задачу и забыл»

```python
asyncio.create_task(write_audit_event(event))
return response
```

Запрос завершился, а задача осталась без явного владельца. При shutdown она может быть отменена; её исключение никто не прочитает; повторный запрос нельзя связать с результатом. Для обязательной записи нужно дождаться результата или передать событие надёжной очереди. Для best-effort работы нужен явный lifecycle и наблюдаемость.

### Ожидать, что `gather` отменит соседей при первой ошибке

По умолчанию этого не происходит. Если операция атомарна на уровне бизнес-смысла, используй TaskGroup или явно реализуй отмену и ожидание всех созданных задач.

### Читать `.result()` слишком рано

До завершения Task метод поднимает `InvalidStateError`. В TaskGroup читай результаты после выхода из блока либо используй `await task`, если тебе действительно нужен результат раньше.

### Создать тысячи задач и считать это ограничением нагрузки

Task — лёгкий объект, но каждая задача удерживает данные и может открыть соединение. `gather(*(fetch(x) for x in items))` планирует всю коллекцию. Ограничение нагрузки — отдельная задача; см. [asyncio-backend-patterns-practice](asyncio-backend-patterns-practice.md).

## Самопроверка

1. Почему `TaskGroup` называется структурной конкурентностью?
2. Что происходит с задачами B и C, если A падает внутри TaskGroup?
3. Что по умолчанию происходит с B и C при ошибке A внутри `gather`?
4. Когда `return_exceptions=True` является разумным контрактом?
5. Какие три обязанности появляются у владельца свободной фоновой задачи?

<details>
<summary>Ориентиры ответов</summary>

1. Жизненный цикл детей ограничен блоком, а родитель гарантированно ждёт их завершения.
2. Незавершённые соседи отменяются, группа ждёт cleanup и поднимает группу ошибок.
3. Первое исключение передаётся вызывающему коду, соседи автоматически не отменяются.
4. Когда частичные результаты допустимы и каждый элемент будет явно классифицирован как значение или ошибка.
5. Хранить ссылку, наблюдать результат/ошибку, отменить и дождаться на shutdown.

</details>

## Краткое резюме

Конкурентность — это не только запуск, но и владение. Последовательный `await` выражает зависимость, TaskGroup — общую область жизни и fail-fast, `gather` — агрегацию результатов с настраиваемой политикой ошибок, а свободный Task требует отдельного supervisor. Предпочитай конструкцию, чья семантика отказа совпадает с бизнес-операцией.

## Ресурсы

- [Task groups — Python 3.14](https://docs.python.org/3.14/library/asyncio-task.html#task-groups) — точная семантика выхода, отказов и вложенных групп.
- [Running tasks concurrently — Python 3.14](https://docs.python.org/3.14/library/asyncio-task.html#running-tasks-concurrently) — поведение `gather` и сравнение с TaskGroup.
- [PEP 654 — Exception Groups and `except*`](https://peps.python.org/pep-0654/) — мотивация и модель обработки конкурентных ошибок.

---

Предыдущая часть: [asyncio-mental-model](asyncio-mental-model.md) · Следующая часть: [asyncio-cancellation-timeouts](asyncio-cancellation-timeouts.md).
