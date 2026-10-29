---
type: topic
title: Тулинг и проектная гигиена
tier: 1
impact: 2
effort: 1
status: todo
confidence: 0
tags: [python, tier1]
updated: 2026-10-29
---

# Тулинг и проектная гигиена

> **Зачем на собесе:** короткий разговор «как ты заводишь проект с нуля». Показывает зрелость за 3 минуты. Отдельной секции нет, но слабый ответ бросается в глаза.

## Минимум для собеса

- **Менеджер зависимостей:** `uv` (быстрый, умеет Python-версии, lock-файл, `uv run`), альтернативы — poetry, pip-tools. Всё описывается в `pyproject.toml` (PEP 621).
- **Lock-файл vs диапазоны:** `pyproject.toml` задаёт *допустимые* версии (`fastapi>=0.115,<1`), lock фиксирует *точные* версии всего дерева с хешами → воспроизводимые сборки. Lock коммитится для приложений, не для библиотек.
- **Линтер/форматтер:** `ruff` (заменяет flake8, isort, black; `ruff check`, `ruff format`). Тайпчекер: `mypy` или `pyright`.
- **pre-commit** — хуки перед коммитом (ruff, mypy, проверки секретов).
- **Структура проекта:** `src/`-layout, `tests/` рядом, `pyproject.toml`, `Dockerfile`, `Makefile`/`justfile` или `uv run` скрипты, `.env.example`.
- **Конфигурация:** pydantic-settings из env, никаких секретов в репо, 12-factor.
- **Версии Python:** поддержка актуальных (3.12/3.13), `requires-python`.
- **CI:** линт + типы + тесты на каждый PR ([[ci-cd]]).

## Вопросы с собесов

- [ ] Как устроен твой проект с нуля: менеджер, линтер, тайпчекер, где фиксируются версии, почему так? → *uv + pyproject + lock, ruff, mypy, pre-commit, CI*
- [ ] Чем lock-файл отличается от диапазонов версий в pyproject? → *допустимые vs точные; воспроизводимость*
- [ ] uv vs poetry vs pip — почему выбрал? → *скорость, единый инструмент, стандарты*
- [ ] Зачем `src/`-layout? → *тесты гоняются на установленном пакете, не на рабочей папке*
- [ ] Как хранишь конфиги и секреты? → *env, pydantic-settings, vault/secret manager, никогда в git*
- [ ] Как обновляешь зависимости и ловишь уязвимости? → *dependabot/renovate, pip-audit, регулярные обновления*

## Углубление

- Сборка пакетов: `hatchling`/`setuptools`, wheels, публикация в приватный индекс.
- Монорепо с несколькими сервисами на uv workspaces.
- Reproducible Docker-сборки с uv (`uv sync --frozen`, кэш слоёв) — см. [[docker]].
- Semantic versioning, changelog, conventional commits.

## Ресурсы

- https://docs.astral.sh/uv/ и https://docs.astral.sh/ruff/
- Python Packaging User Guide: https://packaging.python.org/

## Связи

[[typing]] · [[ci-cd]] · [[docker]] · [[pytest]]

## Мои заметки

