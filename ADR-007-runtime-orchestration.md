# ADR-007: Runtime Orchestration (Minimal Viable)

**Status**: Proposed
**Date**: 2026-05-26
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase 2 (Soul & Connectors)
**Replaces**: Нет
**Related**: ADR-005 (Project Isolation), ROLE-000 (Base AgentRole), ADR-009 (Complexity Budget)

---

## Context

Сейчас Agent Runtime (ROLE-000) определяет базовый цикл `plan → execute → reflect → retry`. Но нет формального описания:
- как стартует/завершается сессия разработчика с Kimi Code
- как происходит retry в рамках одного Plan Agent
- как работает rollback при провале сессии
- как строится task graph в рамках одного плана (max 10 задач по ADR-009)
- как Reviewer (ROLE-003) встраивается в runtime как gate

Это НЕ distributed orchestrator (LangGraph отложен в ADR-005). Это state machine для **одного разработчика + AI в рамках одной сессии Kimi Code**.

---

## Decision

### 1. Agent Lifecycle (сессия разработки)
```text
CREATED        → Сессия инициирована (разработчик открыл Kimi Code)
CONTEXT_LOADED → Загружены RULES, ADR-INDEX, ROLE-INDEX, контекст проекта
PLANNING       → Plan Agent строит план (≤10 задач, ADR-009)
EXECUTING      → Выполнение задач (код, тесты, рефакторинг)
REVIEWING      → Reviewer Engineer (ROLE-003) проверяет результат
COMPLETED      → Задача принята, git commit сделан
FAILED         → Ошибка, требуется retry или abort
ARCHIVED       → Сессия завершена, лог записан (ADR-008)
```

### 2. Task State Machine (в рамках одного плана)
```text
PENDING   → Задача в плане
RUNNING   → AI или разработчик выполняет
REVIEW    → Ожидание review (обязательно для кода)
RETRY     → Не прошла review, повторная попытка (счётчик)
SUCCESS   → Review пройден
FAILED    → Max retries достигнут
DEAD_LETTER → Ручной разбор, не автоматический retry
```

### 3. Retry Policy
```yaml
max_retries: 3
backoff: fixed 30s (не exponential — слишком сложно для ручного режима)
retryable_errors:
  - test_failure
  - lint_error
  - compilation_error
non_retryable_errors:
  - hallucination_detected  → → DEAD_LETTER
  - security_violation      → → DEAD_LETTER
  - complexity_limit_exceeded → → DEAD_LETTER (ADR-009)
```

### 4. Timeout Policy (сессионные, не жёсткие)
```yaml
planning: 10 min   (если план не построен — упростить задачу)
codegen: 15 min    (если AI не выдал код — разбить на меньшие задачи)
review: 5 min      (если review затянулся — пропустить некритичные пункты)
```

### 5. Compensation / Rollback
- **Уровень файла**: `git checkout -- <file>` (если AI испортил файл)
- **Уровень коммита**: `git revert HEAD` (если коммит сломан)
- **Уровень сессии**: откат к предыдущему состоянию `context/` (если вся сессия провальна)

### 6. Queue Architecture
- **FIFO** в рамках Plan Agent (уже реализовано в Kimi Code).
- **Priority**: Security → Architecture → Feature → Style.
- **Dead Letter**: задачи, не прошедшие 3 retry, записываются в `dead_letter_YYYY-MM-DD.md` для ручного разбора.

### 7. Emergency Override (Human Override Mode)
```yaml
emergency_override:
  enabled: true
  trigger: "hotfix / demo / critical bug"
  bypass: [review, complexity_check]
  required: [manual_approval, post_hoc_adr]
  ttl: 24h  (через сутки override автоматически снимается)
```

---

## Consequences

### Positive
- Формализован lifecycle сессии разработки с Kimi Code.
- Retry и rollback защищают от "испорченного кода AI".
- DEAD_LETTER не даёт зациклиться на нерешаемых задачах.
- Emergency Override сохраняет гибкость для critical ситуаций.

### Negative
- Дополнительная дисциплина: разработчик должен записывать логи (ADR-008).
- Timeout'ы не жёсткие — требуют самодисциплины.

---

## Compliance
- **Event Sourcing:** `session_started`, `task_completed`, `retry_attempted`, `dead_letter_created` — все логируются как Events (ADR-001).
- **Security First:** Non-retryable errors включают security violation.
- **Spec-Driven:** ADR фиксирует решение до кода.
- **Fallback:** Если runtime не используется — fallback на ручной режим (как сейчас).
- **Transparent:** Dead Letter и trace доступны через файловую систему (ADR-008).

---

## Implementation Tasks (для Kimi Code)
- [ ] Создать `runtime/session.py` — state machine сессии.
- [ ] Создать `runtime/task_fsm.py` — Task State Machine.
- [ ] Создать `runtime/retry.py` — retry policy с backoff.
- [ ] Создать `runtime/rollback.py` — compensation strategies.
- [ ] Создать `runtime/dead_letter.py` — DEAD_LETTER logger.
- [ ] Обновить `ROLE-003-reviewer-engineer.md` — добавить Mode A/B/C как runtime gates.
- [ ] Обновить `agents/runtime.py` (ROLE-000) — интегрировать с Task FSM.
- [ ] Написать тесты: `test_agents_runtime.py`, `test_agents_state_machine.py`.

---

**Дата:** 2026-05-26 | **Автор:** Human (AndreyVoyage) + AI Architect
