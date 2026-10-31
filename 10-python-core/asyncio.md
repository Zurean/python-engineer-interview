---
type: topic
title: Asyncio
tier: 1
impact: 5
effort: 3
status: todo
confidence: 0
tags: [python, concurrency, tier1]
updated: 2026-10-31
---

# Asyncio

> **Зачем на собесе:** главный фильтр для FastAPI-вакансий (5 из 7). Спрашивают на техскрининге и в live coding. Типовая ловушка — «блокирующий вызов в async-обработчике».

## Минимум для собеса

- **Event loop** — один поток, цикл выполняет корутины по очереди; переключение только на `await`. Поэтому один блокирующий вызов (`time.sleep`, `requests.get`, тяжёлый CPU) останавливает *весь* сервис.
- **Корутина vs задача:** `async def` возвращает корутину — она ничего не делает, пока её не `await`-нут или не обернут в `asyncio.create_task()`. Задача планируется в loop сразу.
- **Конкурентно запустить несколько:** `asyncio.gather(*coros)` или `async with asyncio.TaskGroup() as tg: tg.create_task(...)` (3.11+). TaskGroup — структурная конкурентность: при ошибке в одной задаче остальные отменяются, исключения собираются в `ExceptionGroup`. `gather` по умолчанию при ошибке *не* отменяет остальные.
- **Отмена:** `task.cancel()` бросает `CancelledError` внутри корутины на ближайшем `await`. Её нельзя глотать — иначе ломаются таймауты и TaskGroup. Если ловишь для cleanup — перебрасывай дальше.
- **Таймауты:** `async with asyncio.timeout(5):` (3.11+) или `asyncio.wait_for()`.
- **Блокирующий код в async:** выносить в `await asyncio.to_thread(func)` (обычно блокирующий I/O) или в `ProcessPoolExecutor` через `loop.run_in_executor` (CPU-bound чистого Python).
- **Ограничение параллелизма:** `asyncio.Semaphore(50)` + `gather`, либо пул воркеров через `asyncio.Queue`.
- **Async-библиотеки:** `httpx.AsyncClient`, `aiohttp`, `asyncpg`, SQLAlchemy async, `aiokafka`, `redis.asyncio`. Синхронный `requests` или `psycopg2` в async-коде — классическая ошибка.
- **Когда asyncio не нужен:** CPU-bound задачи, простые скрипты, код без I/O. Async не ускоряет вычисления, он экономит потоки на ожидании I/O.

## Учебный маршрут

Модуль рассчитан на Python 3.11+ и использует `TaskGroup` и `asyncio.timeout()`. Его удобно пройти за две сессии: сначала части 1–2, затем части 3–4.

1. [[asyncio-mental-model]] — построить модель event loop, корутины, задачи и точки кооперативного переключения.
2. [[asyncio-tasks-structured-concurrency]] — научиться выбирать между последовательным `await`, `gather` и `TaskGroup`, понимать владение задачами и распространение ошибок.
3. [[asyncio-cancellation-timeouts]] — рассматривать отмену как часть протокола управления, корректно реализовывать cleanup и таймауты.
4. [[asyncio-backend-patterns-practice]] — не блокировать loop, ограничивать нагрузку, применять backpressure и закрепить материал на backend-сценарии.

## Вопросы с собесов

- [ ] Что такое event loop и почему asyncio работает в одном потоке? → *кооперативная многозадачность, переключение на await*
- [ ] В async-обработчике FastAPI вызывается синхронный HTTP-клиент. Что произойдёт и как исправить? → *весь loop встанет на время запроса, все остальные запросы ждут; заменить на httpx.AsyncClient или to_thread*
- [ ] Чем `gather` отличается от `TaskGroup`? → *TaskGroup отменяет остальных при ошибке, ExceptionGroup; gather — нет, return_exceptions*
- [ ] Как работает отмена задачи и почему нельзя глотать `CancelledError`? → *бросается на await; глотание ломает timeout/TaskGroup; ловить, cleanup, re-raise*
- [ ] Обработать 10 000 URL с лимитом 50 одновременных. Как? → *Semaphore(50) + gather, или Queue с 50 воркерами; AsyncClient один на всех*
- [ ] Чем корутина отличается от задачи (Task)? → *корутина — объект, не выполняется без await; Task — обёртка, запланированная в loop*
- [ ] Что будет, если забыть `await`? → *корутина создана, не выполнена, RuntimeWarning: coroutine was never awaited*
- [ ] Как запустить CPU-тяжёлую функцию, не блокируя loop? → *run_in_executor с ProcessPoolExecutor; to_thread не поможет из-за GIL*
- [ ] Что такое `asyncio.Lock` и зачем он, если один поток? → *защита инварианта между await-точками, например, чтобы не инициализировать ресурс дважды*
- [ ] Как сделать graceful shutdown async-сервиса? → *сигнал → отменить задачи → дождаться cleanup через finally; lifespan в FastAPI*

## Углубление

- Как устроен loop внутри: `selectors`, `call_soon`, `_ready` очередь, `Future` и callbacks. `uvloop` как замена на libuv.
- `asyncio.shield()`, `asyncio.wait()` с `FIRST_COMPLETED`, `as_completed`.
- Контекстные переменные `contextvars` и почему они нужны в async (request id, trace id).
- Async-генераторы, `async for`, `aclose()`, `asynccontextmanager`.
- Backpressure: bounded `asyncio.Queue`, почему unbounded очереди = утечка памяти под нагрузкой.
- Отладка: `PYTHONASYNCIODEBUG=1`, обнаружение slow callbacks, `asyncio.all_tasks()`.
- Разница с trio/anyio и откуда взялась идея структурной конкурентности.

> Запросить углубление: «углуби *Углубление* в `asyncio` про внутренности event loop / contextvars / backpressure».

## Ресурсы

- Документация: https://docs.python.org/3/library/asyncio.html (разделы Coroutines and Tasks, Synchronization)
- Łukasz Langa, серия видео «asyncio internals» на YouTube
- Статья «Structured concurrency» (Nathaniel Smith, trio) — откуда TaskGroup

## Связи

[[gil-threads-processes]] · [[fastapi]] · [[sqlalchemy-alembic]] · [[resilience-patterns]] · [[python-live-coding-tasks]]

## Мои заметки

