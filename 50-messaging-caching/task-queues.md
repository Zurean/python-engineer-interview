---
type: topic
title: Очереди задач
tier: 2
impact: 3
effort: 1
status: todo
confidence: 0
tags: [messaging, python, tier2]
updated: 2026-10-29
---

# Очереди фоновых задач: Celery, Taskiq, arq

> **Зачем на собесе:** «как делаешь фоновые задачи / отложенные / периодические» — почти в каждом backend-собесе. Нужно отличать task-очередь от брокера событий и знать, как не потерять и не задублировать задачу.

## Минимум для собеса

- **Зачем:** вынести из HTTP-запроса долгое/ненадёжное (письма, отчёты, обработка файлов, вызовы внешних API, индексация). Ответ пользователю быстро, работа — воркерами.
- **Task queue vs event streaming:** task-очередь (Celery/Taskiq/arq/RQ) — «выполни эту работу один раз», ack per task, ретраи, результат; брокер событий (Kafka) — «произошло событие», много потребителей, replay. См. [[kafka]].
- **Celery:** брокер (RabbitMQ/Redis), воркеры, `@app.task`, `apply_async(countdown/eta)`, `retry`, `acks_late=True` + идемпотентность (иначе at-most-once при падении воркера), `task_reject_on_worker_lost`, result backend (часто не нужен), Celery Beat для периодических, prefetch и `worker_concurrency`, routing по очередям (быстрые/медленные отдельно), таймауты (`soft_time_limit`). Sync по природе; async-таски — костыли.
- **Async-альтернативы:** **Taskiq** (async-native, брокеры Redis/NATS/Kafka, интеграция с FastAPI/Depends), **arq** (asyncio + Redis, простой), **Dramatiq**, **RQ**. Для FastAPI-стека — Taskiq/arq логичнее, чем Celery.
- **Гарантии:** at-least-once по умолчанию при правильной настройке → задачи идемпотентные ([[resilience-patterns]]). Дедупликация по ключу задачи в Redis/БД.
- **Ретраи:** с backoff, ограничением, разделением временных/постоянных ошибок; DLQ/failed-очередь с алертом.
- **Наблюдаемость:** размер очереди, время ожидания, успех/ошибки, длительность — метрики и алерты; Flower для Celery.
- **Постгрес как очередь:** таблица + `FOR UPDATE SKIP LOCKED` — надёжно и просто для малых/средних нагрузок, транзакционно с данными (нет outbox-проблемы). Знать как вариант.
- **Периодические задачи:** Beat / Taskiq scheduler / K8s CronJob; защита от двойного запуска (лок).
- **BackgroundTasks в FastAPI** — только для лёгкого и некритичного (теряется при рестарте).

## Вопросы с собесов

- [ ] Как организуешь фоновые задачи в FastAPI-проекте? → *Taskiq/arq/Celery + Redis/RabbitMQ; не BackgroundTasks для важного*
- [ ] Воркер упал посреди задачи. Что с задачей? → *acks_late → переисполнится; значит, идемпотентность*
- [ ] Celery vs Kafka — это одно и то же? → *нет: работа vs события*
- [ ] Как не выполнить задачу дважды? → *дедупликация по ключу, условные UPDATE, лок*
- [ ] Как сделать отложенную задачу на 3 дня? → *eta/countdown с оговорками (Redis-брокер держит в памяти) / sorted set / таблица с due_at*
- [ ] Как не дать медленным задачам блокировать быстрые? → *отдельные очереди и пулы воркеров*
- [ ] Как мониторишь очередь? → *длина, время в очереди, ошибки; алерты*
- [ ] Когда Postgres-очередь лучше Redis/RabbitMQ? → *транзакционность с данными, малые объёмы, меньше инфры*

## Углубление

- Celery canvas: chains, groups, chords — и почему chord с Redis-бэкендом хрупкий.
- Prefetch multiplier, late ack и visibility timeout у Redis-брокера (задача «воскресает» при долгом исполнении).
- Workflow-оркестраторы (Temporal, Prefect) для долгих многошаговых процессов.
- Приоритеты и honest SLA очередей.

## Ресурсы

- Celery docs: https://docs.celeryq.dev/ (Tasks → Retrying, acks_late; Monitoring)
- Taskiq: https://taskiq-python.github.io/
- arq: https://arq-docs.helpmanual.io/

## Связи

[[kafka]] · [[redis]] · [[resilience-patterns]] · [[fastapi]] · [[transactions-isolation-locks]]

## Мои заметки

