---
type: topic
title: Индексы и EXPLAIN
tier: 1
impact: 5
effort: 3
status: todo
confidence: 0
tags: [db, postgres, tier1]
updated: 2026-10-29
---

# PostgreSQL: индексы и EXPLAIN

> **Зачем на собесе:** «какие индексы знаешь», «почему индекс не используется», «прочитай план» — стабильный набор в любом backend-собесе. Это самый дешёвый способ выглядеть сеньорно в блоке БД.

## Минимум для собеса

- **B-tree** — дефолт: равенство, диапазоны, сортировка, `LIKE 'abc%'` (префикс). Не работает для `LIKE '%abc'`, функций над колонкой (`lower(email)`) без функционального индекса, неявных приведений типов.
- **Составной индекс** `(a, b)`: работает для условий по `a` и по `a, b`, но не по одному `b` (правило левого префикса). Порядок колонок: сначала равенства, потом диапазон.
- **Покрывающий индекс** (`INCLUDE`) → Index Only Scan, если visibility map свежая (после VACUUM).
- **Частичный индекс** `WHERE status = 'active'` — маленький и быстрый для горячей выборки.
- **Функциональный индекс** `ON lower(email)`.
- **GIN** — для `jsonb`, массивов, полнотекстового поиска, `pg_trgm` (`LIKE '%abc%'`). **GiST** — геометрия, диапазоны. **BRIN** — огромные append-only таблицы по времени. **Hash** — только равенство, редко нужен.
- **Уникальный индекс** = ограничение + индекс. Первичный ключ — всегда индекс. FK — индекс *не* создаётся автоматически → частая причина медленных JOIN и DELETE.
- **Цена индексов:** замедляют INSERT/UPDATE/DELETE, занимают место, bloat. Не индексировать всё подряд.
- **EXPLAIN** показывает план, **EXPLAIN ANALYZE** — реально выполняет и показывает фактические времена и строки. `BUFFERS` — чтение из кэша/диска. Читать снизу вверх и изнутри наружу.
- **Узлы плана:** Seq Scan, Index Scan, Index Only Scan, Bitmap Heap Scan (много строк через индекс), Nested Loop (мало строк, есть индекс), Hash Join (большие наборы, equi-join), Merge Join (отсортированные), Sort, HashAggregate.
- **Почему индекс не используется:** мало строк в таблице (seq scan дешевле); низкая селективность (возвращается >5–10 % таблицы); устаревшая статистика (`ANALYZE`); функция над колонкой; несовпадение типов; `OR` по разным колонкам; `random_page_cost` на SSD слишком высокий.
- **Статистика:** `ANALYZE`, `pg_stats`, планировщик ошибается в оценке rows → плохой join. Autovacuum делает и vacuum, и analyze.
- `pg_stat_statements` — какие запросы жрут время в проде. `pg_stat_user_indexes` — неиспользуемые индексы.

## Вопросы с собесов

- [ ] Какие типы индексов в Postgres знаешь и когда какой? → *B-tree, GIN, GiST, BRIN, Hash + частичные/функциональные/покрывающие*
- [ ] Есть индекс `(a, b)`. Поможет ли он запросу `WHERE b = ?` → *нет, левый префикс*
- [ ] Запрос медленный, индекс есть, но Seq Scan. Почему? → *селективность, статистика, функция, тип, размер таблицы*
- [ ] Чем EXPLAIN отличается от EXPLAIN ANALYZE? Что такое BUFFERS? → *план vs реальное выполнение; кэш/диск*
- [ ] Nested Loop vs Hash Join vs Merge Join — когда планировщик выбирает что? → *размер наборов, наличие индекса, сортировка*
- [ ] Что такое Index Only Scan и почему он может всё равно ходить в таблицу? → *visibility map, heap fetches*
- [ ] Индекс на FK создаётся автоматически? → *нет, только на PK/UNIQUE*
- [ ] `LIKE '%abc%'` — как индексировать? → *pg_trgm + GIN/GiST*
- [ ] Как найти неиспользуемые индексы и самые тяжёлые запросы в проде? → *pg_stat_user_indexes, pg_stat_statements*
- [ ] Что такое bloat и что с ним делать? → *MVCC мёртвые строки; VACUUM, REINDEX CONCURRENTLY, pg_repack*
- [ ] Как создать индекс на большой таблице без блокировки записи? → *CREATE INDEX CONCURRENTLY, нюансы при ошибке (invalid index)*

## Углубление

- Устройство B-tree в Postgres: страницы, fillfactor, HOT updates и почему индексы на обновляемых колонках дороги.
- Оценка стоимости: `seq_page_cost`, `random_page_cost`, `effective_cache_size`, `work_mem` (sort на диске), `cpu_*`.
- Multicolumn statistics (`CREATE STATISTICS`) для коррелированных колонок.
- Partitioning: declarative partitioning, partition pruning, когда помогает, а когда вредит.
- Полнотекстовый поиск: `tsvector`, `to_tsquery`, GIN, ранжирование.
- Чтение планов сложных запросов: CTE (materialized/not с 12-й), подзапросы, window functions, `LATERAL`.

> Запросить углубление: «углуби *Углубление* в `postgres-indexes-explain` про партиционирование / устройство B-tree».

## Ресурсы

- https://use-the-index-luke.com/ — обязательно, целиком (несколько вечеров)
- Документация Postgres: https://www.postgresql.org/docs/current/using-explain.html
- https://explain.dalibo.com/ и https://explain.depesz.com/ — визуализация планов
- Егор Рогов, «PostgreSQL изнутри» (главы про индексы) — для фазы гринда

## Связи

[[transactions-isolation-locks]] · [[query-optimization-n-plus-1]] · [[sqlalchemy-alembic]] · [[sd-building-blocks]]

## Мои заметки

