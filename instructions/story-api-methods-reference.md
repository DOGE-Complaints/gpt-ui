# Story Intake API — SSOT reference (DOGEstonia / GPT Actions)

**Version:** 1.7 · 2026-06-09  
**Scope:** Story-first runtime intake API SSOT  
**HTTP executor module:** [`api-orchestrator.md`](api-orchestrator.md)

This file is the **HTTP source of truth for story intake runtime** inside the instruction bundle. Import the deployed story intake OpenAPI into GPT Actions (or equivalent) and treat that imported contract as authoritative for `operationId`, paths, schemas, and security. Until the node publishes canonical OpenAPI, paths below are **candidates**; before production, reconcile with `GET /openapi.json` on the deployed API.

**Lock:** track `info.version` on the imported OpenAPI and this document’s **Version** line together (`info.version` is **0.4.2** at current instruction alignment — REQ-31 metadata + REQ-23 on REQ-22 wire v2). When a live node is available, prefer locking to `GET /openapi.json` for story routes.

---

## 1. Operations (parity matrix, story-first)

| operationId | Method | Path | Request contract | Success envelope | Purpose |
|-------------|--------|------|------------------|------------------|---------|
| `postStoryIntake` | POST | `/intake/stories` | `StoryIntakeRequest` | `SuccessEnvelope_StoryIntake` (`202`) | Send a citizen story to DOGEstonia for processing (REQ-31 citizen-facing label; operationId unchanged). |

Additional operations (search/reference/update) — add only when runtime contract exists; mirror in this table and in imported Actions contract with one-to-one method/path lock.

### 1.1 `StoryIntakeRequest` field lock

`StoryIntakeRequest` is aligned with runtime story intake contract and orchestrator story transform rules.

