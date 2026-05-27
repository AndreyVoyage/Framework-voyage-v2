# ROLE-002: Security Engineer (AppSec / SecOps)

**Status**: Planned  
**Date**: 2026-05-26  
**Authors**: AndreyVoyage (Human) + AI Architect  
**Phase**: Phase 2 (Soul & Connectors)  
**Replaces**: Нет  
**Related**: ROLE-001 (QA Engineer), ADR-005 (Project Isolation), TECH-001 (AppSec Stack)

---

## Purpose
Security Sandbox блокирует *опасные команды* (rm -rf, curl), но не аудитит *код* на уязвимости. Security Engineer закрывает этот пробел:
- Статический анализ кода на OWASP Top 10.
- Поиск hardcoded secrets и токенов.
- Аудит зависимостей: "3 критических CVE в пакетах".
- Проверка конфигурации: CORS, CSP, HTTPS, headers.
- Compliance с ADR-005 security measures (RAG Guard, PyPI Validation).
- Защита от "vibe coding" — аудит кода, сгенерированного AI.

**Почему сейчас:** Пользователь (AndreyVoyage) явно интересуется безопасностью: спрашивал про "типичные уязвимости", "хакерские атаки", предоставил детальный AppSec stack. Solo-разработчик склонен забывать о безопасности — Security Engineer делегирует это AI.

---

## Responsibilities

### В зоне ответственности

#### 1. SAST (Static Application Security Testing) — "белый ящик"
- Сканирование Python-кода: `bandit`, `semgrep`.
- Сканирование TypeScript/React: `eslint-security`, `sonarjs`.
- Поиск уязвимых паттернов: SQL-инъекции, XSS, CSRF, path traversal, eval/exec.
- Интеграция с CI/CD: блокировка merge при critical findings.

#### 2. DAST (Dynamic Application Security Testing) — "чёрный ящик"
- Сканирование работающего приложения: OWASP ZAP, Burp Suite (community edition).
- Проверка API endpoints на инъекции, авторизацию, rate limiting.
- Проверка Telegram WebApp: валидация `initData`, подписи HMAC.

#### 3. SCA (Software Composition Analysis)
- Аудит Python-зависимостей: `pip-audit`, `safety`.
- Аудит JS-зависимостей: `npm audit`, `yarn audit`.
- Проверка устаревших пакетов: "django 3.2 имеет CVE-2023-XXXX".

#### 4. Secrets Management
- Поиск hardcoded secrets: `gitleaks`, `trufflehog`.
- Проверка `.env` файлов: не закоммичены ли SECRET_KEY, BOT_TOKEN.
- Рекомендации по HashiCorp Vault / StarVault для production.

#### 5. Configuration Security
- Проверка CORS: "разрешены все домены — это риск".
- Проверка CSP (Content Security Policy) для React SPA.
- Проверка security headers: HSTS, X-Frame-Options, X-Content-Type-Options.
- Проверка SSL/TLS конфигурации на VPS.

#### 6. AI-Generated Code Audit
- Проверка кода, написанного Kimi Code / Kimi Agent, на скрытые уязвимости.
- Валидация: "AI сгенерировал `eval(user_input)` — блокировать".

### Вне зоны ответственности
- Не пишет production код (только security-скрипты и конфиги).
- Не деплоит WAF (это DevOps, но Security Engineer даёт рекомендации).
- Не делает penetration testing на уровне инфраструктуры (это внешний аудит).
- Не заменяет human judgment для critical security decisions.

---

## Tools

| Адаптер | Инструменты | Требует approval | Описание |
|---------|-------------|------------------|----------|
| **bandit.py** *(новый)* | `bandit_scan`, `bandit_report` | Нет | SAST для Python (OWASP, CWE) |
| **semgrep.py** *(новый)* | `semgrep_scan`, `semgrep_rules` | Нет | SAST multi-language (custom rules) |
| **zap.py** *(новый)* | `zap_spider`, `zap_scan`, `zap_report` | Нет | DAST (OWASP ZAP) |
| **sca.py** *(новый)* | `pip_audit`, `npm_audit`, `safety_check` | Нет | Аудит зависимостей |
| **secrets.py** *(новый)* | `gitleaks_scan`, `trufflehog_scan` | Нет | Поиск hardcoded secrets |
| **config_audit.py** *(новый)* | `check_cors`, `check_csp`, `check_headers` | Нет | Аудит конфигурации |
| **python.py** | `mypy`, `ruff_check` | Нет | Type safety (indirect security) |
| **files.py** | `cat_file`, `grep_code` | Нет | Чтение конфигов |

