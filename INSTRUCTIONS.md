# INSTRUCTIONS.md — Инструкции по разработке Voyage Framework v4.0

> **Назначение:** Пошаговая инструкция для Human (AndreyVoyage) — как пользоваться фреймворком от идеи до деплоя.  
> **Формат:** Чек-листы + команды + примеры. Без теории — только действия.  
> **Обновляется:** При смене Phase или добавлении нового workflow.

---

## Содержание

1. [Ежедневный workflow](#1-ежедневный-workflow)
2. [Как начать новую задачу](#2-как-начать-новую-задачу)
3. [Как использовать Kimi Code](#3-как-использовать-kimi-code)
4. [Как использовать Kimi Agent](#4-как-использовать-kimi-agent)
5. [Как добавить новую роль](#5-как-добавить-новую-роль)
6. [Как добавить новый инструмент](#6-как-добавить-новый-инструмент)
7. [Как обновить ADR](#7-как-обновить-adr)
8. [Как работать с Sandbox](#8-как-работать-с-sandbox)
9. [Как работать с Approval Flow](#9-как-работать-с-approval-flow)
10. [Как запустить тесты](#10-как-запустить-тесты)
11. [Как деплоить](#11-как-деплоить)
12. [Как использовать LangGraph (Phase 2+)](#12-как-использовать-langgraph-phase-2)

---

## 1. Ежедневный workflow

### Утро (15 минут)
```bash
# 1. Открыть ADR-INDEX — проверить, нет ли новых ADR
# 2. Открыть ROLE-INDEX — проверить статусы ролей
# 3. Открыть FUTURE_ROLES_TECH_REGISTRY — что в работе?
# 4. Выбрать 1–2 задачи на день
```

### День (основная работа)
```bash
# 1. Сгенерировать TASK.md через specs/task_generator.py
# 2. Скопировать TASK.md + CONTEXT.json в VS Code
# 3. Kimi Code пишет код → RESULT.md
# 4. Reviewer Agent проверяет → approve / reject
# 5. DevOps Agent деплоит (с approval)
```

### Вечер (10 минут)
```bash
# 1. Проверить EventEngine — все ли события залогированы?
# 2. Проверить RULES.md — добавил ли Self-Improving Engine новые правила?
# 3. Закоммитить изменения
# 4. Обновить FUTURE_ROLES_TECH_REGISTRY (если статус изменился)
```

---

## 2. Как начать новую задачу

### Шаг 1: Определить тип задачи
| Тип | Workflow | Роли |
|-----|----------|------|
| Новая фича | `feature.yaml` | Architect → Developer → Reviewer → DevOps |
| Баг | `bugfix.yaml` | Developer → Reviewer → DevOps |
| Рефакторинг | `refactor.yaml` | Architect → Developer → Reviewer |
| Хотфикс | `hotfix.yaml` | Developer → DevOps (skip review) |

### Шаг 2: Сгенерировать TASK.md
```python
from voyage_framework.specs.task_generator import TaskGenerator
from voyage_framework.core.event_engine import EventEngine

engine = EventEngine()
engine.set_project_context("skilltracer")

generator = TaskGenerator(engine)
task = generator.generate(
    role="developer",
    task="Add FSM persistence to bot",
    micro_phase="M4-Bot-Engine",
    project_id="skilltracer"
)

# Записать в рабочую директорию
open("TASK.md", "w").write(task.task_markdown)
open("CONTEXT.json", "w").write(json.dumps(task.context_json))
```

### Шаг 3: Проверить TASK.md
- [ ] Acceptance criteria чёткие и измеримы?
- [ ] ADR references актуальны?
- [ ] Rules из RULES.md применимы?
- [ ] Relevant files существуют?

---

## 3. Как использовать Kimi Code

### Вариант А: Файловый протокол (рекомендуется)

```bash
# 1. Сгенерировать TASK.md + CONTEXT.json (см. раздел 2)
# 2. Открыть VS Code с Kimi Code
# 3. Скопировать содержимое TASK.md в чат Kimi Code
# 4. Kimi Code читает CONTEXT.json автоматически (если в workspace)
# 5. Kimi Code пишет код → RESULT.md
# 6. Framework читает RESULT.md и логирует implementation_done
```

### Вариант Б: MCP (Model Context Protocol)

```bash
# 1. Запустить MCP сервер фреймворка
python -m voyage_framework.integrations.kimi_mcp

# 2. Настроить VS Code settings.json:
# {
#   "kimi.mcpServers": {
#     "voyage": {
#       "command": "python",
#       "args": ["-m", "voyage_framework.integrations.kimi_mcp"]
#     }
#   }
# }

# 3. Kimi Code вызывает tools через MCP:
# - query_ast
# - semantic_search
# - get_context
```

### Что делать, если Kimi Code не понял задачу
1. Проверь TASK.md — достаточно ли контекста?
2. Добавь relevant_files в CONTEXT.json.
3. Упрости acceptance criteria (1 критерий = 1 действие).
4. Если не помогло — переключись на Kimi Agent (Web).

---

## 4. Как использовать Kimi Agent

### Когда использовать
- Kimi Code в VS Code недоступен (работаешь с телефона).
- Задача слишком сложная для одного промпта (нужен многошаговый workflow).
- Нужен Agent Runtime (plan → execute → reflect → retry).

### Как запустить
```bash
# 1. Запустить Agent Runtime
python -m voyage_framework.agents.runtime   --workflow workflows/definitions/feature.yaml   --project-id skilltracer   --task "Add FSM persistence to bot"

# 2. Агент выполняет workflow автоматически
# 3. Мониторить через Visualizer или логи
# 4. При approval_required — проверить .voyage_approval_pending.json
```

### Как мониторить
```bash
# SSE поток событий
curl http://localhost:8000/events/stream?project_id=skilltracer

# Или через Visualizer (Phase 3)
# Открыть http://localhost:3000 (Electron) или мобильное приложение
```

---

## 5. Как добавить новую роль

### Пример: добавляем QA Engineer (ROLE-001)

```bash
# Шаг 1: Создать файл роли
cat > voyage_framework/agents/roles/qa.py << 'EOF'
from .base import AgentRole

class QAEngineerRole(AgentRole):
    name = "qa"
    allowed_tools = ["pytest", "playwright_test", "coverage_report"]

    async def plan(self, task: str) -> list[str]:
        return ["Write tests", "Run tests", "Analyze coverage"]

    async def execute(self, step: str) -> ToolResult:
        # implementation
        pass
EOF

# Шаг 2: Зарегистрировать в RoleRegistry
cat >> voyage_framework/agents/roles/__init__.py << 'EOF'
from .qa import QAEngineerRole
RoleRegistry.register("qa", QAEngineerRole)
EOF

# Шаг 3: Добавить workflow
# Создать workflows/definitions/feature_with_qa.yaml

# Шаг 4: Написать тесты
# tests/test_agents_roles_qa.py

# Шаг 5: Обновить ROLE-INDEX.md
# Добавить строку в таблицу, статус = Implemented

# Шаг 6: Обновить FUTURE_ROLES_TECH_REGISTRY.md
# Статус ROLE-001 = Implemented
```

### Чек-лист
- [ ] Файл роли создан (`agents/roles/xxx.py`)
- [ ] Зарегистрирован в `RoleRegistry`
- [ ] Workflow-определения используют новую роль
- [ ] Тесты написаны (`test_agents_roles_xxx.py`)
- [ ] `security/policy.py` обновлён
- [ ] `ROLE-INDEX.md` обновлён
- [ ] `FUTURE_ROLES_TECH_REGISTRY.md` обновлён

---

## 6. Как добавить новый инструмент

### Пример: добавляем `bandit` (Security Engineer)

```bash
# Шаг 1: Создать адаптер
cat > voyage_framework/tools/adapters/bandit.py << 'EOF'
from ..registry import tool
from ..engine import ToolResult

@tool(name="bandit_scan")
async def bandit_scan(path: str = ".") -> ToolResult:
    import subprocess
    result = subprocess.run(
        ["bandit", "-r", path, "-f", "json"],
        capture_output=True, text=True
    )
    return ToolResult(
        success=result.returncode == 0,
        stdout=result.stdout,
        stderr=result.stderr
    )
EOF

# Шаг 2: Auto-discovery подхватит адаптер при следующем запуске
# (registry.discover() сканирует tools/adapters/)

# Шаг 3: Добавить в ROLE (если нужно)
# ROLE-002: добавить "bandit_scan" в allowed_tools

# Шаг 4: Написать тесты
# tests/test_tools_adapters_bandit.py

# Шаг 5: Обновить TECH-001 (если инструмент новый)
```

---

## 7. Как обновить ADR

### Создание нового ADR
```bash
# Шаг 1: Создать файл
cat > ADR/ADR-006-hmac-signing.md << 'EOF'
# ADR-006: HMAC-SHA256 Event Signing

**Status**: Proposed  
**Date**: 2026-05-26  
**Authors**: AndreyVoyage (Human) + AI Architect  
**Phase**: Phase 2  
**Replaces**: Нет  
**Related**: ADR-005

## Context
...
## Decision
...
## Consequences
...
EOF

# Шаг 2: Обновить ADR-INDEX.md
# Добавить строку в таблицу

# Шаг 3: Обсудить (если нужно)
# Если ADR критичный — запустить Reviewer Agent на анализ

# Шаг 4: Принять
# Статус = Accepted

# Шаг 5: Генерировать код
# TASK.md ссылается на ADR-006
```

### Обновление существующего ADR
- Меняешь только статус (Accepted → Deprecated → Superseded).
- НЕ переписываешь содержимое — создаёшь новый ADR с `Replaces`.

---

## 8. Как работать с Sandbox

### Запуск безопасной команды
```python
from voyage_framework.security.sandbox import SecureExecutor
from voyage_framework.security.factory import SandboxFactory

backend = SandboxFactory.get_backend("docker")
executor = SecureExecutor(
    SecurityPolicy(),
    project_root=Path("."),
    backend=backend
)

# Безопасная команда — выполняется
result = executor.execute(["pytest", "backend/tests"])
print(result.success)  # True
```

### Запуск опасной команды
```python
# Опасная команда — блокируется + требует approval
result = executor.execute(["systemctl", "restart", "skilltracer"])
print(result.blocked)  # True

# Human читает .voyage_approval_pending.json
# approve → AI retry
# reject → AI abort
```

### Чек-лист безопасности
- [ ] Команда в whitelist?
- [ ] Аргументы не выходят за project_root?
- [ ] Нет dangerous patterns (rm -rf, eval)?
- [ ] Если dangerous tier — approval получен?

---

## 9. Как работать с Approval Flow

### Для Human
```bash
# 1. Проверить pending approvals
cat .voyage_approval_pending.json

# Пример вывода:
# {
#   "requests": [
#     {
#       "id": "apr-001",
#       "command": ["systemctl", "restart", "skilltracer"],
#       "agent_id": "devops-001",
#       "timestamp": "2026-05-26T18:00:00Z",
#       "status": "pending"
#     }
#   ]
# }

# 2. Approve
python -c "
import json
with open('.voyage_approval_pending.json') as f:
    data = json.load(f)
data['requests'][0]['status'] = 'approved'
data['requests'][0]['approved_by'] = 'human'
with open('.voyage_approval_pending.json', 'w') as f:
    json.dump(data, f)
"

# 3. AI автоматически retry с approved request
```

### Для AI
```python
# AI видит APPROVAL_REQUIRED → создаёт ApprovalRequest → ждёт
# После approve → retry с той же командой
# После reject → abort, логирует agent_failed
# После 24h timeout → abort, логирует agent_failed
```

---

## 10. Как запустить тесты

### Все тесты
```bash
pytest tests/ -v --tb=short
```

### Тесты конкретного модуля
```bash
pytest tests/test_core_models.py -v
pytest tests/test_sandbox.py -v
```

### С coverage
```bash
pytest tests/ --cov=voyage_framework --cov-report=html
# Открыть htmlcov/index.html
```

### С mypy
```bash
mypy voyage_framework/ --strict
```

### С ruff
```bash
ruff check voyage_framework/
ruff format voyage_framework/
```

### Pre-commit (рекомендуется)
```bash
# Установить hooks
pre-commit install

# Запустить вручную
pre-commit run --all-files
```

---

## 11. Как деплоить

### Через DevOps Agent
```bash
# 1. DevOps Agent проверяет pre-deploy
python -m voyage_framework.agents.roles.devops   --action pre_deploy   --project-id skilltracer

# 2. Human approves deploy
# (см. раздел 9)

# 3. DevOps Agent деплоит
python -m voyage_framework.agents.roles.devops   --action deploy   --project-id skilltracer   --target vps://212.109.198.69

# 4. Health-check
python -m voyage_framework.agents.roles.devops   --action health_check   --project-id skilltracer
```

### Ручной деплой (fallback)
```bash
# 1. Git push
git push origin main

# 2. SSH на сервер
ssh root@212.109.198.69

# 3. Pull
cd /opt/skilltracer && git pull

# 4. Restart
systemctl restart skilltracer

# 5. Health-check
curl http://localhost:8000/health
```

---

## 12. Как использовать LangGraph (Phase 2+)

### Когда включать LangGraph
- Phase 1: Custom Runtime (plan → execute → reflect → retry).
- Phase 2+: LangGraph через OrchestratorAdapter ТОЛЬКО если:
  - Нужен human-in-the-loop на уровне workflow (не только dangerous tier).
  - Нужна time-travel debugging.
  - Нужна sub-graph composition (граф внутри графа).
  - Custom Runtime не справляется с branching complexity.

### Как переключиться
```python
# Phase 1: Custom Runtime
from voyage_framework.agents.runtime import CustomAgentRuntime
runtime = CustomAgentRuntime()

# Phase 2+: LangGraph через адаптер
from voyage_framework.integrations.orchestrator_adapter import LangGraphAdapter
runtime = LangGraphAdapter(
    config={
        "checkpoint_store": "postgresql",  # или "sqlite"
        "human_in_the_loop": True,
        "observability": "langsmith"  # опционально
    }
)

# Всё остальное (models, tools, memory) остаётся без изменений!
```

### Как написать workflow для LangGraph
```python
# workflows/definitions/feature_langgraph.py
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    task: str
    plan: list[str]
    code: str
    tests_passed: bool
    confidence: float

def architect_plan(state: AgentState):
    # ...
    return {"plan": [...]}

def developer_implement(state: AgentState):
    # ...
    return {"code": "..."}

def reviewer_check(state: AgentState):
    # ...
    return {"tests_passed": True, "confidence": 0.9}

def route_after_review(state: AgentState) -> str:
    if state["tests_passed"] and state["confidence"] > 0.7:
        return "devops_deploy"
    return "developer_fix"

workflow = StateGraph(AgentState)
workflow.add_node("architect_plan", architect_plan)
workflow.add_node("developer_implement", developer_implement)
workflow.add_node("reviewer_check", reviewer_check)
workflow.add_node("devops_deploy", devops_deploy)

workflow.add_edge("architect_plan", "developer_implement")
workflow.add_edge("developer_implement", "reviewer_check")
workflow.add_conditional_edges("reviewer_check", route_after_review)

app = workflow.compile()
```

### Чек-лист LangGraph integration
- [ ] OrchestratorAdapter stub реализован (`integrations/orchestrator_adapter.py`).
- [ ] LangGraph установлен: `pip install langgraph`.
- [ ] State mapping работает (наш AgentState → LangGraph TypedDict).
- [ ] Checkpoint persistence настроен (PostgreSQL/SQLite).
- [ ] Human-in-the-loop интегрирован с нашим Approval Flow.
- [ ] Все события LangGraph логируются в EventEngine.
- [ ] Fallback на Custom Runtime работает (отключить LangGraph = 1 строка).

### Важно: не делай этого
- ❌ Не используй LangGraph напрямую в `agents/runtime.py` — только через адаптер.
- ❌ Не заменяй EventEngine на LangSmith — LangSmith дополнительно, EventEngine primary.
- ❌ Не используй LangChain models вместо наших — мы model-agnostic.
- ❌ Не переписывай tools/memory под LangGraph — они независимы.

---

## Приложение A: Quick Reference

### Команды
```bash
# Запуск тестов
pytest tests/ -v

# Запуск линтеров
mypy voyage_framework/ && ruff check voyage_framework/

# Генерация TASK.md
python -m voyage_framework.specs.task_generator --task "..." --project-id skilltracer

# Запуск Agent Runtime
python -m voyage_framework.agents.runtime --workflow feature.yaml --project-id skilltracer

# Проверка pending approvals
cat .voyage_approval_pending.json

# Деплой (ручной)
ssh root@212.109.198.69 && cd /opt/skilltracer && git pull && systemctl restart skilltracer
```

### Файлы
| Файл | Назначение | Когда обновлять |
|------|-----------|-----------------|
| ADR-INDEX.md | Реестр ADR | При новом ADR |
| ROLE-INDEX.md | Реестр ролей | При новой роли |
| FUTURE_ROLES_TECH_REGISTRY.md | Дорожная карта | Еженедельно |
| RULES.md | Правила кода | При новой ошибке / правиле |
| INSTRUCTIONS.md | Этот файл | При смене Phase |
| TASK.md | Задача для Kimi Code | Per-task |
| CONTEXT.json | Контекст для Kimi Code | Per-task |

---

**Версия:** 1.0  
**Дата:** 2026-05-26  
**Авторы:** AndreyVoyage (Human) + AI Architect  
**Следующее обновление:** После перехода к Phase 2
