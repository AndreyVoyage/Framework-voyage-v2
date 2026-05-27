# ADR-009: Complexity Budget

**Status**: Proposed
**Date**: 2026-05-26
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase 2 (Soul & Connectors)
**Replaces**: Нет
**Related**: Все ADR, все ROLE, TEST_STRATEGY.md

---

## Context

Framework растёт экспоненциально. Главный риск: `framework complexity > project value`.
Анализ показал, что на текущий момент:
- 5 Accepted ADR (001..005)
- 4 Proposed ADR (006..009)
- 1 Implemented роль (000)
- 3 Planned роли (001..003)
- 2 Concept роли (004..005)

Без hard limits следующий этап создаст complexity debt: microservices, distributed agents, meta-framework.

---

## Decision

### 1. Hard Limits

```yaml
max_active_adr: 9          # Сейчас 9 (5 Accepted + 4 Proposed). Новый ADR только после архивации.
max_roles_planned_active: 5  # Сейчас 4 (000 Implemented + 001..003 Planned). Запас 1.
max_role_switches_per_session: 3
max_rule_dependencies: 3   # Одно правило не может зависеть от >3 других
max_review_depth: 2        # Reviewer не может вызывать sub-reviewer >2 уровней
max_plan_tasks: 10         # Один план — не более 10 задач. Больше — разбивать.
max_adr_dependencies: 3    # Новый ADR не может зависеть от >3 других ADR
```

### 2. Anti-Abstraction Rules (запрещено без исключения)
1. **No microservices** — пока проект не достиг 50k строк.
2. **No premature interfaces** — не создавать `interface IService` ради «красоты».
3. **No distributed queue** — пока 1 разработчик.
4. **No self-modifying AI** — AI не может менять RULES.md, ADR, ROLE без human approval.
5. **No meta-framework** — не создавать фреймворк для управления фреймворком.
6. **No LangGraph in Phase 1** — уже зафиксировано в ADR-005, но повторяем: собственный runtime (ADR-007) до Phase 2+.

### 3. Cost of Governance Gate
Перед добавлением любого компонента >50 строк или нового артефакта:
```markdown
### Complexity Justification
- Компонент: ___
- Строк кода: ___
- Новых зависимостей: ___
- Новых ADR: ___ (проверить: не превысит ли лимит 9?)
- Новых ролей: ___ (проверить: не превысит ли лимит 5?)
- Что упрощается: ___
- Что ускоряется: ___
- Что станет безопаснее: ___
- Риск усложнения: ___ / 10
- Решение: [ ] Принять [ ] Отклонить [ ] Отложить
```

### 4. ADR Slot Management
```text
ADR-001..005: Accepted (Phase 1, foundation)
ADR-006..009: Proposed (Phase 2, governance & runtime)

ADR-010+: ЗАПРЕЩЕНЫ до архивации одного из ADR-001..009.

Архивация возможна, если:
- ADR Superseded новым (например, ADR-002 заменён на pgvector ADR)
- ADR Deprecated и код мигрирован
- ADR перенесён в Phase 3 и больше не актуален для текущей фазы
```

### 5. Role Slot Management
```text
ROLE-000: Implemented (Base, не считается в "активных" для governance)
ROLE-001..003: Planned (Phase 2, core team)
ROLE-004..005: Concept (Phase 2+/3, не переводить в Planned без слота)

ROLE-006+: Можно создать как Concept, но НЕ переводить в Planned/Implemented без:
  - Архивации одной из 001..003, ИЛИ
  - Увеличения max_roles (требует новый ADR и strong justification)
```

---

## Consequences

### Positive
- Framework остаётся управляемым при росте.
- Single-developer не тонет в собственной архитектуре.
- Каждый новый компонент обоснован.

### Negative
- Иногда придётся отказывать себе в "крутой" технологии.
- Архивация ADR требует миграционной работы.

---

## Compliance
- **Event Sourcing:** `complexity_limit_exceeded` — event тип для аудита.
- **Security First:** Anti-abstraction rule #4 защищает от self-modifying AI.
- **Spec-Driven:** ADR-009 фиксирует лимиты до их превышения.
- **Fallback:** Если лимит мешает критичной задаче — Emergency Override (ADR-007).
- **Transparent:** Complexity Justification хранится в `voyage/complexity_log.md`.

---

## Implementation Tasks (для Kimi Code)
- [ ] Создать `voyage/complexity_log.md` — журнал всех justification decisions.
- [ ] Обновить `ADR-INDEX.md` — добавить complexity budget status.
- [ ] Обновить `ROLE-INDEX.md` — добавить complexity budget status.
- [ ] Обновить `FUTURE_ROLES_TECH_REGISTRY.md` — добавить complexity budget status.
- [ ] Создать `tools/complexity_checker.py` — скрипт проверки лимитов перед добавлением.
- [ ] Написать тесты: `test_complexity_budget.py`.

---

**Дата:** 2026-05-26 | **Автор:** Human (AndreyVoyage) + AI Architect
