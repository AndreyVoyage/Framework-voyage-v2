# TECH-001: AppSec Stack — Технологический реестр безопасности

> **Назначение:** Реестр инструментов AppSec, которые Security Engineer (ROLE-002) использует для аудита.  
> **Правило:** Инструмент не используется в framework, пока не внесён в TECH-001 и не создан адаптер в `tools/adapters/`.  
> **Обновляет:** Security Engineer Agent + Human при внедрении нового инструмента.

---

## Как пользоваться этим файлом

### Для Human (AndreyVoyage)
1. **При выборе инструмента** смотри TECH-001: уже ли инструмент в списке? Какой статус?
2. **При добавлении инструмента** создай раздел в этом файле или отдельный `TECH-NNN` при масштабировании.
3. **Приоритет:** Phase 2 использует только инструменты со статусом `Ready`. `Research` — для будущего.

### Для Kimi Code / Kimi Agent
1. **На вход** получай `CONTEXT.json["tech_stack"]` — список инструментов для текущей задачи.
2. **Не предлагай** инструменты в статусе `Research` для production кода.

---

## Сводная таблица инструментов

| Категория | Инструмент | Статус | Phase | Роль | Адаптер | Описание |
|-----------|-----------|--------|-------|------|---------|----------|
| **SAST** | bandit | Ready | Phase 2 | Security | `bandit.py` | Python security linter (PyCQA) |
| **SAST** | semgrep | Ready | Phase 2 | Security | `semgrep.py` | Multi-language SAST с custom rules |
| **DAST** | OWASP ZAP | Ready | Phase 2 | Security | `zap.py` | Web app scanner, API testing |
| **DAST** | Burp Suite | Research | Phase 2+ | Security | — | Enterprise DAST (платный) |
| **SCA** | pip-audit | Ready | Phase 2 | Security | `sca.py` | Аудит Python-зависимостей (PyPA) |
| **SCA** | npm audit | Ready | Phase 2 | Security | `sca.py` | Встроенный аудит npm |
| **SCA** | Snyk | Research | Phase 2+ | Security | — | Cloud SCA (требует API key) |
| **SCA** | OWASP Dependency-Check | Research | Phase 2+ | Security | — | Универсальный SCA |
| **Secrets** | gitleaks | Ready | Phase 2 | Security | `secrets.py` | Поиск secrets в git history |
| **Secrets** | trufflehog | Ready | Phase 2 | Security | `secrets.py` | Поиск secrets + entropy analysis |
| **Secrets** | HashiCorp Vault | Research | Phase 2+ | DevOps | — | Хранилище секретов (production) |
| **Config** | custom scripts | Ready | Phase 2 | Security | `config_audit.py` | CORS, CSP, headers checker |
| **CI/CD** | GitLab CI | Research | Phase 2+ | DevOps | — | Pipeline automation |
| **CI/CD** | DefectDojo | Research | Phase 2+ | Security | — | Управление findings |
| **Cloud** | Cloudflare WAF | Research | Phase 2+ | DevOps | — | WAAP, DDoS protection |
| **Container** | Trivy | Research | Phase 2+ | DevOps | — | Сканирование Docker-образов |
| **Container** | Falco | Research | Phase 2+ | DevOps | — | Runtime security для Kubernetes |
| **AI Security** | ImmuniWeb | Research | Phase 2+ | Security | — | AI-помощник для аудита |

---

## Детальное описание по категориям

### 1. SAST (Static Application Security Testing)

#### bandit
- **Установка:** `pip install bandit`
- **Запуск:** `bandit -r backend/ -f json -o bandit-report.json`
- **Что находит:** hardcoded passwords, eval/exec, SQL-инъекции (по паттернам), weak crypto, assert в production.
- **Интеграция:** Адаптер `bandit.py` парсит JSON-отчёт и генерирует `error_logged` events для Self-Improving Engine.
- **Кастомизация:** `.bandit` файл с исключениями (например, `assert` в тестах — OK).

