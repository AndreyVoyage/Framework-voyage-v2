# ADR-006: Rule Governance & Enforcement Architecture

**Status**: Proposed  
**Date**: 2026-05-26  
**Authors**: AndreyVoyage (Human) + AI Architect  
**Phase**: Phase 2 (Soul & Connectors)  
**Replaces**: Нет  
**Related**: ADR-005 (Project Isolation), ROLE-002 (Security Engineer), ROLE-003 (Reviewer Engineer)

---

## Context

RULES.md — один из сильнейших компонентов framework. Но анализ показал, что сейчас он **декларативный**, а не **executable**:

- Нет rule engine.
- Нет automatic validation.
- Нет CI enforcement.
- Нет auto-fix pipeline.
- Нет severity levels enforcement.
- Нет rule versioning.
- Нет conflict resolution.

Self-Improving Engine автоматически добавляет правила, но без governance это создаёт риски:
- Rule explosion (слишком много правил).
- Conflicting rules (правила противоречат друг другу).
- Overfitting (правила слишком специфичны).
- Dead rules (правила больше не актуальны).

Нужна архитектура, которая превращает RULES.md из документа в **runtime governance system**.

---

## Decision

### 1. Rule Engine Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RULE GOVERNANCE SYSTEM                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │  Rule       │───→│  Rule       │───→│  Rule       │   │
│  │  Parser     │    │  Validator  │    │  Enforcer   │   │
│  │  (parse     │    │  (validate  │    │  (enforce   │   │
│  │   RULES.md) │    │   rules)    │    │   in CI)    │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
│         │                  │                  │            │
│         ▼                  ▼                  ▼            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              Rule Registry (in-memory)               │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │  │
│  │  │ Active  │ │Deprecated│ │Experimental│ │Conflict │  │  │
│  │  │ Rules   │ │ Rules   │ │ Rules   │ │ Rules   │  │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │  Conflict   │───→│  Auto-Fix   │───→│  CI/CD      │   │
│  │  Resolver   │    │  Engine     │    │  Integration│   │
│  │             │    │             │    │             │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. Rule Lifecycle

```
Proposed → Experimental → Active → Deprecated → Archived
    ↑___________↓___________________↓
    (human review)        (auto-detect dead rules)
```

- **Proposed:** Self-Improving Engine предлагает правило.
- **Experimental:** Правило активно 7 дней, собирает статистику.
- **Active:** Правило подтверждено (0 конфликтов, положительный impact).
- **Deprecated:** Правило устарело (детекция через AST: код больше не нарушает правило).
- **Archived:** Правило удалено из active set, но сохранено в истории.

### 3. Rule Severity Enforcement

| Severity | Enforcement | CI Action | Human Approval |
|----------|-------------|-----------|----------------|
| **MUST** | Блокирует merge | Fail pipeline | Required for override |
| **SHOULD** | Warning | Warning in pipeline | Not required |
| **MAY** | Info | Info in pipeline | Not required |
| **EXPERIMENTAL** | Log only | No CI impact | N/A |

### 4. Conflict Resolution

- **Detection:** При добавлении нового правила проверяется на конфликт с существующими.
- **Types:**
  - **Direct conflict:** "Всегда используй async SQLAlchemy" vs "Используй sync SQLAlchemy для миграций".
  - **Scope conflict:** "Все функции должны иметь type hints" vs "@pytest.fixture не требует type hints".
  - **Temporal conflict:** Старое правило противоречит новому ADR.
- **Resolution:**
  - Auto-resolve: если одно правило deprecated → удалить.
  - Human review: если оба active → flag для Architect.
  - ADR priority: ADR > RULES.md (architectural truth > operational constraint).

### 5. ADR vs RULES Priority

**Проблема:** Анализ показал, что RULES.md имеет более высокий приоритет, чем ADR. Это опасно: ADR описывает архитектурную истину, RULES — operational constraints.

**Решение:**
```
Priority (высший → низший):
1. ADR (architectural decisions)
2. Architectural Rules (в RULES.md, помечены [ARCH])
3. Operational Rules (в RULES.md, помечены [OPS])
4. Style Rules (в RULES.md, помечены [STYLE])
```

