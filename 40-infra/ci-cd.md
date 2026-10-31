---
type: topic
title: CI/CD
tier: 2
impact: 3
effort: 1
status: todo
confidence: 0
tags: [infra, tier2]
updated: 2026-10-29
---

# CI/CD

> **Зачем на собесе:** 3 из 7 вакансий (X5, Циан с Jenkins, банк). Вопрос один: «как выглядит ваш пайплайн от коммита до прода». Нужно описать свой и понимать принципы. Вечер подготовки.

## Минимум для собеса

- **CI:** на каждый PR — lint (ruff), типы (mypy), unit-тесты, интеграционные тесты (сервис-контейнеры), сборка Docker-образа, сканирование (trivy, pip-audit). Быстро (< 10 мин), с кэшем зависимостей (uv cache) и слоёв образа.
- **CD:** после merge в main — сборка образа с тегом = commit sha, push в registry, деплой в staging автоматически, в prod — автоматически или по кнопке (approval). Миграции — отдельным шагом до раскатки ([kubernetes](kubernetes.md)).
- **Инструменты:** GitHub Actions (workflow, jobs, steps, matrix, services, secrets, environments), GitLab CI (`.gitlab-ci.yml`, stages, runners), Jenkins (Jenkinsfile, declarative pipeline) — знать, что это, уметь прочитать.
- **Стратегии деплоя:** rolling (дефолт K8s), blue/green (два окружения, переключение трафика), canary (часть трафика на новую версию + метрики), feature flags (деплой ≠ релиз).
- **Откат:** redeploy предыдущего тега; схема БД совместима ([zero-downtime-migrations](../20-databases/zero-downtime-migrations.md)).
- **Ветвление:** trunk-based с короткими ветками и feature flags vs git flow; релизные теги, semver.
- **Секреты в CI:** secrets хранилища CI, OIDC-федерация в облако вместо долгоживущих ключей; не логировать.
- **Качество пайплайна:** воспроизводимость (lock-файлы, pinned actions), кэш, параллельные job-ы, `fail fast`, артефакты (coverage, отчёты).
- **GitOps (обзорно):** состояние кластера в git, ArgoCD/Flux применяют.
- **Pre-commit** локально дублирует быстрые проверки CI ([tooling](../10-python-core/tooling.md)).

## Вопросы с собесов

- [ ] Опиши пайплайн от коммита до прода в твоём проекте. → *по шагам с инструментами и временем*
- [ ] Как ускорил бы медленный CI (30 минут)? → *кэш, параллелизм, разделение unit/integration, выборочный запуск*
- [ ] Blue/green vs canary vs rolling — когда что? → *мгновенный откат / проверка на доле трафика / дефолт*
- [ ] Как откатываешь неудачный релиз? → *предыдущий тег; совместимость схемы*
- [ ] Где запускаются миграции в пайплайне? → *job до деплоя, один раз*
- [ ] Что такое feature flag и зачем отделять деплой от релиза? → *снижение риска, постепенное включение*
- [ ] Как хранишь секреты для CI? → *secrets, OIDC, scoping*
- [ ] Trunk-based или git flow — что практиковал и почему? → *честно*

## Углубление

- Build matrices и reusable workflows; monorepo и выборочный запуск по изменённым путям.
- Подпись образов (cosign), SLSA, provenance.
- Progressive delivery с Argo Rollouts/Flagger: автоматический canary по метрикам.
- Тестирование инфраструктурного кода (Terraform plan в PR) — [cloud-terraform](../95-optional/cloud-terraform.md).
- DORA-метрики: lead time, deployment frequency, MTTR, change failure rate.

## Ресурсы

- https://docs.github.com/en/actions (Workflow syntax, Caching, Services)
- Книга «Continuous Delivery» (Humble, Farley) — по оглавлению
- https://trunkbaseddevelopment.com/

## Связи

[docker](docker.md) · [kubernetes](kubernetes.md) · [tooling](../10-python-core/tooling.md) · [integration-testing](../60-testing/integration-testing.md) · [zero-downtime-migrations](../20-databases/zero-downtime-migrations.md)

## Мои заметки

