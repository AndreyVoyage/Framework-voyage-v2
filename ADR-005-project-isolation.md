# ADR-005: Project Isolation & Multi-Project Core Architecture9

**Status**: Accepted  
**Date**: 2026-05-25  
**Authors**: AndreyVoyage (Human) + AI Architect  
**Phase**: Phase 1 (Foundation) — заложка фундамента  
**Replaces**: Нет (новое решение)  
**Related**: ADR-001 (PostgreSQL Events), ADR-004 (Dual-Location)

---

## Context

Voyage Framework v4.0 создан внутри SkillTracer как `tools/voyage_framework/`. 
Для перехода от `single-project tool` к `multi-project AI OS` необходимо заложить 
изоляцию на уровне данных, памяти и правил. Решение должно:

- Не тормозить текущие 10 AC Phase 1;
- Работать на SQLite (dev) и PostgreSQL (prod) без дублирования логики;
- Сохранять принцип `Fallback` (ядро не зависит от внешних orchestrator);
- Ограничить security-scope Phase 1 deterministic фильтрами, без криптографии.

---

## Decision

### 1. Project Isolation: Composite Key
- `project_id: str` — обязательное поле в `Event`, `AgentState`, `JournalEntry`, `ProjectContext`.
- Уникальность на уровне хранилища: `(project_id, event_id)`. `event_id` (ULID) остаётся глобально уникальным.
- `causation_id` и `correlation_id` работают внутри одного `project_id` и не меняются.

### 2. Event Engine: Application-Level Filtering
- `EventEngine` фильтрует все операции чтения/записи по `project_id` на уровне приложения (SQLAlchemy query filter).
- PostgreSQL RLS считается **optional enhancement** для production, не требованием Phase 1.
- SQLite fallback остаётся полностью функциональным для dev и single-project режима.

### 3. Semantic Memory: Namespace per Project
- ChromaDB: коллекция именуется `f"voyage_{project_id}"`.
- Cross-project semantic query запрещена на уровне runtime. Админский доступ — через отдельный API вне агентского цикла.

### 4. Self-Improving Engine: Scoped RULES.md
- Каждый проект имеет свой `RULES.md` (например, `projects/skilltracer/RULES.md`).
- При отсутствии project-specific файла — fallback на глобальный `RULES.md`.
- `error_logged` → `rule_added` chain работает внутри одного `project_id`.

### 5. LangGraph: Not in Phase 1
- Phase 1 использует **собственный Agent Runtime** (`plan → execute → reflect → retry`).
- LangGraph (или любой другой orchestrator) рассматривается как **optional integration** Phase 2.
- В Phase 1 создаётся абстрактный интерфейс `OrchestratorAdapter` в `integrations/orchestrator_adapter.py` (stub). 
  Это позволит в Phase 2 подключить LangGraph без изменения `agents/runtime.py`.

### 6. Sandbox: Strategy Pattern
- Создаётся абстрактный класс `SandboxBackend` в `security/sandbox.py`.
- Phase 1 реализует только `DockerSandbox` (универсальный dev-режим).
- Phase 2 добавит `SeccompSandbox` (Linux production) и `WindowsJobSandbox` (Windows fallback) 
  как отдельные файлы без переписывания ядра.

### 7. Security Phase 1: Deterministic Filters Only
В Phase 1 реализуются только детерминированные (rule-based) защиты:

- **RAG Guard**: pre-embedding regex-фильтр в Semantic Memory 
  (`ignore previous instructions`, `bypass sandbox` и т.д.).
- **Context Wrapping**: retrieved context оборачивается в `<<retrieved_context>>` 
  с явной инструкцией «not executable».
- **PyPI Validation**: `python.py` adapter проверяет имя пакета через PyPI JSON API 
  перед `pip install` (защита от typosquatting).
- **Path Traversal + Whitelist**: существующие L1-L5 Sandbox (уже в Master-документе).

### 8. Security Phase 2 (отложено): Cryptography & ML
Следующие меры переносятся в Phase 2 / v4.1:
- HMAC-SHA256 event signing;
- Hash chain для Audit Trail;
- PostgreSQL RLS как primary изоляция;
- ML-based anomaly detection.

---

## Consequences

### Positive
- Мультипроектность работает уже на SQLite без установки PostgreSQL.
- Phase 1 остаётся в рамках 10 AC, не раздувается.
- Fallback принцип сохранён: ядро не зависит от LangGraph, Docker, RLS.
- Security закрывает 90% реальных угроз минимальным кодом.

### Negative
- Application-level filtering требует дисциплины: разработчик не должен забывать 
  `.filter(project_id=...)` в новых запросах.
- Без HMAC подделка `error_logged` теоретически возможна при компрометации БД 
  (mitigated: доступ к БД только у админа-соло-разработчика).

---

## Implementation Tasks (для Kimi Code)

- [ ] Добавить `project_id: str = "default"` в `core/models.py` (`Event`, `AgentState`, `JournalEntry`).
- [ ] Обновить `EventEngine`: фильтрация `read/write` по `project_id`; метод `set_project_context(project_id)`.
- [ ] Обновить `memory/semantic.py`: динамическое имя коллекции `f"voyage_{project_id}"`.
- [ ] Обновить `memory/context_tiers.py`: `project_id` в `ProjectContext`.
- [ ] Обновить `specs/task_generator.py`: добавить `<<retrieved_context>>` wrapping.
- [ ] Обновить `tools/adapters/python.py`: PyPI validation перед `pip install`.
- [ ] Добавить `security/backends/docker.py` и `security/factory.py` (Strategy Pattern).
- [ ] Добавить `integrations/orchestrator_adapter.py` (stub interface).
- [ ] Обновить тесты Phase 1: изоляция по `project_id` не должна сломать существующие 10 AC.
- [ ] Не реализовывать: HMAC signing, hash chain, RLS DDL, LangGraph adapter.

---

## Compliance

- **Event Sourcing**: `project_id` — shard key, не нарушает append-only семантику.
- **Security First**: deterministic filters закрывают L3.5 (RAG Injection) и L2.5 (Supply Chain).
- **Spec-Driven**: ADR фиксирует решение до кода.
- **Fallback**: SQLite + собственный runtime + Docker sandbox = работает без PostgreSQL, LangGraph, seccomp.
- **Transparent**: scoped RULES.md и project-isolated events делают аудит понятным.

---

## Notes

- `project_id = "default"` используется для обратной совместимости с существующими событиями SkillTracer.
- При миграции на Phase 2 все `default` события могут быть явно привязаны к `project_id="skilltracer"` через миграционный скрипт.
