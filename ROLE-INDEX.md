# ROLE-INDEX — Реестр ролей агентов Voyage Framework v4.0+

> **Назначение:** Единый источник правды для всех ролей AI-агентов.
> **Правило:** Если роль не внесена в ROLE-INDEX — её нет в системе.
> **Обновляет:** Human (AndreyVoyage) + Architect Agent при добавлении новой роли.
> **Связь с ADR:** Роли реализуют архитектурные решения, но не являются ими. ROLE ≠ ADR.
> **Complexity Budget:** Макс. 5 ролей в статусе Planned/Implemented/Active. Сейчас 4 — запас 1.

---

## Жизненный цикл роли
```
Concept → Planned → Implemented → Active → Deprecated → Archived
    ↑___________↓
    (можно вернуть из Deprecated, если отмена была ошибкой)
```

---

## Реестр ролей

| ID | Название | Статус | Phase | Инструменты | Технологии | Краткая суть |
|----|----------|--------|-------|-------------|------------|--------------|
| [ROLE-000](ROLE-000-base-role.md) | Base AgentRole (ABC) | Implemented | Phase 1 | — | Python ABC | Базовый класс для всех ролей |
| [ROLE-001](ROLE-001-qa-engineer.md) | QA Engineer | Planned | Phase 2 | pytest, playwright, coverage, grep_code | Playwright, Cypress, Jest | Тесты, coverage, flaky analysis |
| [ROLE-002](ROLE-002-security-engineer.md) | Security Engineer (AppSec) | Planned | Phase 2 | bandit, semgrep, gitleaks, npm_audit, grep_code | SAST, DAST, SCA, IAST, Vault | Уязвимости, secrets, compliance |
| [ROLE-003](ROLE-003-reviewer-engineer.md) | Reviewer Engineer (Architecture & Merge Guardian) | Planned | Phase 2 | adr_validator, semantic_diff, coupling, debt_scorer, hallucination, merge_gate | AST, Git, ADR validation | Architectural review, merge gate, hallucination detection |
| ROLE-004 | Performance Engineer | Concept | Phase 2+ | locust, k6, explain_query, chrome_lighthouse | Locust, k6, PostgreSQL EXPLAIN | Нагрузка, profiling, оптимизация |
| ROLE-005 | Product Manager | Concept | Phase 3 | semantic_query, event_engine, github | Metrics, Backlog, Prioritization | Приоритизация, бэклог, метрики |

---

## ROLE-003 — Режимы работы (дополнение)

Одна роль, три режима (Complexity Budget: не плодим отдельные роли).

| Режим | Когда активируется | Что проверяет | Gate |
|-------|-------------------|---------------|------|
| **Mode A: Architecture Reviewer** | Изменения в архитектуре / новый ADR | ADR compliance, coupling, layering | Блок merge |
| **Mode B: Code Reviewer** | После каждого codegen | Bugs, anti-patterns, quality, coverage | Блок merge при критических |
| **Mode C: AI Safety Reviewer** | Подозрение на hallucination | Fake APIs, ghost functions, unsafe execution | DEAD_LETTER немедленно |

**Переключение:**
```yaml
default_mode: B                    # Code Reviewer — чаще всего
architecture_changes: A            # При затронутых ADR
ai_output_suspicious: C            # Если structured log показывает hallucination risk
```

---

## Complexity Budget (ADR-009)
- **Активных/Planned ролей:** 4 / 5 (запас 1)
- **Concept ролей (ROLE-004, ROLE-005):** не переводить в Planned без архивации существующей или увеличения лимита (требует ADR)

---

## Шаблон новой роли

При создании `ROLE-NNN-kebab-name.md` используй следующую структуру:

```markdown
# ROLE-NNN: Название роли

**Status**: Concept | Planned | Implemented | Active | Deprecated | Archived
**Date**: YYYY-MM-DD
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase N
**Replaces**: ROLE-XXX (или "Нет")
**Related**: ROLE-YYY, ADR-ZZZ, TECH-WWW

---

## Purpose
Зачем эта роль? Какую задачу решает, которую не решают текущие роли?

## Responsibilities
- Что именно делает агент?
- Что НЕ делает (границы ответственности)?

## Tools
| Адаптер | Инструменты | Требует approval |
|---------|-------------|------------------|
| xxx.py | `tool_a`, `tool_b` | Да / Нет |

## Prompt Focus
- Что спрашивать у агента?
- Какие acceptance criteria для его работы?

## Policy (что может / не может)
- **Может:** ...
- **Не может:** ...

## Implementation Tasks (для Kimi Code)
- [ ] Создать `agents/roles/xxx.py`
- [ ] Зарегистрировать в `RoleRegistry`
- [ ] Добавить workflow-определения
- [ ] Написать тесты
- [ ] Обновить `security/policy.py`

## Notes
```

---

## Частые ошибки (авто-правила для Self-Improving Engine)

| Ошибка | Правило |
|--------|---------|
| ROLE создан, но не добавлен в ROLE-INDEX | Всегда обновляй ROLE-INDEX в том же коммите |
| Роль в статусе Planned, но код уже написан | Статус должен быть Implemented ДО merge |
| Дублирование ROLE в мастер-документе | Мастер-документ ссылается на ROLE-INDEX, не дублирует текст |
| Роль реализована, но не зарегистрирована в RoleRegistry | Регистрация — обязательный шаг в Implementation Tasks |
| Роль не имеет merge gate permissions | Reviewer Engineer должен проверять permissions новой роли |

---

**Версия индекса:** 1.2
**Дата:** 2026-05-26
**Следующее обновление:** После реализации ROLE-001, ROLE-002 или ROLE-003
