---
type: hub
title: Инфраструктура
tier: 1
updated: 2026-10-29
---

# Инфраструктура — хаб

> **Зачем:** Docker (5/7) и Kubernetes на уровне пользователя (5/7) — уровень 1. CI/CD (3/7) и observability (4/7) — уровень 2. На собесе это 10–15 минут вопросов, но «не знаю Docker» — красный флаг.

## Порядок прохождения (Парето)

1. [docker](docker.md) — Dockerfile для Python, слои, multi-stage, compose. Вечер.
2. [kubernetes](kubernetes.md) — Deployment/Service/Ingress/ConfigMap/Secret, probes, ресурсы, как читать логи. Два вечера.
3. [observability](observability.md) — структурные логи, метрики, трейсинг, Sentry. Уровень 2, но спрашивают в 4/7.
4. [ci-cd](ci-cd.md) — пайплайн: lint → test → build → deploy. Уровень 2.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", tier AS "Ур.", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "40-infra" WHERE type = "topic" SORT tier ASC, impact DESC
```

Без Dataview: [docker](docker.md) · [kubernetes](kubernetes.md) · [observability](observability.md) · [ci-cd](ci-cd.md)

## Практика

Всё это закрывается одним прогоном [rag-service-project](../capstone/rag-service-project.md): Dockerfile → compose → kind/k3s → GitHub Actions → Prometheus/Grafana/Sentry.
