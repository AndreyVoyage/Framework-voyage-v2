# ADR-INDEX — Реестр архитектурных решений Voyage Framework v4.0+

> **Назначение:** Единый источник правды для всех Architecture Decision Records.
> **Правило:** Если решение не внесено в ADR-INDEX — его нет в системе.
> **Обновляет:** Human (AndreyVoyage) + Architect Agent после каждого принятого ADR.
> **Complexity Budget:** Макс. 9 активных ADR. Сейчас 9 — лимит достигнут.

---

## Жизненный цикл ADR
```
Proposed → Accepted → [Deprecated | Superseded] → Archived
    ↑___________↓
    (можно вернуть из Deprecated)
```

---

## Реестр ADR

| ID | Название | Статус | Дата | Фаза | Связанные ADR | Краткая суть |
|----|----------|--------|------|------|---------------|--------------|
| [ADR-001](ADR-001-postgresql-events.md) | PostgreSQL для Event Store | Accepted | 2026-05-21 | Phase 1 | ADR-002, ADR-004 | Primary: PostgreSQL; Fallback: SQLite WAL |
| [ADR-002](ADR-002-chromadb-semantic.md) | ChromaDB для Semantic Memory | Accepted | 2026-05-21 | Phase 1 | ADR-001, ADR-003 | Primary: ChromaDB; Fallback: SQLite keyword search |
| [ADR-003](ADR-003-tree-sitter-ast.md) | tree-sitter для AST Indexing | Accepted | 2026-05-21 | Phase 1 | ADR-004 | Unified facade ASTPython + ASTTypeScript |
| [ADR-004](ADR-004-dual-location.md) | Dual-Location Strategy | Accepted | 2026-05-21 | Phase 1 | ADR-001, ADR-002 | Отдельный репозиторий + git submodule в SkillTracer |
| [ADR-005](ADR-005-project-isolation.md) | Project Isolation & Multi-Project Core | Accepted | 2026-05-25 | Phase 1 | ADR-001, ADR-004 | `project_id` как composite key; LangGraph отложен в Phase 2 |
| [ADR-006](ADR-006-rule-governance.md) | Rule Governance & Enforcement Architecture | Proposed | 2026-05-26 | Phase 2 | ADR-005, ROLE-002, ROLE-003 | RULES.md → executable governance system; ADR > RULES priority |
| [ADR-007](ADR-007-runtime-orchestration.md) | Runtime Orchestration (Minimal Viable) | Proposed | 2026-05-26 | Phase 2 | ADR-005, ROLE-000 | State machine для сессии разработки |
| [ADR-008](ADR-008-observability.md) | Observability (Minimal Viable) | Proposed | 2026-05-26 | Phase 2 | ADR-007 | Файловая observability: logs, trace, metrics |
| [ADR-009](ADR-009-complexity-budget.md) | Complexity Budget | Proposed | 2026-05-26 | Phase 2 | Все ADR | Hard limits: 9 ADR, 5 ролей, 10 задач в плане |

---

## Complexity Budget (ADR-009)
- **Активных ADR:** 9 / 9 (лимит достигнут, новые только после архивации)
- **ADR-010+:** ЗАПРЕЩЕНЫ до архивации одного из ADR-001..009

---

## Шаблон нового ADR

При создании `ADR-NNN-title.md` используй следующий frontmatter и структуру:

```markdown
# ADR-NNN: Краткое название решения

**Status**: Proposed | Accepted | Deprecated | Superseded | Archived
**Date**: YYYY-MM-DD
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase N
**Replaces**: ADR-XXX (или "Нет")
**Related**: ADR-YYY, ADR-ZZZ

---

## Context
Что за проблема? Какие требования? Какие ограничения?

## Decision
Что именно решено? Однозначно, без "может быть".

## Consequences
### Positive
- ...
### Negative
- ...

## Alternatives
- **Вариант А:** ... (почему отвергнут)
- **Вариант Б:** ... (почему отвергнут)

## Compliance
Как это решение влияет на существующие принципы фреймворка?
- Event Sourcing: ...
- Security First: ...
- Spec-Driven: ...
- Fallback: ...
- Transparent: ...

## Implementation Tasks (для Kimi Code)
- [ ] Конкретное действие 1
- [ ] Конкретное действие 2

## Notes
Всё, что не влезло в формальные разделы.
```

---

## Частые ошибки (авто-правила для Self-Improving Engine)

| Ошибка | Правило |
|--------|---------|
| ADR создан, но не добавлен в ADR-INDEX | Всегда обновляй ADR-INDEX в том же коммите, что и новый ADR |
| Статус ADR = Accepted, но код не соответствует решению | Добавь `Compliance`-проверку в Reviewer Engineer Agent |
| Название файла ADR не по шаблону | Используй `ADR-NNN-kebab-case.md`, без пробелов и амперсандов |
| Дублирование ADR в мастер-документе | Мастер-документ ссылается на ADR-INDEX, не дублирует текст |
| ADR конфликтует с RULES.md | ADR имеет более высокий приоритет (ADR-006); RULES автоматически deprecated |

---

**Версия индекса:** 1.2
**Дата:** 2026-05-26
**Следующее обновление:** После принятия ADR-006 или архивации одного из ADR-001..005
