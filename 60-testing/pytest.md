---
type: topic
title: pytest и unit-тесты
tier: 1
impact: 4
effort: 2
status: todo
confidence: 0
tags: [testing, python, tier1]
updated: 2026-10-29
---

# pytest, фикстуры, моки

> **Зачем на собесе:** «как тестируешь», «что мокаешь», «как тестируешь async-код и FastAPI». Вопросы практические, ответы короткие — нужно иметь свои примеры.

## Минимум для собеса

- **Фикстуры:** `@pytest.fixture`, области (`function`/`module`/`session`), `yield` для teardown, `conftest.py` для общих, `autouse`, фабрики-фикстуры, зависимость фикстур друг от друга.
- **Параметризация:** `@pytest.mark.parametrize` — таблица случаев вместо копипасты; `ids` для читаемых имён.
- **Маркеры:** `@pytest.mark.slow`, `-m "not slow"`, `skipif`, `xfail`.
- **Моки:** `unittest.mock` / `pytest-mock` (`mocker.patch`). Патчить там, где *используется*, а не где определено. `AsyncMock` для корутин. `spec=`/`autospec` чтобы мок не принимал любые вызовы.
- **Что мокать, а что нет:** мокать внешние границы (HTTP-API, время, случайность, очереди), не мокать свою бизнес-логику и БД (для БД — реальная в контейнере, см. [integration-testing](integration-testing.md)). Когда моки начинают вредить: тест проверяет взаимодействие, а не поведение; ломается при рефакторинге; «зелёный тест, красный прод».
- **Фейки вместо моков:** in-memory репозиторий, фейковый шлюз — проще и устойчивее к рефакторингу.
- **Async:** `pytest-asyncio` (`asyncio_mode = auto`) или `anyio`; async-фикстуры; одна event loop на сессию при необходимости.
- **FastAPI:** `httpx.AsyncClient(transport=ASGITransport(app=app))`, `app.dependency_overrides[get_session] = ...`, lifespan через `LifespanManager` (asgi-lifespan).
- **HTTP-моки:** `respx` (для httpx), `pytest-httpx`, `responses` (requests).
- **Время:** `freezegun`/`time-machine`; не `sleep` в тестах.
- **Структура:** AAA (arrange/act/assert), один сценарий на тест, имена-предложения, тест-данные через фабрики (`factory_boy`, `polyfactory`).
- **Покрытие:** `pytest-cov`; процент — не цель, цель — покрыть граничные случаи и критичные пути.
- **Property-based:** `hypothesis` — для парсеров, сериализации, инвариантов (знать, что это и где применимо).
- **Flaky-тесты:** причины (время, порядок, общая БД, сеть), как лечить (изоляция, `pytest-randomly`).

## Вопросы с собесов

- [ ] Что ты мокаешь, а что нет? Когда моки начинают вредить? → *границы; тесты на взаимодействие хрупкие*
- [ ] Как тестируешь код с БД: реальная база, SQLite или моки? Плюсы и минусы. → *реальная в контейнере; SQLite — другой диалект; моки — бесполезны для SQL*
- [ ] Как тестируешь async-код и FastAPI-эндпоинты? → *pytest-asyncio, AsyncClient + ASGITransport, overrides*
- [ ] Что такое фикстура, какие области видимости? → *function/module/session, yield-teardown*
- [ ] Где патчить: `mocker.patch("module.requests.get")` или `("requests.get")`? → *где используется*
- [ ] Как тестируешь код, зависящий от времени / внешнего API? → *freezegun, respx*
- [ ] Что такое пирамида тестов и как выглядит в твоём проекте? → *много unit, меньше интеграционных, мало e2e; честно*
- [ ] Тест флакает. Твои действия? → *найти причину: изоляция, порядок, время; не retry*
- [ ] Что такое hypothesis и где бы применил? → *property-based; парсеры, round-trip*
- [ ] TDD практикуешь? → *честно; когда полезно: багфиксы, чистая логика*

## Углубление

- Архитектура тестируемого кода: зависимости наружу, чистое ядро, «functional core, imperative shell».
- Contract-тесты между сервисами (pact), snapshot-тесты (`syrupy`).
- Mutation testing (`mutmut`) — проверка качества тестов.
- Тестирование consumer'ов Kafka и фоновых задач.
- Параллельный запуск `pytest-xdist` и изоляция БД по воркерам.

## Ресурсы

- https://docs.pytest.org/ (How-to: fixtures, parametrize, monkeypatch)
- Brian Okken, «Python Testing with pytest» (2nd ed.)
- cosmicpython.com, главы про тестирование и фейки

## Связи

[integration-testing](integration-testing.md) · [fastapi](../30-backend-design/fastapi.md) · [architecture-layers](../30-backend-design/architecture-layers.md) · [ci-cd](../40-infra/ci-cd.md) · [code-review-exercise](../90-live-coding/code-review-exercise.md)

## Мои заметки

