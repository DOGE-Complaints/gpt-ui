# STORY-GPT-SUBMIT-02 — Полная зачистка legacy `POST /intake/stories` + `submitter` из GPT-контракта

> **Источник:** live-тест Custom GPT (2026-07-07) — модель всё ещё видит `submitter` в OpenAPI и conditional «omit on stash / include for service» в инструкциях; это создаёт путаницу и риск лишнего поля в stash-payload. Follow-up к [GPT-SUBMIT-01](./STORY-GPT-SUBMIT-01-redirect-handoff.md) (handoff ✅, зачистка неполная).

## Meta
- **Key:** `STORY-GPT-SUBMIT-02-legacy-intake-submitter-cleanup`
- **Пакет:** [`story-submit-handoff/`](./INDEX.md)
- **Status:** 🟢 Done (Verified) — static 8/8 + REG-05 · [audit](../../../analysis/audit-gpt-submit-02-legacy-cleanup-code-2026-07-07.md) · [re-audit](../../../analysis/audit-gpt-submit-02-reaudit-gap-closure-2026-07-07.md) 2026-07-07
- **Severity:** 🟠 MED (live GPT confusion; не блокирует stash, но ломает payload hygiene)
- **Приоритет:** P2 (следом за operator cutover GPT-SUBMIT-01; до широкого пилота)
- **Тип:** instruction + OpenAPI contract cleanup (trio lockstep + downstream grep)
- **Источник:** operator live-test report 2026-07-07; [re-audit #2](../../../analysis/audit-gpt-submit-01-redirect-handoff-reaudit2-2026-07-06.md) (runtime PASS, metadata/doc drift на service-path prose)
- **Зависит от:** [GPT-SUBMIT-01](./STORY-GPT-SUBMIT-01-redirect-handoff.md) 🟢 (stash+redirect primary path)
- **Связано:** [gpt-story-submit-handoff-operator-guide.md](../../../gpt-story-submit-handoff-operator-guide.md) · [interview reconciliation](../../../analysis/interview-gpt-submit-handoff-reconciliation-2026-07-06.md) N2 (omit `submitter`)

## Зачем простыми словами

GPT-SUBMIT-01 **переключил wire** на `POST /story-drafts`, но в Actions YAML и инструкциях остались **двойные контракты**: «для stash omit submitter, для service include submitter», таблицы `POST /intake/stories`, схемы `SuccessEnvelope_StoryIntake` (202 + `story_id`), §5.2.4.B service post-submit.

Для **Custom GPT** с единственной операцией `postStoryDraftStash` это **мертвое наследие**: модель читает OpenAPI → видит `Submitter` в `StoryIntakeRequest` → может слать поле, хотя автор определяется только в браузере через `/me`.

Цель стори — **один безусловный контракт stash**: без `submitter`, без упоминаний user-facing `POST /intake/stories` / `postStoryIntake` в hot-path инструкций и OpenAPI.

- **Pipeline:** [`STORY-M1-06-26-legacy-intake-submitter-cleanup`](../../../analysis/tasks/epics/EPIC-M1-06-orchestrator-openapi-web2/stories/STORY-M1-06-26-legacy-intake-submitter-cleanup/STORY-M1-06-26-legacy-intake-submitter-cleanup.md) (🟢 Done · pkg-000026 · GIM-201…206 · reaudit GIM-207…208)

## Что наблюдаю сейчас (verified — re-audit 2026-07-07)

- **OpenAPI v0.6.0:** [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml) — `StoryDraftStashRequest`; **нет** `Submitter`, `StoryIntakeResponseData`, `SuccessEnvelope_StoryIntake`; rg `submitter` → 0.
- **Trio lockstep:** [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md) v1.9; [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) v0.4.0 — stash-only §5.1/§5.2; §5.2.1 explicit «no `submitter` field»; §6.1 single 201 success; error example `session_language` (N3).
- **N1 rename (REG-05 ✅):** [`story-normalizer.md`](../../../../instructions/story-normalizer.md) v0.2.15 (GIM-207); [`story-data-model.md`](../../../../instructions/story-data-model.md) v0.14 L78 (GIM-208) — `StoryDraftStashRequest` в hot path; rg `StoryIntakeRequest` в instructions hot path → 0 (N2 version-history L115 api-methods — допустимо).
- **Scoped downstream:** [`story-lifecycle-instructions.md`](../../../../instructions/story-lifecycle-instructions.md) §2.2; [`ingest-validation.md`](../../../../instructions/ingest-validation.md) L55; [`gpt-story-submit-handoff-operator-guide.md`](../../../gpt-story-submit-handoff-operator-guide.md) — без service `/intake/stories` runtime prose.
- **Gate:** [`STORY-M1-06-26-acceptance-gate.md`](../../../analysis/tasks/epics/EPIC-M1-06-orchestrator-openapi-web2/stories/STORY-M1-06-26-legacy-intake-submitter-cleanup/STORY-M1-06-26-acceptance-gate.md) static PASS; N2 version-history policy.

## Что наблюдалось до реализации (baseline 2026-07-07 — pre-pkg-000026)

### OpenAPI [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml) v0.5.0

| Артефакт | Строки / факт | Проблема |
|----------|---------------|----------|
| `components.schemas.Submitter` | L54–65 | Существует; description ссылается на `POST /intake/stories` scripts |
| `StoryIntakeRequest.submitter` | L169–172 | `$ref: Submitter` — поле в requestBody `postStoryDraftStash` |
| `StoryIntakeResponseData` | L209–239 | Success schema для legacy intake (`story_id`, `status`) |
| `SuccessEnvelope_StoryIntake` | L241–247 | Не используется ни одним path в yaml |
| `paths./story-drafts.post` description | L277 | «Same service Bearer as **legacy intake**» |
| Единственный path | L271+ | Только `/story-drafts` — intake success schemas = мёртвый код |

### [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md) v1.8

| Место | Факт |
|-------|------|
| §1 parity | `StoryIntakeRequest (omit submitter on user path)` — conditional |
| §1.1 field table L30–32 | Три строки `submitter*` + «Service POST /intake/stories scripts may include submitter» |
| §1 intro L19 | Блок «Not in GPT Actions: postStoryIntake · POST /intake/stories — service-only» |
| Version history L111–119 | `postStoryIntake`, `/intake/stories` (история — см. открытые вопросы) |

### [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) v0.3.9

| Категория | Примеры (grep) |
|-----------|----------------|
| Endpoint tables §5.1, §5.2 | Строки `Submit Story (service-only)` · `POST /intake/stories` (L237, L252) |
| §5.2.1 field mapping | `submitter` + `submitter.external_user_id` + `submitter.identity_issuer` с колонкой Service-only (L350–352) |
| §5.2.3 / §5.2.4.B | Service-only legacy 202 flow (L601, L636–640) |
| §6.1 schemas | Два success response: stash **и** intake service channel (L708–729) |
| §6.1 error example | `"Missing or invalid root.submitter."` (L699) — без контекста «не для stash» |
| Header / checklist | `postStoryIntake` в forbidden lexicon, operator checklist (L31, L471, L764) |

### Downstream instructions (остаточные service-path фразы)

| Файл | Строка | Текст |
|------|--------|-------|
| [`story-lifecycle-instructions.md`](../../../../instructions/story-lifecycle-instructions.md) | L56 | «Service seed/sim may use POST /intake/stories» |
| [`ingest-validation.md`](../../../../instructions/ingest-validation.md) | L55 | «(service seed/sim only)» после negation |
| [`story-i18n-policy.md`](../../../../instructions/story-i18n-policy.md) | changelog L99 | историческая ссылка (hot path чистый) |
| [`story-interview-flow.md`](../../../../instructions/story-interview-flow.md) | changelog L360 | историческая ссылка |
| [`gpt-story-submit-handoff-operator-guide.md`](../../../gpt-story-submit-handoff-operator-guide.md) | L64, L200 | service-only пояснения |

### Вне scope hot-path (не трогать в этой стори без отдельного решения)

- **Gateway runtime:** `POST /intake/stories` **существует** на ноде (GW-DRAFT-04 service channel) — [`asgi_app.py`](../../../../../doge-complaints-gateway/src/core/api/asgi_app.py). Зачистка **GPT Actions artifact** не удаляет server route.
- **REQ-30/31/40/42** product headers — всё ещь содержат `postStoryIntake` / `POST /intake/stories` в названиях/примерах (requirements layer, не runtime instructions).

## Требование / целевое состояние

### Принцип: один контракт для GPT Actions

```
StoryDraftStashRequest (или очищенный StoryIntakeRequest)
  schema_version
  narrative
  origin
  privacy
  gpt_signals
  live_story_context
  (no submitter — ever, in GPT artifact)
```

### Целевые изменения (GPT UI only)

1. **OpenAPI v0.6.0** (version bump): удалить `Submitter`, `StoryIntakeResponseData`, `SuccessEnvelope_StoryIntake`; убрать `submitter` из request schema; переименовать request schema → `StoryDraftStashRequest` **или** оставить имя `StoryIntakeRequest` без поля (решение N1).
2. **story-api-methods-reference.md**: единственная операция; field table **без** строк `submitter*`; без блока «service POST /intake/stories».
3. **api-orchestrator.md**: §5.1/§5.2 tables — только `/story-drafts`; §5.2.1 mapping — без submitter rows; удалить §5.2.4.B service intake templates и §6.1 intake success JSON; одна success schema (201 `draft_id`); negation «do not send submitter» → **schema has no submitter field**.
4. **Downstream grep:** `story-lifecycle`, `ingest-validation`, operator guide — убрать service-path parentheticals из **runtime** prose (changelog history — по N2).
5. **Lockstep + operator:** bump versions; re-import Actions; acceptance gate extended grep.

### Поведение после зачистки

| Актор | Поведение |
|-------|-----------|
| Custom GPT | Видит только stash schema **без** submitter → не может «легально» добавить поле из OpenAPI |
| Browser submit | Автор = `sub` из identity `/me` (без изменений, GW-DRAFT-02) |
| Gateway service scripts | `POST /intake/stories` остаётся на ноде; документация — **gateway runtime-docs**, не GPT Actions yaml |

## Граница и контракт

- ✅ **В scope:** [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md), [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md), [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml), downstream instruction grep (lifecycle, ingest, i18n hot path), [`gpt-story-submit-handoff-operator-guide.md`](../../../gpt-story-submit-handoff-operator-guide.md), story acceptance gate / grep guards.
- ❌ **Не gateway:** не удалять `POST /intake/stories` route, не менять `contracts.py` на ноде.
- ❌ **Не REQ-* bulk rewrite:** REQ-30/31/40/42 — emergent follow-up (E2), не блокер этой стори.
- ❌ **Не git commit / operator re-import:** отдельный operator step после merge (как GAP-OP-01).

## Подзадачи (черновик)

- **T01 — OpenAPI schema purge:** удалить `Submitter`, intake success envelopes, `submitter` property; bump `info.version` 0.6.0; обновить `postStoryDraftStash` requestBody `$ref`; убрать «legacy intake» из description.
- **T02 — Request schema naming (N1):** либо rename `StoryIntakeRequest` → `StoryDraftStashRequest`, либо оставить имя без submitter — lockstep в api-methods + orchestrator §6.
- **T03 — api-orchestrator purge:** service-only endpoint rows; §5.2.1 submitter mapping rows; §5.2.4.B + service 202 blocks; §6.1 dual success; simplify error examples (stash-relevant only).
- **T04 — story-api-methods-reference purge:** field table, §1 intro, conditional omit language → unconditional «no submitter in Actions contract».
- **T05 — Downstream propagation:** lifecycle §2.2, ingest-validation L55, operator guide — runtime prose без `/intake/stories` service qualifiers.
- **T06 — Grep gate:** расширить acceptance gate / live grep — `submitter` в OpenAPI components = 0; `POST /intake/stories` в `instructions/` hot path = 0 (changelog lines — policy N2).
- **T07 — Version history policy (N2):** сжать или оставить historical rows в §version tables — зафиксировать в gate.
- **T08 — Live replay:** operator smoke — stash payload **без** `submitter` в God Mode preview и в фактическом Action call.

## Acceptance Criteria

- [x] OpenAPI Actions artifact: **нет** `Submitter` schema; **нет** `StoryIntakeResponseData` / `SuccessEnvelope_StoryIntake`; requestBody **не** содержит `submitter`; `info.version` 0.6.0; only `/story-drafts`. — GIM-201 · [audit](../../../analysis/audit-gpt-submit-02-legacy-cleanup-code-2026-07-07.md)
- [x] `story-api-methods-reference.md` lockstep: no conditional omit; no `submitter*` rows; no service intake §1 prose. — GIM-202
- [x] `api-orchestrator.md`: stash-only tables; no submitter §5.2.1; no §5.2.4.B; single §6.1 success; explicit no submitter in schema. — GIM-203
- [x] Downstream lifecycle §2.2 + ingest L55 — no service `/intake/stories` parentheticals. — GIM-204 (scoped)
- [x] Live grep `submitter` hot path = negation only (N2). — GIM-205
- [x] Live grep `POST /intake/stories` outside version-history = 0 (N2). — GIM-205
- [x] Operator guide: single stash contract. — GIM-204
- [x] Version lockstep + re-import noted. — GIM-206

## Открытые вопросы

- **N1 — Имя request schema:** `StoryDraftStashRequest` — **закрыто** (GIM-201 trio + GIM-207…208 normalizer/data-model; REG-05 ✅ · [re-audit](../../../analysis/audit-gpt-submit-02-reaudit-gap-closure-2026-07-07.md) §1).
- **N2 — Version history:** **решено** — audit trail в §version tables; hot-path grep excludes (GIM-205 gate).
- **N3 — Error example `root.submitter`:** **закрыто** — §6.1 использует `narrative.session_language` ([`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) L641).
- **N4 — Gateway cross-doc:** нужна ли ссылка в GPT docs «service intake → см. gateway server-env-quickstart» одной строкой вместо удалённых блоков?

## Эмерджентные запросы (cross-app)

| # | Запрос | Кому | Sev |
|---|--------|------|-----|
| **E1** | REQ-30/31/40/42 headers — устаревшие `postStoryIntake` в названиях/requirements prose | GPT requirements hygiene | 🟡 |
| **E2** | Gateway runtime-docs: явно отделить «GPT Actions contract» от «service POST /intake/stories scripts» | gateway docs | 🟡 |

## Швы (lockstep — обновить все вместе)

- [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml)
- [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md)
- [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) §5.1–§5.2, §5.2.1, §5.2.3–§5.2.4.A, §6.1
- Downstream: [`story-lifecycle-instructions.md`](../../../../instructions/story-lifecycle-instructions.md), [`ingest-validation.md`](../../../../instructions/ingest-validation.md)
- Operator: [`gpt-story-submit-handoff-operator-guide.md`](../../../gpt-story-submit-handoff-operator-guide.md)
- Gate: расширение [`STORY-M1-06-25-acceptance-gate.md`](../../../analysis/tasks/epics/EPIC-M1-06-orchestrator-openapi-web2/stories/STORY-M1-06-25-submit-redirect-handoff/STORY-M1-06-25-acceptance-gate.md) или новый gate row в follow-up pkg

## Проверка после реализации (команды)

```bash
# OpenAPI — no dead intake success schemas
rg -n 'Submitter|SuccessEnvelope_StoryIntake|StoryIntakeResponseData' \
  "GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml"

# Request body must not mention submitter
rg -n 'submitter' "GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml"

# Instructions hot path (tune per N2 changelog policy)
rg -n 'submitter|POST /intake/stories|postStoryIntake' "GPT UI/instructions/" \
  | rg -v 'version|changelog|0\.3\.|0\.2\.|history'
```
