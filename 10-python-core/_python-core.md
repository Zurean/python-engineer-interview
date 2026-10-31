---
type: hub
title: Python core
tier: 1
updated: 2026-10-29
---

# Python core — хаб

> **Зачем:** первая и обязательная секция любого собеса. Требуют все 7 вакансий. Здесь же главный фильтр для FastAPI-вакансий — asyncio.

## Порядок прохождения (Парето)

1. [asyncio](asyncio.md) — главный фильтр, учить первым.
2. [data-model](data-model.md) — классика скрининга, дешёвая.
3. [generators-context-managers-decorators](generators-context-managers-decorators.md) — спрашивают всегда.
4. [gil-threads-processes](gil-threads-processes.md) — один вопрос, задают в 90 % случаев.
5. [typing](typing.md) — на Middle+ ожидают уверенности, на Senior — Protocol/дженерики.
6. [memory-performance](memory-performance.md) — «сервис медленный, что делать» + утечки.
7. [tooling](tooling.md) — uv, ruff, pyproject: короткий разговор, но показывает зрелость.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "10-python-core" WHERE type = "topic" SORT impact DESC, effort ASC
```

Без Dataview:
- [asyncio](asyncio.md) · [data-model](data-model.md) · [generators-context-managers-decorators](generators-context-managers-decorators.md) · [gil-threads-processes](gil-threads-processes.md) · [typing](typing.md) · [memory-performance](memory-performance.md) · [tooling](tooling.md)

## Практика

Весь набор скрининговых вопросов одним списком — [python-live-coding-tasks](../90-live-coding/python-live-coding-tasks.md). Упражнение на код-ревью с типовыми багами — [code-review-exercise](../90-live-coding/code-review-exercise.md).
