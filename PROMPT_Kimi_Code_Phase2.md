# PROMPT: Continue Voyage Framework v4.0 — Phase 2 Implementation

> **Роль:** AI-разработчик (Developer Agent)  
> **Задача:** Реализовать Phase 2 компоненты Voyage Framework  
> **Контекст:** Phase 1 MVP завершён (EventEngine, Sandbox, TaskGenerator, AgentRuntime)  
> **Целевой проект:** SkillTracer (Python backend + React frontend)  
> **Framework:** https://github.com/AndreyVoyage/Framework-voyage-v2  

---

## Что уже реализовано (Phase 1 — Foundation)

| Компонент | Файл | Статус |
|-----------|------|--------|
| Event Engine | `core/event_engine.py` | ✅ SQLite + JSONL, append-only |
| Models | `core/models.py` | ✅ Pydantic: Event, AgentState, ToolResult, SecurityPolicy |
| Storage | `core/storage.py` | ✅ Atomic writes, frontmatter, journal rotation |
| Sandbox | `security/sandbox.py` | ✅ 5 уровней, SubprocessBackend |
| Policies | `security/policy.py` | ✅ 6 ролей |
| Audit | `security/audit.py` | ✅ JSONL trail |
| Approval | `security/approval.py` | ✅ .voyage_approval_pending.json |
| Task Generator | `specs/task_generator.py` | ✅ TASK.md + CONTEXT.json |
| Tracker | `specs/tracker.py` | ✅ Acceptance criteria |
| Agent Runtime | `agents/runtime.py` | ✅ Plan→Execute→Reflect→Retry |
| CLI | `cli.py` | ✅ voyage init/run/task/status/events/approve |

---

## Что нужно реализовать (Phase 2 — Soul & Connectors)

### Приоритет 1: Memory Layer

#### 1.1 Context Tiers (`memory/context_tiers.py`)
```python
# Реализовать 4-tier memory model:
# - Short-term: текущая сессия (correlation_id), 10 событий, никогда не обрезается
# - Working: текущая микро-фаза, 20 событий
# - Long-term: ADR + архитектура, 50 событий за 90 дней
# - Semantic: похожие решения, векторный поиск

class ContextCompiler:
    def compile(self, project_id: str, correlation_id: str, max_tokens: int = 12000) -> CompiledContext

class CompiledContext(BaseModel):
    short_term: list[Event]
    working: list[Event] 
    long_term: list[Event]
    semantic: list[dict]  # retrieved from ChromaDB
    total_tokens: int
    truncated: bool  # были ли обрезаны lower tiers
```

**AC:**
- [ ] tiktoken считает токены каждого tier
- [ ] Если total > max_tokens — обрезает снизу (semantic → long-term → working)
- [ ] Short-term никогда не обрезается
- [ ] Все операции фильтруются по project_id (ADR-005)
- [ ] Тесты: test_context_tiers.py (покрытие >80%)

#### 1.2 Semantic Memory (`memory/semantic.py`)
```python
# VectorStore abstraction → ChromaDB (namespace per project)

class SemanticMemory:
    def __init__(self, project_id: str, db_path: str = ".voyage/chroma")
    def add(self, text: str, metadata: dict) -> str  # return id
    def query(self, query_text: str, n_results: int = 5) -> list[dict]
    def delete(self, ids: list[str]) -> None

# Fallback: SQLite keyword search если ChromaDB недоступен
class SQLiteSemanticFallback:
    def query(self, query_text: str) -> list[dict]
```

**AC:**
- [ ] Namespace per project: `f"voyage_{project_id}"`
- [ ] RAG Guard: pre-embedding фильтр инъекций
- [ ] Retrieved context оборачивается в `<<retrieved_context>>`
- [ ] Fallback на SQLite keyword search
- [ ] Тесты: test_semantic.py

#### 1.3 AST Manager (`memory/ast_manager.py`)
```python
# tree-sitter для Python + TypeScript
# Impact analysis: "что сломается при изменении файла X?"

class ASTManager:
    def __init__(self, project_root: Path)
    def index_file(self, path: Path) -> ASTNode
    def get_dependencies(self, file_path: str) -> list[str]  # кто зависит от файла
    def get_impact(self, changed_file: str) -> ImpactReport

class ImpactReport(BaseModel):
    changed_file: str
    directly_affected: list[str]  # файлы, которые импортируют changed_file
    transitively_affected: list[str]  # через цепочку зависимостей
    tests_to_run: list[str]  # тесты, связанные с изменёнными файлами
    risk_score: float  # 0.0-1.0
```

**AC:**
- [ ] Поддержка Python (.py) и TypeScript (.ts, .tsx)
- [ ] Incremental re-indexing (по mtime, не полный rebuild)
- [ ] Impact analysis за <2 секунд для проекта с 50 файлами
- [ ] Тесты: test_ast_manager.py

---

### Приоритет 2: Tool Engine & Adapters

#### 2.1 Tool Engine (`tools/engine.py`)
```python
# Единая точка входа для всех инструментов
# @tool decorator + auto-discovery

class ToolEngine:
    def __init__(self, executor: SecureExecutor)
    def register(self, tool: Tool) -> None
    def execute(self, tool_name: str, args: dict) -> ToolResult
    def list_tools(self) -> list[ToolInfo]

@dataclass
class Tool:
    name: str
    description: str
    handler: Callable[..., ToolResult]
    required_args: list[str]
    approval_required: bool = False
```

