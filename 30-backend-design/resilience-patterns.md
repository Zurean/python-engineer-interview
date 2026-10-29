---
type: topic
title: Отказоустойчивость интеграций
tier: 1
impact: 5
effort: 2
status: todo
confidence: 0
tags: [backend, architecture, tier1]
updated: 2026-10-29
---

# Отказоустойчивость интеграций: таймауты, ретраи, идемпотентность

> **Зачем на собесе:** «обработка отказов зависимостей» — уровень 1 из анализа вакансий. На архитектурной секции и в системном дизайне это главный признак сеньорности: не «как сделать», а «что будет, когда сломается». Тема компактная, импакт огромный.

## Минимум для собеса

- **Таймауты везде.** Любой сетевой вызов без таймаута — висящий воркер при деградации зависимости. Отдельно connect и read timeout (`httpx.Timeout(connect=1, read=5)`), таймаут на запрос к БД (`statement_timeout`), таймаут на всю операцию (`asyncio.timeout`). Бюджет времени: таймаут на внешний вызов < таймаут на входящий запрос.
- **Ретраи — только для идемпотентных операций и временных ошибок** (5xx, таймауты, connection reset; не 4xx). **Exponential backoff + jitter**, ограниченное число попыток. Ретраи без бюджета = retry storm, который добивает деградирующую зависимость. Библиотеки: `tenacity`, встроенные в httpx transport.
- **Идемпотентность:** повтор операции даёт тот же результат. HTTP: GET/PUT/DELETE идемпотентны, POST — нет → `Idempotency-Key` в заголовке, хранить результат по ключу (Redis/БД с уникальным индексом). В обработке очередей — at-least-once доставка → дедупликация по id сообщения, или операции вида `UPDATE ... WHERE status = 'pending'`.
- **Circuit breaker:** после N ошибок перестать дёргать зависимость на время (open), потом пробный запрос (half-open). Защищает и себя, и её. Библиотеки: `pybreaker`, `aiobreaker`; на уровне инфры — service mesh.
- **Bulkhead:** изолировать пулы/семафоры под разные зависимости, чтобы одна медленная не съела все воркеры.
- **Fallback и graceful degradation:** закэшированный ответ, дефолт, «фича временно недоступна», вместо 500.
- **Rate limiting** на входе (token bucket в Redis) и уважение лимитов внешних API (429 + Retry-After).
- **Transactional outbox:** нельзя атомарно «записать в БД и отправить в Kafka». Пишем событие в таблицу outbox в той же транзакции, отдельный процесс публикует. Inbox для дедупликации на приёме. **Saga** вместо распределённых транзакций.
- **Health checks:** liveness (процесс жив) vs readiness (готов принимать трафик: БД/брокер доступны).
- **Backpressure:** bounded очереди, отказ (503) лучше бесконечного роста latency. Load shedding.
- **Dead letter queue** для сообщений, которые не обработались после N попыток.

## Вопросы с собесов

- [ ] Внешний API стал отвечать по 30 секунд. Что случится с твоим сервисом и как защититься? → *исчерпание воркеров/соединений; таймауты, circuit breaker, bulkhead, fallback*
- [ ] Задача из очереди может выполниться дважды. Как сделать обработку идемпотентной? → *ключ идемпотентности, уникальный индекс, условный UPDATE, inbox*
- [ ] Когда ретраить нельзя? → *неидемпотентные операции, 4xx, без backoff/лимита*
- [ ] Как гарантировать, что событие в Kafka отправится, если запись в БД прошла? → *transactional outbox*
- [ ] Что такое circuit breaker, какие у него состояния? → *closed/open/half-open*
- [ ] Чем liveness отличается от readiness? → *перезапуск vs исключение из балансировки*
- [ ] Как сделать POST идемпотентным? → *Idempotency-Key + сохранённый ответ*
- [ ] Что такое exponential backoff с jitter и зачем jitter? → *разнести повторы, избежать синхронных волн*
- [ ] Как сервис должен вести себя, когда Redis (кэш) недоступен? → *деградация до БД, не падать; таймаут на кэш маленький*
- [ ] Что такое saga и чем отличается от 2PC? → *цепочка локальных транзакций с компенсациями*

## Углубление

- Retry budgets и adaptive concurrency (по мотивам Google SRE, Netflix concurrency-limits).
- Deadline propagation через заголовки между сервисами (gRPC-style).
- Exactly-once semantics в Kafka: идемпотентный продюсер, транзакции, почему end-to-end exactly-once всё равно требует идемпотентного консюмера ([[kafka]]).
- Chaos engineering на минималках: убить зависимость в docker-compose и посмотреть, что будет.
- Паттерны из «Release It!» (Nygard): Steady State, Fail Fast, Handshaking.

## Ресурсы

- Michael Nygard, «Release It!» (2nd ed.) — главы про stability patterns
- https://microservices.io/patterns/ (Outbox, Saga, Circuit Breaker)
- Google SRE book, глава «Handling Overload» и «Addressing Cascading Failures»
- Документация tenacity: https://tenacity.readthedocs.io/

## Связи

[[kafka]] · [[redis]] · [[task-queues]] · [[sd-building-blocks]] · [[observability]] · [[transactions-isolation-locks]]

## Мои заметки

