# LangGraph Integration Analysis — Voyage Framework v4.0

> **Назначение:** Анализ целесообразности использования LangGraph для разработки Voyage Framework.  
> **Дата:** 2026-05-26  
> **Авторы:** AndreyVoyage (Human) + AI Architect  
> **Статус:** Рекомендация принята (ADR-005, пункт 5) — LangGraph как optional integration Phase 2

---

## 1. Что такое LangGraph (кратко)

LangGraph — это orchestration framework от команды LangChain. Моделирует workflow как направленный граф:
- **Nodes** = агенты или функции (Architect, Developer, Reviewer, DevOps).
- **Edges** = переходы между нодами (успех → дальше, ошибка → retry).
- **State** = общее состояние, которое передаётся между нодами.
- **Checkpointing** = сохранение состояния после каждого шага (можно resume после crash).
- **Human-in-the-loop** = встроенный механизм паузы для human approval.

---

## 2. Сопоставление с вашей архитектурой

### Ваш Custom Runtime (Phase 1)
```
plan → execute → reflect → retry → done
   ↑___________________________↓
```
- Простой цикл с 4 состояниями.
- Checkpoint после каждого node.
- Retry при confidence < 0.7 или failed tests.
- Human approval только для dangerous tier.

### LangGraph (Phase 2+)
```
┌─────────┐    ┌──────────┐    ┌───────────┐    ┌────────┐
│  idle   │───→│ planning │───→│ executing │───→│reflect │
└─────────┘    └──────────┘    └───────────┘    └───┬────┘
                                                     │
                              ┌──────────────────────┘
                              │
                         ┌────┴────┐
                         │ retrying│←── (conditional edge)
                         └────┬────┘
                              │
                         ┌────┴────┐
                         │  done   │
                         └─────────┘
```
- Тот же цикл, но явно описан как граф.
- Conditional edges определяют routing (успех/ошибка).
- Checkpointing встроен (PostgresSaver, AsyncSqliteSaver).
- Human-in-the-loop через `interrupt()`.

**Вывод:** LangGraph решает ТЕ ЖЕ задачи, что и ваш Custom Runtime, но с большей детерминированностью и observability.

---

## 3. Ускорит ли LangGraph разработку вашего фреймворка?

### ✅ Да, в следующих сценариях:

#### 1. Human-in-the-loop на уровне workflow
- **Ваш Runtime:** Human approval только для dangerous tier (systemctl, ssh). Для всего остального — автоматика.
- **LangGraph:** Можно вставить `interrupt()` на ЛЮБОМ шаге. Например: "Architect спланировал → human проверил план → Developer начал код". Это полезно для критичных архитектурных решений.
- **Ускорение:** Human не нужно проверять весь результат — только ключевые точки. Экономит время на review.

#### 2. Time-travel debugging
- **Ваш Runtime:** Checkpoint сохраняет состояние, но нет визуального дебаггера.
- **LangGraph:** LangGraph Studio — визуальный IDE для дебага графа. Можно кликнуть на любой node и посмотреть состояние. Можно "отмотать" назад и изменить решение.
- **Ускорение:** Дебаг сложных workflow (например, "Architect → Developer → Reviewer → Developer (fix) → Reviewer → DevOps") в 2–3 раза быстрее.

#### 3. Sub-graph composition
- **Ваш Runtime:** Один workflow = один цикл.
- **LangGraph:** Граф может быть node внутри другого графа. Например: "Feature workflow" содержит "Testing sub-graph", который содержит "Unit tests sub-graph" + "E2E tests sub-graph".
- **Ускорение:** Переиспользование workflow-компонентов. Не писать "тестирование" заново для каждого workflow.

#### 4. Deterministic routing
- **Ваш Runtime:** Routing (куда идти после reflect) зависит от confidence score, который вычисляет LLM. Это может быть недетерминировано.
- **LangGraph:** Routing — это явная Python-функция. Compliance-аудитор может прочитать код и понять, почему агент пошёл туда, а не сюда.
- **Ускорение:** Меньше time spent на объяснение "почему агент сделал X".

