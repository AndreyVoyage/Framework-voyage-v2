# TEST_STRATEGY.md — Test Strategy Voyage Framework v4.0

> **Purpose:** Close gaps in test infrastructure identified by analysis.  
> **Date:** 2026-05-26  
> **Authors:** AndreyVoyage (Human) + AI Architect  
> **Status:** Phase 2 (Soul & Connectors)  
> **Problem:** Test strategy described conceptually, but execution architecture incomplete.

---

## 1. Test Pyramid

```
        +-------------+
        |   E2E       |  <- Playwright, aiogram TestClient
        |   (5%)      |     "User adds photo -> deletes"
        +-------------+
        | Integration |  <- pytest + testcontainers
        |   (15%)     |     "API endpoint + DB + cache"
        +-------------+
        |   Contract  |  <- schemathesis, pact
        |   (10%)     |     "API schema not broken"
        +-------------+
        |    Unit     |  <- pytest, pytest-asyncio
        |   (50%)     |     "Function X returns Y on Z"
        +-------------+
        |   Security  |  <- bandit, semgrep, zap
        |   (10%)     |     "No SQL injections"
        +-------------+
        |  Mutation   |  <- mutmut, cosmic-ray
        |   (5%)      |     "Tests catch mutations"
        +-------------+
        |   Replay    |  <- EventEngine replay
        |   (5%)      |     "Deterministic replay"
        +-------------+
```

---

## 2. Missing Components

### 2.1 CI Test Gates

**What is needed:**
```yaml
# .github/workflows/test-gates.yaml
name: Test Gates
on: [pull_request]
jobs:
  test-gates:
    runs-on: ubuntu-latest
    steps:
      - name: Unit Tests
        run: pytest tests/unit/ --cov=voyage_framework --cov-fail-under=80

      - name: Integration Tests
        run: pytest tests/integration/ --testcontainers

      - name: Contract Tests
        run: schemathesis run http://localhost:8000/openapi.json

      - name: Security Tests
        run: |
          bandit -r voyage_framework/
          semgrep --config=auto voyage_framework/

      - name: Mutation Tests
        run: mutmut run --paths-to-mutate=voyage_framework/

      - name: Flaky Test Detection
        run: pytest tests/ --count=10

      - name: Merge Gate
        run: python -m voyage_framework.review.merge_gate
```

**Acceptance Criteria:**
- [ ] Pipeline fail if coverage < 80%.
- [ ] Pipeline fail if mutation score < 70%.
- [ ] Pipeline fail if flaky tests > 5%.
- [ ] Pipeline warning if security findings > 0 (critical/high).

### 2.2 Snapshot Testing

**For AI-generated outputs:**
```python
# tests/test_snapshots.py
import syrupy

def test_task_generator_snapshot(snapshot):
    task = generator.generate(role="developer", task="Add FSM persistence")
    assert task.task_markdown == snapshot

def test_context_json_snapshot(snapshot):
    context = generator.generate_context(task="Add FSM persistence")
    assert context == snapshot
```

**Why:** AI may generate different text each run. Snapshot testing fixes "golden output" and detects drift.

### 2.3 AI Evaluation Tests

**Critical for AI-native framework:**

```python
# tests/test_ai_evaluation.py

def test_prompt_regression():
    result = agent.run("Add FSM persistence to bot")
    assert evaluate(result, golden_dataset["fsm_persistence"]) > 0.9

def test_hallucination_detection():
    code = agent.generate_code(task="Add FSM persistence")
    assert not contains_hallucinated_imports(code)

def test_context_assembly():
    context = memory.compile_context(project_id="skilltracer")
    assert context.short_term is not None
    assert context.semantic is not None
    assert len(context.short_term) <= 10

def test_task_decomposition():
    plan = architect.plan("Add FSM persistence")
    assert len(plan.steps) >= 3
    assert all(step.has_acceptance_criteria for step in plan.steps)
```

**Golden Datasets:**
- `datasets/golden_tasks.json` — 50 typical tasks with expected results.
- `datasets/golden_contexts.json` — 50 contexts with correct assembly.
- `datasets/golden_prompts.json` — 50 prompts with expected responses.

### 2.4 Contract Testing

**For multi-agent communication:**

