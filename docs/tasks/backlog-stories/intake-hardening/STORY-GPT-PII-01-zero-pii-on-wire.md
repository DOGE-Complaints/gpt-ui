# STORY-GPT-PII-01 — Zero PII on wire (GPT-only MVP)

## Meta
- **Key:** `STORY-GPT-PII-01-zero-pii-on-wire`
- **Пакет:** [`intake-hardening/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** Critical — GAP-IH-04 (operator refine 2026-08-10)
- **Тип:** product requirement (privacy / GDPR-relevant wire)
- **Gap:** [`gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md`](../../../analysis/gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md) §4
- **Основание:** as-built [`story-pii-processing-before-send-as-built.md`](../../../runtime-docs/story-pii-processing-before-send-as-built.md); [REQ-23](../../../requirements/REQ-23-gpt-behavioral-extensions-pii-signals-consistency.md) Done (flags + ask/edit — **слабее** этого target); PII types [`story-normalizer.md`](../../../../instructions/story-normalizer.md) §4.4
- **Не путать с:** [REQ-21](../../../requirements/REQ-21-post-demo-privacy-pii-and-web3-vault-policy.md) Deferred (vault / post-demo)
- **MVP enforcement seam:** **только Custom GPT instructions** — gateway residual-PII reject / backend AI **не использовать** (дорого по токенам)
- **Pipeline / GIM:** GIM-240…245 (pkg-000035)

## Зачем простыми словами

Пользователь не должен никаким разумным путём через GPT протолкнуть в `POST /story-drafts` текст с ФИО, телефоном, email, адресом или ID. Сейчас REQ-23 позволяет отказ от редактуры и отправку сырого PII с флагами. На MVP весь контроль — на стороне GPT; серверный ИИ-детект не берём.

## Текущее состояние (verified)

| Элемент | Реальность |
|---------|------------|
| GPT detect + ask/edit | Instruction §4.4 + orchestrator §5.2.0 |
| Decline edit | §5.2.0 п.3 → всё равно шлёт текст + `redaction_requested=false` |
| Wire `privacy.*` | Optional flags; default false |
| Stash / stories | **Raw** `narrative.original_text` (+ i18n) |
| Server `redact_pii` | Только logs / embeddings if flag — **не** scrub persist |
| Backend AI PII reject | **В src нет**; на MVP **не планируем** |

## Требование / целевое состояние (MVP)

1. Перед **каждым** `postStoryDraftStash`: в wire-полях narrative (`original_text`, `title`/`description`/`summary` i18n и прочих text sidecars на том же draft body) **нет** типов normalizer §4.4 (person name, address street+number, phone, email, personal ID).
2. Если пользователь **отказывается** очистить PII → **STOP** (не вызывать Action). Ужесточение vs Done REQ-23 §1.3 decline→send.
3. Флагов `privacy.contains_pii` / `redaction_requested` **недостаточно** как замена scrub: либо текст очищен, либо отправки нет.
4. Enforcement MVP = GPT instruction modules only. Gateway **не** обязан детектировать residual PII (без backend AI / без оплаты токенов с сервера).

## Векторы проталкивания PII (investigated)

| # | Вектор | Почему дыра сейчас | MVP close? |
|---|--------|--------------------|------------|
| V1 | Decline redact + backend confirmation | §5.2.0 п.3 шлёт сырой PII | **Да** — STOP on decline |
| V2 | LLM miss → `contains_pii=false`, omit `privacy` | Нет второго контура | **Да** — conservative detect + pre-send re-scan all wire text fields |
| V3 | PII только в title / description / summary / одной локали | Scan может смотреть не все поля | **Да** — полный scan wire text |
| V4 | Re-inject PII после «очистки» перед confirm | Нет финального re-check | **Да** — re-scan immediately before HTTP |
| V5 | PII в `consistency_notes`, `location_query`, institution, free-text labels | Sidecars вне узкого scan | **Да** — включить sidecars |
| V6 | God Mode preview / debug path | «No redaction» в preview; риск слабой дисциплины | **Да** — HTTP всё равно zero-PII; preview ≠ send exemption |
| V7 | Minimal assent / «отправь как есть» | Давление обойти scrub | **Да** — не считать согласие на PII-send |
| V8 | Obfuscation (пробелы в телефоне, «собака» в email) | Модель может не пометить | **Да** — treat obfuscated identifiers as PII |
| V9 | Прямой HTTP `POST /story-drafts` в обход GPT | Вне GPT control | **Residual** — documented; citizen path = Actions only |
| V10 | SPA `submit` body `{}` | Не scrub; доверяет stash | **Residual** — зависит от содержимого stash (V1–V8) |

## Acceptance Criteria (requirements)

- [x] Instructions закрывают V1–V8: decline→STOP; full-field re-scan; no send with §4.4 PII present.
- [x] Evidence matrix: сценарии телефон/email/имя в `original_text` и в `description.ru` → Action **не** вызывается; civic story без PII → Action OK.
- [x] Documented residual: V9 (direct client), V10 (SPA trusts stash); REQ-21 vault остаётся Deferred.
- [x] Meta/Швы явно: **no backend AI PII reject on MVP** (cost).
- [x] Не регрессирует opaque submitter ids (не narrative PII).

## Границы

- **In scope:** GPT instructions (normalizer, orchestrator, safety, field lock); product outcome zero-PII on Action body для citizen GPT path.
- **Вне scope:** gateway LLM/heuristic PII reject; OpenAI chat transcript вне wire; identity phone HMAC; REQ-21 vault; redesign God Mode activation.
- **Не трогаем:** Done Status REQ-23 (исторический); as-built факт raw persist при обходе GPT.

## Швы

- GPT: `story-normalizer.md` §4.4, `api-orchestrator.md` §5.2.0 / admission, `safety-compliance.md` privacy, OpenAPI `Privacy` semantics (flags не заменяют scrub).
- Gateway: **MVP — не использовать** для residual PII AI; as-built `redact_pii` logs остаётся как есть.
- Related: GPT-MISSION-01 (standing ≠ PII), GPT-GUARD-01 (junk holes).
