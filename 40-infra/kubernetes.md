---
type: topic
title: Kubernetes (уровень пользователя)
tier: 1
impact: 3
effort: 3
status: todo
confidence: 0
tags: [infra, tier1]
updated: 2026-10-29
---

# Kubernetes на уровне пользователя

> **Зачем на собесе:** 5 из 7 вакансий, но «на уровне пользователя»: задеплоить сервис, прочитать логи, понять, почему под в CrashLoopBackOff. Не нужно администрировать кластер. Два вечера с kind/k3s дают достаточно.

## Минимум для собеса

- **Объекты:** Pod (один+ контейнер, эфемерный), Deployment (управляет ReplicaSet → Pod-ами, rolling update), Service (стабильный адрес и балансировка: ClusterIP/NodePort/LoadBalancer), Ingress (HTTP-роутинг снаружи), ConfigMap/Secret (конфигурация → env или файлы), Namespace, Job/CronJob (миграции, периодические задачи), StatefulSet (БД, брокеры — обычно не ваше дело), HPA (автомасштабирование по CPU/метрикам).
- **Probes:** liveness (перезапустить, если завис), readiness (убрать из Service, если не готов — БД недоступна, прогрев), startup (для медленного старта). Неправильные probes = каскадные перезапуски.
- **Ресурсы:** `requests` (для планирования) и `limits` (CPU throttling, OOMKill по памяти). Python-сервис: память растёт — ставить limit с запасом, следить за RSS.
- **Rolling update:** `maxSurge`/`maxUnavailable`, новая версия поднимается рядом со старой → схема БД должна быть совместима с обеими ([[zero-downtime-migrations]]). Graceful shutdown: SIGTERM → `terminationGracePeriodSeconds` → приложение дорабатывает in-flight запросы; `preStop` hook.
- **Конфигурация и секреты:** env из ConfigMap/Secret; секреты — не в git в открытом виде (sealed-secrets, external-secrets, Vault).
- **Миграции при деплое:** Job или init-container, один раз, до раскатки Deployment; не из каждого пода.
- **Отладка:** `kubectl get pods`, `describe pod` (events: ImagePullBackOff, OOMKilled, probe failed), `logs -f --previous`, `exec -it`, `port-forward`, `top pods`. CrashLoopBackOff → смотреть логи предыдущего контейнера.
- **Helm / Kustomize:** шаблонизация манифестов; values для окружений. Знать, что это, уметь прочитать chart.
- **Логи и метрики:** stdout → сборщик (Loki/ELK); `/metrics` → Prometheus через ServiceMonitor ([[observability]]).
- **Сеть на пальцах:** Pod получает IP, Service — виртуальный IP + DNS (`svc.namespace.svc.cluster.local`), Ingress-контроллер (nginx/traefik) терминирует HTTP.

## Вопросы с собесов

- [ ] Что такое Pod, Deployment, Service, Ingress — как они связаны? → *контейнеры → реплики → адрес → внешний HTTP*
- [ ] Чем liveness отличается от readiness, что будет, если перепутать? → *рестарт vs трафик; каскад*
- [ ] Под в CrashLoopBackOff. Твои действия? → *describe, logs --previous, причины: конфиг, миграция, OOM*
- [ ] Что такое requests/limits и что происходит при превышении? → *throttling / OOMKilled*
- [ ] Как происходит деплой новой версии без даунтайма? → *rolling update, readiness, graceful shutdown, совместимая схема*
- [ ] Где и как запускаешь миграции БД в K8s? → *Job/init-container до раскатки*
- [ ] Как приложение получает конфиг и секреты? → *ConfigMap/Secret → env/volume*
- [ ] Как масштабируешь Python-сервис в K8s? → *реплики/HPA; воркеры внутри пода ≈ CPU limit*
- [ ] Как попасть в под и посмотреть, что он делает? → *exec, port-forward, py-spy dump*
- [ ] Helm — пользовался? Что такое values? → *шаблоны + параметры окружений*

## Углубление

- Устройство control plane (apiserver, etcd, scheduler, controller-manager), kubelet, CNI/CSI — обзорно.
- PodDisruptionBudget, anti-affinity, topology spread для устойчивости.
- Service mesh (Istio/Linkerd): mTLS, ретраи, circuit breaking на уровне сети.
- GitOps (ArgoCD/Flux), environments через Kustomize overlays.
- Vertical/Horizontal autoscaling по кастомным метрикам (KEDA по лагу Kafka).
- Операторы и CRD — что это.

## Ресурсы

- https://kubernetes.io/docs/concepts/ (Workloads, Services, Configuration)
- kind: https://kind.sigs.k8s.io/ — локальный кластер за 2 минуты
- Marko Lukša, «Kubernetes in Action» (главы 1–9 достаточно)

## Связи

[[docker]] · [[ci-cd]] · [[observability]] · [[zero-downtime-migrations]] · [[resilience-patterns]] · [[rag-service-project]]

## Мои заметки

