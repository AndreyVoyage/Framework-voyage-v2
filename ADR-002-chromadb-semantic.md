# ADR-002: ChromaDB для Semantic Memory

## Статус
Accepted

## Контекст

AI-агенты framework'а должны находить релевантные решения из прошлого. Пример: "Почему отказались от UUID?" → найти все related decisions.

Требования:
- Semantic search (по смыслу, не по ключевым словам)
- Локальное хранение (solo-dev, нет команды)
- Простота настройки
- Возможность миграции на pgvector в будущем

Варианты:
- **ChromaDB:** локальный, zero-config, embeddings из коробки
- **pgvector (PostgreSQL):** unified БД, но требует настройки
- **Pinecone/Weaviate:** cloud, платно, overkill
- **Keyword search (SQLite):** просто, но нет semantic understanding

## Решение

**ChromaDB как primary semantic store. SQLite keyword fallback.**

Архитектура:
- `VectorStore` (ABC) — абстракция
- `ChromaDBStore` — реализация через `chromadb.Client`
- `KeywordFallback` — SQLite с `LIKE` search, если ChromaDB недоступен

Коллекции ChromaDB:
- `decisions` — ADR и архитектурные решения
- `errors` — ошибки и их решения
- `implementation` — код и паттерны
- `plans` — планы и спецификации
- `architecture` — общая архитектура

## Последствия

### Положительные
- Semantic search работает из коробки (default embedding: all-MiniLM-L6-v2)
- Локальное хранение — никаких API keys
- Абстракция VectorStore позволит перейти на pgvector без переписывания кода

### Отрицательные
- ChromaDB тяжёлая зависимость (~100MB)
- Embedding модель загружается при первом запуске
- Не масштабируется на кластер (для solo-dev не критично)

## Миграция на pgvector (v4.1+)

```python
# Сегодня
semantic = ChromaDBStore()

# Завтра (без изменений в остальном коде)
semantic = PGVectorStore(dsn="postgresql://...")
```

## Связанные ADR

- ADR-001 (PostgreSQL) — unified БД в будущем
- ADR-003 (tree-sitter) — AST индексация тоже требует local storage

---
**Дата:** 2026-05-21 | **Автор:** Human (AndreyVoyage) + AI Architect
