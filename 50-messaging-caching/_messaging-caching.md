---
type: hub
title: Очереди и кэширование
tier: 2
updated: 2026-10-29
---

# Очереди, брокеры, кэширование — хаб

> **Зачем:** Kafka/RabbitMQ — 4 из 7, Redis — 3 из 7. Уровень 2, но без них не ответить на системный дизайн и на «как сервис переживёт нагрузку». Банки спрашивают Kafka подробно.

## Порядок прохождения

1. [[kafka]] — модель, consumer groups, гарантии, ретраи, лаг.
2. [[redis]] — структуры, кэш и инвалидация, локи, rate limiting, pub/sub.
3. [[task-queues]] — Celery/Taskiq/arq: фоновые задачи, отличие от брокера событий.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "50-messaging-caching" WHERE type = "topic" SORT impact DESC
```

Без Dataview: [[kafka]] · [[redis]] · [[task-queues]]