#### semgrep
- **Установка:** `pip install semgrep`
- **Запуск:** `semgrep --config=auto --json backend/ frontend/`
- **Что находит:** OWASP Top 10, CWE, custom rules (например: "не используй `requests.get` без timeout").
- **Кастомные правила:** Хранятся в `tools/security_rules/semgrep/`. Примеры:
  ```yaml
  rules:
    - id: no-eval-in-bot
      pattern: eval(...)
      languages: [python]
      message: "eval() запрещён в коде бота"
      severity: ERROR
  ```

### 2. DAST (Dynamic Application Security Testing)

#### OWASP ZAP
- **Установка:** Docker `owasp/zap2docker-stable`
- **Запуск:** `zap-baseline.py -t http://localhost:8000`
- **Что находит:** XSS, SQL-инъекции в API, небезопасные headers, открытые директории.
- **API scanning:** ZAP может сканировать OpenAPI спецификацию FastAPI (`/openapi.json`).
- **Ограничение:** Требует запущенного приложения. Не используется в unit-тестах, только в CI/CD или staging.

### 3. SCA (Software Composition Analysis)

#### pip-audit
- **Установка:** `pip install pip-audit`
- **Запуск:** `pip-audit -r requirements.txt --format=json`
- **Что находит:** CVE в PyPI-пакетах, устаревшие версии.
- **Интеграция:** Адаптер `sca.py` вызывает `pip-audit` и сравнивает findings с whitelist (некоторые CVE могут быть accepted risk).

#### npm audit
- **Встроено:** `npm audit --json`
- **Что находит:** CVE в JS-зависимостях.

### 4. Secrets Scanning

#### gitleaks
- **Установка:** `brew install gitleaks` или Docker
- **Запуск:** `gitleaks detect --source . --verbose --redact`
- **Что находит:** AWS keys, GitHub tokens, Telegram bot tokens, пароли в коде.
- **История:** Сканирует весь git history, не только текущий коммит.

#### trufflehog
- **Установка:** `pip install trufflehog`
- **Запуск:** `trufflehog filesystem .`
- **Что находит:** То же + entropy analysis (случайные строки, похожие на ключи).
- **Преимущество:** Меньше false positives на случайных строках.

### 5. Configuration Security

#### CORS Checker
- **Проверка:** `Access-Control-Allow-Origin: *` → BLOCKED.
- **Правило:** Для SkillTracer разрешены только `*.skilltracer.art-artel.su` и `https://t.me`.

#### CSP Checker
- **Проверка:** `Content-Security-Policy` header для React SPA.
- **Правило:** `script-src 'self'` + явный hash для inline scripts.

#### Security Headers
- **Проверка:** HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy.

### 6. DevSecOps Pipeline (Phase 2+)

```
Developer пушит код
    │
    ▼
┌─────────────────┐
│  CI/CD Pipeline │
│  (GitHub Actions│
│   или GitLab CI)│
├─────────────────┤
│ 1. bandit_scan  │
│ 2. semgrep_scan │
│ 3. gitleaks_scan│
│ 4. pip_audit    │
│ 5. npm_audit    │
│ 6. pytest       │ ← QA Engineer
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
 Clean    Finding
    │         │
    ▼         ▼
 Merge   Security Engineer
          анализирует → human approval
```

### 7. AI Security

#### Защита от "vibe coding"
- **Проблема:** Kimi Code может сгенерировать `eval(user_input)` или `subprocess.call(user_input)`.
- **Решение:** Security Engineer сканирует ВЕСЬ код, сгенерированный AI, перед merge.
- **Правило:** Любой код с `eval`, `exec`, `subprocess`, `os.system` требует human approval, даже если написан AI.

---

## Чек-лист внедрения инструмента

Когда добавляешь новый инструмент (например, Trivy для Docker):

- [ ] Добавить инструмент в таблицу TECH-001 (статус: Ready или Research).
- [ ] Создать адаптер в `tools/adapters/` (если Ready).
- [ ] Написать тесты для адаптера.
- [ ] Обновить Security Engineer ROLE — добавить инструмент в список Tools.
- [ ] Обновить CI/CD pipeline (если применимо).
- [ ] НЕ трогать мастер-документ — только TECH-001 и ROLE-002.

---

**Версия TECH:** 1.0  
**Дата:** 2026-05-26  
**Следующее обновление:** После внедрения первого инструмента из списка (bandit/semgrep)
