# 04 — Actions OpenAPI wire

**Зона:** [`architecture/`](./README.md)  
**As-of:** 2026-09-16  
**Artifact:** [`custom-gpt-story-intake-actions.openapi.yaml`](../custom-gpt-story-intake-actions.openapi.yaml)  
**Companion:** [`story-api-methods-reference.md`](../../instructions/story-api-methods-reference.md)

---

## 1. Роль файла

Hand-crafted **GPT Actions import** subset для Story Intake — **не** live FastAPI `GET /openapi.json` (yaml L29–31: auto schema omits body / wrong ids / no security).

| Property | Value | Lines |
|----------|--------|-------|
| `info.version` | **0.8.0** | L13 |
| Path | `POST /story-drafts` | L412+ |
| `operationId` | `postStoryDraftStash` | L415 |
| Consequential | `x-openai-isConsequential: true` | L421 |
| Default server | `https://dogestonia-tallinn.up.railway.app` | L29–30 |
| Security | `GptActionsBearer` HTTP Bearer | L33–45 |

Auth mapping: ChatGPT Actions API Key (Bearer) = host `GPT_ACTIONS_BEARER_SECRET` — yaml L3–4, L37–45; [`gpt-actions-bot-api-auth-mapping.md`](../gpt-actions-bot-api-auth-mapping.md).

---

## 2. Что делает операция

- Stash `StoryDraftStashRequest` → **201** + `draft_id` (story **не** создаётся).  
- User finishes в SPA (`POST …/submit`).  
- Required root includes `schema_binding` (GPT-SSR-02, description L19).  
- Envelope behavioral version axis separate from server implementation version (L23–26).

Lockstep: bump yaml version when operationId/method/path/required/response/auth change — sync `story-api-methods-reference.md` §4; **re-import Actions** after contract bump (L20).

---

## 3. Operator surface

| Step | Where |
|------|--------|
| Import YAML | ChatGPT → Configure → Actions |
| Bearer secret | Actions auth = gateway `GPT_ACTIONS_BEARER_SECRET` |
| `SPA_BASE` | Configure → Instructions (not Actions YAML) — [03](./03-init-and-runtime-order.md) |
| Pack-only change | Instruction/Knowledge re-upload; OpenAPI re-import **обычно не** нужен (manual) |

Handoff checklist: [`gpt-story-submit-handoff-operator-guide.md`](../gpt-story-submit-handoff-operator-guide.md).

**Verified discrepancy:** handoff guide prose всё ещё цитирует старые OpenAPI **0.5.0 / 0.6.0** в местах — live artifact **0.8.0** (этот yaml). Не принимать guide version как SSOT контракта.

---

## 4. Bridge reuse

`doge-ai-bridge` держит wire copy под `DOGESTONIA_OPENAPI_PATH` (часто [`doge-ai-bridge/docs/openapi/story-intake-actions.openapi.yaml`](../../../doge-ai-bridge/docs/openapi/story-intake-actions.openapi.yaml)) — тот же `postStoryDraftStash`, исполняет **приложение**, не Actions UI ([00](./00-two-channels-overview.md), [05](./05-python-bridge-instruction-delivery.md)).

Channel façade n8n↔bridge — **отдельный** OAS (`aibridge-channel-v1`) — не смешивать со story-intake wire ([aibridge 00-overview](../../../doge-ai-bridge/docs/architecture/runtime/00-overview.md)).
