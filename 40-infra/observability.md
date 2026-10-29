---
type: topic
title: Observability
tier: 2
impact: 4
effort: 2
status: todo
confidence: 0
tags: [infra, tier2]
updated: 2026-10-29
---

# Observability: логи, метрики, трейсинг

> **Зачем на собесе:** 4 из 7 вакансий; на Senior-собесе — «как узнаёшь, что сервис сломался, раньше пользователей» и «как расследовал инцидент». Хороший ответ строится вокруг трёх столпов и SLO.

## Минимум для собеса

- **Три столпа:** логи (что случилось), метрики (сколько/как быстро, агрегаты), трейсы (путь запроса через сервисы). Плюс ошибки (Sentry) и алерты.
- **Логи:** структурные (JSON) через `structlog`/`logging` + JSON-форматтер; в stdout; уровни; обязательные поля — timestamp, level, service, request_id/trace_id, user_id (без PII). `contextvars` для request_id через async-код. Не логировать секреты и тела. Сборщики: Loki, ELK. Логировать ошибку один раз с контекстом, не на каждом слое.
- **Метрики:** Prometheus (pull, `/metrics`), типы — Counter, Gauge, Histogram (latency buckets, p95/p99 через `histogram_quantile`), Summary. **RED** для сервисов: Rate, Errors, Duration; **USE** для ресурсов. Клиент `prometheus_client`, middleware для FastAPI; метрики бизнес-уровня (заказов/мин, лаг консюмера). В multi-process (gunicorn) — `PROMETHEUS_MULTIPROC_DIR`. Grafana — дашборды.
- **Трейсинг:** OpenTelemetry — SDK + автоинструментирование (FastAPI, SQLAlchemy, httpx, aiokafka) → Jaeger/Tempo. Span, trace_id, пропагация контекста через заголовки (W3C traceparent) и через сообщения Kafka.
- **Ошибки:** Sentry — исключения с контекстом, группировка, release tracking, performance.
- **Алерты:** по симптомам (error rate, p99, доступность), не по причинам (CPU 80 %). Alertmanager → Telegram/PagerDuty. Избегать alert fatigue.
- **SLI / SLO / error budget:** SLI — измеримый показатель (доля успешных запросов < 300 мс), SLO — цель (99.9 % за 30 дней), error budget — сколько можно «потратить». Алерты на скорость сгорания бюджета (burn rate).
- **Health endpoints:** `/health/live`, `/health/ready` ([[kubernetes]]).
- **Расследование инцидента:** алерт → дашборд (RED) → трейсы медленных запросов → логи по trace_id → гипотеза → фикс → постмортем без обвинений (blameless), action items.
- **Для LLM-сервисов:** дополнительно метрики токенов, стоимости, латентности модели, доли фолбэков ([[evals-cost-latency]]).

## Вопросы с собесов

- [ ] Как узнаёшь, что сервис деградировал, раньше пользователей? → *RED-метрики, алерты на SLO, Sentry*
- [ ] Чем метрики отличаются от логов и трейсов, когда что использовать? → *агрегаты / события / путь запроса*
- [ ] Какие типы метрик в Prometheus и как считать p99? → *Counter/Gauge/Histogram; histogram_quantile*
- [ ] Как пробрасываешь request_id/trace_id через async-код и между сервисами? → *contextvars, OTel propagation, заголовки*
- [ ] Что такое SLO и error budget? → *цель на SLI; допустимый объём ошибок*
- [ ] Расскажи про инцидент, который расследовал. → *[[stories]], история 1; по шагам*
- [ ] Как настраиваешь алерты, чтобы они не доставали? → *по симптомам, burn rate, severity*
- [ ] Как логировать в async-сервисе, чтобы не тормозить? → *stdout, неблокирующий handler/QueueHandler, без лишнего в hot path*
- [ ] Что такое OpenTelemetry и почему он, а не vendor SDK? → *стандарт, автоинструментирование, vendor-neutral*

## Углубление

- Exemplars (связь метрик и трейсов), tail-based sampling.
- Cardinality: почему нельзя класть user_id в лейблы метрик.
- Profiling в проде как четвёртый столп (Pyroscope, py-spy).
- Log-based metrics vs настоящие метрики; стоимость хранения логов.
- SRE-практики: on-call, runbooks, постмортемы, game days.

## Ресурсы

- Google SRE Book, главы «Monitoring Distributed Systems», «SLOs»
- https://opentelemetry.io/docs/languages/python/
- https://prometheus.io/docs/practices/ (naming, histograms, alerting)
- structlog docs: https://www.structlog.org/

## Связи

[[kubernetes]] · [[resilience-patterns]] · [[memory-performance]] · [[fastapi]] · [[evals-cost-latency]] · [[stories]]

## Мои заметки

