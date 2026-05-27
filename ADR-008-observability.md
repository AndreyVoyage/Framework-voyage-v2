# ADR-008: Observability (Minimal Viable)

**Status**: Proposed
**Date**: 2026-05-26
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase 2 (Soul & Connectors)
**Replaces**: Нет
**Related**: ADR-007 (Runtime Orchestration), ADR-001 (PostgreSQL Events)

---

## Context

Без observability AI-framework невозможно дебажить. Но OpenTelemetry и полноценный tracing — перебор для single-developer (нарушает ADR-009 Complexity Budget: no over-engineering).

---

## Decision

### 1. Structured Log
Файл: `voyage/traces/voyage-log-YYYY-MM-DD.jsonl` (одна строка = один event)

```json
{"ts":"2026-05-26T21:10:00Z","session":"sess_abc123","agent":"planner","task_id":"T-007","event":"plan_generated","duration_ms":4500,"status":"ok"}
{"ts":"2026-05-26T21:15:00Z","session":"sess_abc123","agent":"reviewer","task_id":"T-007","event":"hallucination_detected","severity":"high","detail":"fake_api_call: getUserByMagicId()"}
```

### 2. Execution Trace Tree
Файл: `voyage/traces/trace-YYYY-MM-DD.md`

```markdown
# Trace 2026-05-26

## Session: sess_abc123
- [21:10] Planner → Plan generated (5 tasks)
  - [21:12] Architect → ADR-006 updated
  - [21:14] Codegen → auth.ts modified
    - [21:15] Reviewer → ❌ Hallucination detected (fake API)
    - [21:16] Codegen → Retry #1
    - [21:17] Reviewer → ✅ Approved
  - [21:20] QA → Tests passed
- [21:22] Session COMPLETED
```

### 3. Minimal Metrics (ручные)
В конце каждой сессии разработчик записывает в `voyage/metrics.md`:
```markdown
| Date | Tasks | Success | Retry | Hallucinations | Avg Latency |
|:---|:---|:---|:---|:---|:---|
| 2026-05-26 | 5 | 4 | 3 | 1 | 12 min |
```

### 4. Reasoning Audit
Для спорных решений AI: копировать reasoning chain в `voyage/reasoning/YYYY-MM-DD_<task>.md`.
Нужно, чтобы понять «почему AI предложил именно это».

### 5. Event Sourcing Integration (ADR-001)
Все runtime-события (ADR-007) дублируются в EventEngine как `event_type: "runtime_*"`.
Это даёт replay capability: можно воспроизвести сессию по events.

---

## Что НЕ делаем (Phase 3 / при нарушении Complexity Budget)
- OpenTelemetry collector
- Grafana / Prometheus
- Distributed tracing
- Автоматические alerting
- Real-time dashboards

---

## Compliance
- **Event Sourcing:** Все observability-события — это Events (ADR-001).
- **Security First:** Логи не содержат secrets (gitleaks scan перед commit).
- **Spec-Driven:** ADR фиксирует формат до реализации.
- **Fallback:** Если jsonl сломан — fallback на plain markdown trace.
- **Transparent:** Все файлы human-readable, хранятся в git.

---

## Implementation Tasks (для Kimi Code)
- [ ] Создать `voyage/traces/` директорию.
- [ ] Создать `observability/logger.py` — structured jsonl logger.
- [ ] Создать `observability/tracer.py` — markdown trace generator.
- [ ] Создать `observability/metrics.py` — metrics.md updater.
- [ ] Обновить `EventEngine` (ADR-001) — добавить `runtime_*` event types.
- [ ] Написать тесты: `test_observability_logger.py`, `test_observability_tracer.py`.

---

**Дата:** 2026-05-26 | **Автор:** Human (AndreyVoyage) + AI Architect
