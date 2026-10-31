---
type: theory
title: Backend-паттерны и практика asyncio
parent: "[[asyncio]]"
topic: asyncio
order: 4
status: review
tags:
  - python
  - asyncio
updated: 2026-10-31
---

# Backend-паттерны и практика asyncio

Родительская тема: [[asyncio]].

## Результаты обучения

После этой части ты сможешь:

- выбрать способ интеграции синхронного I/O и CPU-bound кода;
- ограничить число запросов через `Semaphore` и объяснить предел этого решения;
- построить bounded worker pool с backpressure;
- объяснить, зачем `asyncio.Lock` нужен в одном потоке;
- диагностировать блокировку loop и применить модель к FastAPI-сценарию.

## Неблокирующий путь должен быть неблокирующим целиком

Асинхронный обработчик полезен, только если его зависимости тоже кооперативны. Один синхронный HTTP-клиент, драйвер базы, файловая операция или CPU-тяжёлая сериализация может заблокировать loop.

Порядок выбора:

1. **Нативная async-библиотека** — лучший вариант для сетевого I/O: она интегрируется с loop, отменой и таймаутами.
2. **`asyncio.to_thread()`** — адаптер для блокирующего I/O, когда async-аналога нет или миграция постепенная.
3. **`ProcessPoolExecutor`** — для CPU-bound чистого Python, если стоимость сериализации и запуска оправданна.
4. Отдельный worker/service — для тяжёлых, долгих или требующих надёжного выполнения задач.

### Блокирующий I/O в потоке

```python
import asyncio
from pathlib import Path


def read_config(path: Path) -> str:
    return path.read_text(encoding="utf-8")


async def main() -> None:
    text = await asyncio.to_thread(read_config, Path("pyproject.toml"))
    print(len(text))


asyncio.run(main())
```

`to_thread` не делает функцию неблокирующей — она блокирует другой поток. Отмена ожидающей Task не умеет принудительно остановить уже работающую Python-функцию в потоке. Поэтому функция всё равно должна иметь собственные сетевые таймауты и ограниченный runtime.

Из-за GIL `to_thread` обычно не ускоряет CPU-bound Python-код. Он может быть уместен для C-расширения, которое освобождает GIL, но это свойство нужно проверять по документации библиотеки. Связь с выбором потоков и процессов разобрана в [[gil-threads-processes]].

## Ограничение конкурентности через Semaphore

Пусть нужно запросить много URL, но внешний API разрешает не более 50 одновременных запросов. Семафор ограничивает число задач внутри критической области:

```python
import asyncio
from collections.abc import Awaitable, Callable, Iterable
from typing import TypeVar

T = TypeVar("T")
R = TypeVar("R")


async def map_limited(
    items: Iterable[T],
    limit: int,
    operation: Callable[[T], Awaitable[R]],
) -> list[R]:
    semaphore = asyncio.Semaphore(limit)

    async def run(item: T) -> R:
        async with semaphore:
            return await operation(item)

    return await asyncio.gather(*(run(item) for item in items))
```

Семафор ограничивает активные операции, но пример всё равно создаёт Task для каждого элемента. Для 10 000 коротких URL это может быть приемлемо после измерения; для миллионов элементов — нет: ожидающие задачи удерживают память и входные данные.

Кроме того, семафор ограничивает **concurrency**, но не **rate**. Пять слотов могут выполнить сотни быстрых запросов в секунду. Для лимита «100 запросов в минуту» нужен rate limiter/token bucket и корректная обработка `Retry-After`.

## Bounded Queue и backpressure

Backpressure означает: если consumers не успевают, producer вынужден замедлиться, а не накапливает неограниченную очередь в памяти. `asyncio.Queue(maxsize=N)` приостанавливает `put()`, когда буфер заполнен.

```python
import asyncio


async def worker(name: str, queue: asyncio.Queue[str]) -> None:
    while True:
        item = await queue.get()
        try:
            await asyncio.sleep(0.05)  # вместо реального I/O
            print(name, item)
        finally:
            queue.task_done()


async def main() -> None:
    queue: asyncio.Queue[str] = asyncio.Queue(maxsize=10)

    async with asyncio.TaskGroup() as group:
        workers = [group.create_task(worker(f"w{i}", queue)) for i in range(3)]

        for item in (f"job-{i}" for i in range(20)):
            await queue.put(item)  # ждёт, если consumers отстают

        await queue.join()
        for task in workers:
            task.cancel()


asyncio.run(main())
```

