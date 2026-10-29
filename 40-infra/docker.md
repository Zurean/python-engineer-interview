---
type: topic
title: Docker
tier: 1
impact: 4
effort: 1
status: todo
confidence: 0
tags: [infra, tier1]
updated: 2026-10-29
---

# Docker и Docker Compose

> **Зачем на собесе:** требуют 5 из 7. Вопросы простые: «напиши Dockerfile для FastAPI», «что такое слои», «образ весит 2 ГБ — что делать». Готовится за вечер, провал выглядит плохо.

## Минимум для собеса

- **Образ vs контейнер:** образ — неизменяемые слои (файловая система + метаданные), контейнер — запущенный процесс поверх образа с writable-слоем.
- **Слои и кэш:** каждая инструкция `RUN/COPY/ADD` — слой; кэш инвалидируется с первой изменившейся инструкции вниз → сначала копировать `pyproject.toml`/lock и ставить зависимости, потом код.
- **Хороший Dockerfile для Python:** базовый `python:3.12-slim`; `uv sync --frozen --no-dev` (или `pip install --no-cache-dir`); **multi-stage** (сборочные зависимости не попадают в финальный образ); не root (`USER app`); `ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1`; `.dockerignore`; `CMD ["uvicorn", "app:app", "--host", "0.0.0.0"]` в exec-форме (сигналы доходят до процесса → graceful shutdown); `HEALTHCHECK` или probes в K8s.
- **`CMD` vs `ENTRYPOINT`**, shell vs exec форма, PID 1 и сигналы (`tini`/`--init`).
- **Compose:** сервисы (app, postgres, redis, kafka), `depends_on` с `condition: service_healthy`, volumes для данных, `env_file`, сети, профили. Для локальной разработки и интеграционных тестов.
- **Сети:** bridge по умолчанию, сервисы видят друг друга по имени; публикация портов `-p 8000:8000`.
- **Volumes vs bind mounts;** данные БД — в volume; код в dev — bind mount + reload.
- **Размер образа:** slim/alpine (alpine — musl, проблемы с wheels, осторожно), multi-stage, `--no-cache-dir`, не ставить dev-зависимости, объединять `RUN` и чистить apt-списки.
- **Безопасность:** не root, минимальный образ, не хранить секреты в образе (`ARG` тоже попадает в историю), сканирование (`trivy`), pin версий базовых образов.
- **Логи:** в stdout/stderr, `docker logs`; ресурсы: `--memory`, `--cpus`; отладка: `docker exec -it`, `docker inspect`, `docker system prune`.
- **Регистр:** теги, `latest` — не использовать в проде, тег = commit sha/версия.

## Вопросы с собесов

- [ ] Напиши Dockerfile для FastAPI-приложения. Что сделаешь, чтобы сборка была быстрой и образ маленьким? → *порядок слоёв, multi-stage, slim, no-cache*
- [ ] Чем образ отличается от контейнера? → *шаблон vs процесс*
- [ ] `CMD` vs `ENTRYPOINT`? Shell vs exec форма? → *дефолт-аргументы vs исполняемый; сигналы*
- [ ] Почему приложение не завершается по `docker stop` 10 секунд? → *shell-форма, PID 1 не получает SIGTERM*
- [ ] Как организуешь локальное окружение с БД, Redis, Kafka? → *compose, healthcheck, volumes*
- [ ] Куда пишешь логи в контейнере? → *stdout, собирает платформа*
- [ ] Почему нельзя запускать от root и как это исправить? → *USER, права на файлы*
- [ ] Alpine или slim для Python? → *slim; alpine = musl, нет wheels, медленнее*
- [ ] Как попадают секреты в контейнер? → *env/secret manager в рантайме; не ARG/ENV в образе*

## Углубление

- Устройство: namespaces, cgroups, overlayfs, OCI-спецификация; containerd vs Docker.
- BuildKit: cache mounts (`--mount=type=cache` для uv/pip), secrets при сборке, multi-platform (`buildx`).
- Distroless-образы, минимизация attack surface, SBOM.
- Reproducible builds: `uv.lock`, digest-pinning базовых образов.
- Docker в CI: кэш слоёв в registry, layer reuse.

## Ресурсы

- https://docs.docker.com/build/building/best-practices/
- uv + Docker guide: https://docs.astral.sh/uv/guides/integration/docker/
- https://pythonspeed.com/docker/ (Itamar Turner-Trauring — лучшая серия про Python в Docker)

## Связи

[[kubernetes]] · [[ci-cd]] · [[tooling]] · [[integration-testing]] · [[rag-service-project]]

## Мои заметки

