---
type: topic
title: Строительные блоки распределённых систем
tier: 1
impact: 4
effort: 3
status: todo
confidence: 0
tags: [system-design, tier1]
updated: 2026-10-29
---

# Строительные блоки: что это и какова цена

> **Зачем на собесе:** на секции дизайна нужно быстро доставать кирпичики и — главное — называть их цену. Ниже каждый блок в формате «зачем → как → чем платим».

## Минимум для собеса

**Балансировка и stateless-сервисы.** L4/L7 балансировщик (nginx, envoy, cloud LB), round-robin/least-connections, health checks. Сервисы без состояния → масштабирование репликами. Сессии — в Redis, не в памяти. Цена: нет.

**Кэширование.** Где: клиент/CDN, API-слой, приложение (in-memory), распределённый (Redis), БД (shared buffers). Стратегии: cache-aside (читаем кэш → промах → БД → кладём), write-through, write-behind. TTL vs явная инвалидация. Цена: инвалидация (самая сложная проблема), stale data, cache stampede (lock/early refresh), горячие ключи. См. [redis](../50-messaging-caching/redis.md).

**Репликация БД.** Primary + read replicas (streaming), чтение с реплик → лаг → read-your-writes проблема (читать с primary после записи своего). Failover (Patroni). Multi-primary — почти никогда. Цена: лаг, сложность роутинга.

**Партиционирование и шардирование.** Партиционирование — внутри одной БД (по времени, по ключу). Шардирование — по разным инстансам: ключ шардирования (user_id, tenant), hash vs range, consistent hashing. Цена: cross-shard запросы и транзакции, ребалансировка, горячие шарды. Делать, когда вертикальное масштабирование и реплики исчерпаны.

**Очереди и брокеры.** Разгрузка синхронного пути, буферизация пиков, развязка сервисов, ретраи. Kafka (лог, партиции, consumer groups, replay) vs RabbitMQ (очереди, роутинг, ack) vs SQS/Redis Streams. Гарантии: at-most/at-least/exactly-once (последняя — почти всегда «at-least + идемпотентность»). Порядок — в рамках партиции/ключа. Цена: eventual consistency, дубликаты, мониторинг лага, DLQ. См. [kafka](../50-messaging-caching/kafka.md), [task-queues](../50-messaging-caching/task-queues.md).

**Консистентность.** CAP на пальцах: при сетевом разделении выбираем доступность или консистентность. Сильная vs eventual; где строгая обязательна (деньги, остатки) — транзакции в одной БД, идемпотентность, outbox; где можно eventual — ленты, счётчики, поиск. Saga для многошаговых операций между сервисами. См. [transactions-isolation-locks](../20-databases/transactions-isolation-locks.md), [resilience-patterns](../30-backend-design/resilience-patterns.md).

**Хранилища по задаче.** Реляционная (Postgres) — дефолт; документная (Mongo) — гибкая схема, редко нужна; key-value (Redis) — кэш/сессии/счётчики; поиск (Elasticsearch/OpenSearch) — полнотекст, фасеты; колоночные (ClickHouse) — аналитика; объектное (S3) — файлы; time-series; графовые; векторные (pgvector/Qdrant) — [vector-dbs](../95-optional/vector-dbs.md). Ответ «Postgres + S3 + Redis» закрывает 80 % кейсов.

**Поиск и индексация.** CDC/outbox → индексатор → Elasticsearch; eventual.

**Rate limiting и защита.** Token bucket / sliding window в Redis, на gateway. Backpressure, load shedding.

**Уникальные ID.** Автоинкремент (не шардируется), UUIDv4 (случайный → плохо для B-tree), UUIDv7/ULID (упорядоченные), Snowflake.

**Файлы и медиа.** Загрузка напрямую в S3 по presigned URL, обработка асинхронно, CDN для раздачи.

**Realtime.** Polling → long polling → SSE → WebSocket; pub/sub через Redis/Kafka между репликами.

**Наблюдаемость и деплой** — одной фразой в каждом дизайне ([observability](../40-infra/observability.md), [kubernetes](../40-infra/kubernetes.md)).

## Вопросы с собесов

- [ ] Чем Kafka отличается от RabbitMQ, когда что? → *лог с replay и партициями vs очередь с роутингом*
- [ ] Как инвалидировать кэш при обновлении данных? Что такое cache stampede? → *delete on write / TTL / версии; lock или early refresh*
- [ ] Когда шардировать БД и по какому ключу? → *когда исчерпаны реплики/вертикаль; по ключу доступа, равномерному*
- [ ] Что такое CAP и как это применить к выбору хранилища? → *при партиции — A или C*
- [ ] Как обеспечить exactly-once обработку события? → *at-least-once + идемпотентный консюмер*
- [ ] Читаем с реплики после записи и не видим свои данные. Что делать? → *read-your-writes: primary для автора, sticky session, версия*
- [ ] Как сгенерировать уникальные ID в распределённой системе? → *UUIDv7/Snowflake*
- [ ] Как загрузить файл 2 ГБ через API? → *presigned URL, multipart, асинхронная обработка*
- [ ] Чем партиционирование отличается от шардирования? → *внутри инстанса vs между*

## Углубление

- Консенсус (Raft) на уровне «что решает», leader election, распределённые локи (и почему Redlock спорный).
- Quorum reads/writes, векторные часы, CRDT — обзорно.
- CDC через Debezium.
- Геораспределённость, multi-region active-active и почему это дорого.
- Паттерны данных из DDIA: LSM vs B-tree, compaction, что такое write amplification.

## Ресурсы

- Martin Kleppmann, «Designing Data-Intensive Applications» — главы 5–9 (репликация, партиционирование, транзакции, проблемы распределённых систем, консистентность). Обязательно, но можно по одной главе в неделю.
- Alex Xu, «System Design Interview» — главы 4–6 (rate limiter, consistent hashing, key-value store)
- https://github.com/donnemartin/system-design-primer

## Связи

[sd-framework](sd-framework.md) · [sd-practice-cases](sd-practice-cases.md) · [kafka](../50-messaging-caching/kafka.md) · [redis](../50-messaging-caching/redis.md) · [transactions-isolation-locks](../20-databases/transactions-isolation-locks.md) · [resilience-patterns](../30-backend-design/resilience-patterns.md) · [postgres-indexes-explain](../20-databases/postgres-indexes-explain.md)

## Мои заметки