`task_done()` должен вызываться ровно один раз для каждого успешного `get()`, поэтому он находится в `finally`. После `queue.join()` незавершённой работы нет, workers можно отменить; TaskGroup дождётся их завершения.

В Python 3.13 появился `Queue.shutdown()`, который формализует остановку producers/consumers. В учебном примере используется переносимый для 3.11+ шаблон с отменой workers.

## Зачем Lock, если поток один

Между двумя участками кода без `await` другая asyncio-задача в том же loop не вклинится. Но инвариант часто пересекает `await`:

```python
cache: dict[str, str] = {}
lock = asyncio.Lock()


async def get_token(key: str) -> str:
    async with lock:
        if key not in cache:
            cache[key] = await request_token(key)
        return cache[key]
```

Без Lock две задачи могут обе проверить отсутствие ключа, приостановиться в `request_token()` и выполнить двойную инициализацию. `asyncio.Lock` защищает инвариант между await-точками, но не предназначен для синхронизации OS threads.

Держать Lock во время медленного I/O может чрезмерно сериализовать разные ключи. Более зрелый дизайн хранит отдельную Task/Future на ключ (single-flight) или использует per-key locks. Сначала сформулируй инвариант, затем выбирай область блокировки.

## Backend-сценарий: агрегирующий endpoint

Endpoint должен параллельно получить профиль и заказы, ответить за 700 мс и не перегрузить downstream. Архитектурные решения:

- один переиспользуемый async HTTP/DB client создаётся в lifespan, а не на каждый запрос;
- TaskGroup выражает, что профиль и заказы принадлежат одному запросу;
- общий `asyncio.timeout(0.7)` ограничивает бюджет операции;
- клиентские connection limits и/или semaphore ограничивают запросы к зависимости;
- отмена входящего запроса распространяется к дочерним операциям;
- ошибки преобразуются в HTTP-ответ на границе, а cleanup остаётся в клиентах и контекстных менеджерах.

Псевдокод без привязки к фреймворку:

```python
async def build_dashboard(user_id: int) -> dict[str, object]:
    async with asyncio.timeout(0.7):
        async with asyncio.TaskGroup() as group:
            profile_task = group.create_task(load_profile(user_id))
            orders_task = group.create_task(load_orders(user_id))

    return {
        "profile": profile_task.result(),
        "orders": orders_task.result(),
    }
```

Если заказы опциональны, контракт меняется: ошибку этой операции можно обработать внутри дочерней корутины и вернуть fallback. Не меняй TaskGroup на `gather(return_exceptions=True)` механически — сначала реши, является ли частичный успех частью API.

## Диагностика проблем event loop

Признаки блокировки loop: одновременно растут latency всех запросов, lag таймеров и heartbeat; CPU одного потока может быть высоким или поток может ждать внутри синхронной библиотеки.

Минимальный набор:

- запускать тестовый/стейджинг-код с `asyncio.run(..., debug=True)` или `PYTHONASYNCIODEBUG=1`;
- следить за сообщениями о slow callbacks; порог задаёт `loop.slow_callback_duration`;
- включать предупреждения о неосвобождённых ресурсах;
- снимать stack traces и профили, а не угадывать по наличию `async def`;
- проверять метрики pool exhaustion, queue depth, in-flight requests и event-loop lag.

Debug mode имеет накладные расходы и не заменяет production observability. Связанные практики: [[observability]] и [[resilience-patterns]].

## Типичные ошибки — короткий чеклист

- синхронный клиент внутри `async def`;
- новый HTTP/DB client на каждый вызов вместо пула с lifecycle;
- тысячи/миллионы задач без ограничения входного потока;
- semaphore ошибочно принят за rate limiter;
- неограниченная Queue скрывает перегрузку до OOM;
- Lock удерживается на слишком широкой области;
- `create_task()` используется как гарантия доставки фоновой работы;
- CPU-bound работа отправлена в `to_thread` без проверки GIL и измерений.

