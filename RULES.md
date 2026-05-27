# RULES.md — Живой документ правил Voyage Framework v4.0

> **Назначение:** Все правила, которые AI-агенты обязаны соблюдать при генерации кода.  
> **Обновляется:** Self-Improving Engine автоматически + Human вручную.  
> **Приоритет:** Если правило в RULES.md конфликтует с мастер-документом — RULES.md выигрывает (он живой, мастер-документ статичен).  
> **Project-scoped:** Каждый проект имеет свой RULES.md (fallback на глобальный).

---

## Как пользоваться этим файлом

### Для Human (AndreyVoyage)
1. **Перед code review** проверь, что код соответствует RULES.md.
2. **При новой ошибке** — добавь правило вручную или дождись Self-Improving Engine.
3. **Не удаляй** правила без замены — помечай как `[DEPRECATED: replaced by rule N]`.

### Для Kimi Code / Kimi Agent
1. **Перед написанием кода** прочитай RULES.md полностью.
2. **После написания кода** проверь: нарушено ли какое-либо правило?
3. **Если правило невозможно соблюсти** — логируй `rule_violation` event с обоснованием.

---

## Раздел 1: Код (Python)

### 1.1 Type Hints (MUST)
- Все `async def` функции обязаны иметь return type annotation.
- Все `def` функции длиной >3 строк обязаны иметь type hints для аргументов и возврата.
- Исключение: `@pytest.fixture`, `@app.get()` декораторы FastAPI (type hints опциональны).
- Пример: `async def handle_message(update: Update, context: Context) -> None:`

### 1.2 Async SQLAlchemy (MUST)
- В async коде использовать ТОЛЬКО `async_sessionmaker` и `AsyncSession`.
- Запрещено: `Session` (sync), `create_engine` без `create_async_engine`.
- Пример: `async with async_session() as session:`

### 1.3 No eval/exec (MUST)
- Запрещено использовать `eval()`, `exec()`, `compile()` с user input.
- Исключение: `ast.literal_eval()` для безопасного парсинга.
- Self-Improving Engine автоматически добавляет это правило при первом `eval` в коде.

### 1.4 Secrets Management (MUST)
- Все секреты через `pydantic-settings` + `.env` файл.
- Запрещено: hardcoded `BOT_TOKEN`, `SECRET_KEY`, пароли в коде.
- Self-Improving Engine + gitleaks сканируют репозиторий на нарушения.

### 1.5 Error Handling (MUST)
- Все async функции, вызывающие внешние API, обязаны иметь `try/except`.
- Логирование через `structlog` или `logging` с `correlation_id`.
- Не использовать голый `except:` — всегда указывай тип исключения.

### 1.6 Testing (MUST)
- Каждая новая функция длиной >10 строк обязана иметь ≥1 тест.
- Coverage не должен падать ниже 80%.
- Использовать `pytest-asyncio` для async тестов.

---

## Раздел 2: Архитектура

### 2.1 Event Sourcing (MUST)
- Все изменения состояния логируются как `Event` через `EventEngine.append()`.
- Запрещено: прямое обновление БД в обход EventEngine.
- `project_id` обязателен во всех событиях (ADR-005).

### 2.2 Project Isolation (MUST)
- Все запросы к БД фильтруются по `project_id`.
- Semantic Memory использует namespace: `f"voyage_{project_id}"`.
- Cross-project query запрещён на уровне runtime.

### 2.3 Spec-Driven (MUST)
- Сначала спецификация (TASK.md), потом код.
- Acceptance criteria определяются ДО реализации.
- Код не мержится без прохождения всех AC.

### 2.4 Fallback (MUST)
- Все внешние зависимости (PostgreSQL, ChromaDB) обязаны иметь fallback.
- PostgreSQL → SQLite WAL.
- ChromaDB → SQLite keyword search.
- Framework не должен падать при недоступности внешнего сервиса.

---

## Раздел 3: Безопасность

### 3.1 Sandbox (MUST)
- Все команды через `SecureExecutor`.
- Dangerous tier (systemctl, ssh, curl) требует human approval.
- Логирование ВСЕХ попыток выполнения в audit trail.

### 3.2 RAG Guard (MUST)
- Pre-embedding фильтр: `ignore previous instructions`, `bypass sandbox` → SANITIZED.
- Retrieved context оборачивается в `<<retrieved_context>>` с пометкой "not executable".

