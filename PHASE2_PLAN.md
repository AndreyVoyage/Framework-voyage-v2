# Phase 2 Plan — Soul & Connectors

> **Phase 1 Status:** 85% complete (M1-M3, M5-M6 done; M4 70%)
> **Цель Phase 2:** Memory, Tools, Workflows, Integrations
> **Дата:** 2026-05-28

---

## Архитектура Phase 2

Memory (M7) -> Tools (M8) -> Workflows (M9) -> Integrations (M10)

---

## M7: Memory Layer

### M7.1 Context Tiers
- 4-tier model: short-term, working, long-term, semantic
- tiktoken budgeting, truncation rules
- Фильтрация по project_id

### M7.2 Semantic Memory
- ChromaDB с namespace per project
- RAG Guard (injection filter)
- SQLite fallback

### M7.3 AST Manager
- tree-sitter: Python + TypeScript
- Impact analysis (<2s для 50 файлов)
- Incremental re-indexing

---

## M8: Tool Engine & Adapters

### M8.1 Tool Engine
- @tool decorator
- Auto-discovery
- Все calls через EventEngine

### M8.2 Adapters (5 штук)
- git.py — git_diff, git_log, git_status
- python.py — pytest, mypy, ruff, pip (+ PyPI validation)
- system.py — df, ps
- files.py — cat, grep, find
- deploy.py — systemctl, ssh, curl (approval required)

---

## M9: Workflow Engine

### M9.1 Engine
- YAML-driven execution
- Deterministic transitions
- Checkpoint + resume

### M9.2 Definitions (4 штуки)
- feature.yaml — Plan->Dev->Test->Review->Deploy
- bugfix.yaml — Reproduce->Fix->Test->Review
- refactor.yaml — AST->Plan->Migrate->Test
- hotfix.yaml — Fix->Test->Deploy (skip review)

---

## M10: Integrations

### M10.1 Kimi Code
- Файловый протокол: TASK.md + CONTEXT.json
- RESULT.md чтение
- implementation_done логирование

### M10.2 Orchestrator Adapter
- ABC для всех адаптеров
- CustomRuntimeAdapter (полная реализация)
- LangGraphAdapter (stub)

### M10.3 GitHub
- PR analysis
- Commit log reading

---

## Приоритет выполнения

1. M7.1 Context Tiers (foundation для всего)
2. M8.1 Tool Engine (foundation для адаптеров)
3. M7.2 Semantic Memory (зависит от M7.1)
4. M8.2 Adapters (зависит от M8.1)
5. M7.3 AST Manager (зависит от M7.1)
6. M9.1 Workflow Engine (зависит от M8)
7. M9.2 Workflow Definitions (зависит от M9.1)
8. M10.1 Kimi Code (зависит от M3 + M9)
9. M10.2 Orchestrator Adapter (зависит от M4)
10. M10.3 GitHub (зависит от M8.2 git adapter)

---

## AC для завершения Phase 2

- [ ] M7: Memory Layer — все компоненты
- [ ] M8: Tool Engine — 5 адаптеров
- [ ] M9: Workflow Engine — 4 definitions
- [ ] M10: Integrations — все адаптеры
- [ ] mypy clean
- [ ] ruff clean
- [ ] pytest all passed (coverage >80%)
- [ ] SelfImprovingEngine интегрирован с Rule Engine
- [ ] OrchestratorAdapter компилируется без ошибок

---

**Версия:** 1.0 | **Дата:** 2026-05-28
