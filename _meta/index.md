---
type: meta
title: Карта проекта и память агента
updated: 2026-10-31
---

# Карта проекта и память агента

Эта заметка — входная точка для понимания устройства проекта и продолжения работы в новой сессии. Пользовательская главная страница — [Home](../Home.md).

## Быстрый старт для агента

1. Прочитать `AGENTS.md` в корне проекта.
2. Открыть [state](state.md) и продолжить незавершённую работу.
3. Проверить приоритеты и критерии в [tasks](tasks.md).
4. Для теоретических материалов следовать [theory-note-template](theory-note-template.md).
5. Перед коммитом синхронизировать [tasks](tasks.md), эту карту и [state](state.md).

## Память и инструкции

- `AGENTS.md` — обязательные правила работы агента, безопасность редактирования и рабочий цикл.
- [tasks](tasks.md) — активные и отложенные задачи проекта.
- [state](state.md) — последнее изменение, текущая работа, следующий шаг и блокеры.
- [theory-note-template](theory-note-template.md) — назначение, структура и критерии качества теоретических конспектов.
- [conventions](conventions.md) — общие конвенции структуры и редактирования волта.
- [how-to-use](how-to-use.md) — пользовательский цикл обучения.
- [sources](sources.md) — происхождение программы и материалов.

## Пользовательская навигация и управление прогрессом

- [Home](../Home.md) — главная страница волта.
- [pareto-plan](../00-strategy/pareto-plan.md) — порядок прохождения программы.
- [sprint](sprint.md) — текущий фокус на 1–2 недели.
- [progress](progress.md) — ручная сводка прохождения тем.
- [log](log.md) — журнал занятий и собеседований.

## Стратегия подготовки

- [target-profile](../00-strategy/target-profile.md) — целевая позиция и рынок.
- [vacancy-analysis](../00-strategy/vacancy-analysis.md) — основание приоритетов программы.
- [interview-formats](../00-strategy/interview-formats.md) — структура собеседований.
- [self-presentation](../00-strategy/self-presentation.md) — рассказ о себе.
- [stories](../00-strategy/stories.md) — поведенческие истории по STAR.

## Карта учебных направлений

| Направление | Хаб | Роль |
|---|---|---|
| Python | [_python-core](../10-python-core/_python-core.md) | Язык, конкурентность, типизация, память и инструменты |
| Базы данных | [_databases](../20-databases/_databases.md) | PostgreSQL, SQLAlchemy, транзакции, индексы и миграции |
| Backend design | [_backend-design](../30-backend-design/_backend-design.md) | FastAPI, API, архитектура, надёжность и безопасность |
| Инфраструктура | [_infra](../40-infra/_infra.md) | Docker, Kubernetes, CI/CD и observability |
| Сообщения и кэш | [_messaging-caching](../50-messaging-caching/_messaging-caching.md) | Kafka, Redis и очереди задач |
| Тестирование | [_testing](../60-testing/_testing.md) | pytest и интеграционные тесты |
| LLM engineering | [_llm-engineering](../70-llm-engineering/_llm-engineering.md) | RAG, tool calling, evals и AI-assisted development |
| System design | [_system-design](../80-system-design/_system-design.md) | Фреймворк, строительные блоки и практические кейсы |
| Live coding | [_live-coding](../90-live-coding/_live-coding.md) | Задачи, алгоритмическая база и code review |
| Дополнительно | [_optional](../95-optional/_optional.md) | Cloud, Terraform, Go, ML, vector DB и английский |

## Сквозная практика

- [rag-service-project](../capstone/rag-service-project.md) — проект, связывающий большую часть тем уровней 1–2 и создающий материал для портфолио и рассказа на собеседовании.

## Текущий маршрут развития содержимого

1. Пользователь проходит пилотный теоретический модуль [asyncio](../10-python-core/asyncio.md); первая обратная связь по ментальной модели внесена в материал.
2. Собрать оставшуюся обратную связь по глубине, объёму, практике и навигации, затем скорректировать или утвердить [theory-note-template](theory-note-template.md).
3. После утверждения формата последовательно наполнять остальные темы по [pareto-plan](../00-strategy/pareto-plan.md).
4. После стабилизации формата вернуться к отложенной задаче развития структуры из [tasks](tasks.md).

## Правило актуальности карты

Обновлять эту заметку, когда появляется новый документ верхнего уровня, меняется роль существующего документа, добавляется новый учебный хаб либо изменяется общий маршрут работы. Не перечислять здесь каждую дочернюю теоретическую заметку: её место должно быть видно из карточки родительской темы.

## Контроль версий

Git хранит содержательные заметки волта, `AGENTS.md` и проектные метадокументы. Внутренние связи используют стандартные относительные Markdown-ссылки, совместимые с Obsidian, VS Code и GitHub. Локальная конфигурация `.obsidian/`, установленные плагины, каталоги Copilot и локальные агентные навыки исключены через `.gitignore`, чтобы diff отражал изменения учебного проекта.
