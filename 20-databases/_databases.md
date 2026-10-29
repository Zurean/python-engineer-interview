---
type: hub
title: Базы данных
tier: 1
updated: 2026-10-29
---

# Базы данных (PostgreSQL) — хаб

> **Зачем:** требование №1 после самого Python (6 из 7 вакансий). На собесе — отдельный блок в техскрининге и опора для системного дизайна. Именно глубина БД отличает Middle от Senior.

## Порядок прохождения (Парето)

1. [[postgres-indexes-explain]] — индексы и чтение планов: спрашивают всегда.
2. [[transactions-isolation-locks]] — уровни изоляции, deadlock, FOR UPDATE.
3. [[sqlalchemy-alembic]] — стек из вакансий: сессии, async, миграции.
4. [[query-optimization-n-plus-1]] — N+1, пагинация, батчинг.
5. [[zero-downtime-migrations]] — «переименуй колонку на живой базе»: Senior-вопрос.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "20-databases" WHERE type = "topic" SORT impact DESC, effort ASC
```

Без Dataview:
- [[postgres-indexes-explain]] · [[transactions-isolation-locks]] · [[sqlalchemy-alembic]] · [[query-optimization-n-plus-1]] · [[zero-downtime-migrations]]

## Практика

Поднять Postgres в Docker, взять таблицу на 1–5 млн строк (сгенерировать), и каждый тезис из заметок проверить руками через `EXPLAIN (ANALYZE, BUFFERS)`. Одной такой сессии хватает, чтобы на собесе говорить уверенно.
