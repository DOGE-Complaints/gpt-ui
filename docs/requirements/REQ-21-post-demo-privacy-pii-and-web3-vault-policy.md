# REQ-21 (post-demo): Privacy, PII handling, and protected web3 payload policy

> **Назначение:** зафиксировать **post-demo** продуктово-инженерные требования к обработке персональных/чувствительных данных в связке GPT → intake → downstream, с приоритетом **безопасности пользователей** и **минимизации рисков**, включая целевую модель **отдельного защищённого web3 JSON-слоя** с жёстким контролем доступа.  
> **Юридическая оговорка:** этот документ **не является** юридической консультацией. Перед production необходим внешний privacy/legal review под вашу юрисдикцию, роли контроллера/обработчика, и фактическую архитектуру хранения/ключей.

**Версия:** 0.1 · 2026-04-26  
**Статус:** **post-demo** (не является обязательным gate для текущего demo MVP; задаёт целевую рамку и критерии готовности к production-grade privacy).  
**Связанные документы:** [REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md) (PII minimization, `privacy`), [REQ-19](./REQ-19-security-auth-boundaries-and-user-identity-flow.md) (identity vs channel auth), [REQ-15](./REQ-15-working-assumptions.md), [REQ-16](./REQ-16-open-decisions.md)

**Фактический контрактный якорь (intake):** в `StoryIntakeRequest` блок `privacy` опционален; в OpenAPI для `contains_pii` / `redaction_requested` заданы defaults `false` (см. `doge-complaints-gateway/docs/runtime-docs/api-reference/openapi.yaml`, компонент `Privacy`).

---

## 1. Термины (рабочие определения для продукта)

| Термин | Смысл в этом REQ |
|--------|-------------------|
| **PII** | Данные, которые позволяют идентифицировать человека напрямую или через сочетание фактов (включая «косвенные» кейсы вроде уникального адреса/расписания/редких деталей). |
| **Sensitive non-PII** | Не обязательно идентифицирует человека, но повышает риск вреда (например, описание насилия, острый health-context, уязвимость несовершеннолетних) — обрабатывается как **high-risk narrative**, даже если формально «PII мало». |
| **`privacy.contains_pii` (wire)** | **Сигнал риска для приёмника**, не «юридическое заключение» и не доказательство отсутствия PII. |
| **Operational store** | Базы/логи/аналитика продукта с более широким доступом и более длинным жизненным циклом. |
| **Vault payload (web3 layer)** | Отдельный **минимальный** JSON-фрагмент с чувствительными данными/метаданными, который **не дублирует** весь narrative, хранится под **отдельной политикой доступа** и **строгими ACL** (см. §4). |

---

## 2. Стартовая юридически-консервативная позиция (product north star)

1. **User safety first:** лучше потерять часть «удобства аналитики», чем создать траекторию деанона/ретенции/утечки.  
2. **Data minimization by default:** не собирать PII «на всякий случай»; не просить идентификаторы, если они не нужны для цели.  
3. **Separation of evidence vs projection:** `narrative.original_text` остаётся **источником правды пользователя** (как в REQ-18/§19), а любые структурированные privacy-решения — **отдельный, проверяемый слой**.  
4. **No silent widening:** расширение полей intake / хранения чувствительных данных — только через **явный ADR + OpenAPI lockstep + privacy review**.  
5. **Honest semantics:** если продукт не может гарантировать качество PII-детекции, `contains_pii` не должен позиционироваться как «100% корректно».

---

## 3. Целевая модель после demo (двухконтурная)

### 3.1 Контур A — публичный/операционный слой (минимум чувствительного)

**Цель:** в операционном контуре хранить и обрабатывать в основном **де-идентифицированный** или **redacted** narrative + структурные сигналы.

**Требования:**

- **A1. Redaction-first:** если `privacy.redaction_requested=true` или safety pipeline требует редактуры, в operational store попадает **redacted** текст/ссылка на vault, а не «сырой» идентификатор.  
- **A2. Strict boolean wire policy:** если `privacy` передаётся, `contains_pii` и `redaction_requested` — строго boolean (как зафиксировано в runtime audit intake parser).  
- **A3. Conservative default:** если GPT/bridge **не уверены**, предпочтительнее трактовать кейс как **high-risk** (включить защитные меры), чем «успокоить» систему ложным `false`.

