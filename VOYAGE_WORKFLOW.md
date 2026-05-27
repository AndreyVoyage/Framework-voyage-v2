# Voyage Framework — Пошаговое руководство по применению

> **Для кого:** один разработчик + Kimi Code в VS Code
> **Цель:** превратить фреймворк из документов в рабочий процесс каждой сессии
> **Время на старт:** 15 минут подготовки + 5 минут в начале каждой сессии

---

## Phase 0. Подготовка (делается один раз на проект)

### Шаг 0.1. Создай папку фреймворка в корне проекта
```bash
mkdir -p voyage/{adr,roles,tech,rules,traces,reasoning,dead_letter}
touch voyage/.gitkeep
```

### Шаг 0.2. Скопируй базовые артефакты
Помести в `voyage/` следующие файлы (скачанные ранее):
- `ADR-INDEX.md`, `ROLE-INDEX.md`, `FUTURE_ROLES_TECH_REGISTRY.md`
- `ADR-001..009.md`
- `ROLE-000..003.md`
- `TEST_STRATEGY.md`, `RULES.md`, `TECH-001-appsec-stack.md`
- `INSTRUCTIONS.md`, `LANGGRAPH_INTEGRATION_ANALYSIS.md`

### Шаг 0.3. Инициализируй логи
```bash
touch voyage/traces/voyage-log-$(date +%Y-%m-%d).jsonl
touch voyage/traces/trace-$(date +%Y-%m-%d).md
touch voyage/metrics.md
```

### Шаг 0.4. Проверь Complexity Budget
Открой `FUTURE_ROLES_TECH_REGISTRY.md` → убедись, что:
- ADR: 9 / 9 (лимит достигнут)
- Роли: 4 / 5 (запас 1)

---

## Phase 1. Первая сессия с Kimi Code (алгоритм)

### Шаг 1.1. Запуск сессии (2 минуты)
```bash
export VOYAGE_SESSION=$(date +%s)
export VOYAGE_DATE=$(date +%Y-%m-%d)
touch voyage/dead_letter/dead_letter-${VOYAGE_DATE}.md
```

### Шаг 1.2. Загрузка контекста в Kimi Code (3 минуты)
В чате Kimi Code загрузи файлы в таком порядке:

1. **FUTURE_ROLES_TECH_REGISTRY.md** — лимиты сложности
2. **ADR-009.md** — ограничения (Kimi не будет предлагать лишнее)
3. **ADR-007.md** — runtime (Kimi знает lifecycle сессии)
4. **RULES.md** — текущие правила проекта
5. **ROLE-003.md** — reviewer (Kimi знает, что review обязателен)

> **Порядок важен:** сначала ограничения, потом процесс, потом правила.

### Шаг 1.3. Формулировка задачи (2 минуты)
```markdown
## Сессия: [короткое название]

### Намерение
Что нужно сделать: ___

### Роль
Кто ведёт: ROLE-001 (Architect) / ROLE-002 (Security) / ROLE-003 (Reviewer)

### Ограничения
- Макс. задач в плане: 10 (ADR-009)
- Макс. retry: 3 (ADR-007)
- Нельзя: [перечисли запреты из ADR-009 Anti-Abstraction]
```

### Шаг 1.4. Plan Mode
Если задача сложная:
1. «Включи Plan Mode. Построй план не более 10 задач.»
2. Проверь: не больше 10 пунктов? Нет ли запрещённого?
3. Утверди или упрости план.

### Шаг 1.5. Execution
Для каждой задачи:
```text
1. Kimi генерирует код / изменения
2. Ты проверяешь глазами (или просишь Kimi review)
3. Если ок → git add + git commit
4. Если не ок → Retry (max 3 раза)
5. Если 3 retry не помогли → DEAD_LETTER
```

---

## Phase 2. Чек-лист каждой сессии

```markdown
### Pre-flight
- [ ] Git статус чистый или закоммичен
- [ ] Дата и session ID записаны
- [ ] Complexity Budget не превышен

### Context Load
- [ ] Загружен FUTURE_ROLES_TECH_REGISTRY
- [ ] Загружен ADR-009 (limits)
- [ ] Загружен ADR-007 (runtime)
- [ ] Загружен RULES.md
- [ ] Загружен ROLE-003 (reviewer)

### Execution
- [ ] Задача сформулирована по шаблону
- [ ] План не более 10 задач
- [ ] Каждая задача прошла review
- [ ] Нет hallucination

### Post-flight
- [ ] Все изменения закоммичены
- [ ] Записан voyage-log.jsonl
- [ ] Записан trace.md
- [ ] Метрики обновлены
- [ ] DEAD_LETTER записан, если были непройденные retry
```

---

## Phase 3. Работа с артефактами

| Артефакт | Когда обновлять | Кто обновляет |
|:---|:---|:---|
| **ADR-INDEX.md** | Новый статус ADR, архивация | Ты вручную |
| **ROLE-INDEX.md** | Новая роль или изменение | Ты вручную |
| **FUTURE_ROLES_TECH_REGISTRY** | Еженедельно или при смене статуса | Ты вручную |
| **RULES.md** | Новое правило, истечение TTL | Ты вручную (AI не трогает без approval) |
| **voyage-log-*.jsonl** | Каждая сессия | Ты вручную или скрипт |
| **trace-*.md** | Каждая сессия | Ты вручную или скрипт |
| **metrics.md** | Каждая сессия | Ты вручную |
| **dead_letter-*.md** | Когда задача ушла в DEAD_LETTER | Ты вручную |

---

## Phase 4. Emergency Override

```markdown
### Override Log
- Дата: ___
- Причина: hotfix / demo / critical bug
- Что bypass: review / complexity_check
- Ручное подтверждение: [ ] Да
- Post-hoc ADR требуется: [ ] Да
- TTL: 24 часа
```

---

## Phase 5. Ретроспектива (раз в неделю, 10 минут)

1. Открой `metrics.md` — растёт ли retry? hallucinations?
2. Открой `dead_letter/` — какие задачи висят?
3. Проверь `FUTURE_ROLES_TECH_REGISTRY` — есть ли Planned >2 недель?
4. Проверь `RULES.md` — есть ли истёкшие TTL?

---

## Быстрый старт

```bash
mkdir -p voyage/{adr,roles,tech,rules,traces,reasoning,dead_letter}
export VOYAGE_DATE=$(date +%Y-%m-%d)
touch voyage/traces/voyage-log-${VOYAGE_DATE}.jsonl
# В Kimi Code загрузи: FUTURE_ROLES_TECH_REGISTRY → ADR-009 → ADR-007 → RULES → ROLE-003
```

---

## Приоритеты для Kimi Code (P0..P4)

| Приоритет | Что делать | Почему |
|:---|:---|:---|
| **P0** | Загрузить ADR-009 (Complexity Budget) | Без этого Kimi будет генерировать лишнее |
| **P1** | Загрузить ADR-007 (Runtime) | Kimi понимает retry, timeout, rollback |
| **P2** | Загрузить ROLE-003 (Reviewer) | Kimi знает, что review — не опция |
| **P3** | Загрузить RULES.md | Конкретные проверяемые правила |
| **P4** | Загрузить FUTURE_ROLES_TECH_REGISTRY | Общая картина |

---

*Версия фреймворка: Voyage 1.0 (Phase 1)*