```python
# tests/test_contracts.py
import schemathesis

@schemathesis.hook
def test_api_contract():
    schema = schemathesis.from_path("openapi.json")
    schema.parametrize()

def test_event_contract():
    event = Event(event_type="plan_created", payload={})
    assert EventSchema.validate(event)

def test_message_contract():
    msg = AgentMessage(from_role="architect", to_role="developer", payload={})
    assert MessageSchema.validate(msg)
```

### 2.5 Replay Testing

**Critical for Event Sourcing:**

```python
# tests/test_replay.py

def test_deterministic_replay():
    events = event_engine.get_events(project_id="skilltracer")
    state1 = replay(events)
    state2 = replay(events)
    assert state1 == state2

def test_event_stream_replay():
    workflow = load_workflow("feature.yaml")
    events = run_workflow(workflow)
    replayed = replay(events)
    assert replayed.final_state == workflow.expected_state

def test_prompt_replay():
    prompt = load_prompt("TASK-001")
    result1 = agent.run(prompt)
    result2 = agent.run(prompt)
    assert result1 == result2
```

### 2.6 Chaos Testing

**For async orchestration:**

```python
# tests/test_chaos.py
import pytest

@pytest.mark.chaos
def test_agent_failure_recovery():
    kill_agent("developer")
    assert system.state == "recovered"

@pytest.mark.chaos
def test_memory_corruption():
    corrupt_chromadb()
    result = semantic_memory.query("test")
    assert result.source == "sqlite_fallback"

@pytest.mark.chaos
def test_partial_execution():
    checkpoint = run_workflow_partial("feature.yaml", stop_at="developer_implement")
    resumed = resume_from_checkpoint(checkpoint)
    assert resumed.completed
```

---

## 3. Test Infrastructure

### 3.1 tests/ Structure

```
tests/
├── conftest.py
├── unit/
│   ├── test_core_models.py
│   ├── test_event_engine.py
│   ├── test_storage.py
│   ├── test_sandbox.py
│   ├── test_sandbox_backends.py
│   ├── test_policy.py
│   ├── test_audit.py
│   ├── test_approval.py
│   ├── test_tools_engine.py
│   ├── test_tools_adapters.py
│   ├── test_context_tiers.py
│   ├── test_semantic.py
│   ├── test_ast_manager.py
│   ├── test_specs_adr.py
│   ├── test_specs_plan.py
│   ├── test_specs_task_generator.py
│   ├── test_specs_tracker.py
│   ├── test_agents_runtime.py
│   ├── test_agents_state_machine.py
│   ├── test_agents_roles_architect.py
│   ├── test_agents_roles_developer.py
│   ├── test_agents_roles_reviewer.py
│   ├── test_agents_roles_devops.py
│   ├── test_agents_roles_qa.py
│   ├── test_agents_roles_security.py
│   ├── test_agents_roles_reviewer_engineer.py
│   ├── test_workflows_engine.py
│   ├── test_self_improving.py
│   ├── test_rules_parser.py
│   ├── test_rules_validator.py
│   ├── test_rules_enforcer.py
│   ├── test_rules_conflicts.py
│   ├── test_visualizer_api.py
│   └── test_project_isolation.py
├── integration/
│   ├── test_event_engine_postgres.py
│   ├── test_event_engine_sqlite.py
│   ├── test_semantic_chromadb.py
│   ├── test_semantic_sqlite_fallback.py
│   ├── test_full_workflow_feature.py
│   ├── test_full_workflow_bugfix.py
│   └── test_api_endpoints.py
├── e2e/
│   ├── test_telegram_bot_flow.py
│   ├── test_react_spa_flow.py
│   └── test_full_user_journey.py
├── security/
│   ├── test_bandit_scan.py
│   ├── test_semgrep_scan.py
│   ├── test_zap_scan.py
│   ├── test_secrets_scan.py
│   └── test_config_audit.py
├── contract/
│   ├── test_api_contract.py
│   ├── test_event_contract.py
│   └── test_message_contract.py
├── mutation/
│   └── test_mutation_score.py
├── replay/
│   ├── test_deterministic_replay.py
│   ├── test_event_stream_replay.py
│   └── test_prompt_replay.py
├── chaos/
│   ├── test_agent_failure.py
│   ├── test_memory_corruption.py
│   └── test_partial_execution.py
├── ai_evaluation/
│   ├── test_prompt_regression.py
│   ├── test_hallucination_detection.py
│   ├── test_context_assembly.py
│   └── test_task_decomposition.py
├── snapshots/
│   ├── test_task_generator_snapshot.py
│   └── test_context_json_snapshot.py
└── datasets/
    ├── golden_tasks.json
    ├── golden_contexts.json
    └── golden_prompts.json
```

