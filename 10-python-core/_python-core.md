---
type: hub
title: Python core
tier: 1
updated: 2026-10-29
---

# Python core — хаб

> **Зачем:** первая и обязательная секция любого собеса. Требуют все 7 вакансий. Здесь же главный фильтр для FastAPI-вакансий — asyncio.

## Порядок прохождения (Парето)

1. [[asyncio]] — главный фильтр, учить первым.
2. [[data-model]] — классика скрининга, дешёвая.
3. [[generators-context-managers-decorators]] — спрашивают всегда.
4. [[gil-threads-processes]] — один вопрос, задают в 90 % случаев.
5. [[typing]] — на Middle+ ожидают уверенности, на Senior — Protocol/дженерики.
6. [[memory-performance]] — «сервис медленный, что делать» + утечки.
7. [[tooling]] — uv, ruff, pyproject: короткий разговор, но показывает зрелость.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "10-python-core" WHERE type = "topic" SORT impact DESC, effort ASC
```

Без Dataview:
- [[asyncio]] · [[data-model]] · [[generators-context-managers-decorators]] · [[gil-threads-processes]] · [[typing]] · [[memory-performance]] · [[tooling]]

## Практика

Весь набор скрининговых вопросов одним списком — [[python-live-coding-tasks]]. Упражнение на код-ревью с типовыми багами — [[code-review-exercise]].
