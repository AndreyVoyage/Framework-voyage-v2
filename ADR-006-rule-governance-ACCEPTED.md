# ADR-006: Rule Governance & Enforcement Architecture

**Status**: Accepted
**Date**: 2026-05-26 (updated 2026-05-28)
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase 1 (Foundation)
**Replaces**: None
**Related**: ADR-005 (Project Isolation), ROLE-002 (Security Engineer), ROLE-003 (Reviewer Engineer), RULES.md v1.2

---

## Context

RULES.md is one of the strongest framework components. But analysis showed it is currently **declarative**, not **executable**:

- No rule engine.
- No automatic validation.
- No CI enforcement.
- No auto-fix pipeline.
- No severity levels enforcement.
- No rule versioning.
- No conflict resolution.

Self-Improving Engine automatically adds rules, but without governance this creates risks:
- Rule explosion (too many rules).
- Conflicting rules (rules contradict each other).
- Overfitting (rules too specific).
- Dead rules (rules no longer relevant).

**Critical conflict discovered (2026-05-28):** RULES.md v1.1 stated: "If a rule in RULES.md conflicts with the master document — RULES.md wins." This contradicts the architectural principle: ADR describes architectural truth, RULES — operational constraints. ADR cannot be overridden by a rule.

An architecture is needed that turns RULES.md from a document into a **runtime governance system** with a clear priority hierarchy.

---

## Decision

### 1. Rule Engine Architecture

Rule Parser -> Rule Validator -> Rule Enforcer -> Rule Registry
                  |                  |
                  v                  v
          Conflict Resolver    Auto-Fix Engine
                  |                  |
                  v                  v
            CI/CD Integration

### 2. Rule Lifecycle

Proposed -> Experimental -> Active -> Deprecated -> Archived

- **Proposed:** Self-Improving Engine proposes a rule.
- **Experimental:** Rule is active for 7 days, collecting statistics.
- **Active:** Rule is confirmed (0 conflicts, positive impact).
- **Deprecated:** Rule is outdated.
- **Archived:** Rule removed from active set but kept in history.

### 3. Rule Severity Enforcement

| Severity | Enforcement | CI Action | Human Approval |
|----------|-------------|-----------|----------------|
| **MUST** | Blocks merge | Fail pipeline | Required for override |
| **SHOULD** | Warning | Warning in pipeline | Not required |
| **MAY** | Info | Info in pipeline | Not required |
| **EXPERIMENTAL** | Log only | No CI impact | N/A |

### 4. Conflict Resolution

- **Direct conflict:** two rules contradict each other.
- **Scope conflict:** rule does not apply to all code.
- **Temporal conflict:** old rule contradicts new ADR.
- **Resolution:** ADR priority, human review, auto-deprecate.

### 5. ADR vs RULES Priority — HIERARCHY (updated 2026-05-28)

```
Priority (highest -> lowest):
1. ADR (Architecture Decision Records)
   — Immutable without new ADR.
   — Can override any rules in RULES.md.

2. [ARCH] rules (Architectural Rules in RULES.md)
   — Sections 2.x, 4.x in RULES.md.
   — Require new ADR for changes.

3. [OPS] rules (Operational Rules in RULES.md)
   — Sections 1.x, 3.x, 5.x, 6.x in RULES.md.
   — Can be updated by Self-Improving Engine.

4. [STYLE] rules (Style Rules in RULES.md)
   — Lowest priority.
   — Can be updated by Self-Improving Engine.
```

**Hierarchy application rules:**
- If a RULES.md rule conflicts with ADR -> ADR wins, rule automatically deprecated.
- If [ARCH] rule conflicts with [OPS] -> [ARCH] wins.
- If [OPS] rule conflicts with [STYLE] -> [OPS] wins.
- Changing [ARCH] rules requires a **new ADR** with justification.
- Changing [OPS] and [STYLE] is sufficient with Self-Improving Engine or human review.

**Important:** RULES.md is a living document, but it **CANNOT override ADR**. Changing ADR requires a new ADR with justification.

### 6. Auto-Fix Engine

- **MUST rules:** Auto-fix on violation.
- **SHOULD rules:** Suggestion, not auto-fix.
- **MAY rules:** Info only.

### 7. CI/CD Integration

```yaml
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
- RULES.md becomes executable, not just documentation.
- Self-Improving Engine cannot degrade the system (governance layer).
- CI/CD automatically blocks violations.
- ADR priority prevents architectural drift.
- Rule lifecycle prevents rule explosion.
- Priority hierarchy resolves ADR vs RULES conflict (v1.1 -> v1.2).

### Negative
- Additional complexity: Rule Engine module needed.
- CI/CD pipeline longer (rule enforcement takes time).
- Human review required for conflict resolution (may slow down).

---

## Implementation Tasks

- [ ] Create `rules/` module: parser, validator, enforcer, conflicts, registry, autofix.
- [ ] Update `core/models.py` — add `Rule` Pydantic model.
- [ ] Update `self_improving/engine.py` — integrate with Rule Engine.
- [ ] Update `ROLE-003-reviewer-engineer.md` — add rule validation.
- [ ] Update `RULES.md` — add categories [ARCH], [OPS], [STYLE] (v1.2 done).
- [ ] Create `.github/workflows/rule-enforcement.yaml`.
- [ ] Write tests for rules module.

---

## Compliance

- **Event Sourcing:** `rule_proposed`, `rule_activated`, `rule_deprecated`, `rule_conflict_detected` — all logged as Events.
- **Security First:** Rule Engine sandboxed validation.
- **Spec-Driven:** ADR-006 fixes decision before code.
- **Fallback:** If Rule Engine unavailable — fallback to manual RULES.md parsing.
- **Transparent:** Rule Registry available via Visualizer API.

---

## Notes

- **Rule ID format:** `RULE-{category}-{NNN}` (e.g., `RULE-ARCH-001`, `RULE-OPS-042`).
- **Rule hash:** SHA256 of `text + pattern` for deduplication.
- **Experimental period:** 7 days or 10 executions (whichever comes first).
- **Dead rule detection:** If rule does not trigger for 30 days -> status "deprecated".
- **Performance:** Rule Engine must run <5 seconds for project with 100 rules.
- **Priority fix date:** 2026-05-28 — hierarchy ADR > [ARCH] > [OPS] > [STYLE] fixed in RULES.md v1.2.

---

**Date:** 2026-05-26 (updated 2026-05-28) | **Author:** Human (AndreyVoyage) + AI Architect
**Status:** Accepted | **Phase:** Phase 1 (Foundation)