### 3.2 Контур B — vault (web3) слой (строгий ACL)

**Цель:** единственное место, где допускается хранение **минимально необходимого** «сырья»/ключей/доказательств целостности, если продукт решает, что это необходимо.

**Требования:**

- **B1. Minimal JSON:** vault payload содержит только поля из allowlist (например: `vault_ref`, `ciphertext`, `content_hash`, `schema_version`, `created_at`, `policy_version`, `access_class`, `expiry`, `key_id`, `pii_class`, `redaction_manifest_ref`).  
- **B2. No duplicate narrative:** vault **не** является второй копией всего `original_text`, если это можно избежать; предпочтительно хранить **ссылку/хэш** + минимальный фрагмент, необходимый для расследования/appeal (если такая функция есть).  
- **B3. Hard ACL:** доступ к vault — отдельные роли, отдельные ключи, отдельный audit; **запрет** «читать vault из админки по умолчанию».  
- **B4. Key governance:** отдельный процесс ротации ключей, emergency revoke, break-glass с двухфакторным контролем и пост-фактум аудитом.  
- **B5. Retention & deletion:** явные сроки хранения; право на удаление/забвение — как отдельный юридический workstream (не заглушать «web3 значит навсегда»).

---

## 4. Политика для `privacy.contains_pii` (post-demo, рекомендуемая)

Этот раздел **заменяет** «один радиобаттон в опросе» на формальное продуктовое правило.

### 4.1 Источник истины для флага

**Рекомендуемый порядок (строже к безопасности):**

1. **Safety/ingest pipeline signals** (например, `pii_detected[]`, redaction policy, high-risk minors/health/violence contexts) → формируют **privacy recommendation**.  
2. **GPT** выставляет wire-флаги **только** если это проходит правила bridge (не «угадайка» в обход safety).  
3. **Backend** валидирует согласованность: если `contains_pii=false`, но narrative содержит запрещённые паттерны/маркеры риска — **reject или принудительный redaction path** (продуктово выбрать один режим).

### 4.2 Правило для demo vs post-demo

- **Demo (как ты уже ответила `false`):** допустимо как **сознательное упрощение**, если параллельно есть политика «не вводить пользователя в состояние, где PII неизбежны» и есть safety stop-lines.  
- **Post-demo (этот REQ):** `false` **не является** «разрешением хранить всё подряд»; это лишь дефолт wire-поля. Истинная защита — **минимизация + redaction + vault**.

---

## 5. Требования к GPT instructions / orchestration (post-demo)

- **G1.** Явно различать: «пользователь сказал» vs «система вывела» (см. REQ-18 принцип разделения).  
- **G2.** Запрет на «добивание» идентификаторов ради заполнения полей.  
- **G3.** Если пользователь вставляет PII добровольно — предложить **redaction**, **не сохранять** лишние копии в operational артефактах.  
- **G4.** `privacy.redaction_requested` должен поддерживаться как пользовательский сигнал (explicit), а не только GPT inference.

---

## 6. Acceptance criteria (post-demo readiness)

1. **AC-1:** Опубликован allowlist полей для vault JSON и запрещённые поля (например, сырой payment data, passport scans — только если это вообще legal и отдельно approved).  
2. **AC-2:** Документирован **redaction path** от `narrative` до operational store + ссылка на vault ref.  
3. **AC-3:** Документирован **break-glass** и audit trail для доступа к vault.  
4. **AC-4:** Политика `contains_pii` согласована между GPT bridge и backend validation (нет противоречивых «всегда false» при явном PII).  
5. **AC-5:** Проведён внешний privacy/legal review (трекер задачи/ссылка).

---

## 7. Открытые вопросы (намеренно не закрываем в этом REQ)

- Юрисдикция и роль оператора как controller/processor.  
- Нужна ли **криптографическая** защита на уровне content (envelope encryption) vs только ACL.  
- Нужен ли пользователю UI для управления redaction/consent на web3 vault.  
- Как совмещается «immutability web3» с правом на удаление.

---

## 8. История версий

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.1 | 2026-04-26 | Первый выпуск post-demo privacy policy: PII semantics, dual-layer model, vault constraints, `privacy.contains_pii` governance, acceptance criteria. |