#### 5. Production observability
- **Ваш Runtime:** EventEngine логирует события, но нет визуального trace.
- **LangGraph:** LangSmith — полный trace каждого запуска графа. Видно: какая нода сколько времени заняла, сколько токенов сожрала, где упала.
- **Ускорение:** Оптимизация workflow на основе данных, а не интуиции.

### ❌ Нет, в следующих сценариях:

#### 1. Простые workflow (feature, bugfix)
- **Ваш Runtime:** 4 состояния, линейный цикл. Работает идеально.
- **LangGraph:** Добавляет boilerplate (StateGraph, TypedDict, compile). Для простого цикла — overkill.
- **Замедление:** +20–30% времени на написание workflow-определения.

#### 2. Phase 1 (Foundation)
- **Ваш Runtime:** Нужен СЕЙЧАС. LangGraph — дополнительная зависимость, которую нужно изучать.
- **LangGraph:** Learning curve 1–2 недели для graph-based мышления.
- **Замедление:** Phase 1 затянется на 1–2 недели.

#### 3. Vendor lock-in
- **LangGraph:** Tightly coupled с LangChain экосистемой. Хотя model-agnostic, но структуры данных (TypedDict, BaseMessage) — от LangChain.
- **Ваш Runtime:** Полностью независим. Можно перейти на CrewAI, AutoGen или свой runtime без переписывания.
- **Риск:** Если LangGraph устареет или изменит API — придётся рефакторить.

---

## 4. Рекомендация: как использовать LangGraph

### Стратегия: Adapter Pattern (ADR-005)

