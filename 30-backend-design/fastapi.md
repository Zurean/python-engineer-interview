---
type: topic
title: FastAPI
tier: 1
impact: 5
effort: 2
status: todo
confidence: 0
tags: [backend, python, tier1]
updated: 2026-10-29
---

# FastAPI (и ASGI вообще)

> **Зачем на собесе:** основной фреймворк в 5 из 7 вакансий. Вопросы: `def` vs `async def`, Depends, lifespan, валидация, как устроен ASGI/uvicorn. Плюс Django на уровне «понимаю отличия».

## Минимум для собеса

- **ASGI** — async-аналог WSGI: приложение — корутина `app(scope, receive, send)`. Сервер — uvicorn (один процесс) / gunicorn с uvicorn-воркерами (несколько процессов = обход GIL). Starlette — основа FastAPI.
- **`def` vs `async def` эндпоинты:** `async def` выполняется в event loop — любой блокирующий вызов внутри останавливает всех. `def` FastAPI сам гоняет в threadpool (anyio, по умолчанию 40 потоков) — безопасно для sync-библиотек, но ограничено пулом. Тяжёлый CPU в `async def` — блокирует loop; выносить в процессы.
- **Dependency Injection:** `Depends()` — функции/генераторы с `yield` (cleanup после ответа), кэширование в рамках запроса, `Annotated[Session, Depends(get_session)]`, переопределение в тестах через `app.dependency_overrides`.
- **Валидация:** Pydantic-модели для body, `Query/Path/Header` для параметров, `response_model` фильтрует вывод, 422 при ошибке валидации. Граница валидации — здесь.
- **Lifespan:** `@asynccontextmanager async def lifespan(app)` — создать engine, http-клиент, consumer при старте, закрыть при остановке. Не глобалы на импорте.
- **Middleware** (CORS, request id, логирование, таймауты), `BackgroundTasks` (для лёгких задач; для тяжёлых — очередь, см. [[task-queues]]).
- **Ошибки:** `HTTPException`, `exception_handler` для доменных ошибок → единый формат ответа ([[api-design]]).
- **Роутеры** (`APIRouter`), версионирование через префиксы, OpenAPI из коробки.
- **Производительность:** один `httpx.AsyncClient` на приложение, пул БД, `orjson`-ответы, не логировать тела запросов в hot path, настроить количество воркеров ≈ CPU.
- **Django-отличия** (если спросят): монолит с батарейками (ORM, admin, auth, миграции), sync-first, Django ORM vs SQLAlchemy, DRF vs FastAPI; async в Django ограничен ORM.

## Вопросы с собесов

- [ ] Как FastAPI обрабатывает `def` и `async def` эндпоинты? Что будет при тяжёлом CPU в `async def`? → *threadpool vs loop; loop встанет*
- [ ] Что такое ASGI и чем отличается от WSGI? → *async-интерфейс; scope/receive/send; websockets*
- [ ] Как работает `Depends`, что такое dependency с `yield`? → *DI-граф, cleanup, кэш в рамках запроса*
- [ ] Где создаёшь engine/клиенты и почему не на уровне модуля? → *lifespan; тесты, порядок инициализации, форки*
- [ ] Как тестируешь FastAPI-приложение? → *TestClient/httpx ASGITransport, dependency_overrides, см. [[pytest]]*
- [ ] Как сделать единый формат ошибок? → *exception handlers, доменные исключения → HTTP-коды*
- [ ] Сколько воркеров uvicorn ставить и зачем gunicorn? → *≈CPU, процесс-менеджер, обход GIL*
- [ ] Как прокинуть request id через все логи? → *middleware + contextvars*
- [ ] BackgroundTasks vs Celery/очередь — когда что? → *лёгкое и не критичное vs надёжное/тяжёлое*
- [ ] Чем FastAPI отличается от Django/DRF, когда выбрать Django? → *батарейки, admin, монолит*

## Углубление

- Устройство Starlette: routing, middleware stack, `Request`/`Response`, streaming responses, websockets.
- Pydantic v2 в FastAPI: `model_config`, `computed_field`, `TypeAdapter`, производительность сериализации.
- Зависимости с областью видимости приложения (singleton) и альтернативные DI-контейнеры (dishka, punq).
- Graceful shutdown: завершение in-flight запросов, таймауты uvicorn, readiness probe в K8s.
- Rate limiting и защита API (см. [[redis]], [[auth-security]]).
- Litestar/BlackSheep как альтернативы — трейдоффы.

## Ресурсы

- https://fastapi.tiangolo.com/ (Tutorial → Advanced, особенно Dependencies, Lifespan, Concurrency and async/await)
- https://github.com/zhanymkanov/fastapi-best-practices
- ASGI spec: https://asgi.readthedocs.io/

## Связи

[[asyncio]] · [[sqlalchemy-alembic]] · [[typing]] · [[api-design]] · [[pytest]] · [[architecture-layers]]

## Мои заметки

