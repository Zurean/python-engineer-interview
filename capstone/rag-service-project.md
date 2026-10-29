---
type: project
title: Сквозной проект — RAG-сервис по документам
status: todo
updated: 2026-10-29
---

# Сквозной проект: RAG-сервис по документам

> **Зачем:** один проект вместо разрозненных курсов. Если был опыт обработки документов через LLM — это его естественное продолжение. Закрывает почти все темы уровней 1–2 и даёт то, чего не даёт теория: *истории для собеса* («а у нас в проекте…»). Это также портфолио на GitHub.

## Что делает сервис

Пользователь загружает документы (PDF/DOCX/MD) → они индексируются → можно задавать вопросы и получать ответы с цитатами источников. Multi-tenant: пользователь видит только свои документы.

## Стек (по вакансиям)

| Компонент | Выбор | Закрывает |
|---|---|---|
| API | FastAPI, async, Pydantic v2 | [[fastapi]], [[typing]], [[api-design]] |
| БД | Postgres + pgvector, SQLAlchemy 2.0 async, Alembic | [[_databases]], [[rag]] |
| Очередь индексации | Kafka (aiokafka) или Redpanda; retry-topic + DLQ | [[kafka]], [[resilience-patterns]] |
| Кэш и лимиты | Redis: кэш ответов, rate limiting, идемпотентность | [[redis]] |
| Фоновые задачи | Taskiq/arq для переиндексации и отчётов | [[task-queues]] |
| LLM-слой | SDK напрямую: эмбеддинги, structured output, tool calling для «поиск → ответ» | [[rag]], [[tool-calling-structured-output]] |
| Evals | датасет 50+ вопросов, recall@k + LLM-judge, запуск в CI | [[evals-cost-latency]] |
| Тесты | pytest, testcontainers (Postgres, Redis, Kafka) | [[_testing]] |
| Контейнеры | Dockerfile multi-stage, docker-compose для dev | [[docker]] |
| Оркестрация | kind/k3s: Deployment, Service, Ingress, probes, Job для миграций | [[kubernetes]] |
| CI/CD | GitHub Actions: lint → types → tests → build → deploy в kind | [[ci-cd]] |
| Observability | structlog JSON, Prometheus + Grafana (RED + токены/стоимость), OTel-трейсы, Sentry | [[observability]] |
| Auth | JWT access/refresh, owner-check на документах | [[auth-security]] |

## Этапы (каждый — рабочий инкремент, 1–2 недели)

- [ ] **0. Скелет.** uv, ruff, mypy, pre-commit, src-layout, FastAPI «hello», Dockerfile, compose с Postgres/Redis, CI с линтом и тестами. → [[tooling]], [[docker]], [[ci-cd]]
- [ ] **1. Документы.** Загрузка файла (presigned-подобный поток или прямой upload), таблицы `documents`/`chunks`, миграции, JWT-auth, owner-check, интеграционные тесты на реальном Postgres. → [[sqlalchemy-alembic]], [[auth-security]], [[integration-testing]]
- [ ] **2. Индексация асинхронно.** Событие «документ загружен» → Kafka → воркер: парсинг, чанкинг, эмбеддинги → pgvector. Outbox для события, идемпотентность по hash, retry-topic, DLQ. → [[kafka]], [[resilience-patterns]]
- [ ] **3. Вопрос-ответ.** Гибридный поиск (вектор + tsvector), rerank, structured output с цитатами, стриминг ответа (SSE). Таймауты/ретраи/фолбэк на вызовы модели. → [[rag]], [[tool-calling-structured-output]]
- [ ] **4. Кэш и лимиты.** Redis: кэш ответов по (tenant, normalized question), rate limiting на пользователя, Idempotency-Key на загрузку. → [[redis]]
- [ ] **5. Evals.** Датасет вопросов по своим документам, recall@k, faithfulness через judge, отчёт в CI, сравнение двух стратегий чанкинга. → [[evals-cost-latency]]
- [ ] **6. Observability.** JSON-логи с request_id/trace_id, /metrics (RED + токены + стоимость), OTel → Jaeger, Sentry, дашборд в Grafana, алерт на error rate. → [[observability]]
- [ ] **7. Kubernetes.** Манифесты/Helm, probes, requests/limits, Job для миграций, rolling update без даунтайма, HPA по CPU. → [[kubernetes]], [[zero-downtime-migrations]]
- [ ] **8. Нагрузка и профилирование.** locust/k6, найти узкое место, EXPLAIN ANALYZE на поиске, py-spy на воркере, оптимизировать, задокументировать «до/после». → [[memory-performance]], [[query-optimization-n-plus-1]]
- [ ] **9. README как кейс для собеса.** Архитектурная схема, решения и трейдоффы (ADR), цифры (латентность, стоимость на запрос, recall), что бы сделал иначе. → [[stories]], [[architecture-layers]]

## Как превращать проект в ответы на собесе

После каждого этапа — записать в «Мои заметки» 2–3 «истории»: с какой проблемой столкнулся, как решил, что измерил. Это и есть материал для секций «опыт» и «архитектура». Проект без таких заметок — просто код.

## Мои заметки