### 3.2 pytest.ini

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
addopts = -v --tb=short --strict-markers
markers =
    unit: Unit tests (fast, isolated)
    integration: Integration tests (require DB/services)
    e2e: End-to-end tests (require full system)
    security: Security tests (require scanning tools)
    contract: Contract tests (require API schema)
    mutation: Mutation tests (slow)
    replay: Replay tests (require event history)
    chaos: Chaos tests (destructive)
    ai_eval: AI evaluation tests (require LLM)
    snapshot: Snapshot tests (require golden data)
```

### 3.3 Makefile

```makefile
.PHONY: test test-unit test-integration test-e2e test-security test-contract test-mutation test-replay test-chaos test-ai-eval test-all

test-unit:
	pytest tests/unit/ -m unit -v

test-integration:
	pytest tests/integration/ -m integration -v

test-e2e:
	pytest tests/e2e/ -m e2e -v

test-security:
	pytest tests/security/ -m security -v

test-contract:
	pytest tests/contract/ -m contract -v

test-mutation:
	mutmut run --paths-to-mutate=voyage_framework/
	mutmut results

test-replay:
	pytest tests/replay/ -m replay -v

test-chaos:
	pytest tests/chaos/ -m chaos -v

test-ai-eval:
	pytest tests/ai_evaluation/ -m ai_eval -v

test-snapshot:
	pytest tests/snapshots/ -m snapshot -v

test-all:
	pytest tests/ -v --cov=voyage_framework --cov-report=html --cov-fail-under=80

test-fast:
	pytest tests/unit/ -m unit -x -q
```

---

## 4. CI/CD Pipeline

```yaml
# .github/workflows/test-pipeline.yaml
name: Test Pipeline
on: [pull_request, push]
jobs:
  unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -e ".[dev]"
      - run: make test-unit

  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env: { POSTGRES_PASSWORD: test }
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e ".[dev]"
      - run: make test-integration

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e ".[dev]"
      - run: make test-security

  contract:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e ".[dev]"
      - run: make test-contract

  mutation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e ".[dev]"
      - run: make test-mutation

  ai-eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e ".[dev]"
      - run: make test-ai-eval

  merge-gate:
    needs: [unit, integration, security, contract]
    runs-on: ubuntu-latest
    steps:
      - run: echo "All gates passed -> merge allowed"
```

---

## 5. Quality Metrics

| Metric | Threshold | CI Gate |
|--------|-----------|---------|
| Unit test coverage | >= 80% | Fail |
| Integration test coverage | >= 60% | Warning |
| Mutation score | >= 70% | Fail |
| Flaky test rate | <= 5% | Fail |
| Security findings (critical/high) | = 0 | Fail |
| Security findings (medium) | <= 3 | Warning |
| Contract violations | = 0 | Fail |
| Replay determinism | 100% | Fail |
| AI evaluation score | >= 0.9 | Warning |
| Architectural score | >= 0.7 | Fail |

---

## 6. Implementation Checklist

- [ ] Create tests/ structure.
- [ ] Write conftest.py with fixtures.
- [ ] Configure pytest.ini with markers.
- [ ] Create Makefile with commands.
- [ ] Configure GitHub Actions pipeline.
- [ ] Create golden datasets (50 tasks, 50 contexts, 50 prompts).
- [ ] Configure snapshot testing (syrupy).
- [ ] Configure mutation testing (mutmut).
- [ ] Configure contract testing (schemathesis).
- [ ] Configure chaos testing (pytest-chaos).
- [ ] Integrate with Reviewer Engineer (merge gate).
- [ ] Update ROLE-001-qa-engineer.md with test orchestration.

---

**Version:** 1.0  
**Date:** 2026-05-26  
**Authors:** AndreyVoyage (Human) + AI Architect  
**Next update:** After CI pipeline implementation
