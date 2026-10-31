---
type: topic
title: Kafka
tier: 2
impact: 4
effort: 3
status: todo
confidence: 0
tags: [messaging, tier2]
updated: 2026-10-29
---

# Kafka (и брокеры сообщений вообще)

> **Зачем на собесе:** 4 из 7 вакансий (Альфа, X5, Циан, банк). Вопросы: модель топиков/партиций, consumer groups, гарантии доставки, порядок, ретраи, лаг. Этого не было в первоначальном плане, а рынок просит.

## Минимум для собеса

- **Модель:** topic → партиции (упорядоченный append-only лог, offset) → реплики (leader/follower, ISR). Сообщение = key + value + headers + timestamp. **Ключ определяет партицию** (hash) → порядок гарантирован только внутри партиции, для одного ключа.
- **Producer:** `acks=0/1/all` (durability vs латентность), батчинг (`linger.ms`), идемпотентный продюсер (`enable.idempotence`), ретраи. Сериализация — JSON / Avro / Protobuf + Schema Registry.
- **Consumer groups:** партиция читается одним консюмером группы → параллелизм = число партиций. Rebalance при изменении состава группы (пауза обработки; cooperative sticky снижает). Offset commit: auto vs manual (`commit` после обработки → at-least-once).
- **Гарантии:** at-most-once (commit до обработки), at-least-once (commit после; дубликаты возможны → идемпотентный консюмер), exactly-once (транзакции Kafka, только внутри Kafka→Kafka; end-to-end всё равно идемпотентность на стороне потребителя).
- **Ретраи и ошибки в консюмере:** нельзя бесконечно блокировать партицию одним плохим сообщением → retry-топики с задержкой, DLQ после N попыток, логирование и алерт.
- **Лаг консюмера** — главная метрика (`consumer_lag`); растёт → не успеваем → больше партиций/консюмеров или быстрее обработка. Burrow/Prometheus exporter.
- **Retention** по времени/размеру, compacted topics (последнее значение по ключу — для состояния).
- **Kafka vs RabbitMQ:** Kafka — лог с replay, высокий throughput, порядок по ключу, много консюмеров одного потока; RabbitMQ — очередь с ack per message, роутинг (exchanges), приоритеты, проще для task-очередей. Redis Streams / NATS — лёгкие альтернативы.
- **Python:** `aiokafka` (async), `confluent-kafka` (librdkafka, быстрый, sync), `faststream` (фреймворк над брокерами).
- **Паттерны:** transactional outbox (из [resilience-patterns](../30-backend-design/resilience-patterns.md)), CDC (Debezium), event-carried state transfer, партиционирование по tenant/user для порядка.
- **Схемы и совместимость:** backward/forward compatible изменения событий, Schema Registry.

## Вопросы с собесов

- [ ] Что такое партиция и как гарантируется порядок? → *лог; порядок внутри партиции, ключ → партиция*
- [ ] Как работают consumer groups и что такое rebalance? → *партиция → один консюмер; пауза при изменении состава*
- [ ] Какие гарантии доставки есть и как получить exactly-once? → *at-least + идемпотентность; транзакции внутри Kafka*
- [ ] Сообщение не обрабатывается (bug/невалидное). Что делать, чтобы не встала партиция? → *retry-topic, DLQ, skip с алертом*
- [ ] Консюмер отстаёт (лаг растёт). Действия? → *метрики, партиции, параллелизм, батчи, профилирование*
- [ ] `acks=all` — что это и чем платим? → *подтверждение ISR; латентность*
- [ ] Как записать в БД и отправить событие атомарно? → *outbox*
- [ ] Kafka vs RabbitMQ — когда что? → *поток событий/replay vs task-очередь/роутинг*
- [ ] Сколько партиций делать? → *по целевому параллелизму и throughput; менять потом сложно (ключи перераспределятся)*
- [ ] Как версионируешь схему событий? → *совместимые изменения, registry*

## Углубление

- KRaft (без ZooKeeper), ISR и `min.insync.replicas`, unclean leader election.
- Kafka Streams / ksqlDB, stateful processing, windowing (обзорно).
- Транзакции Kafka в деталях (`transactional.id`, zombie fencing).
- Cooperative rebalancing, static membership, `max.poll.interval.ms` и «consumer kicked out».
- Тестирование консюмеров ([integration-testing](../60-testing/integration-testing.md)), локальная Kafka в compose (redpanda как лёгкая замена).

## Ресурсы

- Kleppmann, DDIA, глава 11 (Stream Processing)
- https://kafka.apache.org/documentation/#design (Design — короткий и точный)
- Confluent blog: «Exactly-once semantics are possible», «Error handling patterns»
- aiokafka docs: https://aiokafka.readthedocs.io/

## Связи

[resilience-patterns](../30-backend-design/resilience-patterns.md) · [task-queues](task-queues.md) · [sd-building-blocks](../80-system-design/sd-building-blocks.md) · [integration-testing](../60-testing/integration-testing.md) · [observability](../40-infra/observability.md)

## Мои заметки

