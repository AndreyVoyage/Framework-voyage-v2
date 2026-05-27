# Framework-voyage-v2

> **AI-Native Engineering Operating System** для solo-разработчика.  
> AI пишет ТЗ для AI. Framework генерирует `TASK.md` + `CONTEXT.json`, которые Kimi Code в VS Code читает и выполняет.

[![Status](https://img.shields.io/badge/Phase-1%20Foundation-yellow)](./PHASE_STATUS.md)
[![Python](https://img.shields.io/badge/Python-3.11%2B-blue)](./pyproject.toml)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)

---

## 🚀 Что это?

**Voyage Framework v4.0** — фреймворк для разработки с AI-ассистентами, который:

- 📋 **Планирует** задачи через микро-фазы с acceptance criteria
- 🧠 **Помнит** всё: решения, ошибки, контекст, структуру кода
- 🛡 **Безопасно выполняет** команды через sandbox с human approval
- 📝 **Генерирует ТЗ** для Kimi Code / Kimi Agent
- 🔄 **Самоулучшается**: каждая ошибка → новое правило
- 📊 **Визуализирует** работу через отдельное приложение

**Целевой проект:** [SkillTracer](https://github.com/AndreyVoyage/skilltracer) — платформа трекинга жизни с Telegram-ботом и React SPA.

---

## 📁 Структура репозитория

```
Framework-voyage-v2/
├── ADR/                          # Architecture Decision Records
│   ├── ADR-INDEX.md              # Реестр всех ADR
│   ├── ADR-001-postgresql-events.md
│   ├── ADR-002-chromadb-semantic.md
│   ├── ADR-003-tree-sitter-ast.md
│   ├── ADR-004-dual-location.md
│   ├── ADR-005-project-isolation.md
│   ├── ADR-006-rule-governance.md
│   ├── ADR-007-runtime-orchestration.md
│   ├── ADR-008-complexity-budget.md
│   └── ADR-009-complexity-budget-v2.md
│
├── ROLE/                         # Ролевые спецификации
│   ├── ROLE-INDEX.md             # Реестр ролей
│   ├── ROLE-000-base-role.md     # Базовая роль (ABC)
│   ├── ROLE-001-architect.md
│   ├── ROLE-002-security-engineer.md
│   └── ROLE-003-reviewer-engineer.md
│
├── TECH/                         # Технологические реестры
│   └── TECH-001-appsec-stack.md  # AppSec инструменты
│
├── FUTURE_ROLES_TECH_REGISTRY.md # Реестр будущих ролей и технологий
│
├── RULES.md                      # Живой документ правил (v1.2)
│   # Приоритет: ADR > [ARCH] > [OPS] > [STYLE]
│
├── INSTRUCTIONS.md               # Инструкции по использованию
│
├── VOYAGE_WORKFLOW.md            # Workflow: feature / bugfix / refactor / hotfix
│
├── TEST_STRATEGY.md              # Стратегия тестирования (пирамида, CI gates)
│
├── PROMPT_ANALYZE_FRAMEWORK.md   # System Prompt для Kimi Agent
│
├── LANGGRAPH_INTEGRATION_ANALYSIS.md  # Анализ интеграции LangGraph
│
└── VOYAGE_FRAMEWORK_MASTER_DOCUMENT_v4.0b.md  # ← МАСТЕР-ДОКУМЕНТ
```

---

## 🏗 Архитектура

### Компоненты (Phase 1 — Foundation)

| Компонент | Статус | Документ |
|-----------|--------|----------|
| **Event Store** | 🟡 Planned | [ADR-001](./ADR/ADR-001-postgresql-events.md) |
| **Semantic Memory** | 🟡 Planned | [ADR-002](./ADR/ADR-002-chromadb-semantic.md) |
| **AST Manager** | 🟡 Planned | [ADR-003](./ADR/ADR-003-tree-sitter-ast.md) |
| **Dual-Location** | 🟡 Planned | [ADR-004](./ADR/ADR-004-dual-location.md) |
| **Project Isolation** | 🟡 Planned | [ADR-005](./ADR/ADR-005-project-isolation.md) |
| **Rule Governance** | ✅ Accepted | [ADR-006](./ADR/ADR-006-rule-governance.md) |
| **Runtime Orchestration** | 🟡 Proposed | [ADR-007](./ADR/ADR-007-runtime-orchestration.md) |
| **Complexity Budget** | ✅ Accepted | [ADR-008](./ADR/ADR-008-complexity-budget.md), [ADR-009](./ADR/ADR-009-complexity-budget-v2.md) |
| **Security Sandbox** | 🟡 Planned | [ROLE-002](./ROLE/ROLE-002-security-engineer.md) |
| **Reviewer Engineer** | 🟡 Planned | [ROLE-003](./ROLE/ROLE-003-reviewer-engineer.md) |
| **AppSec Stack** | 🟡 Planned | [TECH-001](./TECH/TECH-001-appsec-stack.md) |

### Роли

| Роль | Статус | Описание |
|------|--------|----------|
| **Base Role** | ✅ Implemented | [ROLE-000](./ROLE/ROLE-000-base-role.md) — ABC для всех ролей |
| **Architect** | 🟡 Planned | [ROLE-001](./ROLE/ROLE-001-architect.md) — Spec-driven, ADR-aware |
| **Security Engineer** | 🟡 Planned | [ROLE-002](./ROLE/ROLE-002-security-engineer.md) — AppSec, SecOps |
| **Reviewer Engineer** | 🟡 Planned | [ROLE-003](./ROLE/ROLE-003-reviewer-engineer.md) — Governance layer |

---

## 🛠 Runnable MVP (Python)

**Код фреймворка** находится в отдельном репозитории:
👉 **[voyage-framework](https://github.com/AndreyVoyage/voyage-framework)** (скоро)

### Быстрый старт

```bash
# 1. Клонировать код
pip install -e .

# 2. Инициализировать проект
voyage init

# 3. Сгенерировать задачу для Kimi Code
voyage task developer --task "Implement auth" --phase M1

# 4. Запустить агента
voyage run developer --task "Test echo" --plan "echo hello"

# 5. Проверить статус
voyage status
```

### Что уже работает в MVP

| Компонент | Статус |
|-----------|--------|
| Event Engine (SQLite + JSONL) | ✅ |
| Security Sandbox (5 уровней) | ✅ |
| Task Generator (TASK.md + CONTEXT.json) | ✅ |
| Agent Runtime (Plan→Execute→Reflect→Retry) | ✅ |
| Role Policies (6 ролей) | ✅ |
| Audit Log | ✅ |
| Approval Flow | ✅ |
| CLI (voyage init/run/task/status/events/approve) | ✅ |
| Tests (50+ unit + integration) | ✅ |

---

## 📖 Как пользоваться

### Для Human (AndreyVoyage)

1. **Читайте [VOYAGE_WORKFLOW.md](./VOYAGE_WORKFLOW.md)** — основной workflow
2. **Проверяйте [RULES.md](./RULES.md)** перед code review
3. **Следите за [ADR-INDEX.md](./ADR/ADR-INDEX.md)** — реестр архитектурных решений
4. **Используйте [PROMPT_ANALYZE_FRAMEWORK.md](./PROMPT_ANALYZE_FRAMEWORK.md)** для запуска Kimi Agent

### Для Kimi Code / Kimi Agent

1. **Читайте [RULES.md](./RULES.md)** полностью перед кодом
2. **Следуйте [VOYAGE_WORKFLOW.md](./VOYAGE_WORKFLOW.md)** — пошагово
3. **Проверяйте [ADR](./ADR/)** при архитектурных изменениях
4. **Генерируйте TASK.md** через TaskGenerator перед каждой задачей

---

## 🧪 Тестирование

Стратегия описана в [TEST_STRATEGY.md](./TEST_STRATEGY.md):

- **Пирамида тестов:** Unit → Integration → E2E
- **CI Gates:** mypy → ruff → pytest → coverage
- **Mutation Testing:** Phase 2
- **Replay Testing:** Phase 2
- **Chaos Testing:** Phase 2

---

## 📊 Фазы разработки

| Фаза | Статус | Цель |
|------|--------|------|
| **Phase 1: Foundation** | 🟡 In Progress | Всё для мышления и безопасности |
| **Phase 2: Soul & Connectors** | ⚪ Not Started | Всё для действия и общения |
| **Phase 3: Eyes (Visualizer)** | ⚪ Not Started | Интерактивное учебное пособие |

---

## 🔗 Связанные репозитории

| Репозиторий | Описание |
|-------------|----------|
| [SkillTracer](https://github.com/AndreyVoyage/skilltracer) | Целевой проект (Python + React) |
| voyage-framework (скоро) | Runnable MVP на Python |
| voyage-visualizer (скоро) | Desktop + Mobile приложение |

---

## 📝 Лицензия

MIT License — AndreyVoyage

---

**Версия документации:** 2.0  
**Дата:** 2026-05-28  
**Статус:** Phase 1 — Foundation (в разработке)