---

## Prompt Focus

**Примеры запросов к агенту:**
- "Просканируй `backend/app/bot/handlers.py` на OWASP Top 10."
- "Найди все hardcoded secrets в репозитории."
- "Проверь CORS конфигурацию FastAPI — разрешены ли все домены?"
- "Аудит зависимостей: есть ли критические CVE в `requirements.txt`?"
- "AI сгенерировал этот код — есть ли скрытые уязвимости?"
- "Проверь Telegram `initData` валидацию на HMAC-подпись."

**Acceptance Criteria для работы Security Engineer:**
- [ ] SAST: 0 critical, 0 high findings (или каждый задокументирован как accepted risk).
- [ ] Secrets: 0 hardcoded secrets в коде.
- [ ] SCA: 0 критических CVE в зависимостях.
- [ ] Config: CORS не разрешён для `*`, CSP настроен для React SPA.
- [ ] DAST: API endpoints защищены от инъекций (ZAP scan clean).

---

## Policy (что может / не может)

**Может:**
- Запускать все security-инструменты без approval.
- Читать любой код и конфиг для аудита.
- Писать security-скрипты в `tools/security_scripts/`.
- Блокировать merge при critical findings.
- Добавлять security-правила в `RULES.md` (например: "Никогда не используй `eval()`").

**Не может:**
- Писать production код (только security-скрипты, конфиги, правила).
- Менять архитектуру без approval Architect.
- Деплоить WAF или менять firewall rules (только рекомендации DevOps).
- Пропускать critical findings без human approval.

---

## Implementation Tasks (для Kimi Code)

- [ ] Создать `agents/roles/security.py`, наследующий `AgentRole`.
- [ ] Зарегистрировать в `RoleRegistry` как `"security"`.
- [ ] Создать адаптер `tools/adapters/bandit.py` (wrapper).
- [ ] Создать адаптер `tools/adapters/semgrep.py` (wrapper + custom rules).
- [ ] Создать адаптер `tools/adapters/zap.py` (OWASP ZAP API wrapper).
- [ ] Создать адаптер `tools/adapters/sca.py` (pip-audit, npm audit).
- [ ] Создать адаптер `tools/adapters/secrets.py` (gitleaks, trufflehog).
- [ ] Создать адаптер `tools/adapters/config_audit.py` (CORS, CSP, headers).
- [ ] Добавить workflow-определение `security_audit.yaml`:
  ```yaml
  nodes:
    - name: security_scan
      role: security
      tools: [bandit_scan, semgrep_scan, gitleaks_scan, pip_audit]
      transitions:
        - condition: success
          target: reviewer_check
        - condition: failure
          target: developer_fix_security
  ```
- [ ] Написать тесты: `test_agents_roles_security.py`.
- [ ] Обновить `security/policy.py` — добавить permissions для Security Engineer.
- [ ] Обновить `ROLE-INDEX.md` — статус изменён на Implemented.

---

## Notes

- **Project isolation (ADR-005):** Security-аудит выполняется в рамках одного `project_id`. Cross-project secrets scanning запрещён.
- **False positives:** Security Engineer должен уметь помечать finding как `accepted_risk` с обоснованием. Это логируется как `security_accepted_risk` event.
- **AI code audit:** При проверке кода, сгенерированного Kimi Code, Security Engineer должен использовать `causation_id` из `implementation_done` event для трассировки.
- **Telegram WebApp security:** Особое внимание к `initData` валидации (HMAC-SHA256) и `window.Telegram.WebApp` API — это критично для SkillTracer.
- **DevSecOps pipeline:** В Phase 2 Security Engineer интегрируется в CI/CD: каждый PR проходит `bandit + semgrep + gitleaks` до merge.

---

**Версия ROLE:** 1.0  
**Дата:** 2026-05-26
