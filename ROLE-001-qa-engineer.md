# ROLE-001: QA Engineer (Quality Assurance Engineer)

**Status**: Planned  
**Date**: 2026-05-26  
**Authors**: AndreyVoyage (Human) + AI Architect  
**Phase**: Phase 2 (Soul & Connectors)  
**Replaces**: Нет  
**Related**: ROLE-002 (Security Engineer), ADR-005 (Project Isolation), TECH-001 (AppSec Stack)

---

## Purpose
Reviewer проверяет *качество кода* (type hints, lint, AST impact), но не *качество продукта*. QA Engineer закрывает этот пробел:
- Генерация тест-кейсов по acceptance criteria.
- Написание E2E-тестов для Telegram-бота (Aiogram) и React SPA.
- Анализ flaky-тестов: "почему этот тест падает 30% времени?"
- Regression testing перед деплоем.
- Coverage tracking: "coverage упал с 85% до 72% — что не покрыто?"

**Почему сейчас:** В SkillTracer уже есть баги (initData invalid, фото не удаляется, красный крестик), которые ручное тестирование через Telegram ловит медленно. Автоматизация тестирования сократит цикл "разработка → баг-репорт → фикс".

---

## Responsibilities

### В зоне ответственности
- Написание unit-тестов (pytest) для новых фич.
- Написание E2E-тестов:
  - Frontend: Playwright/Cypress для React SPA.
  - Backend: Aiogram-тесты для Telegram-бота (mocked updates).
- Анализ flaky-тестов: race conditions, async timing, state leakage между тестами.
- Coverage reports: pytest-cov, генерация HTML-отчёта.
- Regression suite: перед деплоем запускается полный набор тестов.
- Тест-кейсы по acceptance criteria из TASK.md.

### Вне зоны ответственности
- Не пишет production код (только тесты).
- Не деплоит (это DevOps).
- Не аудитит безопасность (это Security Engineer).
- Не оптимизирует производительность (это Performance Engineer).

---

## Tools

| Адаптер | Инструменты | Требует approval | Описание |
|---------|-------------|------------------|----------|
| **python.py** | `pytest`, `pytest_cov`, `pytest_asyncio` | Нет | Unit + integration тесты |
| **playwright.py** *(новый)* | `playwright_test`, `playwright_codegen` | Нет | E2E для React SPA |
| **aiogram_test.py** *(новый)* | `mock_tg_update`, `assert_bot_response` | Нет | Тесты Telegram-бота |
| **coverage.py** *(новый)* | `coverage_report`, `coverage_diff` | Нет | Анализ покрытия |
| **git.py** | `git_diff` | Нет | Находит изменённые файлы для таргетированного тестирования |
| **grep_code.py** | `find_untested_functions` | Нет | Ищет функции без тестов |

---

## Prompt Focus

**Примеры запросов к агенту:**
- "Напиши тест для acceptance criteria: 'PostgresStorage подключён к aiogram Dispatcher'."
- "Почему `test_bot_start` падает 3 раза из 10? Анализируй race condition."
- "Coverage упал с 85% до 72% после PR #42. Что не покрыто?"
- "Сгенерируй E2E-тест для сценария: пользователь добавляет фото → оно отображается → удаляется."
- "Запусти regression suite перед деплоем. Отчёт: сколько прошло, сколько упало."

**Acceptance Criteria для работы QA Engineer:**
- [ ] Все acceptance criteria из TASK.md имеют ≥1 тест.
- [ ] Coverage не падает ниже 80%.
- [ ] Нет flaky-тестов (стабильность >95% за 10 прогонов).
- [ ] E2E-тесты проходят на staging-окружении.

---

## Policy (что может / не может)

**Может:**
- Запускать все тестовые инструменты без approval.
- Читать любой код для написания тестов.
- Писать тесты в `tests/` и `e2e/`.
- Блокировать merge, если coverage упал или тесты падают.

**Не может:**
- Писать production код (только тесты + test fixtures).
- Менять архитектуру.
- Деплоить.
- Пропускать тесты без human approval (exception: hotfix workflow).

---

## Implementation Tasks (для Kimi Code)

- [ ] Создать `agents/roles/qa.py`, наследующий `AgentRole`.
- [ ] Зарегистрировать в `RoleRegistry` как `"qa"`.
- [ ] Создать адаптер `tools/adapters/playwright.py` (wrapper над Playwright CLI).
- [ ] Создать адаптер `tools/adapters/aiogram_test.py` (mocked Telegram updates).
- [ ] Создать адаптер `tools/adapters/coverage.py` (pytest-cov wrapper).
- [ ] Добавить workflow-определение `feature_with_qa.yaml`:
  ```yaml
  nodes:
    - name: run_tests
      role: qa
      tools: [pytest, playwright_test, coverage_report]
      transitions:
        - condition: success
          target: reviewer_check
        - condition: failure
          target: developer_fix
  ```
- [ ] Написать тесты: `test_agents_roles_qa.py`.
- [ ] Обновить `security/policy.py` — добавить permissions для QA.
- [ ] Обновить `ROLE-INDEX.md` — статус изменён на Implemented.

---

## Notes

- **Flaky-test analysis:** QA Engineer должен использовать EventEngine для поиска истории падений теста. Если `test_x` падал 5 раз за неделю — агент автоматически повышает приоритет расследования.
- **Project isolation (ADR-005):** Тесты и coverage reports хранятся в project-scoped директории: `projects/{project_id}/test_reports/`.
- **Telegram WebApp тестирование:** Playwright может тестировать WebApp через `tgWebAppData` mock, но не тестирует сам Telegram-клиент. Для бота используем `aiogram` TestClient.

---

**Версия ROLE:** 1.0  
**Дата:** 2026-05-26