## Практические задания

### 1. Базовое: измерить конкурентность

Напиши `fetch(name, delay)`, которая ждёт через `asyncio.sleep` и возвращает имя. Сравни время трёх последовательных вызовов с TaskGroup.

Критерии готовности:

- результат обоих вариантов одинаков;
- последовательное время близко к сумме задержек, конкурентное — к максимуму;
- ты можешь объяснить, почему это не доказывает параллельное выполнение Python-кода.

### 2. Прикладное: ограниченный агрегатор

Реализуй обработку 100 искусственных запросов с максимум 10 активными операциями, индивидуальным таймаутом и итоговой статистикой успехов/таймаутов/ошибок. Не используй внешнюю сеть: моделируй задержки и отказы.

Критерии готовности:

- измеренный максимум in-flight не превышает 10;
- одна ошибка не теряется и выбранная политика partial failure объяснена;
- после завершения `asyncio.all_tasks()` не показывает твоих незавершённых workers (кроме текущей main-задачи).

### 3. Диагностическое: найти блокировку loop

Возьми программу с heartbeat каждые 50 мс и добавь worker с `time.sleep(0.5)`. Зафиксируй lag, включи debug mode, затем исправь программу двумя способами: `asyncio.sleep` для моделируемого ожидания и `to_thread` для настоящей блокирующей функции.

Критерии готовности:

- до исправления heartbeat имеет заметную паузу;
- после исправления loop остаётся отзывчивым;
- ты можешь назвать ограничение отмены функции, уже работающей в потоке.

## Итоговая самопроверка для собеседования

Попробуй ответить вслух без заметок:

1. Что такое event loop и где происходит переключение задач?
2. Чем coroutine object отличается от Task?
3. Когда выбрать TaskGroup, а когда `gather(return_exceptions=True)`?
4. Как распространяется отмена и почему нельзя глотать `CancelledError`?
5. Как обработать 10 000 URL с лимитом 50 одновременных запросов и что изменится для 10 миллионов URL?
6. Что произойдёт при вызове синхронного HTTP-клиента из async endpoint?
7. Как запустить CPU-bound функцию, не блокируя loop?
8. Зачем нужен `asyncio.Lock` в одном потоке?
9. Чем concurrency limit отличается от rate limit?
10. Как организовать graceful shutdown долгоживущих workers?

Эти вопросы покрывают исходный чеклист в [[asyncio#Вопросы с собесов]]. Чекбоксы отмечаются только после того, как ответы воспроизводятся без подсказки.

## Краткое резюме модуля

`asyncio` эффективно, когда задачи большую часть времени ждут совместимое неблокирующее I/O. Event loop исполняет одну задачу за раз до фактической приостановки. Структурная конкурентность связывает lifecycle детей с родителем, отмена и таймауты задают границы работы, а semaphore и bounded queue защищают зависимости и память от перегрузки. `async` не исправляет блокирующие библиотеки, CPU-bound код и отсутствие эксплуатационных лимитов — эти решения остаются ответственностью архитектуры.

## Ресурсы и дальнейшее углубление

- [Developing with asyncio — Python 3.14](https://docs.python.org/3.14/library/asyncio-dev.html) — debug mode, блокирующий код, потоки и типичные ловушки.
- [Synchronization primitives — Python 3.14](https://docs.python.org/3.14/library/asyncio-sync.html) — точные контракты Lock и Semaphore.
- [Queues — Python 3.14](https://docs.python.org/3.14/library/asyncio-queue.html) — bounded queue, `join`/`task_done` и `shutdown` с Python 3.13.
- [Running in Threads — Python 3.14](https://docs.python.org/3.14/library/asyncio-task.html#running-in-threads) — поведение `to_thread` и замечание о GIL.
- [Executing code in thread or process pools — Python 3.14](https://docs.python.org/3.14/library/asyncio-eventloop.html#executing-code-in-thread-or-process-pools) — выбор executor для блокирующей и CPU-bound работы.

---

Предыдущая часть: [[asyncio-cancellation-timeouts]] · Вернуться к карточке: [[asyncio]].