```
┌─────────────────────────────────────────┐
│  Voyage Framework                       │
│  ┌─────────────────────────────────┐   │
│  │  Agent Runtime (абстракция)     │   │
│  │  plan → execute → reflect → retry│   │
│  └────────────┬────────────────────┘   │
│               │                         │
│    ┌──────────┴──────────┐             │
│    ▼                     ▼             │
│ ┌─────────────┐    ┌─────────────┐    │
│ │ Custom      │    │ LangGraph   │    │
│ │ Runtime     │    │ Adapter     │    │
│ │ (Phase 1)   │    │ (Phase 2+)  │    │
│ └─────────────┘    └─────────────┘    │
│                                        │
│  ┌─────────────────────────────────┐   │
│  │  Core (models, tools, memory)   │   │
│  │  — НЕЗАВИСИМЫ от orchestrator   │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Правила:**
1. **Phase 1:** Custom Runtime. LangGraph НЕ используется.
2. **Phase 2:** LangGraph через `OrchestratorAdapter`. Custom Runtime остаётся fallback.
3. **Core (models, tools, memory, specs):** НЕЗАВИСИМЫ от orchestrator. Меняем только адаптер.
4. **EventEngine:** Остаётся primary source of truth. LangSmith — дополнительно.

### Когда включать LangGraph

| Критерий | Custom Runtime | LangGraph |
|----------|---------------|-----------|
| Простой цикл (4 состояния) | ✅ | ❌ overkill |
| Human approval на каждом шаге | ❌ | ✅ |
| Time-travel debugging | ❌ | ✅ |
| Sub-graph composition | ❌ | ✅ |
| Complex branching (>5 paths) | ❌ сложно | ✅ |
| Production observability | ❌ basic | ✅ LangSmith |
| Vendor independence | ✅ | ❌ lock-in |
| Learning curve | ✅ low | ❌ medium |

**Решение:**
- Phase 1: Custom Runtime (закрываем foundation).
- Phase 2: Добавляем LangGraph Adapter. НЕ удаляем Custom Runtime.
- Phase 2+: Если workflow становится сложным (>5 nodes, >3 conditional edges) → переключаемся на LangGraph. Если простой → остаёмся на Custom Runtime.

---

## 5. Сравнение с другими фреймворками

| Фреймворк | Подходит для Voyage? | Почему |
|-----------|---------------------|--------|
| **LangGraph** | ✅ Да (Phase 2+) | Graph-based, checkpointing, HITL, observability |
| **CrewAI** | ⚠️ Частично | Role-based, быстрый старт, но меньше контроля |
| **AutoGen/AG2** | ❌ Нет | Conversational, не подходит для deterministic workflow |
| **OpenAI Agents SDK** | ❌ Нет | Lock-in на OpenAI, model-agnostic — нет |
| **Claude Agent SDK** | ⚠️ Частично | Хорош для Anthropic, но меньше ecosystem |
| **Pydantic AI** | ❌ Нет | Type-safe, но нет orchestration |
| **Custom Runtime** | ✅ Да (Phase 1) | Полный контроль, нет зависимостей |

**Вывод:** LangGraph — лучший выбор для Phase 2+, но Custom Runtime должен остаться fallback.

---

## 6. План внедрения LangGraph

### Phase 1 (сейчас): Подготовка
- [ ] Создать `integrations/orchestrator_adapter.py` (stub).
- [ ] Определить интерфейс `OrchestratorAdapter` (ABC):
  ```python
  class OrchestratorAdapter(ABC):
      @abstractmethod
      async def run(self, workflow: Workflow, initial_state: AgentState) -> AgentState: ...
      @abstractmethod
      async def checkpoint(self) -> Checkpoint: ...
      @abstractmethod
      async def resume(self, checkpoint_id: str) -> AgentState: ...
  ```
- [ ] Реализовать `CustomRuntimeAdapter` (наш текущий runtime).
- [ ] НЕ устанавливать LangGraph.

### Phase 2 (Q3 2026): Внедрение
- [ ] Установить LangGraph: `pip install langgraph`.
- [ ] Реализовать `LangGraphAdapter` (наследует `OrchestratorAdapter`).
- [ ] Настроить checkpointing: `PostgresSaver` или `AsyncSqliteSaver`.
- [ ] Интегрировать human-in-the-loop с нашим Approval Flow.
- [ ] Настроить LangSmith (опционально).
- [ ] Переписать 1–2 workflow на LangGraph (feature, bugfix).
- [ ] Сравнить: Custom Runtime vs LangGraph на одной задаче.
- [ ] Оставить Custom Runtime как fallback.

### Phase 2+ (Q4 2026): Оптимизация
- [ ] Переписать оставшиеся workflow на LangGraph (если показало себя хорошо).
- [ ] Sub-graph composition для повторяющихся паттернов.
- [ ] Time-travel debugging для сложных багов.

---

## 7. Риски и митигация

| Риск | Вероятность | Влияние | Митигация |
|------|------------|---------|-----------|
| LangGraph API изменится | Средняя | Высокое | Adapter Pattern — меняем только адаптер |
| Learning curve затянет Phase 2 | Средняя | Среднее | Начинать с 1 workflow, не всё сразу |
| Performance хуже Custom Runtime | Низкая | Среднее | Бенчмарк перед полным переходом |
| Vendor lock-in | Средняя | Среднее | Core независим, Custom Runtime fallback |
| LangSmith платный | Низкая | Низкое | EventEngine primary, LangSmith optional |

---

## 8. Итоговое решение

**LangGraph — ДА, но через адаптер и только в Phase 2+.**

**Почему это ускорит разработку:**
1. Human-in-the-loop на уровне workflow → меньше ручного review.
2. Time-travel debugging → быстрее дебаг сложных workflow.
3. Sub-graph composition → переиспользование компонентов.
4. LangSmith observability → data-driven оптимизация.

**Почему НЕ сейчас:**
1. Phase 1 должен быть простым и быстрым.
2. Custom Runtime покрывает 100% нужд Phase 1.
3. Learning curve LangGraph — 1–2 недели, которые лучше потратить на foundation.

**Золотое правило:**
> "Framework'и нужны не для того, чтобы заменить вашу архитектуру, а для того, чтобы ускорить то, что вы уже делаете. Если Custom Runtime справляется — не меняй его. Если workflow становится сложным — LangGraph через адаптер."

---

**Версия документа:** 1.0  
**Дата:** 2026-05-26  
**Авторы:** AndreyVoyage (Human) + AI Architect  
**Следующее обновление:** После бенчмарка Custom Runtime vs LangGraph в Phase 2