| Field | Required | Source |
|---|---:|---|
| `schema_version` | yes | hard-coded `m2.story_intake_envelope.v2` |
| `submitter.external_user_id` | yes | session/user context configured for runtime |
| `submitter.identity_issuer` | yes | hard-coded `dogestonia.gpt.v1` (REQ-22 demo) |
| `narrative.original_text` | yes | `canonical_payload.description[session_language]` |
| `narrative.language` | yes | `normalization_metadata.detected_input_language` |
| `narrative.session_language` | yes | `normalization_metadata.session_language` |
| `narrative.title` | yes | `canonical_payload.title` object `{et, ru, en}` |
| `narrative.description` | yes | `canonical_payload.description` object `{et, ru, en}` |
| `narrative.summary` | no | `canonical_payload.summary`; omit entire block if any `et`/`ru`/`en` slot empty (REQ-25) |
| `narrative.location_query` | no | `normalized_issue_payload.location_query` when user confirmed location (REQ-26); omit if absent |
| `narrative.canonical_type` | no | `canonical_payload.type`; `complaint` / `system_bug` → issue promotion gate (REQ-25) |
| `narrative.canonical_labels` | no | `canonical_payload.labels[]`; canonical disposition only per `story-label-taxonomy.md` (REQ-25) |
| `narrative.institution` | no | **Demo scope: always omit (REQ-28 pre-flight #7).** Post-demo: `canonical_payload.institution` when all `{et,ru,en}` non-empty (REQ-23 §2.5 / gateway REQ-43 (institution)). |
| `privacy.contains_pii` | no | `normalization_metadata.contains_pii` after §5.2.0 flow (REQ-23 §A) |
| `privacy.redaction_requested` | no | user choice in api-orchestrator §5.2.0 |
| `gpt_signals.severity` | no | `non_wire_metadata.severity` (gateway REQ-42 / gpt_signals enums) |
| `gpt_signals.impact_estimation` | no | `non_wire_metadata.impact_estimation` |
| `gpt_signals.problem_status` | no | `non_wire_metadata.problem_status` |
| `live_story_context.consistency_notes` | no | normalizer §4.5; omit if null |
| `origin` | no | traceability sidecar |

The request must not contain backend-issued fields (`id`, `status`, timestamps, txids), raw `non_wire_metadata`, or other instruction-internal metadata not accepted by runtime schema.

---

## 2. Authorization

- **GPT Actions:** `security: GptActionsBearer` in imported Actions contract; UI value = **`GPT_ACTIONS_BEARER_SECRET`** on server.
- Do not assume arbitrary headers from ChatGPT; user identity per node policy / `mock_user` per imported OpenAPI + operator setup.
- **Current identity model (M6-04):** app-level bearer integration key; no per-user OAuth claims are assumed in this contract.
- Any per-user identity model requires explicit backend/proxy design and must not be implied by this SSOT.
- Bearer secret placement is Actions/OpenAPI configuration concern, not runtime user-auth logic in instruction text.
- User authorization/identity flow (if needed by product) must be modeled separately from bearer and mapped into explicit API payload fields.

---

## 2.1 Actions runtime discipline (no filename dependency)

- Runtime behavior must depend on the imported **Actions contract** (`operationId`, method/path, request/response schema, security), not on a local filename string.
- Local OpenAPI artifacts in this repository are **build/import artifacts** for operator workflow only.
- Before release, verify parity across:
  1. Imported Actions schema (UI),
  2. this SSOT table,
  3. deployed API `GET /openapi.json` (when available).

---

## 3. Traceability (instruction modules, story-first)

| Source | Note |
|--------|------|
| REQ-18 / M2 runtime | Story intake handoff; do not invent `story_id` / status. |
| Backend authority | Status outside GPT; API response is truth ([`root.md`](root.md)). |
| [`api-orchestrator.md`](api-orchestrator.md) | Only module that initiates HTTP; DOGEstonia overlay at top of file. |

---

## 4. OpenAPI version lockstep policy

When contract fields change, update **both** artifacts in one change set:

1. Imported OpenAPI (Actions / operator bundle) → `info.version`
2. this file (`story-api-methods-reference.md`) → `Version`
3. changelog rows in both artifacts

Contract fields that require version bump:

- `operationId`
- HTTP method/path
- required request fields
- response envelope assumptions
- authentication semantics

Also record lock source: live `GET /openapi.json` or repository snapshot.

---

## 5. Version history

| Version | Date | Change |
|---------|------|--------|
| 1.7 | 2026-06-09 | **REQ-43 audit follow-up / GIM-185:** gateway REQ-43 (institution) namespace qualifier — §1 `narrative.institution` field lock. Semantics unchanged. |
| 1.6 | 2026-05-31 | **REQ-31 / GIM-137:** citizen-facing purpose text for `postStoryIntake`; lock `info.version` **0.4.2** (summary/description human-readable; operationId unchanged — Actions re-import required). |
| 1.5 | 2026-05-26 | **REQ-28 / GAP-YAML-08:** `narrative.institution` field lock updated with demo-gate note — always omit in demo scope (REQ-28 pre-flight #7); post-demo gate lifted by gateway REQ-43 (institution). |
| 1.4 | 2026-05-24 | **REQ-26 / GIM-120:** `narrative.location_query` source = top-level normalizer `location_query`. |
| 1.3 | 2026-05-24 | **REQ-25 / GIM-116–117:** activate `canonical_type` / `canonical_labels` wire; summary omit entire block if any slot empty. |
| 1.2 | 2026-05-24 | **REQ-23 / GIM-107–111:** `gpt_signals`, `narrative.institution`, `privacy.*`, `live_story_context.consistency_notes`; lock `info.version` **0.4.0**. |
| 1.1 | 2026-05-22 | **REQ-22 / GIM-102–103:** wire v2 field lock — `identity_issuer`, `session_language`, `title`/`description` i18n objects; remove `title_hint*`; success HTTP `202`; lock `info.version` **0.3.0**. |
| 1.0 | 2026-04-27 | Story-first runtime cutover: SSOT switched from `/issues/*` to `/intake/stories` (`postStoryIntake`) and `StoryIntakeRequest`/`SuccessEnvelope_StoryIntake`. |
| 0.1 | 2026-04-10 | First draft + initial Actions contract snapshot 0.1.0 (STORY-M6-01). |
| 0.2 | 2026-04-10 | **English-only** instruction text (repo policy). |
| 0.3 | 2026-04-10 | **M6-02:** one-to-one parity matrix (operationId/method/path/request/response) + lock source note (Actions snapshot 0.1.1, no committed node `/openapi.json`). |
| 0.4 | 2026-04-10 | **M6-04:** explicit app-level bearer identity assumption + version lockstep policy for OpenAPI/SSOT updates. |
| 0.5 | 2026-04-21 | Runtime decoupled from local contract filename; lock source/version updated to 0.1.3 and Actions contract parity rule clarified. |
| 0.6 | 2026-04-22 | **GIM-59:** link to `openapi-lock-snapshot-GIM-59.md` as tracking doc until live `GET /openapi.json` exists for Issues routes. |
| 0.7 | 2026-04-23 | **GIM-46:** linked executable SSOT governance playbook and formalized one-changeset operational flow for contract updates. |
| 0.8 | 2026-04-26 | **GIM-82:** aligned `IssueDraftCreateRequest` field lock with `story-data-model.md`, Actions contract `0.1.4`, and GIM-81 transform; kept PUT semantics as GIM-60 follow-up. |
| 0.9 | 2026-04-26 | **GIM-90:** clarified labels must be taxonomy/validation-approved and label extraction metadata is not part of `IssueDraftCreateRequest`. |
