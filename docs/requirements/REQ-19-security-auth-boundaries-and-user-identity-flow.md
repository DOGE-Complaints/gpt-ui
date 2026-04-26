# REQ-19: Security boundaries — server bearer vs user identity flow

> **Назначение:** зафиксировать границы двух разных security-плоскостей в GPT UI интеграции:
> 1) server/app bearer для Actions канала,  
> 2) user authorization flow и перенос user identity в intake payload.

**Версия:** 0.1 · 2026-04-21  
**Статус:** active draft  
**Связанные документы:** [REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md), [Gateway API Reference](../../../doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md)

---

## 1. Разделение security-слоёв (обязательно)

| Слой | Что это | Что не является этим слоем |
|------|---------|----------------------------|
| Server bearer authorization | Интеграционный секрет канала Actions/API | Пользовательская авторизация |
| User authorization and identity | Идентификация пользователя и передача `external_user_id` в доменный контракт | Канальная сервисная аутентификация |

Требование:
- Документация и инструкции не должны смешивать эти два слоя в одном утверждении.

---

## 2. Server bearer authorization (Actions/OpenAPI уровень)

### 2.1 Норматив

- Bearer для GPT Actions является app-level/server-level интеграционным секретом.
- Значение bearer задаётся на уровне конфигурации Actions и OpenAPI security.
- Runtime instructions не должны описывать bearer как user-auth механизм.

### 2.2 Практическое правило формулировок

- В instructions допускается только boundary-уточнение:
  - bearer = server/app integration key;
  - bearer != user identity / user OAuth claim.
- Подробные operator-нюансы значения секрета остаются в Actions/OpenAPI конфигурации, а не в conversational runtime-логике.

---

## 3. User authorization flow (demo redirect pattern)

### 3.1 Целевой demo-flow

Для demo допускается внешний redirect flow:
1. Из диалога пользователь переходит по ссылке на внешнюю mock IdP страницу (веб-приложение).
2. Mock IdP выполняет user-auth (демо-режим для фиксированного пользователя).
3. IdP возвращает redirect обратно в диалог с параметром user id.
4. Полученный user id маппится в payload story intake как `submitter.external_user_id`.

### 3.2 Контрактная привязка

Для `POST /intake/stories` (planned HTTP binding, existing contract):
- `submitter.external_user_id` — required, non-empty string.
- `submitter.identity_issuer` — optional.

Источник: `doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md` (раздел `POST /intake/stories`, identity linkage path).

### 3.3 Boundary-правило

- User identity не выводится из server bearer.
- User identity передаётся явно в payload уровня доменной истории.

---

## 4. Требования к документации и приемке

- Все документы Module 1/Module 2 должны явно различать:
  - server bearer channel auth;
  - user identity author linkage.
- В интеграционных проверках необходимо отдельно валидировать:
  1. корректность bearer-конфигурации канала;
  2. корректность маппинга user id в `submitter.external_user_id`.

---

## 5. Трассируемость

| Тема | Где зафиксировано |
|------|-------------------|
| Story intake envelope и handoff | [REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md) |
| Входной контракт intake (`submitter.*`) | [Gateway API Reference](../../../doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md) |
| M1 Actions bearer boundary | `GPT UI/instructions/api-orchestrator.md`, `GPT UI/instructions/issue-api-methods-reference.md` |

---

## 6. История версий

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.1 | 2026-04-21 | Первый выпуск: фиксированы границы server bearer vs user identity flow, добавлен demo redirect pattern для user id handoff. |