- **ADR** может override RULES.
- **Architectural Rules** могут override Operational/Style.
- **Conflict:** если RULES противоречит ADR → RULES автоматически deprecated.

### 6. Auto-Fix Engine

- **MUST-правила:** Auto-fix при нарушении.
  - Пример: "Все async def должны иметь type hints" → auto-fix: добавить `-> None`.
- **SHOULD-правила:** Suggestion, не auto-fix.
- **MAY-правила:** Info only.

### 7. CI/CD Integration

```yaml
# .github/workflows/rule-enforcement.yaml
name: Rule Enforcement
on: [pull_request]
jobs:
  rules:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Parse RULES.md
        run: python -m voyage_framework.rules.parse
      - name: Validate Rules
        run: python -m voyage_framework.rules.validate
      - name: Enforce Rules
        run: python -m voyage_framework.rules.enforce
      - name: Check Conflicts
        run: python -m voyage_framework.rules.conflicts
      - name: Report
        run: python -m voyage_framework.rules.report
```

---

## Consequences

### Positive
- RULES.md становится executable, не just documentation.
- Self-Improving Engine не может деградировать систему (governance layer).
- CI/CD автоматически блокирует нарушения.
- ADR priority предотвращает architectural drift.
- Rule lifecycle предотвращает rule explosion.

### Negative
- Дополнительная сложность: нужен Rule Engine (новый модуль).
- CI/CD pipeline дольше (rule enforcement занимает время).
- Human review required для conflict resolution (может замедлить).

---

## Implementation Tasks (для Kimi Code)

- [ ] Создать `rules/` модуль:
  - `rules/parser.py` — парсинг RULES.md (severity, category, pattern).
  - `rules/validator.py` — валидация правил (syntax, scope, conflicts).
  - `rules/enforcer.py` — enforcement в CI (lint integration).
  - `rules/conflicts.py` — conflict detection & resolution.
  - `rules/registry.py` — in-memory rule registry (active/deprecated/experimental).
  - `rules/autofix.py` — auto-fix engine для MUST-правил.
- [ ] Обновить `core/models.py` — добавить `Rule` Pydantic model:
  ```python
  class Rule(BaseModel):
      id: str
      text: str
      severity: Literal["must", "should", "may", "experimental"]
      category: Literal["arch", "ops", "style"]
      status: Literal["proposed", "experimental", "active", "deprecated", "archived"]
      pattern: str  # regex или AST pattern
      autofix: Optional[str]  # шаблон auto-fix
      created_at: datetime
      project_id: str
  ```
- [ ] Обновить `self_improving/engine.py` — интегрировать с Rule Engine:
  - Новые правила → статус "proposed".
  - После 7 дней → "experimental".
  - После подтверждения → "active".
- [ ] Обновить `ROLE-003-reviewer-engineer.md` — добавить rule validation в responsibilities.
- [ ] Обновить `RULES.md` — добавить категории [ARCH], [OPS], [STYLE].
- [ ] Создать `.github/workflows/rule-enforcement.yaml`.
- [ ] Написать тесты: `test_rules_parser.py`, `test_rules_validator.py`, `test_rules_enforcer.py`, `test_rules_conflicts.py`.

---

## Compliance

- **Event Sourcing:** `rule_proposed`, `rule_activated`, `rule_deprecated`, `rule_conflict_detected` — все логируются как Events.
- **Security First:** Rule Engine не может execute arbitrary code (sandboxed validation).
- **Spec-Driven:** ADR-006 фиксирует решение до кода.
- **Fallback:** Если Rule Engine недоступен — fallback на manual RULES.md parsing.
- **Transparent:** Rule Registry доступен через Visualizer API.

---

## Notes

- **Rule ID format:** `RULE-{category}-{NNN}` (например: `RULE-ARCH-001`, `RULE-OPS-042`).
- **Rule hash:** SHA256 от `text + pattern` для deduplication.
- **Experimental period:** 7 дней или 10 executions (что наступит раньше).
- **Dead rule detection:** Если правило не срабатывает 30 дней → статус "deprecated".
- **Performance:** Rule Engine должен работать <5 секунд для проекта с 100 правилами.

---

**Дата:** 2026-05-26 | **Автор:** Human (AndreyVoyage) + AI Architect
