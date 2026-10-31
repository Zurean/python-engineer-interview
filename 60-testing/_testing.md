---
type: hub
title: Тестирование
tier: 1
updated: 2026-10-29
---

# Тестирование — хаб

> **Зачем:** 4 из 7 вакансий требуют unit + интеграционные тесты; спрашивают почти на каждом собесе, а готовиться дёшево. Отношение к тестам — быстрый маркер зрелости.

## Порядок прохождения

1. [pytest](pytest.md) — фикстуры, параметризация, моки, async-тесты, тестирование FastAPI.
2. [integration-testing](integration-testing.md) — реальная БД в тестах, testcontainers, стратегия пирамиды.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "60-testing" WHERE type = "topic" SORT impact DESC, effort ASC
```

Без Dataview: [pytest](pytest.md) · [integration-testing](integration-testing.md)
