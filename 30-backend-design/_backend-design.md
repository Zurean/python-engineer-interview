---
type: hub
title: Backend design
tier: 1
updated: 2026-10-29
---

# Backend design — хаб

> **Зачем:** FastAPI + проектирование API и микросервисов + обработка отказов — уровень 1 (5–6 из 7 вакансий). Эти темы спрашивают и на техскрининге («как устроен FastAPI»), и на архитектурной секции («как сервис переживёт падение зависимости»).

## Порядок прохождения (Парето)

1. [fastapi](fastapi.md) — стек из вакансий, конкретные вопросы.
2. [resilience-patterns](resilience-patterns.md) — таймауты, ретраи, идемпотентность, circuit breaker: самый «сеньорный» блок при низких усилиях.
3. [api-design](api-design.md) — REST/HTTP-гигиена, версии, ошибки, пагинация.
4. [architecture-layers](architecture-layers.md) — где живёт бизнес-логика, DI, границы; вопрос «что бы ты переделал в своём проекте».
5. [auth-security](auth-security.md) — JWT/OAuth2, OWASP, секреты. Банки спрашивают.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "30-backend-design" WHERE type = "topic" SORT impact DESC, effort ASC
```

Без Dataview:
- [fastapi](fastapi.md) · [resilience-patterns](resilience-patterns.md) · [api-design](api-design.md) · [architecture-layers](architecture-layers.md) · [auth-security](auth-security.md)
