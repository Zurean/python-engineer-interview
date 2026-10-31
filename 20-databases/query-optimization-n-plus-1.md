---
type: topic
title: Оптимизация запросов и N+1
tier: 1
impact: 4
effort: 2
status: todo
confidence: 0
tags: [db, performance, tier1]
updated: 2026-10-29
---

# Оптимизация запросов, N+1, пагинация

> **Зачем на собесе:** N+1 спрашивают всегда; X5, Циан и 1440 отдельно требуют «оптимизацию БД и производительности». Плюс практическая задача «эндпоинт списка тормозит».

## Минимум для собеса

- **N+1:** один запрос за списком + по запросу на каждую связь. Симптом — сотни одинаковых SELECT в логе. Лечение: eager load (`selectinload`/`joinedload`), батчинг (`WHERE id IN (...)`), DataLoader-паттерн; в тестах — `lazy='raise'` или счётчик запросов.
- **Пагинация:** `OFFSET` деградирует на больших смещениях (читает и выбрасывает). **Keyset / cursor** пагинация `WHERE (created_at, id) < (?, ?) ORDER BY created_at DESC, id DESC LIMIT n` — O(log n) с индексом. `COUNT(*)` на большой таблице — дорого: приблизительный count из `pg_class.reltuples` или не показывать total.
- **Выбирать только нужные колонки**, не `SELECT *` (особенно с jsonb/text).
- **Батчинг записи:** multi-row INSERT, `COPY`, `executemany`, upsert; транзакция вокруг батча.
- **Агрегации и аналитика — в БД**, не в Python: `GROUP BY`, window functions, CTE. Перенос фильтрации/сортировки в SQL.
- **Индексы под реальные запросы** ([postgres-indexes-explain](postgres-indexes-explain.md)): смотреть `pg_stat_statements`, искать top по total_time.
- **Кэширование** горячих чтений в Redis с TTL/инвалидацией ([redis](../50-messaging-caching/redis.md)); materialized views для тяжёлых отчётов.
- **Соединения:** пул, не открывать соединение на запрос; `PgBouncer` в transaction-mode (ограничения: нет prepared statements/`SET`/advisory locks в session-семантике).
- **Антипаттерны:** `OR` вместо `IN`, функции над индексируемыми колонками, `NOT IN` с NULL, `DISTINCT` для маскировки плохого JOIN, `SELECT ... FOR UPDATE` всей таблицы, long transactions.
- **Таймауты** на запросы (`statement_timeout`) — защита от одного плохого запроса, кладущего базу.

## Вопросы с собесов

- [ ] Что такое N+1, как найти и как исправить? → *лог SQL, eager load, батчинг, lazy='raise'*
- [ ] Эндпоинт со списком заказов отвечает 3 секунды. Твои шаги? → *лог запросов → EXPLAIN ANALYZE → N+1/индекс/OFFSET → кэш*
- [ ] Почему `OFFSET 100000` медленный и чем заменить? → *keyset-пагинация*
- [ ] Как сделать `COUNT(*)` быстрым на таблице в 50 млн строк? → *reltuples, отдельный счётчик, не показывать*
- [ ] Как вставить миллион строк быстро? → *COPY, батчи, отключить индексы/триггеры на время, транзакция*
- [ ] Что такое PgBouncer и какие ограничения у transaction pooling? → *мультиплексирование соединений; нет session state*
- [ ] `joinedload` vs `selectinload` — когда что? → *to-one через JOIN; коллекции через IN, иначе декартово произведение*
- [ ] Как мониторишь медленные запросы в проде? → *pg_stat_statements, log_min_duration_statement, auto_explain*

## Углубление

- `auto_explain` для логирования планов медленных запросов в проде.
- Window functions и `LATERAL` для «top-N per group» вместо N запросов.
- Denormalization и когда она оправдана; generated columns.
- Read replicas: роутинг чтений, лаг репликации и read-your-writes.
- Partitioning по времени для таблиц событий + `DROP PARTITION` вместо `DELETE`.
- Connection storms при деплое и как PgBouncer/пулы их гасят.

## Ресурсы

- https://use-the-index-luke.com/sql/partial-results/fetch-next-page (keyset pagination)
- https://www.postgresql.org/docs/current/pgstatstatements.html
- Документация SQLAlchemy: Relationship Loading Techniques

## Связи

[postgres-indexes-explain](postgres-indexes-explain.md) · [sqlalchemy-alembic](sqlalchemy-alembic.md) · [redis](../50-messaging-caching/redis.md) · [memory-performance](../10-python-core/memory-performance.md) · [sd-building-blocks](../80-system-design/sd-building-blocks.md)

## Мои заметки

