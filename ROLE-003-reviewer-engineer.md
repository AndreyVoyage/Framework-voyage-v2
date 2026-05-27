# ROLE-003: Reviewer Engineer (Architecture & Merge Guardian)

**Status**: Planned  
**Date**: 2026-05-26  
**Authors**: AndreyVoyage (Human) + AI Architect  
**Phase**: Phase 2 (Soul & Connectors)  
**Replaces**: Нет (новая роль)  
**Related**: ROLE-001 (QA Engineer), ROLE-002 (Security Engineer), ADR-005 (Project Isolation)

---

## Purpose

Существующий Reviewer (ROLE-000 base) проверяет code quality (mypy, ruff, AST impact). Но для AI-native framework нужен **центральный governance layer**:

- Валидация архитектурной целостности.
- Проверка consistency между ADR.
- Semantic diff: "что реально изменилось в коде?"
- Backward compatibility check.
- Coupling growth detection.
- Technical debt scoring.
- Hallucinated code paths detection.
- AI-generated migrations validation.

**Почему сейчас:** Анализ показал, что framework имеет сильную архитектуру, но **governance не enforceится**. Reviewer Engineer — это merge guardian, который не даёт AI деградировать систему.

---

## Responsibilities

### В зоне ответственности

#### 1. Architectural Review
- Проверка: изменения не нарушают ADR.
- Проверка: новый код соответствует ADR-005 (project isolation).
- Проверка: нет architectural drift (код отклоняется от ADR).
- Генерация `architectural_score` (0.0–1.0) для каждого PR.

#### 2. ADR Consistency Validation
- Проверка: новый ADR не конфликтует с существующими.
- Проверка: ADR-INDEX актуален (все ADR упомянуты).
- Проверка: статусы ADR корректны (Accepted → Deprecated → Superseded).

#### 3. Semantic Diff Analysis
- Не просто `git diff`, а **понимание**, что изменилось:
  - "Функция `handle_message` теперь принимает `project_id` — это соответствует ADR-005."
  - "Добавлен новый импорт `chromadb` — это соответствует ADR-002."
- Сравнение с ADR: каждое изменение должно быть обосновано.

#### 4. Backward Compatibility
- Проверка: публичный API не сломан.
- Проверка: Event schema не изменена без миграции.
- Проверка: `project_id` добавлен без breaking changes.

#### 5. Coupling & Cohesion Analysis
- Проверка: новый код не создаёт циклических зависимостей.
- Проверка: модуль не импортирует «всё подряд».
- Метрики: cyclomatic complexity, coupling between objects.

#### 6. Technical Debt Scoring
- Оценка debt для каждого изменения: "+0.05 debt points".
- Аккумуляция debt по проекту: "текущий debt = 2.3/10".
- Рекомендации: "refactor module X — debt превышает threshold".

#### 7. Hallucination Detection
- Проверка: AI не сгенерировал несуществующие функции/классы.
- Проверка: AI не импортировал несуществующие модули.
- Проверка: AI не изменил файлы вне разрешённой директории.

#### 8. Merge Gate
- **Блокировка merge**, если:
  - architectural_score < 0.7
  - ADR consistency check failed
  - backward compatibility broken
  - coupling threshold exceeded
  - debt score > 5.0
  - hallucination detected
- **Разрешение merge** только после human approval (для critical) или auto-approve (для routine).

### Вне зоны ответственности
- Не пишет production код (только review-скрипты и validators).
- Не запускает функциональные тесты (это QA Engineer).
- Не аудитит безопасность (это Security Engineer).
- Не деплоит (это DevOps).

---

## Tools

| Адаптер | Инструменты | Требует approval | Описание |
|---------|-------------|------------------|----------|
| **adr_validator.py** *(новый)* | `validate_adr_consistency`, `check_architectural_drift` | Нет | Проверка ADR на конфликты |
| **semantic_diff.py** *(новый)* | `semantic_diff`, `impact_analysis` | Нет | Понимание изменений, не just diff |
| **compatibility.py** *(новый)* | `check_api_breaking`, `check_event_schema` | Нет | Backward compatibility |
| **coupling.py** *(новый)* | `analyze_coupling`, `detect_cycles` | Нет | Cyclomatic complexity, CBO |
| **debt_scorer.py** *(новый)* | `score_debt`, `debt_report` | Нет | Technical debt accumulation |
| **hallucination.py** *(новый)* | `detect_hallucinated_imports`, `detect_ghost_functions` | Нет | AI hallucination detection |
| **merge_gate.py** *(новый)* | `evaluate_merge`, `block_merge`, `approve_merge` | Да (critical) | Merge gate controller |
| **git.py** | `git_diff`, `git_log` | Нет | Изменения для анализа |
| **ast_manager.py** | `query_ast`, `get_dependencies` | Нет | AST для impact analysis |

