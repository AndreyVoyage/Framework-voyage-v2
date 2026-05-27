# ROLE-000: Base AgentRole (ABC)

**Status**: Implemented
**Date**: 2026-05-21
**Authors**: AndreyVoyage (Human) + AI Architect
**Phase**: Phase 1 (Foundation)
**Replaces**: Нет
**Related**: Все ROLE-NNN

---

## Purpose

Базовый абстрактный класс для всех ролей AI-агентов в Voyage Framework. Определяет интерфейс, который должны реализовывать все конкретные роли.

---

## Interface

```python
from abc import ABC, abstractmethod
from typing import List, Any

class AgentRole(ABC):
    name: str
    allowed_tools: List[str]

    @abstractmethod
    async def plan(self, task: str) -> List[str]:
        """Построить план выполнения задачи."""
        pass

    @abstractmethod
    async def execute(self, step: str) -> Any:
        """Выполнить один шаг плана."""
        pass

    @abstractmethod
    async def reflect(self, result: Any) -> dict:
        """Оценить результат выполнения."""
        pass
```

---

## Base Lifecycle

```text
plan → execute → reflect → retry (если reflect показал проблемы)
```

---

## Implementation Notes

- Все роли наследуют `AgentRole`.
- `RoleRegistry` хранит mapping `name → class`.
- Runtime вызывает `plan()`, затем цикл `execute() → reflect()`.

---

**Версия ROLE:** 1.0
**Дата:** 2026-05-21
