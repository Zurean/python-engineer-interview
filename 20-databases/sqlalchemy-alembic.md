---
type: topic
title: SQLAlchemy 2.0 и Alembic
tier: 1
impact: 4
effort: 2
status: todo
confidence: 0
tags: [db, python, tier1]
updated: 2026-10-29
---

# SQLAlchemy 2.0 (async) и Alembic

> **Зачем на собесе:** стек 4 из 7 вакансий. Вопросы практические: сессия на запрос, транзакции, N+1, async-движок, миграции. Это «проверка, что реально писал», а не теория.

## Минимум для собеса

**SQLAlchemy 2.0**
- Два слоя: **Core** (SQL-выражения, `select()`, `insert()`) и **ORM** (`Mapped[]`, `mapped_column`, `DeclarativeBase`). Стиль 2.0: `select(User).where(...)`, `session.execute(stmt).scalars()`, никакого `query()`.
- **Engine** — один на приложение, держит пул соединений (`pool_size`, `max_overflow`, `pool_pre_ping`, `pool_recycle`). **Session** — единица работы, короткоживущая: одна на HTTP-запрос / задачу.
- **Async:** `create_async_engine` (драйвер `asyncpg` или `psycopg[async]`), `AsyncSession`, `async_sessionmaker`, `await session.execute()`. Lazy load в async невозможен без `await` → `selectinload`/`joinedload` или `AsyncAttrs`.
- **Сессия на запрос в FastAPI:** dependency с `async with session_maker() as session: yield session`; коммит явно в сервисе или в dependency (`session.begin()`), роллбек на исключении.
- **Транзакция:** `async with session.begin():` — commit при выходе, rollback при ошибке. `expire_on_commit=False` для async (иначе доступ к атрибутам после commit = implicit IO).
- **Загрузка связей:** `selectinload` (второй запрос `IN`, хорошо для коллекций), `joinedload` (JOIN, для one-to-one/many-to-one), `lazy='raise'` чтобы ловить N+1 в тестах. Подробнее — [query-optimization-n-plus-1](query-optimization-n-plus-1.md).
- **Identity map**, `flush` vs `commit`, `refresh`, `merge`; unit of work.
- **Bulk:** `session.execute(insert(T), [dicts])`, `insert().on_conflict_do_update()` (upsert, из `sqlalchemy.dialects.postgresql`).

**Alembic**
- `alembic init`, `env.py` (подключение metadata, async-режим), `alembic revision --autogenerate -m "..."`, `alembic upgrade head`, `downgrade -1`.
- Autogenerate не видит всё (переименования трактует как drop+add, не видит изменения server_default, некоторые типы) → ревьюить миграции руками всегда.
- Миграции в CI/деплое: запускать до старта новой версии (init-container/job), один процесс, не из каждого пода.
- Ветвление ревизий (`heads`, `merge`), данные-миграции отдельно от схемы.
- Безопасные миграции без даунтайма — [zero-downtime-migrations](zero-downtime-migrations.md).

## Вопросы с собесов

- [ ] Как управляешь сессией и транзакцией в async SQLAlchemy на уровне запроса? → *session per request через Depends, begin/commit, expire_on_commit=False*
- [ ] Что такое N+1 и как его ловить и лечить в SQLAlchemy? → *selectinload/joinedload, lazy='raise', логирование SQL*
- [ ] Engine vs Session — сколько их должно быть? → *один engine, сессия на единицу работы*
- [ ] Чем `flush` отличается от `commit`? → *отправка SQL без фиксации vs фиксация транзакции*
- [ ] Почему в async нельзя просто обратиться к `user.posts`? → *lazy load = IO без await; eager load или awaitable_attrs*
- [ ] Как настроить пул соединений и что такое `pool_pre_ping`? → *размер, overflow, проверка живости*
- [ ] Что не умеет autogenerate в Alembic? → *rename, server_default, часть типов*
- [ ] Как делаешь upsert? → *insert ... on_conflict_do_update*
- [ ] Где запускаешь миграции при деплое в K8s? → *job/init-container, один раз, до раскатки*
- [ ] Raw SQL vs ORM — когда что? → *сложная аналитика/окна/CTE — Core или text; CRUD — ORM*

## Углубление

- Устройство Unit of Work, identity map и `autoflush` — откуда «неожиданные» SELECT.
- `with_for_update()`, `skip_locked` в ORM, `execution_options(isolation_level=...)`.
- Hybrid properties, column_property, polymorphic mapping.
- Паттерн Repository + Unit of Work поверх SQLAlchemy и когда это лишняя абстракция.
- Alembic: кастомные операции, `include_object`, несколько схем/баз, тестирование миграций up/down в CI.
- Альтернативы: SQLModel, Tortoise, raw asyncpg — трейдоффы.

## Ресурсы

- https://docs.sqlalchemy.org/en/20/orm/queryguide/ и «What's New in 2.0»
- https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html
- https://alembic.sqlalchemy.org/en/latest/autogenerate.html (раздел «What does Autogenerate Detect»)

## Связи

[fastapi](../30-backend-design/fastapi.md) · [query-optimization-n-plus-1](query-optimization-n-plus-1.md) · [zero-downtime-migrations](zero-downtime-migrations.md) · [transactions-isolation-locks](transactions-isolation-locks.md) · [integration-testing](../60-testing/integration-testing.md)

## Мои заметки

