---
type: hub
title: Системный дизайн
tier: 1
updated: 2026-10-29
---

# Системный дизайн — хаб

> **Зачем:** отдельная секция на Middle+/Senior; «проектирование микросервисов, архитектура» — 6 из 7 вакансий. При коротком опыте на Python это обычно одна из двух зон с самым большим пробелом — и, значит, с самым большим приростом.

Главное: секция проверяет **процесс рассуждения**, а не знание «правильной» архитектуры. Кандидат без фреймворка ответа проваливается даже с хорошими знаниями. Поэтому порядок такой:

1. [sd-framework](sd-framework.md) — пошаговый сценарий ответа (45 минут). Выучить как стихотворение.
2. [sd-building-blocks](sd-building-blocks.md) — кирпичики: балансировка, кэш, очереди, шардирование, репликация, консистентность. Каждый — с ценой.
3. [sd-practice-cases](sd-practice-cases.md) — 6 кейсов, прогнать вслух по фреймворку, с таймером.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "80-system-design" WHERE type = "topic" SORT impact DESC
```

Без Dataview: [sd-framework](sd-framework.md) · [sd-building-blocks](sd-building-blocks.md) · [sd-practice-cases](sd-practice-cases.md)

## Как тренироваться

Один кейс = 45 минут вслух + 15 минут разбора. Записывать на диктофон или рисовать в excalidraw. Два кейса в неделю — через месяц секция перестаёт пугать. Опорные темы: [resilience-patterns](../30-backend-design/resilience-patterns.md), [kafka](../50-messaging-caching/kafka.md), [redis](../50-messaging-caching/redis.md), [transactions-isolation-locks](../20-databases/transactions-isolation-locks.md).