**AC:**
- [ ] @tool decorator для регистрации
- [ ] Auto-discovery через registry.discover()
- [ ] Все tool calls логируются в EventEngine
- [ ] Тесты: test_tools_engine.py

#### 2.2 Tool Adapters (`tools/adapters/`)
```python
# git.py — git_diff, git_log, git_status
# python.py — pytest, mypy, ruff_check, pip_install (+ PyPI validation)
# system.py — df_h, ps_aux
# files.py — cat_file, grep_code, find_files
# deploy.py — systemctl_restart, ssh_command, curl_url (approval required!)
```

**AC:**
- [ ] PyPI Validation: проверка имени пакета перед pip install
- [ ] Все адаптеры проходят через SecureExecutor
- [ ] Deploy adapter требует approval (dangerous tier)
- [ ] Тесты: test_tools_adapters.py

---

### Приоритет 3: Workflow Engine

#### 3.1 Workflow Engine (`workflows/engine.py`)
```python
# YAML-driven workflow execution
# feature.yaml, bugfix.yaml, refactor.yaml, hotfix.yaml

class WorkflowEngine:
    def __init__(self, runtime: AgentRuntime)
    def load(self, workflow_path: Path) -> Workflow
    def execute(self, workflow: Workflow, context: dict) -> WorkflowResult
    def get_state(self) -> WorkflowState

class Workflow(BaseModel):
    name: str
    nodes: list[Node]
    transitions: list[Transition]
```

**AC:**
- [ ] Чтение workflow из YAML
- [ ] Deterministic transitions (для audit)
- [ ] Checkpoint после каждого node
- [ ] Resume from checkpoint
- [ ] Тесты: test_workflows_engine.py

#### 3.2 Workflow Definitions
```yaml
# workflows/definitions/feature.yaml
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

**AC:**
- [ ] 4 workflow definition файла
- [ ] Все transitions логируются в EventEngine
- [ ] Human can pause/resume workflow

---

### Приоритет 4: Integrations

#### 4.1 Kimi Code Integration (`integrations/kimi_code.py`)
```python
# Файловый протокол + MCP adapter

class KimiCodeIntegration:
    def write_task(self, task: TaskSpec) -> tuple[Path, Path]  # TASK.md, CONTEXT.json
    def read_result(self) -> Result  # читает RESULT.md
    def notify_complete(self, result: Result) -> None  # логирует implementation_done
```

**AC:**
- [ ] TASK.md + CONTEXT.json в рабочую директорию
- [ ] RESULT.md читается после выполнения
- [ ] Все действия логируются в EventEngine

#### 4.2 Orchestrator Adapter (`integrations/orchestrator_adapter.py`)
```python
# Stub для LangGraph (Phase 2+), сейчас — pass-through

class OrchestratorAdapter(ABC):
    @abstractmethod
    def run(self, workflow: Workflow, state: AgentState) -> NodeResult

class CustomRuntimeAdapter(OrchestratorAdapter):
    # Phase 1: наш собственный runtime
    def run(self, workflow, state): ...

class LangGraphAdapter(OrchestratorAdapter):
    # Phase 2+: LangGraph через adapter
    def run(self, workflow, state): ...
```

**AC:**
- [ ] ABC для всех адаптеров
- [ ] CustomRuntimeAdapter — полная реализация
- [ ] LangGraphAdapter — stub (pass-through)
- [ ] Переключение без изменения agents/runtime.py

---

## Правила разработки (из RULES.md v1.2)

1. **ADR > [ARCH] > [OPS] > [STYLE]** — при конфликте применяй иерархию
2. **Все async def — type hints** (MUST)
3. **Async SQLAlchemy only** (MUST)
4. **No eval/exec** с user input (MUST)
5. **Secrets через pydantic-settings** (MUST)
6. **Каждая функция >10 строк — ≥1 тест** (MUST)
7. **Event Sourcing** — все изменения через EventEngine.append() (MUST)
8. **Project Isolation** — project_id во всех операциях (MUST)
9. **Spec-Driven** — сначала TASK.md, потом код (MUST)
10. **Fallback** — все внешние зависимости имеют fallback (MUST)

---

## Acceptance Criteria для Phase 2

- [ ] Context Tiers работает с tiktoken budgeting
- [ ] Semantic Memory с ChromaDB + SQLite fallback
- [ ] AST Manager для Python + TypeScript
- [ ] Tool Engine с @tool decorator
- [ ] 5 Tool Adapters (git, python, system, files, deploy)
- [ ] Workflow Engine с YAML definitions
- [ ] 4 workflow definitions (feature, bugfix, refactor, hotfix)
- [ ] Kimi Code Integration (файловый протокол)
- [ ] Orchestrator Adapter (CustomRuntime + LangGraph stub)
- [ ] Все компоненты логируются в EventEngine
- [ ] Все компоненты фильтруются по project_id
- [ ] mypy clean
- [ ] ruff clean
- [ ] pytest all passed (coverage >80%)

---

## Как проверить результат

```bash
# 1. Установка
pip install -e ".[dev,semantic,ast]"

# 2. Тесты
pytest tests/ -v --cov=voyage_framework --cov-report=term

# 3. Линтеры
mypy voyage_framework/
ruff check voyage_framework/

# 4. CLI
voyage status
voyage run developer --task "Test Phase 2" --plan "echo Phase 2 ready"
```

---

**Generated by:** Voyage Framework v4.0  
**Phase:** 2 — Soul & Connectors  
**Date:** 2026-05-28
