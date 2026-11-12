# STORY-GPT-SUBMIT-01 — Редирект-handoff вместо прямого POST истории

> **Углублено до build-стандарта (2026-07-06):** решения интервью [`interview-gpt-submit-handoff-reconciliation-2026-07-06.md`](../../../analysis/interview-gpt-submit-handoff-reconciliation-2026-07-06.md) (D-SUBMIT-1…6); состыковка с последствиями ID-12 [`consequences-id12-...`](../../../../../spa-app/docs/analysis/consequences-id12-gpt-module-and-emergent-requests-2026-07-04.md).

## Meta
- **Key:** `STORY-GPT-SUBMIT-01-redirect-handoff`
- **Пакет:** [`story-submit-handoff/`](./INDEX.md)
- **Status:** 🟢 Done (Verified) — runtime + doc-sync 2026-07-06 · [re-audit #2](../../../analysis/audit-gpt-submit-01-redirect-handoff-reaudit2-2026-07-06.md)
- **Severity:** 🔴 HIGH (GPT-сторона M-3)
- **Приоритет:** P1 (критический путь MVP: spa ID-12 → **GPT-SUBMIT-01** → identity IDS-DOC-DRAFT-05)
- **Тип:** instruction update (`api-orchestrator.md` §5.2.x репойнт + OpenAPI-artifact)
- **Источник:** [`mvp-integration-plan-2026-07-02.md` §2](../../../../../docs/analysis/mvp-integration-plan-2026-07-02.md)
- **Основание (решения):** [`interview-gpt-submit-handoff-reconciliation-2026-07-06.md`](../../../analysis/interview-gpt-submit-handoff-reconciliation-2026-07-06.md) (D-SUBMIT-1…6)
- **Актуализация 2026-07-03/05:** под **реальный** gateway (стеш-роут реализован) + отгруженную ID-12 (приёмник переименован) — см. [`summary-story-draft-handoff-...`](../../../../../doge-complaints-gateway/docs/analysis/summary-story-draft-handoff-code-changes-2026-07-03.md), $doc1 §1a.
- **Зависит от:** GW-DRAFT-01 (стеш готов ✅), [SPA-ID-12](../../../../../spa-app/docs/tasks/backlog-stories/identity-auth/STORY-SPA-ID-12-story-draft-handoff-submit.md) (приём — отгружено ✅)
- **Pipeline:** [`STORY-M1-06-25-submit-redirect-handoff`](../../../analysis/tasks/epics/EPIC-M1-06-orchestrator-openapi-web2/stories/STORY-M1-06-25-submit-redirect-handoff/STORY-M1-06-25-submit-redirect-handoff.md) (🟢 Done · pkg-000024+025 · GIM-186…200)

## Зачем простыми словами
GPT больше **не публикует историю сам**. Он собирает тот же готовый JSON (`StoryIntakeRequest`), **той же авторизацией** кладёт его в стеш gateway и **редиректит юзера** на наш веб с `draft_id`. Финальный сабмит под пользовательской Supabase-сессией делает **браузер** (identity `/me` + phone-гейт). Меняется **только endpoint** и хвост потока — вся качественная GPT-машинерия подготовки payload остаётся.

## Что наблюдаю сейчас (verified по коду — post SUBMIT-02, 2026-07-09)
- **User submit = stash+redirect:** `api-orchestrator.md` v0.4.0 §5.2 — `postStoryDraftStash` → `POST /story-drafts`; auth = `GptActionsBearer` (`GPT_ACTIONS_BEARER_SECRET`); §5.2.B redirect `{SPA_BASE}/#/story/submit?draft_id=<id>` ([api-orchestrator.md §5.2](../../../../instructions/api-orchestrator.md)).
- **§5.2.x-машинерия guard'ит stash-payload:** §5.2.0a admission gate (REQ-30), §5.2.0 PII (REQ-23), §5.2.0b dual-mode preview (REQ-31), §5.2.2 pre-flight items 1–17 incl. `session_language` (REQ-40), §5.2.3 `{draft_id}`/201, §5.2.4.A redirect-копи (REQ-42) ([api-orchestrator.md §5.2.x](../../../../instructions/api-orchestrator.md)).
- **OpenAPI-artifact v0.6.0:** [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml) — **единственный** path `POST /story-drafts` (`operationId: postStoryDraftStash`); no `Submitter`. Lockstep: [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md) v1.9.
- **Downstream propagation (REG-01…04):** `story-interview-flow.md` §7.5 L209 stash+redirect; `ingest-validation.md` L55; `story-lifecycle-instructions.md` §2.2 L56; `story-i18n-policy.md` §6.1/6.2 L65, L83.
- **Gateway (готово):** `POST /story-drafts` (стеш) + `session_language` validation ([contracts.py L241–247](../../../../../doge-complaints-gateway/src/core/intake/contracts.py#L241)); `/intake/stories` = service-only (seed/sim), не user-путь.
- **Приёмник SPA (ID-12):** `STORY_SUBMIT_PATH = '/story/submit'` ([storyHandoffFlowState.js:14](../../../../../spa-app/src/auth/storyHandoffFlowState.js#L14)); `draft_id` из hash-query ([StorySubmitPage.jsx:49-59](../../../../../spa-app/src/pages/StorySubmitPage.jsx#L49)).
- **Operator pending:** Actions re-import v0.6.0 + privacy policy URL (GAP-OP-03); `SPA_BASE` в Custom GPT Instructions. **Гайд:** [`gpt-story-submit-handoff-operator-guide.md`](../../../docs/gpt-story-submit-handoff-operator-guide.md) (header drift REG-07).
- **Follow-up closed:** [GPT-SUBMIT-02](./STORY-GPT-SUBMIT-02-legacy-intake-submitter-cleanup.md) — legacy dual-contract зачищен (OpenAPI v0.6.0 + instructions lockstep).

## Требование / целевое состояние (по D-SUBMIT-1…6)
- **Единственный user-submit-флоу:** GPT собирает `StoryIntakeRequest` → **вся §5.2.x-машинерия валидирует/превьюит его** → стеш `POST /story-drafts` (той же service-Bearer) → `{draft_id}` → **редирект** на `{SPA_BASE}/#/story/submit?draft_id=<id>`. Дальше сабмитит браузер; GPT не участвует.
- **Репойнт, не снос (D-SUBMIT-1):** §5.2.0a/§5.2.0/§5.2.0b/§5.2.2 сохраняются как guard'ы **stash-payload**; §5.2.1 transform без изменений (тот же payload).
- **Хвост (D-SUBMIT-2):** §5.2.3 — стеш возвращает `{draft_id}`, не 202; §5.2.4 (post-submit) для user-пути → **redirect-копи** (момент «что дальше» — до редиректа), `cognitive_style`-адаптация наследуется; §5.2.4 остаётся **doc-only** для service seed/sim (N1).
- **`session_language` (D-SUBMIT-4):** обязателен в stash-payload — GPT-инструкция (AC + §5.2.2 pre-flight check) + gateway-валидация (cross-ref E3).
- **OpenAPI (D-SUBMIT-3):** добавить `POST /story-drafts`, убрать `postStoryIntake` из Actions-artifact, переимпорт.
- **Копи (D-SUBMIT-5):** терминальная — GPT не заявляет «история создана», не ждёт результата; 202 у браузера.
- **`/intake/stories` (F старого $doc2):** пометить service-only (seed/sim), не user-путь.

## Граница и контракт
- ✅ **GPT-сторона:** `api-orchestrator.md` §5.2.x + `story-api-methods-reference.md` + OpenAPI-artifact (operationId/method/path/auth в lockstep).
- ❌ **Не gateway/SPA:** стеш-роут, read-DTO, submit-гейт, phone-verify — реализованы (GW-DRAFT-01…04) / SPA (ID-12). Здесь GPT лишь **вызывает** стеш и **редиректит**.
- **`submitter` (N2):** автор = `sub` из `/me` на сабмите (confused-deputy защита, V6) — GPT **не слать** `submitter.external_user_id` в стеше (gateway всё равно перезапишет; чистит контракт).
- **Auth-value (N4):** совпадение `GPT_ACTIONS_BEARER_SECRET` ↔ gateway `SERVICE_API_TOKEN` — **deploy-конфиг** ([`railpack.json`](../../../../../doge-complaints-gateway/railpack.json) + Railway env; GW-DEPLOY-01 [retired](../../../../../doge-complaints-gateway/docs/analysis/retire-STORY-GW-DEPLOY-01-2026-07-10.md)), не эта стори.
- **`SPA_BASE`:** внешний адрес SPA — конфиг на стороне GPT/деплоя (в SPA его нет), см. E4.

## Подзадачи (черновик)
- **T01 — Маршрут redirect (K1):** §B/§C редирект-URL → `{SPA_BASE}/#/story/submit?draft_id=<id>` (HashRouter; менять только сегмент пути; канон [storyHandoffFlowState.js:14](../../../../../spa-app/src/auth/storyHandoffFlowState.js#L14)). Убрать `/story/compose`.
- **T02 — Репойнт wire (D-SUBMIT-1):** в §5.2.x заменить целевой вызов `POST /intake/stories` → `POST /story-drafts` (стеш, service-Bearer, тело = тот же `StoryIntakeRequest`, ответ `{draft_id}`). §5.2.1 transform не трогать. Явно зафиксировать: §5.2.0a/§5.2.0/§5.2.0b/§5.2.2 теперь guard'ят stash-payload.
- **T03 — Хвост §5.2.3/§5.2.4 (D-SUBMIT-2):** §5.2.3 — обработка `{draft_id}` (не 202); §5.2.4 user-путь → redirect-копи с наследованием `cognitive_style`; пометить §5.2.4 «service seed/sim = doc-only» (N1). Cross-ref REQ-42/41/40 (что осталось на стеше vs что переработано).
- **T04 — `session_language` обязателен (D-SUBMIT-4):** AC + новый §5.2.2 pre-flight item «`narrative_session_language` ∈ {en,et,ru} обязателен для stash» + i18n-поля `narrative_{title,summary,description,institution}{en,et,ru}` (V5); cross-ref E3 (gateway-валидация).
- **T05 — OpenAPI-artifact (D-SUBMIT-3):** добавить `POST /story-drafts` (securityScheme Bearer=service; requestBody = полный `StoryIntakeRequest`, `session_language` required; response `{draft_id}`); убрать `postStoryIntake`; version-bump + lockstep с `story-api-methods-reference.md`; переимпорт Actions.
- **T06 — Копи + payload-гигиена (D-SUBMIT-5, N2):** терминальная redirect-копи (не «создана», не ждать результат, продолжить на сайте); `submitter` не слать; one-shot/TTL-предупреждение — опционально (N3, Открытые вопросы).
- **T07 — `/intake/stories` пометка:** в инструкциях пометить service-only (seed/sim), запрет user-историй туда.
- **T08 — story acceptance gate.**

## Acceptance Criteria
- [x] `api-orchestrator.md` описывает **стеш(`POST /story-drafts`)+redirect** как единственный submit-флоу пользовательской истории; §5.2.x-машинерия (admission/PII/preview/pre-flight) сохранена и guard'ит stash-payload. — GIM-187 · [re-audit #2](../../../analysis/audit-gpt-submit-01-redirect-handoff-reaudit2-2026-07-06.md)
- [x] Redirect-URL = `{SPA_BASE}/#/story/submit?draft_id=<id>` (не `/story/compose`). — GIM-186
- [x] `session_language` (+ i18n-поля) **обязателен** в stash-payload: AC + §5.2.2 pre-flight item 17. — GIM-189; gateway E3 ✅
- [x] OpenAPI-artifact содержит `POST /story-drafts` (Bearer=service, `{draft_id}`); `postStoryIntake` убран из Actions paths; lockstep с `story-api-methods-reference.md`. — GIM-190 (⏳ Actions re-import — GAP-OP-01 operator)
- [x] §5.2.3 обрабатывает `{draft_id}` (не 202); §5.2.4 user-путь = redirect-копи (наследует `cognitive_style`); service seed/sim помечен doc-only. — GIM-188
- [x] GPT **не** утверждает «история создана» — терминальная копи; `submitter` не слать (автор = `sub` из /me). — GIM-191
- [x] `/intake/stories` помечен service-only (seed/sim), не user-путь. — GIM-192 + REG-02…04
- [x] Ссылки: GW-DRAFT-01 (стеш), SPA-ID-12 (приём), $doc1 §1a (контракт), парные REQ-40/41/42, эмердженты E1–E5. — GIM-193 gate 8/8

## Открытые вопросы
- **N1 — §5.2.4 service seed/sim = doc-only:** перенесено в [GPT-SUBMIT-02](./STORY-GPT-SUBMIT-02-legacy-intake-submitter-cleanup.md) — целевое состояние: удалить §5.2.4.B и service intake prose из GPT hot path.
- **N3 — one-shot/TTL warning (D-SUBMIT-5):** **опционально; не принято как обязательное** ([reconciliation §4](../../../analysis/interview-gpt-submit-handoff-reconciliation-2026-07-06.md)). `api-orchestrator.md` L622 — `Optional (N3 — not required)`. TTL/one-shot — поведение gateway (V6), не duty GPT. E5 — emergent для gateway→SPA-копи.
- **N4 — token-value alignment:** `GPT_ACTIONS_BEARER_SECRET` ↔ `SERVICE_API_TOKEN` — deploy-конфиг ([`railpack.json`](../../../../../doge-complaints-gateway/railpack.json); GW-DEPLOY-01 retired), вне этой стори.

## Эмерджентные запросы (cross-app, ссылками — $doc1 §3)
| # | Запрос | Кому | Sev |
|---|--------|------|-----|
| **E1** | Заморозить read-DTO `GET /story-drafts/{id}` (поля preview) | gateway | 🟠 |
| **E2** | Cutover роута `/story/submit` (GPT-redirect ↔ SPA ↔ deploy) | gateway/GPT/deploy → [mvp-plan](../../../../../docs/analysis/mvp-integration-plan-2026-07-02.md) | 🔴 |
| **E3** | Gateway-валидация `session_language` на стеше (пара к D-SUBMIT-4) | gateway | ✅ closed — [`contracts.py` L241–247](../../../../../doge-complaints-gateway/src/core/intake/contracts.py#L241) |
| **E4** | Стабильный entry-URL GPT (`VITE_STORY_GPT_URL`) | deploy/SPA | 🟡 |
| **E5** | Surface TTL черновика (для копи «истёк» vs «уже отправлено») | gateway → SPA-копи | 🟡 |

## Швы
- [`api-orchestrator.md` §5.1, §5.2.0a…§5.2.4](../../../../instructions/api-orchestrator.md), [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md), [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml) (operationId/method/path/auth в lockstep — обновить все три вместе).
