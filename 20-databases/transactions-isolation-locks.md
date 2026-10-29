---
type: topic
title: Транзакции, изоляция, блокировки
tier: 1
impact: 5
effort: 3
status: todo
confidence: 0
tags: [db, postgres, tier1]
updated: 2026-10-29
---

# Транзакции, уровни изоляции, блокировки

> **Зачем на собесе:** второй стабильный блок по БД. «Какие уровни изоляции», «что такое deadlock», «как списать деньги со счёта без гонки». Банки (Альфа, скрытый банк) спрашивают особенно пристально.

## Минимум для собеса

- **ACID:** Atomicity (всё или ничего), Consistency (инварианты), Isolation (транзакции не видят промежуточных состояний друг друга), Durability (WAL, fsync).
- **MVCC в Postgres:** читатели не блокируют писателей и наоборот; каждая строка имеет версии (`xmin`/`xmax`); отсюда мёртвые строки и VACUUM.
- **Аномалии:** dirty read, non-repeatable read, phantom read, lost update, write skew.
- **Уровни изоляции в Postgres:**
  - `READ COMMITTED` (дефолт) — каждый *запрос* видит снимок на момент своего начала. Возможны non-repeatable read и lost update при read-modify-write.
  - `REPEATABLE READ` — снимок на начало *транзакции*; при конфликте записи — ошибка сериализации (`could not serialize`), надо ретраить. Фантомов в Postgres на этом уровне нет (строже стандарта).
  - `SERIALIZABLE` — SSI, ловит write skew; ещё больше ретраев.
  - `READ UNCOMMITTED` в Postgres = READ COMMITTED.
- **Lost update и как бороться:** `SELECT ... FOR UPDATE` (пессимистично), атомарный `UPDATE ... SET balance = balance - 100 WHERE balance >= 100` (лучший вариант), optimistic locking через `version`-колонку, REPEATABLE READ + ретрай.
- **Блокировки:** уровень строки (`FOR UPDATE`, `FOR NO KEY UPDATE`, `FOR SHARE`) и уровень таблицы (DDL берёт `ACCESS EXCLUSIVE` → блокирует всё). `FOR UPDATE SKIP LOCKED` — паттерн очереди на Postgres. `NOWAIT`.
- **Deadlock:** две транзакции ждут друг друга; Postgres обнаруживает и убивает одну. Профилактика — брать блокировки в одном порядке (например, сортировать id), короткие транзакции.
- **Advisory locks** — прикладные блокировки по ключу (`pg_advisory_xact_lock`), для «только один воркер делает X».
- **Долгие транзакции** — зло: держат блокировки, мешают VACUUM, раздувают таблицы. `idle in transaction` — типичный баг ORM-кода. `statement_timeout`, `lock_timeout`, `idle_in_transaction_session_timeout`.
- **Savepoints**, вложенные транзакции в ORM — на самом деле savepoints.

## Вопросы с собесов

- [ ] Какие уровни изоляции есть, какой дефолт в Postgres и что он разрешает? → *RC; non-repeatable read, lost update*
- [ ] Два запроса одновременно списывают деньги с одного счёта. Как не уйти в минус? → *атомарный UPDATE с условием / FOR UPDATE / optimistic lock*
- [ ] Что такое deadlock, как Postgres его обрабатывает и как избегать? → *детектор, ошибка одной; порядок блокировок*
- [ ] Что такое MVCC и при чём тут VACUUM? → *версии строк; мёртвые версии надо чистить*
- [ ] Чем REPEATABLE READ в Postgres отличается от стандарта? → *нет фантомов; ошибки сериализации*
- [ ] Что такое write skew и какой уровень его ловит? → *SERIALIZABLE*
- [ ] Как сделать очередь задач на Postgres? → *FOR UPDATE SKIP LOCKED*
- [ ] Что такое `idle in transaction` и почему это плохо? → *держит блокировки/снимок; таймаут*
- [ ] Миграция `ALTER TABLE ... ADD COLUMN ... DEFAULT` заблокировала прод. Почему? → *ACCESS EXCLUSIVE; с 11-й DEFAULT без rewrite, но lock всё равно; lock_timeout*
- [ ] Транзакция в SQLAlchemy: где начинается, где коммитится, что с async-сессией? → *см. [[sqlalchemy-alembic]]*

## Углубление

- SSI в деталях: predicate locks, false positives, `pg_serialization_failure`.
- Lock queue: почему ожидающий `ACCESS EXCLUSIVE` блокирует даже новые `SELECT` (ставится в очередь перед ними) — главная причина «миграция положила прод».
- WAL, checkpoints, `synchronous_commit`, репликация (streaming, logical) и read replicas с лагом → stale reads.
- Two-phase commit и почему распределённые транзакции между сервисами — плохая идея (→ saga, outbox в [[resilience-patterns]]).
- Transaction ID wraparound и зачем агрессивный autovacuum.

## Ресурсы

- Документация: https://www.postgresql.org/docs/current/transaction-iso.html и explicit-locking
- Martin Kleppmann, «Designing Data-Intensive Applications», глава 7 (Transactions) — обязательно
- Егор Рогов, «PostgreSQL изнутри» (MVCC, блокировки)

## Связи

[[postgres-indexes-explain]] · [[zero-downtime-migrations]] · [[sqlalchemy-alembic]] · [[resilience-patterns]] · [[redis]]

## Мои заметки

