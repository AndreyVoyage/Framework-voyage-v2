# PROMPT: Анализ Voyage Framework v4.0 для Kimi Code

> **Назначение:** Этот промпт предназначен для Kimi Code в VS Code.  
> **Задача:** Проанализировать архитектуру фреймворка, найти пробелы и предложить улучшения.  
> **Контекст:** Solo-разработчик (AndreyVoyage) ведёт проект SkillTracer. Фреймворк создаётся как AI-Native Engineering OS.  
> **Phase:** Phase 1 — Foundation & Brain (в разработке).

---

## 1. Что тебе нужно проанализировать

Ты получишь следующие файлы (загрузи их в workspace Kimi Code):

### Обязательные файлы для анализа:
1. **VOYAGE_FRAMEWORK_MASTER_DOCUMENT_v4.0b.md** — мастер-документ (архитектура, компоненты, фазы)
2. **ADR-INDEX.md** — реестр архитектурных решений
3. **ADR-001-postgresql-events.md** — PostgreSQL для Event Store
4. **ADR-002-chromadb-semantic.md** — ChromaDB для Semantic Memory
5. **ADR-003-tree-sitter-ast.md** — tree-sitter для AST Indexing
6. **ADR-004-dual-location.md** — Dual-Location Strategy
7. **ADR-005-project-isolation.md** — Project Isolation & Multi-Project Core
8. **ROLE-INDEX.md** — реестр ролей агентов
9. **ROLE-001-qa-engineer.md** — QA Engineer (Planned)
10. **ROLE-002-security-engineer.md** — Security Engineer (Planned)
11. **TECH-001-appsec-stack.md** — AppSec технологический реестр
12. **FUTURE_ROLES_TECH_REGISTRY.md** — сводная дорожная карта
13. **RULES.md** — живой документ правил
14. **INSTRUCTIONS.md** — инструкции по разработке
15. **LANGGRAPH_INTEGRATION_ANALYSIS.md** — анализ LangGraph
16. **FRAMEWORK_SCOPE_ANALYSIS.md** — анализ применимости фреймворка

### Если есть исходный код проекта:
17. **Любые .py файлы из `voyage_framework/`** — для анализа реализации
18. **`pyproject.toml`** — зависимости
19. **`tests/`** — тесты

---

## 2. Структура твоего анализа

Проведи анализ по следующим разделам. Для каждого раздела дай оценку: ✅ (хорошо), ⚠️ (есть пробелы), ❌ (критично).

### Раздел A: Архитектурная целостность
- [ ] Все ADR согласованы между собой? Есть ли конфликты?
- [ ] ADR-005 (Project Isolation) интегрирован в мастер-документ?
- [ ] Есть ли дублирование информации между ADR и мастер-документом?
- [ ] Все компоненты из раздела 4 мастер-документа имеют соответствующие файлы в структуре проекта?

### Раздел B: Реализация Phase 1 (10 AC)
- [ ] AC 1: Core Models — есть ли `core/models.py`? Все модели определены?
- [ ] AC 2: Event Engine — есть ли `core/event_engine.py`? PostgreSQL + SQLite fallback?
- [ ] AC 3: Storage — есть ли `core/storage.py`? Atomic writes, frontmatter, journal rotation?
- [ ] AC 4: Security Sandbox — есть ли `security/sandbox.py`? L1-L5 уровни защиты?
- [ ] AC 5: Policy + Audit + Approval — есть ли `security/policy.py`, `audit.py`, `approval.py`?
- [ ] AC 6: Tool Engine + Adapters — есть ли `tools/engine.py`, `registry.py`, адаптеры?
- [ ] AC 7: 4-tier Memory + Semantic — есть ли `memory/context_tiers.py`, `memory/semantic.py`?
- [ ] AC 8: AST Manager — есть ли `memory/ast_manager.py`? tree-sitter Python + TypeScript?
- [ ] AC 9: Specs — есть ли `specs/adr.py`, `plan.py`, `task_generator.py`, `tracker.py`?
- [ ] AC 10: Self-Improving Engine — есть ли `self_improving/engine.py`? Auto-rule generation?

### Раздел C: Расширяемость
- [ ] RoleRegistry позволяет добавлять роли без переписывания runtime?
- [ ] OrchestratorAdapter stub создан для LangGraph интеграции?
- [ ] SandboxBackend ABC + Strategy Pattern реализованы?
- [ ] VectorStore abstraction позволяет мигрировать с ChromaDB на pgvector?

### Раздел D: Безопасность
- [ ] Все 5 уровней Sandbox реализованы?
- [ ] PyPI validation для `pip install`?
- [ ] RAG Guard для semantic memory?
- [ ] Path traversal protection?
- [ ] Approval flow для dangerous tier?

