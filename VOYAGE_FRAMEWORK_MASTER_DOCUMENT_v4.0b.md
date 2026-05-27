# Voyage AI Dev Framework v4.0 — Мастер-документ

> **Название:** Voyage AI Dev Framework v4.0 (VADF v4)  
> **Автор:** AndreyVoyage (Human) + AI Architect  
> **Дата создания:** 2026-05-21  
> **Дата обновления:** 2026-05-26 (синхронизация с ADR-005)  
> **Статус:** Phase 1 — Foundation & Brain (в разработке)  
> **Целевой проект:** SkillTracer (Python backend + TypeScript/React frontend)  
> **Философия:** AI пишет ТЗ для AI. Framework генерирует `TASK.md` + `CONTEXT.json`, которые Kimi Code в VS Code читает и выполняет.

---

## Содержание

1. [Видение и миссия](#1-видение-и-миссия)
2. [Архитектура высокого уровня](#2-архитектура-высокого-уровня)
3. [Структура проекта](#3-структура-проекта)
4. [Компоненты системы](#4-компоненты-системы)
5. [Потоки данных (Data Flow)](#5-потоки-данных-data-flow)
6. [Роли агентов](#6-роли-агентов)
7. [Цикл работы агента](#7-цикл-работы-агента)
8. [Безопасность](#8-безопасность)
9. [Память и контекст](#9-память-и-контекст)
10. [Интеграции](#10-интеграции)
11. [Self-Improving Engine](#11-self-improving-engine)
12. [Фазы разработки](#12-фазы-разработки)
13. [Термины и аббревиатуры](#13-термины-и-аббревиатуры)
14. [Roadmap](#14-roadmap)
15. [Чек-листы](#15-чек-листы)

---

## 1. Видение и миссия

### Проблема
Solo-разработчик (1 человек) пытается:
- Вести несколько проектов одновременно
- Помнить архитектурные решения месяцами
- Не повторять одни и те же ошибки
- Делегировать рутину AI, но контролировать критичные моменты
- Обучаться на своём опыте (а не на чужих туториалах)

### Решение
Voyage Framework — это **AI-Native Engineering Operating System**, который:
- **Планирует** задачи через микро-фазы с acceptance criteria
- **Помнит** всё: решения, ошибки, контекст, структуру кода
- **Безопасно выполняет** команды через sandbox с human approval
- **Генерирует ТЗ** для Kimi Code / Kimi Agent в формате `TASK.md` + `CONTEXT.json`
- **Самоулучшается**: каждая ошибка → новое правило в `RULES.md`
- **Визуализирует** свою работу через отдельное приложение (обучение + мониторинг)

### Принципы
1. **Event Sourcing** — единый источник правды. Всё остальное — projection.
2. **Security First** — AI не может навредить. Dangerous tier требует approval.
3. **Spec-Driven** — сначала спецификация, потом код. Критерии до реализации.
4. **Self-Improving** — система учится на своих ошибках без human intervention.
5. **Transparent** — всё видно: что AI делает, почему, что планирует дальше.
6. **Fallback** — если что-то сломалось, система продолжает работать.
7. **Project Isolation** — каждый проект изолирован на уровне данных, памяти и правил (ADR-005).

---

## 2. Архитектура высокого уровня

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         VOYAGE AI DEV FRAMEWORK v4.0                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │  Architect  │───→│  Developer  │───→│  Reviewer   │───→│   DevOps    │ │
│  │  (Планирует)│    │  (Пишет код)│    │  (Проверяет)│    │  (Деплоит)  │ │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘ │
│         │                  │                  │                  │        │
│         └──────────────────┴──────────────────┴──────────────────┘        │
│                                    │                                      │
│                         ┌──────────▼──────────┐                          │
│                         │   AGENT RUNTIME     │                          │
│                         │  plan → execute →   │                          │
│                         │  reflect → retry    │                          │
│                         └──────────┬──────────┘                          │
│                                    │                                      │
│  ┌─────────────────────────────────┼─────────────────────────────────┐  │
│  │                                 │                                 │  │
│  │  ┌─────────────┐  ┌─────────────▼─────────────┐  ┌─────────────┐ │  │
│  │  │   EVENT     │  │        MEMORY             │  │   TOOLS     │ │  │
│  │  │   STORE     │  │  ┌─────────┐ ┌─────────┐ │  │             │ │  │
│  │  │ (PostgreSQL │  │  │ Context │ │Semantic │ │  │  ┌───────┐  │ │  │
│  │  │  + SQLite   │  │  │  Tiers  │ │ Memory  │ │  │  │Sandbox│  │ │  │
│  │  │  fallback)  │  │  │(4-tier) │ │(ChromaDB│ │  │  │(Strategy│ │  │
│  │  │             │  │  └─────────┘ └─────────┘ │  │  │Pattern)│  │ │  │
│  │  │  Append-only │  │  ┌─────────┐              │  │  └───────┘  │ │  │
│  │  │  Replayable  │  │  │  AST    │              │  │  ┌───────┐  │ │  │
│  │  │  JSONL backup│  │  │ Graph   │              │  │  │Tool   │  │ │  │
│  │  └─────────────┘  │  │(tree-   │              │  │  │Engine │  │ │  │
│  │                   │  │ sitter) │              │  │  └───────┘  │ │  │
│  │                   │  └─────────┘              │  │             │ │  │
│  │                   └─────────────────────────────┘  └─────────────┘ │  │
│  │                                                                   │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │  │
│  │  │    SPECS    │  │   WORKFLOW  │  │    SELF-IMPROVING       │  │  │
│  │  │  ┌───────┐  │  │   ENGINE    │  │      ENGINE             │  │  │
│  │  │  │ ADR   │  │  │ (YAML-driven│  │  ┌─────────────────┐    │  │  │
│  │  │  │ Plan  │  │  │  state      │  │  │ Error pattern   │    │  │  │
│  │  │  │ Task  │  │  │  machine)   │  │  │ analysis        │    │  │  │
│  │  │  │Tracker│  │  │             │  │  │ Auto-rule       │    │  │  │
│  │  │  └───────┘  │  │  feature    │  │  │ generation      │    │  │  │
│  │  │             │  │  bugfix     │  │  │ RULES.md update │    │  │  │
│  │  │ Generates   │  │  hotfix     │  │  └─────────────────┘    │  │  │
│  │  │ TASK.md     │  │  refactor   │  │                         │  │  │
│  │  │ CONTEXT.json│  │             │  │                         │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │  │
│  │                                                                 │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                      INTEGRATIONS                                │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐   │  │
│  │  │  Kimi Code  │  │ Kimi Agent  │  │       GitHub            │   │  │
│  │  │  (VS Code)  │  │ (Web/API)   │  │  PR analysis, commits   │   │  │
│  │  │             │  │             │  │                         │   │  │
│  │  │ Reads       │  │ Runs        │  │                         │   │  │
│  │  │ TASK.md     │  │ workflows   │  │                         │   │  │
│  │  │ CONTEXT.json│  │             │  │                         │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘   │  │
│  │  ┌─────────────────────────────────────────────────────────────┐  │  │
│  │  │  Orchestrator Adapter (stub) — LangGraph / Custom Runtime   │  │  │
│  │  │  Phase 1: собственный Runtime                               │  │  │
│  │  │  Phase 2+: опциональный LangGraph через adapter              │  │  │
│  │  └─────────────────────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                    VISUALIZER (отдельное приложение)              │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐   │  │
│  │  │  Desktop    │  │   Mobile    │  │   Shared UI Core        │   │  │
│  │  │  (Electron) │  │(React Native│  │  (React + animations)   │   │  │
│  │  │             │  │             │  │                         │   │  │
│  │  │ Real-time   │  │ Real-time   │  │  EventTimeline          │   │  │
│  │  │ event stream│  │ event stream│  │  AgentStateGraph        │   │  │
│  │  │ animations  │  │ (adapted)   │  │  SandboxInspector       │   │  │
│  │  └─────────────┘  └─────────────┘  │  ContextPyramid         │   │  │
│  │                                      │  ASTGraph               │   │  │
│  │                                      │  TermTooltip            │   │  │
│  │                                      └─────────────────────────┘   │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Структура проекта

### Backend (внутри SkillTracer)

```
tools/voyage_framework/
├── pyproject.toml
├── README.md
├── RULES.md                    # ← Живой документ (auto-updated, project-scoped)
├── ADR/
│   ├── ADR-INDEX.md            # ← Единый реестр (см. Вариант Б)
│   ├── ADR-001-postgresql-events.md
│   ├── ADR-002-chromadb-semantic.md
│   ├── ADR-003-tree-sitter-ast.md
│   ├── ADR-004-dual-location.md
│   └── ADR-005-project-isolation.md
├── projects/                   # ← Project-scoped RULES и контекст (ADR-005)
│   └── skilltracer/
│       └── RULES.md            # Fallback на глобальный RULES.md если отсутствует
├── voyage_framework/
│   ├── __init__.py             # __version__ = "4.0.0"
│   ├── core/
│   │   ├── __init__.py
│   │   ├── models.py           # Все Pydantic-модели (+ project_id из ADR-005)
│   │   ├── event_engine.py     # PostgreSQL primary + SQLite fallback
│   │   └── storage.py          # Atomic writes, frontmatter, journal rotation
│   ├── security/
│   │   ├── __init__.py
│   │   ├── sandbox.py          # SecureExecutor + SandboxBackend (ABC)
│   │   ├── backends/           # ← Strategy Pattern (ADR-005)
│   │   │   ├── __init__.py
│   │   │   ├── docker.py       # DockerSandbox (Phase 1, dev-режим)
│   │   │   ├── seccomp.py      # SeccompSandbox (Phase 2, Linux production)
│   │   │   └── windows_job.py  # WindowsJobSandbox (Phase 2, Windows fallback)
│   │   ├── factory.py          # SandboxFactory: возвращает нужный backend
│   │   ├── policy.py           # Role-based permissions
│   │   ├── audit.py            # Audit trail logger
│   │   └── approval.py         # Human approval flow
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── engine.py           # Tool execution через SecureExecutor
│   │   ├── registry.py         # Discovery + @tool decorator
│   │   └── adapters/
│   │       ├── __init__.py
│   │       ├── git.py
│   │       ├── python.py       # pytest, mypy, ruff, pip (+ PyPI validation ADR-005)
│   │       ├── system.py       # df, ps
│   │       ├── files.py        # cat, grep, find
│   │       └── deploy.py       # systemctl, ssh, curl (approval required!)
│   ├── memory/
│   │   ├── __init__.py
│   │   ├── context_tiers.py    # 4-tier + tiktoken budgeting (+ project_id)
│   │   ├── semantic.py         # VectorStore abstraction → ChromaDB (namespace per project)
│   │   └── ast_manager.py      # Unified facade: ASTPython + ASTTypeScript
│   ├── specs/
│   │   ├── __init__.py
│   │   ├── adr.py              # ADR creation + storage
│   │   ├── plan.py             # Micro-phase planning
│   │   ├── task_generator.py   # Генерация TASK.md для Kimi Code (+ context wrapping ADR-005)
│   │   └── tracker.py          # Acceptance criteria tracker
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── runtime.py          # Plan → Execute → Reflect → Retry
│   │   ├── state_machine.py    # Agent internal states
│   │   ├── checkpoint.py       # Durable execution
│   │   └── roles/
│   │       ├── architect.py    # Spec-driven, ADR-aware
│   │       ├── developer.py    # Code + tests + AST impact
│   │       ├── reviewer.py     # Quality gates + auto-rules
│   │       └── devops.py       # Deploy + health-check + rollback
│   │       # ← Расширяемый регистр: qa.py, security.py, performance.py (v4.1+)
│   ├── workflows/
│   │   ├── __init__.py
│   │   ├── engine.py           # YAML-driven workflow execution
│   │   ├── definitions/
│   │   │   ├── feature.yaml      # Plan→Dev→Test→Review→Deploy
│   │   │   ├── bugfix.yaml     # Reproduce→Fix→Test→Review
│   │   │   ├── refactor.yaml   # AST impact→Plan→Migrate→Test
│   │   │   └── hotfix.yaml     # Emergency: skip review
│   │   └── hooks.py            # Kimi API integration
│   ├── integrations/
│   │   ├── __init__.py
│   │   ├── kimi_code.py        # Файловый протокол (TASK.md, CONTEXT.json)
│   │   ├── kimi_mcp.py         # Model Context Protocol adapter
│   │   ├── github.py           # PR analysis, commit log
│   │   └── orchestrator_adapter.py  # ← Stub (ADR-005): LangGraph / Custom Runtime
│   └── visualizer_api/
│       ├── __init__.py
│       ├── server.py           # FastAPI: SSE/WebSocket
│       ├── events_stream.py    # Real-time event feed
│       ├── state_feed.py       # Agent state feed
│       └── sandbox_visualizer.py # Command inspection API
└── tests/
    ├── __init__.py
    ├── conftest.py
    ├── test_core_models.py
    ├── test_event_engine.py
    ├── test_storage.py
    ├── test_sandbox.py
    ├── test_sandbox_backends.py   # ← Strategy Pattern tests (ADR-005)
    ├── test_policy.py
    ├── test_audit.py
    ├── test_approval.py
    ├── test_tools_engine.py
    ├── test_tools_adapters.py
    ├── test_context_tiers.py
    ├── test_semantic.py
    ├── test_ast_manager.py
    ├── test_specs_adr.py
    ├── test_specs_plan.py
    ├── test_specs_task_generator.py
    ├── test_specs_tracker.py
    ├── test_agents_runtime.py
    ├── test_agents_state_machine.py
    ├── test_workflows_engine.py
    ├── test_self_improving.py
    ├── test_visualizer_api.py
    └── test_project_isolation.py  # ← ADR-005: изоляция по project_id
```

### Visualizer (отдельный репозиторий)

```
voyage-visualizer/
├── packages/
│   ├── shared-ui/
│   │   ├── components/
│   │   │   ├── EventTimeline/
│   │   │   ├── AgentStateGraph/
│   │   │   ├── SandboxInspector/
│   │   │   ├── ContextPyramid/
│   │   │   ├── ASTGraph/
│   │   │   └── TermTooltip/
│   │   ├── hooks/
│   │   └── utils/
│   ├── api-client/
│   ├── desktop/          # Electron shell
│   └── mobile/           # React Native shell
├── docs/
│   └── knowledge_base/
│       ├── terms.json
│       └── adr_summaries.json
└── README.md
```

---

## 4. Компоненты системы

### 4.1 Core (Сердце)

| Компонент | Назначение | Ключевые классы |
|-----------|-----------|-----------------|
| **models.py** | Pydantic-модели всей системы | `Event`, `AgentState`, `ToolResult`, `NodeResult`, `SecurityPolicy`, `ApprovalRequest`, `ProjectContext`, `JournalEntry`, `Frontmatter` |
| **event_engine.py** | Хранение и replay событий | `EventEngine`, `DBBackend`, `PostgreSQLBackend`, `SQLiteBackend` |
| **storage.py** | Файловые операции | `atomic_write()`, `append_entry()`, `parse_frontmatter_entries()`, `journal_rotate()` |

**Event** — центральная сущность. Всё, что происходит в системе, логируется как Event:
- `plan_created` — Architect создал план
- `implementation_done` — Developer закончил код
- `error_logged` — что-то сломалось
- `rule_added` — SelfImprovingEngine добавил правило
- `tool_executed` — запущена команда через Sandbox
- `approval_requested` — dangerous tier требует human approval
- `agent_started` / `agent_completed` / `agent_failed` — lifecycle агента

**Event ID** — ULID (Universally Unique Lexicographically Sortable Identifier). 26 символов, сортируется по времени, не требует координации.

**Correlation ID** — связывает все события одной сессии/задачи. Позволяет replay полного workflow.

**Causation ID** — связывает события в цепочку причинно-следственных связей. Event B вызван Event A → `B.causation_id = A.event_id`.

**Project ID** *(ADR-005)* — `project_id: str` в каждой модели. Composite key: `(project_id, event_id)`. Event Engine фильтрует все операции по `project_id` на уровне приложения. SQLite fallback полностью поддерживает multi-project режим.

### 4.2 Security (Щит)

| Компонент | Назначение |
|-----------|-----------|
| **sandbox.py** | Блокировка опасных команд + `SandboxBackend` ABC |
| **backends/docker.py** | DockerSandbox (Phase 1, dev-режим) |
| **backends/seccomp.py** | SeccompSandbox (Phase 2, Linux production) |
| **backends/windows_job.py** | WindowsJobSandbox (Phase 2, Windows fallback) |
| **factory.py** | `SandboxFactory.get_backend(env)` — Strategy Pattern |
| **policy.py** | Ролевые разрешения |
| **audit.py** | Лог всех попыток выполнения |
| **approval.py** | Human approval для dangerous tier |

**SecureExecutor** проверяет команду в 5 уровней:
1. **Dangerous patterns** — regex: `rm -rf`, `curl`, `sudo`, `eval`, `system`...
2. **Whitelist** — base command должен быть в `allowed_commands`
3. **Path traversal** — аргументы не выходят за `project_root`
4. **Network guard** — флаги `--url`, `http://` блокируются если `allow_network=False`
5. **Dangerous tier** — `systemctl`, `ssh`, `curl` требуют `ApprovalRequest`

**RolePolicy** — что может каждая роль:
- **Architect:** писать ADR, менять архитектуру, доступ ко всем файлам
- **Developer:** писать код, запускать тесты, доступ к `backend/` и `frontend/`
- **Reviewer:** проверять код, запускать линтеры, добавлять правила в `RULES.md`
- **DevOps:** деплоить, мониторить, но dangerous tier всё равно требует approval

*Phase 1 Security: только deterministic filters (ADR-005). Phase 2: HMAC signing, hash chain, PostgreSQL RLS.*

### 4.3 Tools (Руки)

| Адаптер | Инструменты | Требует approval |
|---------|-------------|------------------|
| **git.py** | `git_diff`, `git_log`, `git_status` | Нет |
| **python.py** | `pytest`, `mypy`, `ruff_check`, `pip_install` | Нет |
| **system.py** | `df_h`, `ps_aux` | Нет |
| **files.py** | `cat_file`, `grep_code`, `find_files` | Нет |
| **deploy.py** | `systemctl_restart`, `ssh_command`, `curl_url` | **Да** |

**ToolEngine** — единая точка входа. Все инструменты регистрируются через `@tool` decorator. Auto-discovery через `registry.discover()`.

**PyPI Validation** *(ADR-005)*: `python.py` adapter проверяет имя пакета через PyPI JSON API перед `pip install` (защита от typosquatting).

### 4.4 Memory (Мозг)

#### 4-tier Context

| Tier | Что хранит | TTL | Приоритет |
|------|-----------|-----|-----------|
| **Short-term** | Текущая сессия (correlation_id) | 1 сессия | 🔴 Высший |
| **Working** | Текущая микро-фаза | До завершения фазы | 🟠 Высокий |
| **Long-term** | ADR, архитектура, история | 90 дней | 🟡 Средний |
| **Semantic** | Похожие решения по смыслу | Бессрочно | 🟢 Низкий |

**Token Budgeting:** `tiktoken` считает токены каждого tier. Если сумма > `max_tokens` — обрезает снизу (semantic → long-term → working). Short-term **никогда** не обрезается.

**Semantic Memory:** ChromaDB с коллекциями по типам. Namespace per project: `f"voyage_{project_id}"`. Fallback на SQLite keyword search. Cross-project semantic query запрещена на уровне runtime.

**RAG Guard** *(ADR-005)*: pre-embedding regex-фильтр (`ignore previous instructions`, `bypass sandbox` и т.д.). Retrieved context оборачивается в `<<retrieved_context>>` с явной инструкцией «not executable».

**AST Manager:** tree-sitter для Python + TypeScript. Строит граф зависимостей. Impact analysis: "что сломается при изменении файла X?"

### 4.5 Specs (ТЗ для Kimi Code)

| Компонент | Назначение |
|-----------|-----------|
| **adr.py** | Architecture Decision Records — почему принято решение X |
| **plan.py** | Разбиение фазы на микро-фазы с acceptance criteria |
| **task_generator.py** | Генерация `TASK.md` + `CONTEXT.json` для Kimi Code |
| **tracker.py** | Отслеживание: criteria met? proof? |

**Формат TASK.md:**
```markdown
# TASK: Add FSM persistence to bot

## Context
[CompiledContext: short-term + working + long-term + semantic]

## Acceptance Criteria
- [ ] PostgresStorage подключён к aiogram Dispatcher
- [ ] Graceful shutdown сохраняет state
- [ ] /health endpoint возвращает 200
- [ ] Тесты проходят (pytest)

## Rules (from RULES.md)
- Все async функции должны иметь type hints
- Не использовать sync SQLAlchemy session в async коде

## ADR References
- ADR-001: PostgreSQL для Event Store
- ADR-005: Project Isolation (project_id в моделях)

## Instructions
1. Напиши код согласно критериям.
2. Запусти mypy и pytest перед финализацией.
3. Не меняй файлы вне backend/app/bot/.

---
Generated by Voyage Framework v4.0 | Phase: Phase 2 | Micro-Phase: M4
Project: skilltracer
```

**Формат CONTEXT.json:**
```json
{
  "role": "developer",
  "task": "Add FSM persistence",
  "micro_phase": "M4",
  "project_id": "skilltracer",
  "context_summary": "...",
  "relevant_files": ["backend/app/bot/main.py", "backend/app/bot/storage.py"],
  "rules": ["Type hints required", "Async SQLAlchemy only"],
  "adrs": ["ADR-001", "ADR-005"],
  "criteria": ["PostgresStorage connected", "Graceful shutdown", "Health endpoint", "Tests pass"]
}
```

### 4.6 Agents (Душа)

#### Agent Runtime — цикл работы

```
┌─────────┐    ┌──────────┐    ┌───────────┐    ┌────────┐    ┌─────────┐
│  idle   │───→│ planning │───→│ executing │───→│reflect │───→│  done   │
└─────────┘    └──────────┘    └───────────┘    └────────┘    └─────────┘
                    │                │                │
                    │           ┌────┴────┐           │
                    │           │ retrying│←──────────┘
                    │           └────┬────┘  (if confidence < 0.7
                    │                │        or tests failed)
                    └────────────────┘
```

**Plan:** Architect анализирует задачу, AST graph, контекст → генерирует список шагов.

**Execute:** Developer выполняет шаги, запускает tools (pytest, mypy), читает output.

**Reflect:** Анализирует результат. Что пошло хорошо? Что улучшить? Confidence score?

**Retry:** Если confidence < 0.7 или тесты упали → повторить шаг (max 3 retries).

**Checkpoint:** После каждого node состояние сохраняется. Можно resume после crash.

#### Расширяемая ролевая модель (v4.1+)

Phase 1 реализует 4 базовые роли. Реестр ролей (`RoleRegistry`) позволяет добавлять новые без изменения `agents/runtime.py`:

| Роль | Phase | Назначение |
|------|-------|------------|
| **Architect** | 1 | Spec-driven, ADR-aware |
| **Developer** | 1 | Code + tests + AST impact |
| **Reviewer** | 1 | Quality gates + auto-rules |
| **DevOps** | 1 | Deploy + health-check + rollback |
| **QA Engineer** | 2 | Автотесты, тест-кейсы, flaky-test analysis |
| **Security Engineer** | 2 | SecOps/AppSec: уязвимости, audit, compliance |
| **Performance Engineer** | 2 | Нагрузочное тестирование, оптимизация |
| **Product Manager** | 3 | Приоритизация фич, бэклог, метрики |

### 4.7 Workflows (Сценарии)

| Workflow | Путь | Особенности |
|----------|------|-------------|
| **feature** | Plan→Dev→Test→Review→Deploy | Стандартный, с review |
| **bugfix** | Reproduce→Fix→Test→Review | Требует reproduction steps |
| **refactor** | AST impact→Plan→Migrate→Test | Проверка зависимостей перед началом |
| **hotfix** | Fix→Test→Deploy | Skip review, direct deploy |

Workflow определяется в YAML:
```yaml
nodes:
  - name: start
    role: system
    transitions:
      - condition: always
        target: architect_plan
  - name: architect_plan
    role: ai-architect
    tools: [git_log, cat_file]
    transitions:
      - condition: success
        target: developer_implement
      - condition: failure
        target: end
  - name: developer_implement
    role: ai-developer
    tools: [cat_file, grep_code, pytest]
    transitions:
      - condition: success
        target: run_tests
      - condition: failure
        target: reviewer_fix
```

### 4.8 Self-Improving Engine

**Механизм:**
1. Слушает `error_logged` events (внутри одного `project_id`)
2. Извлекает `error_type` и `pattern`
3. Ищет похожие ошибки в истории проекта
4. Генерирует `RuleSuggestion`:
   - `pattern` — что вызвало ошибку
   - `rule_text` — правило для `RULES.md`
   - `severity` — must/should/may
5. Проверяет, нет ли уже такого правила (по hash)
6. Atomic append в project-scoped `RULES.md`
7. Логирует `rule_added` event

**Пример:**
```
Error: missing_type_hints in async def handle_message
Pattern: async def without ->
→ Rule: "Всегда добавляй return type annotation в async функции"
→ RULES.md updated
→ Event: rule_added logged
```

---

## 5. Потоки данных (Data Flow)

### 5.1 Полный цикл задачи (Feature)

```
Human: "Добавить FSM persistence в бота"
    │
    ▼
┌─────────────────┐
│  Architect      │
│  - Читает CONTEXT (project_id="skilltracer")
│  - Анализирует AST graph
│  - Создаёт PLAN.md
│  - Пишет ADR (если нужно)
│  - Генерирует TASK.md + CONTEXT.json
└────────┬────────┘
         │ log: plan_created (project_id="skilltracer")
         ▼
┌─────────────────┐
│  Kimi Code      │
│  (VS Code)      │
│  - Читает TASK.md
│  - Пишет код
│  - Запускает pytest
│  - Возвращает результат
└────────┬────────┘
         │ log: implementation_done (project_id="skilltracer")
         ▼
┌─────────────────┐
│  Reviewer       │
│  - Запускает mypy, ruff
│  - Проверяет AST impact
│  - Проверяет RULES.md
│  - Если ошибки → error_logged
│  - SelfImprovingEngine → rule_added
└────────┬────────┘
         │ log: error_logged / implementation_done
         ▼
┌─────────────────┐
│  DevOps         │
│  - Проверяет df, ps
│  - Запрашивает approval для deploy
│  - Human approves
│  - Deploy
└────────┬────────┘
         │ log: micro_phase_done (project_id="skilltracer")
         ▼
┌─────────────────┐
│  Event Store    │
│  - Все события append-only (project_id shard)
│  - Replay → ProjectContext
│  - Export → markdown files
└─────────────────┘
```

### 5.2 Self-Improving Flow

```
Developer пишет код
    │
    ▼
pytest FAILED (missing type hints)
    │
    ▼
ToolResult → error_logged event (project_id="skilltracer")
    │
    ▼
SelfImprovingEngine.process_error()
    │
    ▼
Pattern detected: "async def without ->"
    │
    ▼
Generate RuleSuggestion
    │
    ▼
Check project-scoped RULES.md (not exists → fallback global)
    │
    ▼
Atomic append to RULES.md
    │
    ▼
Log rule_added event
    │
    ▼
Next Developer читает RULES.md
    │
    ▼
Не повторяет ошибку
```

### 5.3 Visualizer Data Flow

```
EventEngine (PostgreSQL/SQLite, filtered by project_id)
    │
    ▼ SSE/WebSocket
Visualizer API (FastAPI)
    │
    ├──────→ Desktop (Electron)
    │           Real-time animations
    │           EventTimeline
    │           SandboxInspector
    │
    └──────→ Mobile (React Native)
                Adapted UI
                Push notifications
                Offline mode (cache)
```

---

## 6. Роли агентов

### 6.1 Architect (Principal Architect / Tech Lead)

**Может:**
- Писать ADR
- Менять архитектуру
- Создавать планы и микро-фазы
- Генерировать TASK.md для Kimi Code
- Читать весь проект

**Инструменты:** `git_log`, `cat_file`, `query_ast`, `semantic_query`

**Не может:**
- Писать production код (только specs)
- Деплоить
- Выполнять dangerous-tier команды

**Prompt focus:**
- Анализ требований
- AST graph: куда встраивать новый код
- ADR: обоснование решений
- Acceptance criteria

### 6.2 Developer (Senior Developer)

**Может:**
- Писать код
- Запускать pytest, mypy, ruff
- Читать файлы проекта
- Исправлять баги по traceback

**Инструменты:** `cat_file`, `grep_code`, `pytest`, `mypy`, `ruff_check`, `pip_install`

**Не может:**
- Писать ADR
- Менять архитектуру без approval Architect
- Деплоить

**Prompt focus:**
- Реализация по спецификации
- Проверка кода перед ответом (mypy + pytest)
- AST impact: кто зависит от изменяемого файла

### 6.3 Reviewer (Code Reviewer / QA Engineer)

**Может:**
- Запускать mypy, ruff, pytest
- Анализировать AST impact
- Проверять RULES.md
- Генерировать правила для SelfImprovingEngine
- Блокировать merge

**Инструменты:** `git_diff`, `mypy`, `ruff_check`, `cat_file`, `grep_code`

**Не может:**
- Писать код (только рекомендации)
- Деплоить

**Prompt focus:**
- Quality gates
- Impact analysis
- RULES.md compliance
- Risk assessment

### 6.4 DevOps (DevOps Engineer)

**Может:**
- Мониторить ресурсы (df, ps)
- Деплоить (с approval)
- Health-check
- Rollback

**Инструменты:** `df_h`, `ps_aux`, `systemctl_restart` (approval), `ssh_command` (approval), `curl_url` (approval)

**Не может:**
- Писать код
- Менять архитектуру
- Без approval выполнять dangerous-tier

**Prompt focus:**
- Pre-deploy checks
- Rollback plan
- Resource monitoring
- Human approval для production commands

---

## 7. Цикл работы агента

```
┌─────────────────────────────────────────────────────────────┐
│                     AGENT RUNTIME CYCLE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. PLAN                                                    │
│     ├─ Compile context (4-tier memory, project_id filtered)   │
│     ├─ Analyze AST graph                                    │
│     ├─ Generate steps                                       │
│     └─ Set status: planning → idle                          │
│                                                             │
│  2. EXECUTE                                                 │
│     ├─ Load checkpoint (if resuming)                        │
│     ├─ For each step:                                       │
│     │   ├─ Run tools through SecureExecutor (Strategy)      │
│     │   ├─ Collect ToolResults                              │
│     │   └─ Log tool_executed events (with project_id)       │
│     └─ Set status: executing → reflecting                   │
│                                                             │
│  3. REFLECT                                                 │
│     ├─ Analyze results                                      │
│     ├─ Calculate confidence (0.0-1.0)                       │
│     ├─ What went well?                                      │
│     ├─ What to improve?                                     │
│     └─ Set status: reflecting → retrying / done            │
│                                                             │
│  4. RETRY (if needed)                                       │
│     ├─ If confidence < 0.7 → retry step                     │
│     ├─ If tests failed → analyze traceback → fix            │
│     ├─ Increment retry_count                                │
│     ├─ If retry_count > max_retries → failed                │
│     └─ Set status: retrying → executing / failed            │
│                                                             │
│  5. CHECKPOINT                                              │
│     ├─ Save AgentState to disk (project-scoped path)        │
│     ├─ Save workflow history                                │
│     └─ Can resume from any checkpoint                       │
│                                                             │
│  6. DONE / FAILED                                           │
│     ├─ Log agent_completed or agent_failed                  │
│     ├─ If failed → notify human                             │
│     └─ If done → trigger next workflow node                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Безопасность

### 8.1 Уровни защиты

| Уровень | Что проверяет | Пример блокировки |
|---------|--------------|-------------------|
| **L1: Patterns** | Regex dangerous patterns | `rm -rf /` → BLOCKED |
| **L2: Whitelist** | Base command в allowed set | `nc` → BLOCKED |
| **L3: Path** | Аргументы не выходят за project_root | `cat /etc/passwd` → BLOCKED |
| **L4: Network** | Network flags если запрещено | `curl https://evil.com` → BLOCKED |
| **L5: Dangerous Tier** | systemctl, ssh, curl | `systemctl restart` → APPROVAL_REQUIRED |
| **L3.5: RAG Guard** *(ADR-005)* | Pre-embedding фильтр инъекций | `ignore previous instructions` → SANITIZED |
| **L2.5: PyPI Validation** *(ADR-005)* | Typosquatting пакетов | `reqeusts` → BLOCKED |

### 8.2 Approval Flow

```
AI запрашивает: systemctl restart skilltracer
    │
    ▼
SecureExecutor → APPROVAL_REQUIRED
    │
    ▼
ApprovalQueue.create_request()
    │
    ▼
.voyage_approval_pending.json обновлён
    │
    ▼
Human читает файл
    │
    ├─ approve → status=approved → AI retry
    ├─ reject → status=rejected → AI abort
    └─ 24h timeout → status=expired → AI abort
```

### 8.3 Audit Trail

Все команды логируются:
- `timestamp`, `agent_id`, `role`, `command`, `result`, `exit_code`, `blocked`, `reason`, `approval_id`, `project_id`

Можно replay: "что делал AI вчера в 15:00 в проекте skilltracer?"

---

## 9. Память и контекст

### 9.1 4-Tier Memory Model

```
        ┌─────────────────┐
        │   SHORT-TERM    │  ← Текущая сессия
        │   (короткая)    │     10 последних событий
        │   🔴 Высший     │     Никогда не обрезается
        │   приоритет     │
        └────────┬────────┘
                 │
        ┌────────▼────────┐
        │   WORKING       │  ← Текущая микро-фаза
        │   (рабочая)     │     20 событий фазы
        │   🟠 Высокий    │     Обрезается последним
        │   приоритет     │
        └────────┬────────┘
                 │
        ┌────────▼────────┐
        │   LONG-TERM     │  ← ADR, архитектура, история
        │   (долгосрочная)│     50 событий за 90 дней
        │   🟡 Средний    │     Role-aware + project_id filtering
        │   приоритет     │
        └────────┬────────┘
                 │
        ┌────────▼────────┐
        │   SEMANTIC      │  ← Похожие решения по смыслу
        │   (семантическая)│    Vector search (project namespace)
        │   🟢 Низкий     │    Обрезается первым
        │   приоритет     │
        └─────────────────┘
```

### 9.2 Token Budgeting

```
max_tokens = 12000

short_term_tokens  = count(short_term)   # 500 tokens
working_tokens     = count(working)      # 2000 tokens
long_term_tokens   = count(long_term)   # 4000 tokens
semantic_tokens    = count(semantic)    # 3000 tokens
────────────────────────────────────────
total = 9500 tokens ≤ 12000 ✅  OK

Если total > max_tokens:
  while total > max_tokens:
    if semantic has items:
      remove last semantic item
    elif long_term has items:
      remove last long_term item
    elif working has items:
      remove last working item
    # short-term NEVER removed
```

### 9.3 Semantic Query Example

```
Human: "Почему отказались от UUID?"
    │
    ▼
SemanticMemory.query("Why UUID was rejected", project_id="skilltracer")
    │
    ▼
ChromaDB finds (внутри namespace "voyage_skilltracer"):
  - ADR-005: "UUID vs ULID" (distance: 0.12)
  - Event: decision_made "ULID chosen over UUID" (distance: 0.18)
  - Event: error_logged "UUID collision in tests" (distance: 0.25)
    │
    ▼
Return top 3 with metadata
```

---

## 10. Интеграции

### 10.1 Kimi Code (VS Code)

**Протокол:** Файловый + MCP

**Файловый:**
- Framework пишет `TASK.md` + `CONTEXT.json` в рабочую директорию
- Kimi Code читает их при старте сессии
- Kimi Code пишет результат в `RESULT.md`
- Framework читает `RESULT.md` и логирует `implementation_done`

**MCP (Model Context Protocol):**
- Framework предоставляет tools: `query_ast`, `semantic_search`, `get_context`
- Kimi Code вызывает tools через MCP
- Более тесная интеграция, чем файловый обмен

### 10.2 Kimi Agent (Web)

- Framework запускает Kimi Agent через API
- Kimi Agent выполняет workflow nodes
- Результаты логируются в EventEngine
- Human может мониторить через Visualizer

### 10.3 GitHub

- Чтение PR comments
- Анализ commit log
- Автоматическое summary изменений
- Интеграция с Reviewer: "проверь этот PR"

### 10.4 Orchestrator Adapter (ADR-005)

**Phase 1:** Собственный Agent Runtime (`plan → execute → reflect → retry`).

**Phase 2+:** Опциональный LangGraph через `OrchestratorAdapter`:
```python
# Сегодня
runtime = CustomAgentRuntime()

# Завтра (без изменений в agents/runtime.py)
from integrations.orchestrator_adapter import LangGraphAdapter
runtime = LangGraphAdapter(config)
```

---

## 11. Self-Improving Engine

### 11.1 Механизм обучения

```
┌─────────────────────────────────────────────────────────────┐
│              SELF-IMPROVING ENGINE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Input: error_logged event (project_id scoped)              │
│    {                                                        │
│      "error_type": "missing_type_hints",                    │
│      "pattern": "async def without ->",                     │
│      "file": "backend/app/bot/handlers.py",                 │
│      "line": 42,                                            │
│      "project_id": "skilltracer"                            │
│    }                                                        │
│                                                             │
│  Step 1: Pattern Analysis                                   │
│    - Извлечь error_type + pattern                          │
│    - Найти похожие ошибки в истории проекта                 │
│    - Если >3 похожих → паттерн подтверждён                  │
│                                                             │
│  Step 2: Rule Generation                                    │
│    - Сгенерировать rule_text                                │
│    - Определить severity (must/should/may)                │
│    - Пример:                                                │
│      "Всегда добавляй return type annotation              │
│       в async функции: async def handle(...) -> ResultType" │
│                                                             │
│  Step 3: Deduplication                                      │
│    - Hash rule_text                                         │
│    - Проверить project-scoped RULES.md (fallback global)    │
│    - Если exists → skip                                     │
│                                                             │
│  Step 4: Apply                                              │
│    - Atomic append в RULES.md проекта                     │
│    - Log rule_added event                                   │
│    - Update ProjectContext.forbidden                      │
│                                                             │
│  Output: RULES.md updated, rule_added event                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 11.2 Примеры авто-правил

| Error Pattern | Generated Rule | Severity |
|-------------|--------------|----------|
| `async def without ->` | Всегда добавляй return type annotation в async функции | must |
| `race_condition in tests` | Используй asyncio.Lock для shared state в async тестах | should |
| `missing await` | Проверяй, что все async вызовы имеют await | must |
| `sync SQLAlchemy in async` | Используй только async_sessionmaker в async коде | must |
| `hardcoded secret` | Все секреты через pydantic-settings + .env | must |

---

## 12. Фазы разработки

### Phase 1: Foundation & Brain (в разработке)
**Статус:** 🟡 In Progress  
**Цель:** Всё для мышления и безопасности

| Модуль | Статус | Файлы |
|--------|--------|-------|
| Core Models (+ project_id) | 🟡 | `core/models.py` |
| Event Engine (+ project filter) | 🟡 | `core/event_engine.py` |
| Storage | 🟡 | `core/storage.py` |
| Security Sandbox (+ Strategy Pattern) | 🟡 | `security/sandbox.py`, `security/backends/docker.py`, `security/factory.py` |
| Policy | 🟡 | `security/policy.py` |
| Audit | 🟡 | `security/audit.py` |
| Approval | 🟡 | `security/approval.py` |
| Tool Engine | 🟡 | `tools/engine.py`, `tools/registry.py` |
| Tool Adapters (+ PyPI validation) | 🟡 | `tools/adapters/*.py` |
| Context Tiers (+ project_id) | 🟡 | `memory/context_tiers.py` |
| Semantic Memory (+ namespace per project) | 🟡 | `memory/semantic.py` |
| AST Manager | 🟡 | `memory/ast_manager.py` |
| Specs (+ context wrapping) | 🟡 | `specs/*.py` |
| Self-Improving Engine (+ project scope) | 🟡 | `self_improving/engine.py` |
| Orchestrator Adapter (stub) | 🟡 | `integrations/orchestrator_adapter.py` |

**AC:** 10 acceptance criteria (см. System Prompt) + AC по изоляции проектов (ADR-005).

### Phase 2: Soul & Connectors
**Статус:** ⚪ Not Started  
**Цель:** Всё для действия и общения

| Модуль | Описание |
|--------|----------|
| Agent Runtime | Plan→Execute→Reflect→Retry |
| Agent State Machine | idle→planning→executing→reflecting→... |
| Checkpoint | Durable execution, resume |
| Workflow Engine | YAML-driven, real branching |
| Workflow Definitions | feature, bugfix, refactor, hotfix |
| Kimi Code Integration | Файловый протокол + MCP |
| Kimi Agent Integration | Web API adapter |
| GitHub Integration | PR analysis, commits |
| Visualizer API | FastAPI SSE/WebSocket |
| **QA Engineer Agent** | Автотесты, flaky-test analysis |
| **Security Engineer Agent** | Уязвимости, audit, compliance |
| **Performance Engineer Agent** | Нагрузочное тестирование |
| **LangGraph Adapter** | Опциональная интеграция через OrchestratorAdapter |
| **Seccomp Sandbox** | Linux production backend |

### Phase 3: Eyes (Visualizer)
**Статус:** ⚪ Not Started  
**Цель:** Интерактивное учебное пособие

| Компонент | Стек |
|-----------|------|
| Shared UI Core | React + Framer Motion |
| Desktop App | Electron |
| Mobile App | React Native |
| API Client | FastAPI client |
| Knowledge Base | terms.json, adr_summaries.json |
| **Product Manager Agent** | Приоритизация фич, бэклог |

---

## 13. Термины и аббревиатуры

| Термин | Расшифровка | Описание |
|--------|-------------|----------|
| **VADF** | Voyage AI Dev Framework | Название фреймворка |
| **ADR** | Architecture Decision Record | Документ, объясняющий архитектурное решение |
| **AST** | Abstract Syntax Tree | Дерево разбора кода для анализа структуры |
| **AC** | Acceptance Criteria | Критерии приёмки задачи |
| **ChromaDB** | — | Векторная база данных для semantic search |
| **Event Sourcing** | — | Паттерн: всё состояние — производное от событий |
| **FSM** | Finite State Machine | Конечный автомат (для бота) |
| **MCP** | Model Context Protocol | Протокол интеграции AI с инструментами |
| **NodeResult** | — | Результат выполнения узла workflow |
| **pgvector** | — | Векторное расширение PostgreSQL |
| **Semantic Memory** | — | Поиск по смыслу через embeddings |
| **SSE** | Server-Sent Events | Протокол real-time потока данных |
| **Tauri** | — | Фреймворк для desktop-приложений (альтернатива Electron) |
| **tiktoken** | — | Библиотека подсчёта токенов OpenAI |
| **ToolResult** | — | Результат выполнения инструмента |
| **tree-sitter** | — | Парсер для построения AST |
| **ULID** | Universally Unique Lexicographically Sortable Identifier | Уникальный сортируемый ID |
| **WAL** | Write-Ahead Logging | Режим журналирования БД для durability |
| **YAML** | YAML Ain't Markup Language | Формат конфигурации workflow |

---

## 14. Roadmap

### v4.0 (2026 Q2) — Foundation
- ✅ Core Models + Event Engine (+ project_id)
- ✅ Security Sandbox + Approval (+ Strategy Pattern, Docker backend)
- ✅ Tool Engine + Adapters (+ PyPI validation)
- ✅ 4-tier Memory + Semantic (+ namespace per project)
- ✅ AST Manager (Python + TS)
- ✅ Specs (ADR, Plan, Task, Tracker + context wrapping)
- ✅ Self-Improving Engine (+ project scope)
- ✅ Orchestrator Adapter (stub)
- 🔄 Agent Runtime (Phase 2)
- 🔄 Workflow Engine (Phase 2)
- 🔄 Integrations (Phase 2)

### v4.1 (2026 Q3) — Intelligence
- pgvector вместо ChromaDB (опционально)
- Incremental AST re-indexing (по mtime)
- LSP integration для symbol resolution
- Parallel agent execution
- Auto-retry для flaky tests
- **QA Engineer Agent**
- **Security Engineer Agent**
- **Performance Engineer Agent**
- HMAC-SHA256 event signing
- Hash chain для Audit Trail
- PostgreSQL RLS как primary изоляция
- ML-based anomaly detection

### v4.2 (2026 Q4) — Autonomy
- Real autonomous execution (без human-in-the-loop для routine)
- Semantic diff (AI понимает, что изменилось в коде)
- Predictive error detection (предсказывает ошибки до commit)
- Multi-project support (не только SkillTracer)
- LangGraph adapter (опционально)
- SeccompSandbox + WindowsJobSandbox

### v5.0 (2027) — Ecosystem
- Visualizer v2 (3D graph, VR/AR)
- Community rules marketplace
- Integration с другими AI (Claude, GPT, Gemini)
- Plugin system для custom tools
- **Product Manager Agent**

---

## 15. Чек-листы

### Чек-лист запуска Phase 1

- [ ] Создать папку `F:\WORK\skilltracer\tools\voyage_framework\`
- [ ] Создать подпапки (core, security, security/backends, tools, memory, specs, agents, workflows, integrations, visualizer_api, tests, projects/skilltracer)
- [ ] Скопировать `RULES.md` (глобальный)
- [ ] Создать `projects/skilltracer/RULES.md` (project-scoped, может быть пустым)
- [ ] Скопировать 4 ADR в `ADR/` + создать `ADR-INDEX.md`
- [ ] Создать `pyproject.toml`
- [ ] `pip install -e ".[dev]"`
- [ ] Запустить Kimi Agent с System Prompt
- [ ] Сгенерировать `core/models.py` (+ project_id)
- [ ] Запустить `pytest tests/test_core_models.py`
- [ ] Сгенерировать `security/backends/docker.py` + `security/factory.py`
- [ ] Сгенерировать `integrations/orchestrator_adapter.py` (stub)
- [ ] Сгенерировать остальные модули по порядку
- [ ] Пройти все 10 AC + AC по изоляции проектов

### Чек-лист перехода к Phase 2

- [ ] Все AC Phase 1 пройдены
- [ ] `mypy voyage_framework/` — clean
- [ ] `ruff check voyage_framework/` — clean
- [ ] `pytest tests/` — all passed
- [ ] `RULES.md` (глобальный + project-scoped) содержит ≥5 правил (включая auto-generated)
- [ ] EventEngine логирует события без ошибок (с project_id)
- [ ] Sandbox блокирует dangerous commands (Docker backend)
- [ ] TaskGenerator создаёт валидный TASK.md (с project_id и context wrapping)
- [ ] SelfImprovingEngine добавляет правила из ошибок (в project-scoped RULES.md)
- [ ] OrchestratorAdapter stub компилируется без ошибок

### Чек-лист запуска Visualizer (Phase 3)

- [ ] Phase 2 завершена
- [ ] Visualizer API сервер запущен (`localhost:8000`)
- [ ] SSE `/events/stream` возвращает события (filtered by project_id)
- [ ] Desktop app (Electron) подключается к API
- [ ] Mobile app (React Native) подключается к API
- [ ] EventTimeline показывает анимацию событий
- [ ] SandboxInspector визуализирует проверку команд
- [ ] TermTooltip показывает определения при тапе
- [ ] Knowledge Base загружается без ошибок

---

## Приложение A: Примеры использования

### Пример 1: Запуск задачи "Add FSM persistence"

```python
from voyage_framework.core.event_engine import EventEngine
from voyage_framework.agents.runtime import AgentRuntime
from voyage_framework.workflows.engine import WorkflowEngine
from voyage_framework.specs.task_generator import TaskGenerator

# 1. Инициализация
engine = EventEngine()
engine.set_project_context("skilltracer")  # ADR-005

# 2. Создание задачи
generator = TaskGenerator(engine)
task = generator.generate(
    role="developer",
    task="Add FSM persistence to bot",
    micro_phase="M4-Bot-Engine",
    project_id="skilltracer"
)

# 3. TASK.md записан в рабочую директорию
Path("TASK.md").write_text(task.task_markdown)
Path("CONTEXT.json").write_text(json.dumps(task.context_json))

# 4. Kimi Code читает TASK.md и пишет код
# (человек копирует TASK.md в VS Code)

# 5. После выполнения — логируем результат
engine.append(Event(
    event_type="implementation_done",
    payload={"task": "Add FSM persistence", "criteria_met": True},
    micro_phase="M4-Bot-Engine",
    correlation_id="sess-001",
    project_id="skilltracer"  # ADR-005
))
```

### Пример 2: Self-Improving на ошибке

```python
from voyage_framework.self_improving.engine import SelfImprovingEngine

sim = SelfImprovingEngine(
    engine,
    rules_path=Path("projects/skilltracer/RULES.md"),  # project-scoped
    global_rules_path=Path("RULES.md")  # fallback
)

# Developer допустил ошибку
error_event = Event(
    event_type="error_logged",
    payload={
        "error_type": "missing_type_hints",
        "pattern": "async def without ->",
        "file": "backend/app/bot/handlers.py",
        "line": 42
    },
    project_id="skilltracer"
)

# SelfImprovingEngine обрабатывает
suggestion = sim.process_error(error_event)
if suggestion:
    sim.apply_rule(suggestion)
    # projects/skilltracer/RULES.md обновлён
    # Event "rule_added" залогирован
```

### Пример 3: Sandbox + Approval (Strategy Pattern)

```python
from voyage_framework.security.sandbox import SecureExecutor
from voyage_framework.security.factory import SandboxFactory
from voyage_framework.security.policy import PolicyEnforcer

# Phase 1: Docker backend
backend = SandboxFactory.get_backend("docker")
executor = SecureExecutor(
    SecurityPolicy(),
    project_root=Path("."),
    backend=backend  # Strategy Pattern
)

# Безопасная команда — выполняется
result = executor.execute(["pytest", "backend/tests"])
print(result.success)  # True

# Опасная команда — блокируется + требует approval
result = executor.execute(["systemctl", "restart", "skilltracer"])
print(result.blocked)  # True
print("APPROVAL_REQUIRED" in result.stderr)  # True

# Human approves через файл
# AI retry с approved request
```

---

## Приложение B: Связь с прошлыми версиями

| Версия | Что было | Что стало в v4.0 |
|--------|----------|-------------------|
| **v1.x** | Markdown + Manual | Event Sourcing + Auto |
| **v2.x** | Pydantic + Jinja2 | Pydantic v2 + Specs |
| **v3.0** | Event Store (SQLite), AST (Python ast), Tools (no sandbox), State Machine (broken branching) | PostgreSQL+SQLite, tree-sitter multi-lang, Secure Sandbox (Strategy), real branching |
| **v3.1** | Sandbox (buggy), Models (ULID), Agent Runtime (concept) | Fixed sandbox, full runtime, self-improving |
| **v4.0** | — | Всё выше + Spec-Driven + Visualizer API + Dual-Location + Project Isolation + Orchestrator Adapter |

---

## Приложение C: ADR-INDEX ссылка

Все архитектурные решения вынесены в отдельные файлы. См.:
- **[ADR-INDEX.md](ADR/ADR-INDEX.md)** — реестр всех ADR
- **[ADR-001](ADR/ADR-001-postgresql-events.md)** — PostgreSQL для Event Store
- **[ADR-002](ADR/ADR-002-chromadb-semantic.md)** — ChromaDB для Semantic Memory
- **[ADR-003](ADR/ADR-003-tree-sitter-ast.md)** — tree-sitter для AST Indexing
- **[ADR-004](ADR/ADR-004-dual-location.md)** — Dual-Location Strategy
- **[ADR-005](ADR/ADR-005-project-isolation.md)** — Project Isolation & Multi-Project Core

**Правило:** Мастер-документ описывает *что* делает система. ADR описывают *почему* принято конкретное решение. Не дублируй ADR в мастер-документе.

---

**Версия документа:** 2.0  
**Дата:** 2026-05-26  
**Авторы:** AndreyVoyage (Human) + AI Architect  
**Следующее обновление:** После завершения Phase 1