### 3.3 PyPI Validation (MUST)
- `pip install` только после проверки имени пакета через PyPI JSON API.
- Защита от typosquatting: `reqeusts` → BLOCKED.

### 3.4 CORS / CSP (SHOULD)
- CORS: не разрешать `*` в production.
- CSP: `script-src 'self'` для React SPA.
- Security headers: HSTS, X-Frame-Options, X-Content-Type-Options.

---

## Раздел 4: LangGraph Integration (Phase 2+)

### 4.1 Orchestrator Adapter (MUST)
- LangGraph используется ТОЛЬКО через `OrchestratorAdapter` (ADR-005).
- Прямое использование `langgraph.graph.StateGraph` в `agents/runtime.py` запрещено.
- Это позволяет переключаться между Custom Runtime и LangGraph без переписывания ядра.

### 4.2 State Mapping (MUST)
- `AgentState` (наш) → `TypedDict` (LangGraph) через адаптер.
- `Checkpoint` (наш) → `MemorySaver` / `PostgresSaver` (LangGraph) через адаптер.
- Все transitions через conditional edges должны быть deterministic (для audit).

### 4.3 Human-in-the-Loop (MUST)
- `interrupt()` LangGraph используется ТОЛЬКО для dangerous tier approval.
- Human approval flow остаётся в нашем `security/approval.py` — LangGraph вызывает его через адаптер.
- Не использовать LangGraph `interrupt()` для произвольных пауз — это нарушает наш Approval Flow.

### 4.4 Observability (MUST)
- LangSmith tracing опционально (Phase 2+), но НЕ заменяет наш EventEngine.
- Все события LangGraph (node start, node end, error) логируются в наш EventEngine.
- `LangSmith` — дополнительная observability, `EventEngine` — единый источник правды.

### 4.5 No LangChain Lock-in (MUST)
- LangGraph используется как orchestrator, НЕ как замена наших моделей/тулов.
- `core/models.py`, `tools/engine.py`, `memory/semantic.py` остаются независимыми.
- Если LangGraph устареет — заменяем только `OrchestratorAdapter`, не всю систему.

---

## Раздел 5: Процесс разработки

### 5.1 Git Workflow (MUST)
- Коммиты через `git commit -m "type: description"`.
- Типы: `feat`, `fix`, `docs`, `test`, `refactor`, `security`, `adr`.
- Пример: `git commit -m "feat: add project_id to Event model"`

### 5.2 Code Review (MUST)
- Перед merge: mypy clean, ruff clean, pytest all passed.
- Reviewer Agent блокирует merge при нарушении RULES.md.

### 5.3 Documentation (MUST)
- Каждый новый модуль обязан иметь docstring в начале файла.
- Каждая публичная функция обязана иметь docstring.
- ADR обновляется при изменении архитектурного решения.

---

## Раздел 6: Self-Improving Engine

### 6.1 Auto-Rule Generation (MUST)
- При `error_logged` event → Self-Improving Engine анализирует pattern.
- Если pattern повторяется >3 раза → генерируется rule suggestion.
- Rule добавляется в project-scoped RULES.md (fallback global).

### 6.2 Rule Deduplication (MUST)
- Перед добавлением проверять hash rule_text.
- Если rule уже существует → skip.

### 6.3 Rule Severity (MUST)
- `must` — обязательно, блокирует merge.
- `should` — рекомендуется, warning.
- `may` — опционально, info.

---

## Приложение: История изменений

| Дата | Правило | Добавлено | Причина |
|------|---------|-----------|---------|
| 2026-05-21 | 1.1 Type Hints | Human | Первоначальное правило |
| 2026-05-21 | 1.2 Async SQLAlchemy | Human | Первоначальное правило |
| 2026-05-21 | 1.3 No eval/exec | Human | Первоначальное правило |
| 2026-05-26 | 4.1–4.5 LangGraph | Human | ADR-005: LangGraph через адаптер |
| 2026-05-26 | 3.4 CORS/CSP | Human | ROLE-002: Security Engineer |

---

**Версия:** 1.1  
**Дата:** 2026-05-26  
**Следующее обновление:** При добавлении новой роли или инструмента
