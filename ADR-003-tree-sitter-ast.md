# ADR-003: tree-sitter для AST Indexing

## Статус
Accepted

## Контекст

AI-агенты должны понимать структуру кода проекта SkillTracer:
- Python backend (FastAPI, Aiogram, SQLAlchemy)
- TypeScript/React frontend (React SPA)

Требования:
- Парсинг Python и TypeScript
- Построение графа зависимостей (imports, inheritance, calls)
- Impact analysis: "что сломается, если поменять этот файл?"
- Route discovery: найти все API endpoints

Варианты:
- **tree-sitter:** быстрый, multi-language, инкрементальный парсинг
- **Python `ast` модуль:** только Python, стандартная библиотека
- **TypeScript compiler API:** только TS, тяжёлый
- **LSP (Language Server Protocol):** универсально, но overkill

## Решение

**tree-sitter для обоих языков через unified `ASTManager` facade.**

Структура:
- `ASTManager` — unified API
- `ASTPython` — tree-sitter-python grammar
- `ASTTypeScript` — tree-sitter-typescript grammar

Индексируем:
- **Python:** classes, functions, async functions, FastAPI routes (по декораторам `@app.get`, `@router.post`)
- **TypeScript:** components, hooks, API routes, services

## Последствия

### Положительные
- Один инструмент для обоих языков
- Быстрый парсинг (tree-sitter написан на C, биндинги на Python)
- Инкрементальный: можно перепарсить только изменённые файлы
- Impact analysis помогает Reviewer оценивать риски

### Отрицательные
- Требует установки grammar binaries (tree-sitter-python, tree-sitter-typescript)
- Не разрешает импорты (не знает, что `from app.models import User` → `User` это `app/models.py:User`)
- Для полного symbol resolution нужен LSP (v4.2+)

## Roadmap

- **v4.0:** tree-sitter AST + basic dependency extraction
- **v4.1:** Incremental re-indexing (по mtime)
- **v4.2:** LSP integration для полного symbol resolution

## Связанные ADR

- ADR-001 (PostgreSQL) — AST graph можно хранить в БД для быстрого query
- ADR-004 (Dual-Location) — framework lives в SkillTracer, AST индексирует оба языка

---
**Дата:** 2026-05-21 | **Автор:** Human (AndreyVoyage) + AI Architect
