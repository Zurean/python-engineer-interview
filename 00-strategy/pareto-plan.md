---
type: strategy
title: Парето-план подготовки
updated: 2026-10-29
---

# Парето-план: порядок атаки

Идея: собеседование — это не экзамен по всему стеку, а 4–6 секций с предсказуемыми вопросами. 20 % тем дают 80 % результата. Сначала закрываем их на уровне «уверенно проговорить», потом углубляемся.

## Фаза 0 — упаковка (1–2 вечера, импакт огромный)

Делается до любой технической подготовки, потому что влияет на *каждый* собес.

1. [self-presentation](self-presentation.md) — рассказ о себе на 2 минуты, с честной формулировкой стажа и позиционированием.
2. [stories](stories.md) — две-три истории по STAR из своего опыта (инцидент, AI-кейс, смена стека).
3. [interview-formats](interview-formats.md) — понять, какие секции бывают, чтобы не удивляться.

## Фаза 1 — первый фильтр (2–3 недели)

Темы, которые спрашивают на *каждом* Python-собесе. Провал здесь = отказ после первого этапа.

| Порядок | Тема | Почему первой |
|---|---|---|
| 1 | [asyncio](../10-python-core/asyncio.md) | Главный фильтр для FastAPI-вакансий. 5 из 7 требуют async. Тема компактная, вопросы типовые. |
| 2 | [data-model](../10-python-core/data-model.md) + [generators-context-managers-decorators](../10-python-core/generators-context-managers-decorators.md) | Классика скрининга: мутабельный дефолт, `__slots__`, дескрипторы, генераторы. Готовится за пару вечеров. |
| 3 | [postgres-indexes-explain](../20-databases/postgres-indexes-explain.md) | БД — требование №1 после Python. Индексы и EXPLAIN спрашивают всегда. |
| 4 | [transactions-isolation-locks](../20-databases/transactions-isolation-locks.md) | Уровни изоляции, deadlock, SELECT FOR UPDATE — стабильный набор вопросов. |
| 5 | [gil-threads-processes](../10-python-core/gil-threads-processes.md) | Вопрос про GIL задают в 90 % случаев. Ответ короткий, учится за час. |
| 6 | [fastapi](../30-backend-design/fastapi.md) + [sqlalchemy-alembic](../20-databases/sqlalchemy-alembic.md) | Стек из вакансий. Вопросы про `def` vs `async def`, сессии, N+1. |
| 7 | [pytest](../60-testing/pytest.md) | Спрашивают почти всегда, тема дешёвая. |
| 8 | [docker](../40-infra/docker.md) | Dockerfile, слои, multi-stage, compose — 30 минут, но спрашивают. |

Критерий: пройти [python-live-coding-tasks](../90-live-coding/python-live-coding-tasks.md) без подглядывания.

## Фаза 2 — Middle+/Senior-секции (3–4 недели)

Отличает Middle от Middle+/Senior. У кандидатов с коротким опытом на Python именно здесь обычно самые большие пробелы — и самый большой прирост.

| Порядок | Тема |
|---|---|
| 1 | [sd-framework](../80-system-design/sd-framework.md) — фреймворк ответа на системный дизайн; без него секция провальна независимо от знаний |
| 2 | [resilience-patterns](../30-backend-design/resilience-patterns.md) — таймауты, ретраи, идемпотентность, circuit breaker |
| 3 | [kafka](../50-messaging-caching/kafka.md) — consumer groups, гарантии доставки, ретраи |
| 4 | [redis](../50-messaging-caching/redis.md) — кэш, инвалидация, локи, rate limiting |
| 5 | [sd-building-blocks](../80-system-design/sd-building-blocks.md) + [sd-practice-cases](../80-system-design/sd-practice-cases.md) — 3–4 кейса вслух |
| 6 | [api-design](../30-backend-design/api-design.md) + [architecture-layers](../30-backend-design/architecture-layers.md) |
| 7 | [kubernetes](../40-infra/kubernetes.md) — на уровне пользователя |
| 8 | [integration-testing](../60-testing/integration-testing.md), [ci-cd](../40-infra/ci-cd.md), [observability](../40-infra/observability.md) |
| 9 | [typing](../10-python-core/typing.md), [memory-performance](../10-python-core/memory-performance.md), [tooling](../10-python-core/tooling.md) |

## Фаза 3 — дифференциатор (параллельно с фазой 2)

[_llm-engineering](../70-llm-engineering/_llm-engineering.md): [rag](../70-llm-engineering/rag.md), [tool-calling-structured-output](../70-llm-engineering/tool-calling-structured-output.md), [evals-cost-latency](../70-llm-engineering/evals-cost-latency.md), [ai-assisted-development](../70-llm-engineering/ai-assisted-development.md). Это то, что отличает от других Middle+ кандидатов и открывает AI-позиции. Лучше всего качается через [rag-service-project](../capstone/rag-service-project.md).

## Фаза 4 — гринд (бесконечная)

Разделы «Углубление» во всех темах уровня 1, [_optional](../95-optional/_optional.md), CPython-внутренности, [algorithms-basics](../90-live-coding/algorithms-basics.md) глубже. Сюда — только когда фазы 1–2 закрыты на `confidence: 2`.

## Правило выбора темы на сегодня

1. Есть ли незакрытые темы фазы 1? → берём верхнюю.
2. Нет → фаза 2, верхняя незакрытая.
3. После каждого реального собеса — провалившаяся тема прыгает в начало очереди, независимо от фазы.
4. Если не хочется ничего из списка — делаем [rag-service-project](../capstone/rag-service-project.md): он прокачивает всё сразу.
