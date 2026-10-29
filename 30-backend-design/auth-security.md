---
type: topic
title: Аутентификация и безопасность
tier: 1
impact: 3
effort: 2
status: todo
confidence: 0
tags: [backend, security, tier1]
updated: 2026-10-29
---

# Аутентификация, авторизация, базовая безопасность

> **Зачем на собесе:** банки (Альфа, скрытый банк) и любой Senior-собес: «как устроена авторизация», «JWT или сессии», «какие уязвимости знаешь». Нужен уверенный базовый уровень, не специализация.

## Минимум для собеса

- **Аутентификация** (кто ты) vs **авторизация** (что тебе можно). Авторизация — на уровне ресурса (owner check), не только роли (IDOR — самая частая дыра).
- **Пароли:** хранить только хэш с солью — `argon2id` / `bcrypt`; никогда md5/sha256 без KDF. Лимит попыток, 2FA.
- **Сессии vs JWT:** сессия — состояние на сервере (Redis), легко отозвать; JWT — stateless, подпись (HS256/RS256), сложно отозвать → короткий access (5–15 мин) + refresh token с ротацией и хранением/отзывом. Не хранить в JWT чувствительные данные (payload только закодирован). Проверять `exp`, `aud`, `iss`, алгоритм (атака `alg: none`).
- **OAuth2 / OIDC:** authorization code + PKCE для входа через провайдера; client credentials для сервис-сервис. OIDC = OAuth2 + id_token. В FastAPI — `OAuth2PasswordBearer`, `Security`, scopes.
- **Межсервисная аутентификация:** mTLS / service mesh, или короткоживущие токены; никаких «общих секретов в коде».
- **OWASP Top 10, что уметь назвать:** инъекции (SQL — только параметризованные запросы; ORM защищает, `text()` с f-string — нет), broken access control (IDOR), сломанная аутентификация, SSRF (валидация URL, запрет внутренних адресов), небезопасная десериализация (`pickle` из недоверенного источника), XSS (экранирование, CSP — если отдаёте HTML), CSRF (для cookie-сессий — токен/SameSite), утечки через логи и ошибки.
- **Секреты:** env/secret manager (Vault, K8s Secrets с шифрованием), ротация, никогда в репозитории; `.env` в `.gitignore`; сканирование секретов в pre-commit/CI.
- **Транспорт:** TLS везде, HSTS, безопасные cookie (`HttpOnly`, `Secure`, `SameSite`).
- **Валидация входа** на границе (Pydantic), ограничение размера тела, таймауты, rate limiting ([[redis]]).
- **Зависимости:** `pip-audit`/dependabot, минимальный Docker-образ, не root в контейнере ([[docker]]).
- **Логи и PII:** не логировать токены, пароли, карты; маскирование.

## Вопросы с собесов

- [ ] JWT или сессии — что выбрать и почему? Как отозвать JWT? → *трейдоффы; короткий access + refresh + blacklist/ротация*
- [ ] Как хранишь пароли? → *argon2id/bcrypt с солью*
- [ ] Что такое IDOR и как защищаться? → *проверка владельца на каждом доступе к ресурсу*
- [ ] Защищает ли ORM от SQL-инъекций? Всегда? → *параметризация; text() с конкатенацией — нет*
- [ ] Как устроен OAuth2 authorization code flow? Что такое PKCE? → *редирект, код, обмен на токен; защита публичных клиентов*
- [ ] Как сервисы аутентифицируют друг друга? → *mTLS, токены, mesh*
- [ ] Что такое SSRF и где он возникает в бэкенде? → *сервер ходит по URL пользователя; метаданные облака*
- [ ] Где хранишь секреты и как они попадают в контейнер? → *secret manager → env/файл; не в образе*
- [ ] Какие заголовки/настройки cookie для безопасности знаешь? → *HttpOnly, Secure, SameSite, HSTS, CSP*

## Углубление

- Authorization models: RBAC vs ABAC vs ReBAC (Zanzibar/OpenFGA), policy engines (OPA, Casbin).
- Token binding, DPoP, refresh token rotation с обнаружением кражи.
- Шифрование на уровне приложения (envelope encryption, KMS), шифрование полей в БД.
- Threat modeling (STRIDE) для своего сервиса — упражнение на вечер.
- Supply chain: SBOM, подпись образов (cosign), pinning базовых образов.

## Ресурсы

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP Cheat Sheet Series (Authentication, Session Management, JWT)
- FastAPI Security tutorial: https://fastapi.tiangolo.com/tutorial/security/

## Связи

[[api-design]] · [[fastapi]] · [[redis]] · [[docker]] · [[observability]]

## Мои заметки

