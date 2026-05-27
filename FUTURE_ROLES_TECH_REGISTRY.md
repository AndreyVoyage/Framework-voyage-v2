# FUTURE_ROLES_TECH_REGISTRY — Сводный реестр будущего развития

> **Назначение:** Единая таблица "что запланировано → когда → в каком статусе".
> **Правило:** Это 'дорожная карта' на уровне ролей и технологий, а не фаз.
> **Обновляет:** Human (AndreyVoyage) каждую неделю или при принятии решения.
> **Complexity Budget:** Макс. 9 ADR, 5 ролей. Новые артефакты — только через освобождение слота.

---

## Как читать этот файл

**Строка = одна единица развития** (роль, технология, интеграция).
**Колонки:**
- **ID:** ROLE-NNN, TECH-NNN, INT-NNN (integration).
- **Название:** Человекочитаемое.
- **Тип:** Role / Tech / Integration.
- **Phase:** Когда внедрять (Phase 1/2/3).
- **Статус:** Concept → Planned → In Progress → Implemented → Active.
- **Блокируется:** Что нужно сделать ДО этого (зависимости).
- **AC:** Acceptance Criteria — когда считать "готово".

---

## Сводная таблица

| ID | Название | Тип | Phase | Статус | Блокируется | AC |
|----|----------|-----|-------|--------|-------------|----|
| ROLE-000 | Base AgentRole | Role | Phase 1 | Implemented | — | ABC класс, наследование |
| ROLE-001 | QA Engineer | Role | Phase 2 | Planned | ROLE-000, Agent Runtime stable | Playwright + aiogram tests, coverage ≥80% |
| ROLE-002 | Security Engineer | Role | Phase 2 | Planned | ROLE-000, Agent Runtime stable | bandit + semgrep + gitleaks clean |
| ROLE-003 | Reviewer Engineer | Role | Phase 2 | Planned | ROLE-000, Agent Runtime stable, ADR-006 accepted | Architectural score ≥0.7, merge gate working |
| ROLE-004 | Performance Engineer | Role | Phase 2+ | Concept | ROLE-001, ROLE-002, ROLE-003 | Locust tests, profiling reports |
| ROLE-005 | Product Manager | Role | Phase 3 | Concept | Visualizer API, metrics | Приоритизация по impact/effort |
| ADR-001 | PostgreSQL Events | ADR | Phase 1 | Accepted | — | EventEngine работает |
| ADR-002 | ChromaDB Semantic | ADR | Phase 1 | Accepted | ADR-001 | Semantic search работает |
| ADR-003 | tree-sitter AST | ADR | Phase 1 | Accepted | — | AST graph строится |
| ADR-004 | Dual-Location | ADR | Phase 1 | Accepted | — | Git submodule настроен |
| ADR-005 | Project Isolation | ADR | Phase 1 | Accepted | ADR-001, ADR-004 | project_id везде |
| ADR-006 | Rule Governance | ADR | Phase 2 | Proposed | ADR-005, ROLE-003 | Rule Engine executable |
| ADR-007 | Runtime Orchestration | ADR | Phase 2 | Proposed | ADR-005, ROLE-000 | State machine для сессии |
| ADR-008 | Observability | ADR | Phase 2 | Proposed | ADR-007 | Structured logs, trace tree |
| ADR-009 | Complexity Budget | ADR | Phase 2 | Proposed | Все ADR | Hard limits enforced |
| TECH-001 | AppSec Stack | Tech | Phase 2 | Planned | Tool Engine stable | 6 адаптеров (bandit, semgrep, zap, sca, secrets, config) |
| TECH-002 | Testing Stack | Tech | Phase 2 | Planned | Tool Engine stable | 3 адаптера (playwright, aiogram_test, coverage) |
| TECH-003 | Performance Stack | Tech | Phase 2+ | Concept | TECH-002 | locust, k6, explain_query |
| INT-001 | LangGraph Adapter | Integration | Phase 2+ | Planned | Orchestrator Adapter stub | YAML workflow → LangGraph graph |
| INT-002 | GitHub Actions CI | Integration | Phase 2 | Planned | ROLE-001, ROLE-002, ROLE-003 | Pipeline: lint → test → security → review → deploy |
| INT-003 | HashiCorp Vault | Integration | Phase 2+ | Concept | DevOps role stable | Secrets вне репозитория |
| INT-004 | Cloudflare WAF | Integration | Phase 2+ | Concept | DevOps role stable | DDoS + bot protection |

---

## Complexity Budget Status

```yaml
adr:
  active: 5      # ADR-001..005
  proposed: 4    # ADR-006..009
  total: 9
  limit: 9
  status: LIMIT_REACHED  # Новый ADR только через архивацию

roles:
  implemented: 1   # ROLE-000
  planned: 3     # ROLE-001..003
  concept: 2     # ROLE-004..005
  planned_active_total: 4
  limit: 5
  status: OK  # Запас 1 слот

rules:
  dependencies_max: 3
  review_depth_max: 2
  plan_tasks_max: 10
```

---

## Принцип добавления новой строки

**Запрещено добавлять без освобождения слота:**
- Новый ADR (ADR-010+) — пока не заархивирован один из 001..009.
- Новая роль в статусе Planned/Implemented — пока не заархивирована одна из 000..003.

**Разрешено добавлять:**
- Concept-роли (ROLE-006+) — но они не переводятся в Planned без слота.
- Concept-технологии (TECH-004+) — без ограничений.
- Concept-интеграции (INT-005+) — без ограничений.

---

## Чек-лист еженедельного обзора

Каждую неделю (или при планировании спринта):

- [ ] Открыть FUTURE_ROLES_TECH_REGISTRY.
- [ ] Проверить статусы: что перешло из Planned → In Progress?
- [ ] Проверить блокеры: что разблокировалось?
- [ ] Проверить Complexity Budget: не превышены ли лимиты?
- [ ] Выбрать 1–2 пункта на следующую неделю.
- [ ] Обновить статусы и AC (если достигнуты).
- [ ] НЕ трогать мастер-документ — он обновляется только при смене Phase.

---

**Версия реестра:** 1.2
**Дата:** 2026-05-26
**Следующее обновление:** После перехода ADR-006..009 или ROLE-001..003 в статус Implemented