---

## Prompt Focus

**Примеры запросов к агенту:**
- "Проанализируй PR #42: есть ли architectural drift?"
- "Сравни изменения с ADR-005: добавлен ли project_id везде, где нужно?"
- "Какой architectural score у этого изменения?"
- "Есть ли hallucinated imports? Проверь, что все импорты существуют."
- "Проверь backward compatibility: сломан ли публичный API?"
- "Оцени technical debt: сколько debt points добавляет этот PR?"
- "Блокируй merge: coupling превышает threshold."

**Acceptance Criteria для работы Reviewer Engineer:**
- [ ] Architectural score ≥ 0.7 для каждого PR.
- [ ] 0 ADR conflicts для каждого PR.
- [ ] 0 breaking changes без explicit migration.
- [ ] 0 hallucinated imports/functions.
- [ ] Debt score < 5.0 per project.
- [ ] Merge gate блокирует некорректные PR.

---

## Policy (что может / не может)

**Может:**
- Читать весь код, ADR, git history.
- Запускать все review-инструменты без approval.
- Блокировать merge (auto-block для routine, human approval для critical).
- Генерировать review reports.
- Добавлять architectural rules в `RULES.md`.

**Не может:**
- Писать production код (только validators, скрипты).
- Менять ADR без approval Architect.
- Пропускать merge без проверки (exception: hotfix workflow с human override).
- Изменять git history.

---

## Implementation Tasks (для Kimi Code)

- [ ] Создать `agents/roles/reviewer_engineer.py`, наследующий `AgentRole`.
- [ ] Зарегистрировать в `RoleRegistry` как `"reviewer_engineer"`.
- [ ] Создать адаптер `tools/adapters/adr_validator.py`.
- [ ] Создать адаптер `tools/adapters/semantic_diff.py`.
- [ ] Создать адаптер `tools/adapters/compatibility.py`.
- [ ] Создать адаптер `tools/adapters/coupling.py`.
- [ ] Создать адаптер `tools/adapters/debt_scorer.py`.
- [ ] Создать адаптер `tools/adapters/hallucination.py`.
- [ ] Создать адаптер `tools/adapters/merge_gate.py`.
- [ ] Добавить workflow-определение `review_pipeline.yaml`:
  ```yaml
  nodes:
    - name: architectural_review
      role: reviewer_engineer
      tools: [validate_adr_consistency, semantic_diff, check_api_breaking]
      transitions:
        - condition: score > 0.7
          target: qa_tests
        - condition: score <= 0.7
          target: developer_fix
    - name: merge_gate
      role: reviewer_engineer
      tools: [evaluate_merge]
      transitions:
        - condition: approved
          target: devops_deploy
        - condition: blocked
          target: end
  ```
- [ ] Написать тесты: `test_agents_roles_reviewer_engineer.py`.
- [ ] Обновить `security/policy.py` — добавить permissions для Reviewer Engineer.
- [ ] Обновить `ROLE-INDEX.md` — статус изменён на Implemented.
- [ ] Обновить `FUTURE_ROLES_TECH_REGISTRY.md` — статус ROLE-003 = Implemented.

---

## Notes

- **Architectural score formula:**
  ```
  score = 0.3 * adr_compliance + 0.2 * backward_compat + 0.2 * coupling_score + 0.15 * debt_score + 0.15 * no_hallucination
  ```
- **Merge gate levels:**
  - **Auto-approve:** score > 0.9, no critical findings, routine change.
  - **Auto-block:** score < 0.7, any critical finding, breaking change.
  - **Human review:** 0.7 <= score <= 0.9, or critical change (ADR modification, security fix).
- **ADR conflict detection:**
  - Два ADR с противоречивыми решениями → flag.
  - ADR ссылается на Superseded ADR → flag.
  - ADR не упомянут в ADR-INDEX → flag.
- **Hallucination detection:**
  - Импорт `from nonexistent_module import X` → flag.
  - Вызов `obj.nonexistent_method()` → flag.
  - Изменение файла вне `relevant_files` из CONTEXT.json → flag.

---

**Версия ROLE:** 1.0  
**Дата:** 2026-05-26
