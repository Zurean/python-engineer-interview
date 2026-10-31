---
type: topic
title: Проектирование API
tier: 1
impact: 3
effort: 2
status: todo
confidence: 0
tags: [backend, architecture, tier1]
updated: 2026-10-29
---

# Проектирование API

> **Зачем на собесе:** «спроектируй API для X» — частая разминка перед системным дизайном. Проверяют HTTP-гигиену, формат ошибок, версионирование, пагинацию, и понимание, когда REST не подходит.

## Минимум для собеса

- **Ресурсы и методы:** существительные во множественном числе (`/orders/{id}/items`), GET (безопасный, идемпотентный), POST (создание/действие), PUT (полная замена, идемпотентный), PATCH (частичное), DELETE. Действия, не ложащиеся в CRUD — `POST /orders/{id}/cancel`.
- **Коды ответов:** 200/201/204; 400 (невалидный запрос), 401 (не аутентифицирован), 403 (нет прав), 404, 409 (конфликт/дубликат), 422 (валидация), 429 (лимит), 500/502/503/504. Не возвращать 200 с `{"error": ...}`.
- **Единый формат ошибок:** `{"code": "ORDER_NOT_FOUND", "message": "...", "details": [...]}`; RFC 9457 (Problem Details). Коды ошибок машиночитаемые, сообщения — для людей.
- **Пагинация:** cursor-based для лент и больших таблиц, offset — для админок. Единый ответ `{"items": [], "next_cursor": "..."}`. Фильтрация и сортировка через query-параметры, лимит на `limit`.
- **Версионирование:** в пути (`/v1/`) — проще всего; заголовок — чище, но реже. Обратная совместимость: добавлять поля можно, удалять/переименовывать — нет (expand/contract, как в [zero-downtime-migrations](../20-databases/zero-downtime-migrations.md)).
- **Идемпотентность POST** через `Idempotency-Key` ([resilience-patterns](resilience-patterns.md)).
- **Контракт first:** OpenAPI как источник правды, генерация клиентов, contract-тесты.
- **Длительные операции:** `202 Accepted` + ресурс статуса/`Location`, polling или webhook/SSE.
- **Безопасность:** аутентификация на каждом запросе, авторизация на уровне ресурса (не только роли), rate limiting, не светить внутренние id/стек-трейсы ([auth-security](auth-security.md)).
- **Альтернативы REST:** gRPC (внутренние сервисы, строгие контракты, стриминг, производительность), GraphQL (много клиентов с разными нуждами, цена — сложность и N+1 на сервере), WebSocket/SSE (push). Уметь объяснить, когда что.
- **Межсервисное взаимодействие:** sync (HTTP/gRPC) vs async (события через брокер) — связность, задержка, устойчивость ([kafka](../50-messaging-caching/kafka.md)).

## Вопросы с собесов

- [ ] Спроектируй API для корзины/заказов/комментариев: ресурсы, методы, коды, пагинация. → *проговорить вслух за 5 минут*
- [ ] PUT vs PATCH vs POST? → *замена/частичное/создание-действие; идемпотентность*
- [ ] Как версионируешь API и как не сломать клиентов? → *путь, аддитивные изменения, deprecation*
- [ ] Как оформляешь ошибки? → *единый формат, машиночитаемый код, Problem Details*
- [ ] Как сделать API для долгой операции (генерация отчёта 5 минут)? → *202 + статус-ресурс / webhook*
- [ ] REST vs gRPC vs GraphQL — когда что? → *внешние клиенты / внутренние сервисы / много разных клиентов*
- [ ] Чем аутентификация отличается от авторизации, где они живут в FastAPI? → *Depends на уровне роутера/эндпоинта*
- [ ] Что такое HATEOAS и нужен ли он? → *редко; знать, что это*

## Углубление

- Проектирование событий (event-carried state transfer vs notification), схемы событий, Schema Registry.
- gRPC в Python: protobuf, `grpcio`/`grpclib`, стриминг, deadlines, интерсепторы.
- API gateway, BFF, агрегация.
- Conditional requests (`ETag`, `If-Match`) для optimistic concurrency.
- Webhooks: подпись, ретраи, идемпотентность на стороне получателя.

## Ресурсы

- https://cloud.google.com/apis/design (API Design Guide — лучший короткий гайд)
- RFC 9457 Problem Details
- Microsoft REST API Guidelines (GitHub)

## Связи

[fastapi](fastapi.md) · [resilience-patterns](resilience-patterns.md) · [auth-security](auth-security.md) · [sd-framework](../80-system-design/sd-framework.md) · [kafka](../50-messaging-caching/kafka.md)

## Мои заметки

