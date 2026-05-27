# ADR-001: PostgreSQL для Event Store

## Статус
Accepted

## Контекст

Voyage Framework v4.0 требует надёжного хранилища для append-only event log. Это единственный источник правды (source of truth) — все markdown-файлы (PLAN.md, ERRORS.md, DECISIONS.md) являются materialized views.

Требования к хранилищу:
- Append-only (никаких UPDATE/DELETE)
- ACID транзакции
- Индексация по event_type, phase, micro_phase, timestamp, correlation_id
- Возможность replay всех событий в хронологическом порядке
- Backup и export в human-readable формат (JSONL)

Варианты:
- **PostgreSQL:** production-grade, WAL, индексы, масштабируемость
- **SQLite:** zero-config, embedded, WAL mode доступен
- **JSONL files:** просто, но нет индексов и ACID
- **MongoDB:** overkill для одного разработчика

## Решение

**PostgreSQL как primary backend. SQLite WAL как fallback.**

При инициализации EventEngine:
1. Пробуем подключиться к PostgreSQL (DSN из `VOYAGE_DB__POSTGRESQL_DSN` или `.env`)
2. Если connection failed → логируем warning → переключаемся на SQLite (`PRAGMA journal_mode=WAL`)
3. SQLite файл: `.voyage_events.db` в project root

## Последствия

### Положительные
- PostgreSQL на VPS (212.109.198.69) даёт durability и remote access
- SQLite fallback позволяет работать локально без настройки БД
- Одинаковая схема для обоих backend'ов
- JSONL backup для human-readable архива

### Отрицательные
- Двойная реализация DBBackend (PostgreSQLBackend + SQLiteBackend)
- PostgreSQL требует отдельного сервера (пока не оплачен)
- Миграции схемы придётся поддерживать для двух БД

## Альтернативы

- **Только SQLite:** проще, но нет remote access, не production-grade
- **Только PostgreSQL:** если VPS не оплачен — framework не работает
- **JSONL + in-memory индексы:** слишком медленно при >10k событий

## Связанные ADR

- ADR-002 (ChromaDB) — semantic memory использует отдельное хранилище
- ADR-004 (Dual-Location) — framework lives в SkillTracer repo

---
**Дата:** 2026-05-21 | **Автор:** Human (AndreyVoyage) + AI Architect
