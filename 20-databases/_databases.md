---
type: hub
title: Базы данных
tier: 1
updated: 2026-10-29
---

# Базы данных (PostgreSQL) — хаб

> **Зачем:** требование №1 после самого Python (6 из 7 вакансий). На собесе — отдельный блок в техскрининге и опора для системного дизайна. Именно глубина БД отличает Middle от Senior.

## Порядок прохождения (Парето)

1. [postgres-indexes-explain](postgres-indexes-explain.md) — индексы и чтение планов: спрашивают всегда.
2. [transactions-isolation-locks](transactions-isolation-locks.md) — уровни изоляции, deadlock, FOR UPDATE.
3. [sqlalchemy-alembic](sqlalchemy-alembic.md) — стек из вакансий: сессии, async, миграции.
4. [query-optimization-n-plus-1](query-optimization-n-plus-1.md) — N+1, пагинация, батчинг.
5. [zero-downtime-migrations](zero-downtime-migrations.md) — «переименуй колонку на живой базе»: Senior-вопрос.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "20-databases" WHERE type = "topic" SORT impact DESC, effort ASC
```

Без Dataview:
- [postgres-indexes-explain](postgres-indexes-explain.md) · [transactions-isolation-locks](transactions-isolation-locks.md) · [sqlalchemy-alembic](sqlalchemy-alembic.md) · [query-optimization-n-plus-1](query-optimization-n-plus-1.md) · [zero-downtime-migrations](zero-downtime-migrations.md)

## Практика

Поднять Postgres в Docker, взять таблицу на 1–5 млн строк (сгенерировать), и каждый тезис из заметок проверить руками через `EXPLAIN (ANALYZE, BUFFERS)`. Одной такой сессии хватает, чтобы на собесе говорить уверенно.