### Раздел E: Тестируемость
- [ ] Есть ли `tests/` директория?
- [ ] Есть ли `conftest.py`?
- [ ] Покрытие тестами для каждого модуля?
- [ ] CI/CD pipeline (хотя бы базовый GitHub Actions)?

### Раздел F: Документация
- [ ] README.md с инструкцией по установке?
- [ ] ADR-INDEX актуален?
- [ ] ROLE-INDEX актуален?
- [ ] RULES.md содержит ≥5 правил?
- [ ] INSTRUCTIONS.md понятен для новичка?

---

## 3. Формат ответа

Для каждого раздела (A–F):

```markdown
### Раздел X: Название

**Оценка:** ✅ / ⚠️ / ❌

**Найденные проблемы:**
1. ...
2. ...

**Рекомендации:**
1. ...
2. ...

**Приоритет:** High / Medium / Low
```

В конце:

```markdown
## Сводный рейтинг

| Критерий | Оценка | Приоритет |
|----------|--------|-----------|
| Архитектурная целостность | ✅/⚠️/❌ | ... |
| Реализация Phase 1 | ✅/⚠️/❌ | ... |
| Расширяемость | ✅/⚠️/❌ | ... |
| Безопасность | ✅/⚠️/❌ | ... |
| Тестируемость | ✅/⚠️/❌ | ... |
| Документация | ✅/⚠️/❌ | ... |

**Общая оценка:** X/6 ✅, Y/6 ⚠️, Z/6 ❌

**Топ-3 приоритета для доработки:**
1. ...
2. ...
3. ...
```

---

## 4. Что делать после анализа

1. **Если оценка ≥4 ✅** — Phase 1 почти готова. Остались мелкие доработки.
2. **Если оценка 2–3 ✅** — есть серьёзные пробелы. Сгенерируй TASK.md для каждого ❌.
3. **Если оценка ≤1 ✅** — нужно генерировать foundation с нуля. Используй System Prompt для Phase 1.

---

## 5. Пример хорошего анализа (фрагмент)

```markdown
### Раздел B: Реализация Phase 1 (10 AC)

**Оценка:** ⚠️

**Найденные проблемы:**
1. **AC 4 (Security Sandbox):** Есть `security/sandbox.py`, но L5 (Dangerous Tier) не требует approval — команда просто блокируется. Нужен ApprovalRequest + `.voyage_approval_pending.json`.
2. **AC 7 (Semantic Memory):** `memory/semantic.py` использует ChromaDB, но нет SQLite fallback при недоступности ChromaDB.
3. **AC 10 (Self-Improving Engine):** `self_improving/engine.py` — stub, не реализована логика pattern analysis и auto-rule generation.

**Рекомендации:**
1. Добавить `ApprovalQueue` в `security/approval.py` и интегрировать с `SecureExecutor`.
2. Добавить `KeywordFallback` в `memory/semantic.py` (SQLite LIKE search).
3. Реализовать `SelfImprovingEngine.process_error()` с pattern extraction и rule deduplication.

**Приоритет:** High
```

---

## 6. Контекст проекта для понимания

**Проект:** SkillTracer — платформа трекинга жизни в кругу близких людей.
**Стек:** Python (aiogram 3, FastAPI, SQLAlchemy) + TypeScript/React (SPA) + PostgreSQL.
**Инфраструктура:** VPS FirstVDS (212.109.198.69), Docker (опционально), GitHub.
**Разработчик:** 1 человек (AndreyVoyage), работает из Windows (MINGW64/PowerShell) + VS Code + Kimi Code.
**Текущие проблемы:**
- Telegram-бот: initData invalid, фото не удаляется, красный крестик.
- Нет системного тестирования (ручное через телефон).
- Нет аудита безопасности.
- Хочет перейти от "хакатона" к "production-grade".

**Цель фреймворка:** AI пишет ТЗ для AI. Framework генерирует TASK.md + CONTEXT.json, которые Kimi Code читает и выполняет.

---

## 7. Ограничения анализа

- **Не предлагай** переписать всё на другом языке (Go, Rust). Стек фиксирован: Python + TypeScript.
- **Не предлагай** добавить 10 новых ролей в Phase 1. Phase 1 = 4 роли (Architect, Developer, Reviewer, DevOps).
- **Не предлагай** заменить PostgreSQL на MongoDB. ADR-001 принят.
- **Не предлагай** удалить Event Sourcing. Это core принцип фреймворка.
- **Фокус:** Найти пробелы в текущей архитектуре и предложить конкретные исправления.

---

**Промпт версия:** 1.0  
**Дата:** 2026-05-26  
**Авторы:** AndreyVoyage (Human) + AI Architect  
**Следующее обновление:** После анализа и доработки Phase 1
