# ADR-004: Dual-Location Strategy

## Статус
Accepted

## Контекст

Voyage Framework v4.0 создаётся для проекта SkillTracer, но должен быть reusable для других проектов.

Требования:
- Framework доступен внутри SkillTracer (для Kimi Code в VS Code)
- Framework может быть использован отдельно (для других проектов)
- Обновления framework'а не ломают SkillTracer
- SkillTracer может использовать фиксированную версию framework'а

Варианты:
- **A. Только внутри SkillTracer:** `tools/voyage_framework/` — просто, но не reusable
- **B. Только отдельный repo:** `AndreyVoyage/voyage-ai-dev-framework` — reusable, но сложнее интегрировать
- **C. Git submodule:** отдельный repo + submodule в SkillTracer — лучшее из обоих миров
- **D. PyPI package:** publish на PyPI — overkill для solo-dev

## Решение

**Dual-Location: отдельный репозиторий + git submodule в SkillTracer.**

Структура:
```
github.com/AndreyVoyage/voyage-ai-dev-framework/  ← upstream (primary)
    │
    └── git submodule → skilltracer/tools/voyage_framework/  ← working copy
```

Workflow:
1. Разработка ведётся в отдельном репозитории
2. В SkillTracer: `git submodule add https://github.com/AndreyVoyage/voyage-ai-dev-framework.git tools/voyage_framework`
3. Обновление: `git submodule update --remote`
4. SkillTracer может pin конкретный commit submodule'а

## Последствия

### Положительные
- Reusable: другие проекты могут использовать framework
- Version pinning: SkillTracer не сломается от случайного обновления
- Чистая история: commits framework'а отделены от commits SkillTracer
- Возможность open-source в будущем

### Отрицательные
- Двойное обслуживание: commit в upstream + update submodule
- `git submodule` требует дополнительных команд (`git submodule update --init`)
- Новые разработчики (если появятся) должны понимать submodule workflow

## Инструкция по настройке

```bash
# Внутри skilltracer/
git submodule add https://github.com/AndreyVoyage/voyage-ai-dev-framework.git tools/voyage_framework
git submodule update --init --recursive

# Обновление framework
cd tools/voyage_framework
git pull origin main
cd ../..
git add tools/voyage_framework
git commit -m "Update voyage-framework submodule"
```

## Альтернативы

- **Symlink:** `ln -s ../../voyage-ai-dev-framework tools/voyage_framework` — проще, но не работает на Windows без admin
- **Copy-paste:** копировать файлы вручную — быстро устареет
- **pip install -e:** editable install из отдельного repo — возможно, но submodule надёжнее

## Связанные ADR

- ADR-001 (PostgreSQL) — DSN может быть разным для SkillTracer и других проектов
- ADR-002 (ChromaDB) — persist directory может быть в разных местах

---
**Дата:** 2026-05-21 | **Автор:** Human (AndreyVoyage) + AI Architect
