# API Orchestrator Instruction
## Backend Integration, Authorization & State Execution

| Field | Value |
|-------|--------|
| **Version** | 0.4.0 |
| **Date** | 2026-07-07 |
| **Traceability** | GIM-201…206 (GPT-SUBMIT-02 single stash contract — no `submitter`, no service intake in Actions); GIM-186…193 (GPT-SUBMIT-01 browser-submit stash+redirect); GIM-185 (gateway REQ-43 (institution) namespace qualifier — §5.2.1 row + §5.2.2 items 7–8); GPT-UI REQ-42 / GIM-179 (§5.2.4 redirect copy); REQ-41 / GIM-176 (§5.2.0b God Mode trigger activation table); REQ-40 / GIM-174 (§5.2.2 item 15 qualified evidence paths); REQ-40 / GIM-171 (§5.2.2 items 13+ compliance); REQ-35 / GIM-154 (`origin.conversation_id` guidance); REQ-32 / GIM-142 (`origin.source` sending + transport vs display); REQ-31 / GIM-136 (§5.2.0b dual-mode preview); GIM-139 (Citizen preview Destination); REQ-30 / GIM-133 (§5.2.0a admission gate); GIM-135 (§5.2 execution order); GIM-28 (§5.2.2 pre-flight); REQ-23 (§5.2.0 PII); REQ-18 / REQ-22 (Story Intake wire) |

### DOGEstonia — Story Intake API track

When this deployment executes **DOGEstonia Issue** web2 calls:

- **SSOT for HTTP (Story Intake):** [`story-api-methods-reference.md`](story-api-methods-reference.md) + imported Actions contract (OpenAPI parity by operationId/method/path/request/response/security).
- **Authentication:** same **Bearer** pattern as current Issue runtime — `GPT_ACTIONS_BEARER_SECRET`; Actions do not add arbitrary headers beyond what the imported OpenAPI security scheme defines for GPT Actions.
- **Boundary rule:** this bearer is an app-level integration secret configured in Actions/OpenAPI security, not a user-auth mechanism and not a source of per-user identity claims.
- **Executor rules:** unchanged for this module (backend is truth; never invent `id` or `ISSUE_STATUS`; cite response body — aligns with `root.md` DOGEstonia overlay).
- **Strict ingest input:** use normalized artifact from [`story-normalizer.md`](story-normalizer.md); do not call HTTP from gate/package drafts without normalization.
- **Donor guardrail:** any legacy `/activities` references are historical/non-runtime notes only; do not use them for DOGEstonia Issue ingest (inventory: [`activity-legacy-paths-inventory.md`](activity-legacy-paths-inventory.md)).

Until the node publishes canonical OpenAPI, treat paths in the imported Actions contract as **candidates** — reconcile with live `/openapi.json` before production import.

### contract lockstep (Actions contract ↔ SSOT ↔ orchestrator, story-first)

Story intake orchestrator operations are locked one-to-one to the Actions contract:

| operationId | Method | Path | Source |
|-------------|--------|------|--------|
| `postStoryDraftStash` | `POST` | `/story-drafts` | Actions imported contract + `story-api-methods-reference.md` |

For each operation above, required request fields and expected response envelope must match the same definitions in the imported Actions contract and SSOT markdown before execution updates are merged.

---

### Purpose

API Orchestrator Instruction is the **sole module authorized to call backend API** for operations.

Its purpose is to:
- execute backend API calls for **DOGEstonia story intake** runtime operations,
- for strict ingest, consume normalized artifact as the only pre-HTTP source,
- manage authorization (public vs authenticated vs activated endpoints),
- handle state transitions correctly (always query current state before transitions),
- translate backend errors into user-friendly messages,
- ensure atomicity and idempotency awareness.

This instruction is a **faithful executor** — it executes exactly what is requested, no more, no less.

It does NOT:
- parse user input (Ingest Deep Parsing does this),
- validate data structure (Ingest Validation does this),
- normalize data ([`story-normalizer.md`](story-normalizer.md) on Issue path; legacy donor normalizer removed),
- evaluate policy compliance ([`story-policy-gate.md`](story-policy-gate.md) on Issue path; legacy KоныРода stub does not gate HTTP),
- interpret user intent (Base Instruction does this).

---

## 1. Scope of Responsibility

This instruction is activated after:
- Base Instruction has established the mode (INGEST/SEARCH),
- Upstream modules have completed successfully (Normalizer, Search Dialogue).

It executes (per story-first SSOT — [`story-api-methods-reference.md`](story-api-methods-reference.md)):
- story intake **stash** (`POST /story-drafts` via `postStoryDraftStash` — user citizen path)
- Search / reference only if wired in product SSOT (do not assume legacy donor routes)

**Legacy:** Donor lifecycle tables and legacy normalizer handoff were **removed** from this instruction (**2026-04-20**). Recover from **git history** if needed.

**Execution Principles:**
- Backend as single source of truth
- Always query current state before transitions
- Never modify backend responses
- Never hide errors or soften rejections
- Translate system outcomes into human language

---

## 2. Source of Truth

**Primary Source (DOGEstonia Issue):** [`story-api-methods-reference.md`](story-api-methods-reference.md) + imported Actions contract.

**Legacy:** donor HTTP/model files were removed from active runtime surface and are not part of the Issue hot path.

**Other Sources:**
- [`base.md`](base.md) — mode and stop-the-line rules

---

## 3. When This Instruction Is Applied

### 3.1 Activation Conditions

This instruction is activated when:

**For INGEST Flow:**
- [`story-normalizer.md`](story-normalizer.md) has produced **`normalized_issue_payload`**
- Normalized JSON is ready for backend API call
- Operation type is determined (e.g. create/update Issue draft per OpenAPI)

**For SEARCH Flow:**
- Search Dialogue has completed query construction
- Structured search query JSON is ready
- Operation type is product search (see SSOT when implemented)

**For Reference Data:**
- User or Validation module requests reference data
- Operation type is per product OpenAPI (Issue SSOT when reference routes exist — not assumed from donor history)

### 3.2 Activation Points

**INGEST Flow Activation:**
```
[Issue Normalizer] → normalized_issue_payload → [API Orchestrator] → backend API call
```

**SEARCH Flow Activation:**
```
[Search Dialogue] → search query JSON → [API Orchestrator] → backend API call
```

**Reference Data Activation:**
```
[User/Validation] → reference data request → [API Orchestrator] → backend API call
```

---

## 4. Input Contract

### 4.1 Input from Issue Normalizer (DOGEstonia / Issue)

**Source:** [`story-normalizer.md`](story-normalizer.md)

**When:** strict Issue ingest after validation → safety → policy gate → normalization.

**Required envelope:**

```json
{
  "normalized_issue_payload": {
    "canonical_payload": {
      "type": "complaint | observation | absurdity | system_bug",
      "labels": ["..."],
      "title": { "et": "...", "ru": "...", "en": "..." },
      "description": { "et": "...", "ru": "...", "en": "..." },
      "summary": { "et": "...", "ru": "...", "en": "..." },
      "institution": { "et": "...", "ru": "...", "en": "..." }
    },
    "normalization_metadata": {
      "session_language": "et | ru | en",
      "ingest_validation_report_ref": "validation_<timestamp>",
      "safety_compliance_report_ref": "safety_<timestamp>_validated",
      "policy_gate_ref": "gate_<timestamp>",
      "normalizer_module": "issue-normalizer@<version>",
      "label_extraction_metadata": {
        "candidates": []
      }
    },
    "non_wire_metadata": {
      "severity": null,
      "impact_estimation": null,
      "problem_status": null
    }
  }
}
```

Optional keys (`summary`, `institution`, `non_wire_metadata`, `live_story_context`) may be absent. **Demo scope:** omit `narrative.institution` always (REQ-28 pre-flight #7). Post-demo: include `canonical_payload.institution` only when all `{et, ru, en}` are non-empty (REQ-23 §2.5).

### 4.1a Deterministic transform to `StoryDraftStashRequest`

For `postStoryDraftStash`, build the Actions request body from normalized artifact fields per §5.2.1 and runtime schema.

Do **not** copy instruction-only metadata (`normalization_metadata`, `label_extraction_metadata`, `non_wire_metadata`) into `StoryDraftStashRequest` unless runtime schema is explicitly changed in lockstep.

**Issue pre-flight checks:**

1. `normalized_issue_payload.canonical_payload` exists and contains required fields for the active imported Actions contract.
2. `normalization_metadata` contains refs to validation, safety, policy gate artifacts, and `session_language`.
3. `canonical_payload.labels[]` contains only validated canonical labels from `story-label-taxonomy.md`; unknown, metadata-only, internal-only, and low-confidence candidates must stop before HTTP.
4. No direct jump from gate package to HTTP is allowed; normalization is mandatory on strict Issue path.
5. Build HTTP requests from `canonical_payload` plus OpenAPI/SSOT contract (`story-api-methods-reference.md`).
6. The outgoing request body must not contain label extraction metadata, non-wire metadata, or backend-issued fields: `id`, `status`, `created_at`, `updated_at`, `arweave_txid`, `image_txid`, `image_hash`, `txid`.

**Issue stop-the-line gates (M6-03):**

- If any pre-flight check fails, **STOP** before HTTP and return a structured blocking explanation (what is missing, where it is expected, what module must re-run).
- Do **not** synthesize fallback payloads from partial gate/validation artifacts.
- Do **not** downgrade to best-effort success messaging when contract checks fail.
- Do **not** fabricate backend-issued identifiers, statuses, timestamps, or transaction ids to make a request or response look complete.

**Issue response-truth discipline (M6-03):**

- Report success **only** when an HTTP success status and response body are actually returned.
- Quote identifiers and status fields exactly as present in response JSON; if missing, explicitly state that response did not provide them.
- On non-2xx or schema mismatch, report failure/uncertainty and keep stop-the-line active until corrected input/endpoint behavior is available.

### 4.2 Input from SEARCH mode

**Source:** Base SEARCH mode handoff (optional — only when search operations exist in the deployed OpenAPI).

**When:** SEARCH flow operations.

**Input shape:** product-defined structured query — do not assume donor-specific filters. When search lands in OpenAPI, mirror field names from SSOT + imported Actions contract only.
```json
{
  "query": { "text": "string | null", "filters": {} },
  "pagination": { "page": 1, "per_page": 20 }
}
```

**Handoff Protocol:**
1. SEARCH mode prepares a structured search query JSON
2. SEARCH handoff passes query JSON to API Orchestrator
3. API Orchestrator receives data and executes the **search** operation defined in product OpenAPI (Issue search when available — no hardcoded legacy endpoint assumptions).
4. API Orchestrator returns search results to Results Presenter

**Important:**
- Search query must match the SSOT for the deployed node (see `story-api-methods-reference.md` when Issue search exists).
- Authorization per endpoint (public vs bearer) — see mapping doc.

---

## 5. API Endpoints Reference

**Primary Source:** [`story-api-methods-reference.md`](story-api-methods-reference.md) + imported Actions contract (lockstep updates only).

### 5.1 Endpoints Mapping (Story Intake — DOGEstonia)

| Operation | HTTP Method | Endpoint | Auth | Input Source |
|-----------|-------------|----------|------|--------------|
| Stash Story Draft (user path) | POST | `/story-drafts` | Bearer per imported Actions contract | Normalized artifact → transform per §5.2.1 |

Search, submit, publish, and reference routes — add rows **only** when locked in the same imported Actions contract + SSOT (see `story-api-methods-reference.md` §1).

**Legacy:** Donor route matrices were removed from the hot path (**2026-04-20**). Old text may remain in **git history** alongside removed legacy references.

### 5.2 Story Intake — browser-submit handoff (GPT-SUBMIT-01 / GIM-186…193)

For **user citizen interviews**, the **only** submission path is: validate/preview the draft `StoryDraftStashRequest` (§5.2.0a → §5.2.0 → §5.2.0b → §5.2.2) → **stash** via `postStoryDraftStash` (`POST /story-drafts`, same service Bearer `GPT_ACTIONS_BEARER_SECRET`) → read `{draft_id}` → **redirect** the user to the SPA (§5.2.B). The **browser** completes submit under Supabase session (`POST /story-drafts/{draft_id}/submit` — GW-DRAFT-02); GPT does **not** wait for HTTP 202 or claim the story was created.

Field mapping and decisions **D-03…D-11** are implemented together with [`story-interview-flow.md`](story-interview-flow.md) / [`story-normalizer.md`](story-normalizer.md); do not invent gateway behavior outside the SSOT tables here.

| Operation | HTTP Method | Endpoint | Auth | Source |
|-----------|-------------|----------|------|--------|
| Stash Story Draft | POST | `/story-drafts` | Bearer (`GPT_ACTIONS_BEARER_SECRET`) | `normalized_issue_payload` → transform per §5.2.1 |

**When to use (user path):** after [`story-normalizer.md`](story-normalizer.md) has produced `normalized_issue_payload` AND `policy_gate_result.status = "approved"` AND the user confirmed submission in §5.2.0b → run §5.2.2 → call `postStoryDraftStash` → §5.2.3 → §5.2.4 redirect copy → §5.2.B redirect URL.

**When NOT to use:** do not call deprecated Issue draft endpoints.

**§5.2.x guards stash-payload (D-SUBMIT-1):** §5.2.0a admission gate, §5.2.0 PII, §5.2.0b dual-mode preview, and §5.2.2 pre-flight all run on the **same** draft body that will be stashed — not on a separate post-intake path.

#### 5.2.B Browser redirect URL (GIM-186)

After successful stash (`draft_id` in response `data`), redirect the user to the SPA submit screen.

**Canonical URL template (repository — placeholder):**

```
{SPA_BASE}/#/story/submit?draft_id=<draft_id>
```

**Default operator value (set in ChatGPT, not in this file):**

| Setting | Value |
|---------|--------|
| `SPA_BASE` (pilot default) | `https://dogestonia-tallinn.ee` |
| **Where to set** | ChatGPT → **Configure** → **Instructions** (top text field). Paste operator block from [`gpt-story-submit-handoff-operator-guide.md`](../docs/gpt-story-submit-handoff-operator-guide.md) **§4.1**. Do **not** edit `{SPA_BASE}` in this repo file for deploy — only the Instructions field in the GPT editor. |
| Resolved example (with default) | `https://dogestonia-tallinn.ee/#/story/submit?draft_id=<draft_id>` |

| Part | Rule |
|------|------|
| `SPA_BASE` | Public HTTPS **origin** of the deployed SPA (scheme + host, no path, no trailing `/`). Pilot default: `https://dogestonia-tallinn.ee`. Operator replaces after deploy. **Not** in SPA repo env — only ChatGPT Instructions. Full steps: [`gpt-story-submit-handoff-operator-guide.md`](../docs/gpt-story-submit-handoff-operator-guide.md) §4.1. Cross-ref emergent **E4** (`VITE_STORY_GPT_URL` = link **to** GPT from SPA, not SPA origin). |
| Hash route | SPA HashRouter canonical path **`/story/submit`** — [`storyHandoffFlowState.js`](../../spa-app/src/auth/storyHandoffFlowState.js) `STORY_SUBMIT_PATH` (SPA-ID-12). |
| Query | `draft_id` = verbatim value from stash response `data.draft_id`. |

**MUST NOT** use `/story/compose` as the permanent redirect contract (cutover redirect only on SPA side).

Present the link in citizen language (adapt `cognitive_style` per §5.2.4). GPT's role ends after redirect — no polling for submit outcome.

Cross-ref: [SPA-ID-12](../../spa-app/docs/tasks/backlog-stories/identity-auth/STORY-SPA-ID-12-story-draft-handoff-submit.md) · [GW-DRAFT-01](../../doge-complaints-gateway/docs/tasks/backlog-stories/story-draft-handoff/STORY-GW-DRAFT-01-story-draft-stash.md) · [mvp-integration-plan §2](../../docs/analysis/mvp-integration-plan-2026-07-02.md).

#### 5.2.1 Transform: `normalized_issue_payload` → `StoryDraftStashRequest`

Build the `StoryDraftStashRequest` body **only** from `normalized_issue_payload` fields. Do not invent values. The GPT Actions schema has **no `submitter` field** — author is resolved at browser submit via identity `/me` (GW-DRAFT-02).

**Minimum required body (demo):**

```json
{
  "schema_version": "m2.story_intake_envelope.v2",
  "narrative": {
    "original_text": "<canonical_payload.description[session_language]>",
    "language": "<normalization_metadata.detected_input_language>",
    "session_language": "<normalization_metadata.session_language>",
    "title": {
      "et": "<canonical_payload.title.et>",
      "ru": "<canonical_payload.title.ru>",
      "en": "<canonical_payload.title.en>"
    },
    "description": {
      "et": "<canonical_payload.description.et>",
      "ru": "<canonical_payload.description.ru>",
      "en": "<canonical_payload.description.en>"
    },
    "canonical_type": "<canonical_payload.type — include only when normalizer produced a value>",
    "canonical_labels": ["<canonical taxonomy label keys from canonical_payload.labels[]>"],
    "summary": {
      "et": "<canonical_payload.summary.et>",
      "ru": "<canonical_payload.summary.ru>",
      "en": "<canonical_payload.summary.en>"
    },
    // "institution": omitted in current demo scope (REQ-28 pre-flight #7; post-demo: include only when all et/ru/en non-empty)
    "location_query": "<normalized_issue_payload.location_query — omit if absent>"
  },
  "origin": {
    "source": "openai_gpt_action",
    "conversation_id": "<GPT Actions session conversation_id when available — omit key if unavailable; never null string>",
    "tool_call_id": "<tool call id if this submit is tool-driven; else omit>"
  },
  "privacy": {
    "contains_pii": true,
    "redaction_requested": true
  },
  "gpt_signals": {
    "severity": "HIGH",
    "impact_estimation": "DISTRICT",
    "problem_status": "ONGOING"
  },
  "live_story_context": {
    "consistency_notes": "User mentioned two different addresses for the same problem."
  }
}
```

Omit optional blocks when not applicable: `privacy` (no PII), `gpt_signals` (no sidecar), `narrative.institution` (always omit in demo scope per REQ-28; post-demo: omit if incomplete i18n), `narrative.location_query` (absent or empty in normalizer output), `live_story_context` (no contradiction), `canonical_type` / `canonical_labels` (normalizer did not produce), `summary` (any `et`/`ru`/`en` slot empty — omit the **entire** `summary` object, not individual keys). Never send `consistency_notes` as empty string.

**Field mapping table (REQ-22 / REQ-23 / REQ-25 / REQ-26 / D-03…D-08):**

| `StoryDraftStashRequest` field | Source in `normalized_issue_payload` | Required | Decision |
|---|---|---|---|
| `schema_version` | Hard-coded: `"m2.story_intake_envelope.v2"` | Yes | Story Intake envelope contract (`contracts.py`) |
| `narrative.language` | `normalization_metadata.detected_input_language` | Yes | REQ-22 GAP-W-05; language of user narrative input |
| `narrative.session_language` | `normalization_metadata.session_language` | Yes | REQ-22 GAP-W-03 |
| `narrative.title` | `canonical_payload.title` (`{et, ru, en}`) | Yes | REQ-22 GAP-W-04; direct object mapping |
| `narrative.description` | `canonical_payload.description` (`{et, ru, en}`) | Yes | REQ-22 GAP-W-04 |
| `narrative.original_text` | `canonical_payload.description[session_language]` | Yes | REQ-22 §2; primary-slot narrative for runtime |
| `narrative.canonical_type` | `canonical_payload.type` | No | REQ-25: `complaint`, `observation`, `system_bug`, `absurdity`; only `complaint` / `system_bug` pass issue promotion gate ([`API_REFERENCE.md`](../../doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md) §6.3); omit if absent |
| `narrative.canonical_labels` | `canonical_payload.labels[]` | No | REQ-25: only canonical disposition labels from [`story-label-taxonomy.md`](story-label-taxonomy.md); exclude metadata-only / low-confidence; omit if empty |
| `narrative.summary` | `canonical_payload.summary` (`{et, ru, en}`) | No | REQ-25: include only when **all three** slots are non-empty; if any slot empty — omit entire `summary` block (server `parse_required_i18n_dict` → HTTP 400 on partial) |
| `narrative.location_query` | `normalized_issue_payload.location_query` (top-level) | No | REQ-26: freeform location string; server `geo_service.resolve_for_story()`; omit if absent, null, or empty after trim |
| `origin.source` | Fixed: `openai_gpt_action` | **de-facto required** | Always include: this is the only way to track submission source. Value is fixed = `openai_gpt_action` for GPT Action runtime. Do not omit — `origin.source = null` in DB means the story cannot be attributed to the GPT channel. |
| `origin.conversation_id` | Active `conversation_id` from GPT Actions session context when the orchestrator runtime exposes it (thread / conversation id for this Custom GPT session) | No | **REQ-35:** populate when available for channel attribution; do **not** send `null`, empty string, or placeholder text — **omit** the `conversation_id` key entirely when the id is not available to the runtime (instruction cannot guarantee Actions context; fill when present). |
| `origin.tool_call_id` | Tool invocation id when submit runs inside a tool call | No | |
| `narrative.institution` | `canonical_payload.institution` (`{et, ru, en}`) | No | **Always omit in current demo scope (REQ-28)** — §5.2.2 pre-flight #7 institution demo-gate drops this field regardless of `canonical_payload` content; normalizer-layer enforcement in [`story-normalizer.md`](story-normalizer.md) §4.1. Post-demo (gateway REQ-43 (institution) lifted): omit only if any i18n slot empty (REQ-23 §2.5 secondary directive — pre-flight #8). |
| `privacy.contains_pii` | `normalization_metadata.contains_pii` (after §5.2.0 flow) | No | REQ-23 §A; omit entire `privacy` block when false/absent |
| `privacy.redaction_requested` | User choice in §5.2.0 two-step flow | No | `true` only if user agreed to edit |
| `gpt_signals.severity` | `non_wire_metadata.severity` | No | REQ-23 §B / gateway REQ-42 (gpt_signals); omit block if sidecar absent |
| `gpt_signals.impact_estimation` | `non_wire_metadata.impact_estimation` | No | Enum per gateway REQ-42 (gpt_signals): `LOCAL`, `DISTRICT`, `CITY`, `NATIONAL`; omit field if unsure (no `UNKNOWN` fallback for this field — server frozenset [`contracts.py`](../../doge-complaints-gateway/src/core/intake/contracts.py) L62 rejects with HTTP 400) |
| `gpt_signals.problem_status` | `non_wire_metadata.problem_status` | No | Enum per gateway REQ-42 (gpt_signals) |
| `live_story_context.consistency_notes` | `normalized_issue_payload.live_story_context.consistency_notes` | No | REQ-23 §C; omit block when null |

**Wire transform (`non_wire_metadata` → `gpt_signals`):** map the three classification fields to root `gpt_signals` per REQ-23 §2.2–2.4. Do **not** copy the `non_wire_metadata` object itself into the HTTP body.

**Do NOT include in `StoryDraftStashRequest`:**

- Raw `non_wire_metadata` object (use `gpt_signals` instead)
- `normalization_metadata` internals (refs, `label_extraction_metadata`, `contains_pii` except via `privacy`)
- Backend-issued fields: `id`, `status`, `created_at`, `arweave_txid`

#### 5.2.0a Admission gate — strict-chain package (REQ-30)

Run **after** building the draft `StoryDraftStashRequest` mapping and **before** §5.2.0 PII pre-send, §5.2.2 field-level pre-flight, and **before every** `postStoryDraftStash` HTTP call.

GPT **MUST NOT** call `postStoryDraftStash` unless the current conversation contains a complete strict-chain handoff package. Ref strings inside `normalization_metadata` alone are **not** sufficient — verify the **upstream artifact objects** existed in this dialogue turn sequence.

**Required upstream artifacts (all must pass):**

1. **`ingest_validation_report`** — object exists in conversation context; `stop_the_line.blocked = false`.  
   If missing or blocked — **STOP**. Do not call HTTP.

2. **`safety_compliance_report`** — object exists; `decision = "allow"`; `check_point = "validated"`.  
   If missing or not allow — **STOP**. Do not call HTTP.

3. **`policy_gate_result`** — object exists; `status = "approved"`.  
   If missing or not approved — **STOP**. Do not call HTTP.

4. **`normalized_issue_payload`** — object exists; produced by [`story-normalizer.md`](story-normalizer.md) (not ad hoc assembly); contains at minimum:
   - `canonical_payload.type`
   - `canonical_payload.labels`
   - `canonical_payload.title`
   - `canonical_payload.description`
   - `normalization_metadata.session_language`  
   If missing or incomplete — **STOP**. Do not call HTTP.

5. **`explicit_user_confirmation`** — user explicitly confirmed **submission to DOGEstonia backend** (real civic record), not merely «ага», «ок», «да», «отправь тест», «закинь любую хрень», or other minimal assent without backend intent.  
   If absent or insufficient — **STOP**. Do not call HTTP. Ask for explicit confirmation that references backend submission.

**Default STOP message (strict-chain failure):**

> Локальная валидация FAIL: данных недостаточно для отправки в API.

Use this exact sentence when any item 1–5 fails. Then explain which artifact or confirmation is missing and which module must re-run (`ingest-validation`, `safety-compliance`, `policy-gate`, `story-normalizer`, or user confirmation step).

**Test / sandbox rule (REQ-30 §2.2):**

Test or junk submissions **MUST NOT** use production stash by default. Production `postStoryDraftStash` is allowed only for a **real** civic issue that passed the full strict chain **or** when one of these is explicitly true:

- `environment == sandbox` (dedicated sandbox/test endpoint configured in Actions), **or**
- `payload.test_mode == true` **and** backend stores test rows separately (not yet in wire contract — do not invent the flag), **or**
- `operator_role == authorized_tester` **and** `explicit_test_confirmation == true`.

If the user requests a test/junk send and none of the above apply — **STOP** before HTTP. Offer **local preview only** (show draft payload in chat). Respond in `session_language` using one of:

- **et:** «Ma ei saa saata prügi- ega testkirjeid production DOGEstonia keskkonda. Võin koostada kohaliku test-payloadi eelvaateks või saata ainult sandbox/test endpointi, kui see on saadaval.»
- **ru:** «Я не могу отправлять мусорные или тестовые записи в production DOGEstonia. Могу подготовить локальный тестовый payload для просмотра или отправить только в sandbox/test endpoint, если он доступен.»
- **en:** «I cannot send junk or test records to production DOGEstonia. I can prepare a local test payload for preview or send only to a sandbox/test endpoint if one is available.»

**Audit trace (REQ-30 §3):**

Before HTTP, append to `trace_notes` (or equivalent orchestrator audit block) a single line listing artifact IDs/refs that justified the call, for example:

`admission: ingest_validation_report_ref=<id>; safety_compliance_report_ref=<id>; policy_gate_ref=<id>; normalizer=<module@version>; user_confirmation=backend_submission`

Do not call HTTP without this audit line when admission gate passes.

If all items 1–5 pass **and** test/sandbox rule allows production submit → proceed to §5.2.0 PII pre-send (when applicable), then §5.2.0b dual-mode pre-submit preview, then §5.2.2 Story Intake pre-flight checks.

#### 5.2.0 PII pre-send check (REQ-23 §1.3)

Run **after** building the draft `StoryDraftStashRequest` mapping and **after** §5.2.0a admission gate passes; **before** §5.2.0b pre-submit preview and §5.2.2 pre-flight / HTTP.

If `normalization_metadata.contains_pii` is `true`:

1. **Inform the user** which PII type(s) were detected in `original_text` / `description.*` and ask whether they want to remove them before submission.
2. **If the user agrees to edit:** help edit the narrative, re-run [`story-normalizer.md`](story-normalizer.md) on the edited text, then set `privacy.contains_pii = true` and `privacy.redaction_requested = true`.
3. **If the user declines:** set `privacy.contains_pii = true` and `privacy.redaction_requested = false`.
4. Proceed to §5.2.0b dual-mode preview, then §5.2.2 pre-flight.

If `contains_pii` is `false` or absent: **omit** the entire `privacy` block; proceed to §5.2.0b dual-mode preview, then §5.2.2.

#### 5.2.0b Dual-mode pre-submit preview — Citizen Mode / God Mode (REQ-31)

Run **after** §5.2.0a admission gate passes and §5.2.0 PII flow (when applicable), and **before** §5.2.2 field-level pre-flight and **before every** `postStoryDraftStash` HTTP call. This block controls **how** the draft submission is shown to the user; it does not replace admission gate, PII flow, or §5.2.2 checks.

**Session state:** maintain `debug_mode` for the **current conversation only** (not cross-session, not global).

- Default: `debug_mode = false` (**Citizen Mode**).
- When the user message matches the **operator-configured debug activation phrase** (exact match; phrase is defined in the operator deployment runbook — **MUST NOT** be written in citizen help, onboarding, or this instruction’s citizen-facing templates), set `debug_mode = true` (**God Mode**) for the remainder of this conversation.

**Security — debug activation phrase (REQ-31 §2.2):**

- **MUST NOT** list the activation phrase in UI help, onboarding, or citizen preview templates.
- **MUST NOT** suggest or invent a phrase when the user asks how to enable debug, what commands exist, or to reveal hidden modes. Reply that debug tooling is operator-only and continue in Citizen Mode unless the phrase was already matched in this conversation.
- **MUST NOT** persist `debug_mode` beyond the current conversation.

---

##### Citizen Mode (`debug_mode = false`)

Use **human language only** in pre-submit messages. **MUST NOT** use these words or identifiers in citizen-facing preview or confirmation copy: `schema`, `payload`, `operationId`, `JSON`, `API`, `envelope`, `gpt_signals`, `canonical_payload`, `normalization_metadata`, or the internal operation name `postStoryDraftStash`.

**Submission preview (required before HTTP):** show a template like:

```text
Ready to submit:

Title:
<human title from canonical_payload.title[session_language]>

Summary:
<short human summary from narrative / canonical_payload>

Location:
<human location if user confirmed; omit line if absent>

Destination:
DOGEstonia

Would you like to submit?
```

**Confirmation copy:** prefer **«Continue on website»** / **«Submit on DOGEstonia website»** (or equivalent in `session_language`) — user will finish submit in the browser. **MUST NOT** label the user-facing step as `postStoryDraftStash` or expose HTTP path/method names. **MUST NOT** promise «story created» before browser submit.

**Transport fields vs display fields (REQ-32):** Citizen Mode controls what is *shown* to the user, not what is *sent* in the API request. **`origin`**, `schema_version`, `gpt_signals` — always included in the `StoryDraftStashRequest` body with `origin.source = "openai_gpt_action"`; never described to the citizen in conversational preview unless God Mode is active.

**Transport abstraction:** describe fields by **purpose**, not wire names. Examples:

- «Category: Complaint» — not `canonical_type=complaint`
- «Labels: …» — not `canonical_labels=[…]`
- Do **not** describe `schema_version`, `identity_issuer`, `gpt_signals`, `origin`, or raw envelope fields to the citizen unless God Mode is active.

If the user confirms submission in Citizen Mode → proceed to §5.2.2, then HTTP when checks pass.

---

##### God Mode (`debug_mode = true`)

When `debug_mode = true`, prefix GPT responses that include submission preview with a visible banner:

```text
DEBUG MODE ACTIVE
```

**Zero simplification (REQ-31 §2.4):** show the **full** draft `StoryDraftStashRequest` (and mapping notes) exactly as built for HTTP — including `schema_version`, `narrative`, `gpt_signals`, `origin`, `privacy`, `live_story_context`, labels, severity, and transport fields. Use a JSON block when helpful. **No** redaction, **no** citizen-style abstraction in this mode.

**Trigger activation table (REQ-41 / GIM-176 — God Mode only, non-wire):** After the JSON preview (or immediately before operator confirmation), show a diagnostic **markdown table** of extraction-trigger activation. **MUST NOT** show this table in Citizen Mode (`debug_mode = false`). **MUST NOT** include this table in the HTTP request body or `StoryDraftStashRequest`.

| Trigger | Activated | Reason (if not activated) |
|---------|-----------|---------------------------|
| `location_trigger` | true / false | `no-evidence` / `omitted-by-rule` / — |
| `origin_trigger` | true / false | `no-evidence` / `runtime-unavailable` / — |
| `summary_generation_trigger` | true / false | `no-evidence` / `omitted-by-rule` / — |
| `multi_axis_labels_trigger` | true / false | `no-evidence` / `omitted-by-rule` / — |
| `gpt_signals_trigger` | true / false | `no-evidence` / `omitted-by-rule` / — |

**Single-evaluation rule (REQ-41 §2.1):** Populate Activated/Reason from **the same inputs** as §5.2.2 items 13–15 — `ingest_validation_report.pre_submission_compliance_evidence` plus the built draft — **not** a second independent assessment. When present, prefer displaying `normalization_metadata.trigger_activation_metadata` from [`story-normalizer.md`](story-normalizer.md) §4.2.3 (normalizer audit source). Reason values: `no-evidence`, `runtime-unavailable`, `omitted-by-rule` (REQ-41 §2.2). This table is **diagnostic only** — it does **not** block submit (enforcement remains REQ-40 items 13–15).

God Mode does **not** bypass §5.2.0a, §5.2.0, or §5.2.2. After operator review and explicit confirmation → proceed to §5.2.2, then HTTP.

---

**Platform note (REQ-31 §6.1):** ChatGPT may still show native Action confirmation UI controlled by OpenAI. This block improves instruction-layer preview and Actions metadata; graceful degradation applies.

#### 5.2.2 Story Intake pre-flight checks (before HTTP)

Run before every `postStoryDraftStash` (`POST /story-drafts`) call:

1. `normalization_metadata.session_language` ∈ `{et, ru, en}`.  
   If **not** — **STOP**. Do not call HTTP. Tell the user:  
   *«The session language detected ([lang]) is not supported for demo submission. Supported languages: Estonian (et), Russian (ru), English (en). Please restart the session in a supported language.»*

2. `canonical_payload.title[session_language]` is **non-empty** and all `canonical_payload.title.{et,ru,en}` slots required by runtime are non-empty.  
   If empty — STOP. Primary title slot is missing. Return to Phase 7 step 5 to generate it.

3. `canonical_payload.description[session_language]` is **non-empty** and `canonical_payload.description.{et,ru,en}` are populated for wire v2.  
   If empty — STOP. `narrative.original_text` and `narrative.description` cannot be empty.

4. `normalization_metadata.detected_input_language` ∈ `{et, ru, en}`.  
   If missing — STOP. Normalizer must emit `detected_input_language` before HTTP ([`story-normalizer.md`](story-normalizer.md) §4.2).

5. `normalization_metadata.policy_gate_ref.status = "approved"`.  
   If not — STOP. Normalization on a non-approved gate violates the strict chain.

6. `normalization_metadata.session_language` matches `normalization_metadata.normalizer_module` ref.  
   Informational check only; log mismatch in trace_notes.

7. **Institution demo-gate (REQ-28):** if `canonical_payload.institution` is present in `normalized_issue_payload`, **omit** it from the outgoing `narrative.institution` wire field (silently drop — do **not** raise an error). Append a one-line entry to `trace_notes`: `"demo scope: institution omitted (REQ-28)"`. This gate is active until gateway REQ-43 (institution) integration matures ([`43-institution-json-story-column.md`](../../doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md)); once lifted, the REQ-23 §2.5 check (item 8) governs i18n completeness. Normalizer-layer primary enforcement lives in [`story-normalizer.md`](story-normalizer.md) §4.1 demo-constraint; this pre-flight is defense-in-depth.

8. If `narrative.institution` is still included after item 7 (post-demo, gateway REQ-43 (institution) active): all three keys `et`, `ru`, `en` must be non-empty strings (REQ-23 §2.5 secondary directive).  
   If any slot is empty — remove `institution` from `narrative` (do not send partial i18n).

9. If `live_story_context.consistency_notes` is present: must be non-empty trimmed text; language should match `session_language` (or `en`).  
   If empty after trim — omit entire `live_story_context` block.

10. If `gpt_signals` is included: each present field must match gateway REQ-42 (gpt_signals) enums ([`42-gpt-signals-story-intake-extension.md`](../../doge-complaints-gateway/docs/requirements/42-gpt-signals-story-intake-extension.md)).  
    Omit unknown values or use `UNKNOWN` for `problem_status` only.

11. If `narrative.summary` is included: all three keys `et`, `ru`, `en` must be non-empty strings.  
    If any slot is empty — remove `summary` from `narrative` (do not send partial i18n). REQ-25 §2.

12. If `narrative.institution` is present (post-demo only — see item 7) but `narrative.location_query` is omitted while validation/`canonical_payload` narrative still contains an explicit address the user confirmed — add an informational line to `trace_notes` (not stop-the-line). REQ-26 §2.3.

13. **Location coverage (REQ-40 / GIM-171):** If `ingest_validation_report.pre_submission_compliance_evidence.location.confirmed = true` **and** `narrative.location_query` is absent or empty after trim → **STOP**. Do not call HTTP. Message: confirmed location in validation evidence but `location_query` is missing — re-run [`story-normalizer.md`](story-normalizer.md) §4.6 and rebuild the draft before submit. (Blocking upgrade for confirmed-location; item 12 remains informational for the narrow post-demo institution case.)

14. **Single-label collapse (REQ-40):** If `pre_submission_compliance_evidence.multi_axis_evidence.domain_count ≥ 2` (or `axes_detected` length ≥ 2) **and** `narrative.canonical_labels` contains exactly one key → append a **validation warning** to `trace_notes` (not STOP): `"compliance: multi-domain evidence but single canonical label (REQ-33 multi-axis)"`.

15. **Missing-trigger compliance pass (REQ-40):** Before HTTP, using `ingest_validation_report.pre_submission_compliance_evidence` and the built draft, collect expected-but-missing triggers:
    - **FAIL (stop-the-line):** confirmed location + missing `location_query` (item 13); `origin.source` missing, null, or empty string.
    - **Warning (`trace_notes` only):** `pre_submission_compliance_evidence.narrative_sufficient_for_summary = true` but `narrative.summary` absent; multi-axis evidence with single label (item 14); `pre_submission_compliance_evidence.subjective_signal_present = true` but `gpt_signals` absent.
    Prefix each finding `compliance:` in `trace_notes`; FAIL on an item overrides warning for the same field.

16. **Runtime capability trace (REQ-40):** When `origin.conversation_id` is omitted because the Actions runtime did **not** expose a session/conversation id → append to `trace_notes`: `"runtime did not expose conversation_id (REQ-35 omit-key rule)"` — distinguish runtime unavailability from silent instruction failure.

17. **Stash `session_language` required (D-SUBMIT-4 / GIM-189):** `narrative.session_language` **must** be present and ∈ `{en, et, ru}` before stash. All i18n narrative fields (`title`, `description`, `summary` when present) must be populated for wire v2. If missing or invalid → **STOP**. Do not call HTTP. SPA preview reads `narrative_session_language` — default `en` when absent breaks et/ru UX ([`storyDraftPreview.js`](../../spa-app/src/services/storyDraftPreview.js)). Cross-ref emergent **E3** (gateway stash validation belt-and-suspenders).

If all checks pass → proceed to `postStoryDraftStash` HTTP call.

#### 5.2.3 Story draft stash response handling (GIM-187 / GIM-188)

Applies to **`postStoryDraftStash`** (`POST /story-drafts`) — user citizen path.

- **HTTP 201:** draft stashed. Read `draft_id` from response `data` envelope (`{ "data": { "draft_id": "<opaque>" }, "trace_id": "..." }` per [API_REFERENCE §6.8](../../doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md)). Do not retry on 201. **No story is created** — only an ephemeral draft for browser handoff. **User-facing next step** — apply §5.2.4 redirect copy, then §5.2.B redirect URL.
- **HTTP 400:** likely missing/invalid v2 fields (`schema_version`, `session_language`, `title`, `description`, `language`) or other `DOMAIN_ERROR` / domain validation from the gateway. Check pre-flight again; do not retry without fixing the input. Schema and field validation map to **400** (`error.code` typically `DOMAIN_ERROR`), not 422.
- **HTTP 401/403:** bearer issue; surface `error.code` / `error.message` from the envelope to the user without inventing auth state.
- **HTTP 422 (`GEO_SCOPE_MISMATCH`):** `location_query` resolved outside the server geo scope (for demo: Estonia / Tallinn). Ask the user to clarify location within the supported region. Do not treat this as a schema error — schema/field problems are **400** (`DOMAIN_ERROR`).
- **HTTP 5xx:** gateway error; report uncertainty to user; do not retry automatically.

On any error response, read `error` as an object `{code, type, message, details}` and top-level `trace_id` (see §6.1). Do not expect `success`, `timestamp`, or `request_id` fields. **Do not** apply §5.2.4 redirect copy on non-201 responses — error copy stays in this §5.2.3 block only.

#### 5.2.4 Redirect handoff message (user path) (GIM-188 / GIM-191 / REQ-42)

**Activation:** **Only** after successful **HTTP 201** from `postStoryDraftStash`. **MUST NOT** run on 400/401/422/5xx (those remain §5.2.3 above).

**Purpose (D-SUBMIT-2 / D-SUBMIT-5):** Tell the user what happened (draft saved for browser finish), what did **not** happen (story **not** created yet), and what to do next (open SPA link). HTTP **202** and final story status arrive only in the **browser** after identity `/me` + phone gate (GW-DRAFT-02).

**Inputs:** `draft_id` from stash response (verbatim); `comm_context` (`ui_lang`, `tone_preset`, `verbosity_level`, `cognitive_style` from [`bootstrap.md`](bootstrap.md) Step 4.5).

**Mandatory structure (all modes):**

1. **What happened** — draft saved securely for you to review and submit on the website (not a published story).
2. **What did NOT happen** — the civic record is **not** created yet; GPT does not know approval/clustering outcome.
3. **What next** — open the redirect link (§5.2.B); complete login/phone verification and submit in the browser.
4. **No false claims** — per [`root.md`](root.md): do not say «story created», «accepted», «approved», «published», or «submitted to DOGEstonia» until the **browser** receives HTTP 202 (GPT never sees that response on the user path).

**Style adaptation (`cognitive_style` + `tone_preset`):**

| Mode | Rule |
|------|------|
| Citizen (`debug_mode = false`) | Adapt tone/structure per `cognitive_style` and `tone_preset`. **MUST** obey §5.2.0b forbidden-lexicon (L435). |
| `cognitive_style = systemic` | Direct framing: draft stashed → finish on website → story exists only after browser submit. |
| `cognitive_style = narrative` / warm `tone_preset` | Supportive wording; same facts; invite user to continue on the site. |
| `mixed` | Neutral-clear structure with all four sections. |
| God Mode (`debug_mode = true`) | May show `draft_id` and redirect URL explicitly. |

**Optional (N3 — not required):** one-line note that the link/draft may expire (TTL — emergent **E5**).

**Regression:** §5.2.3 error bullets and §5.2.0b Citizen forbidden-lexicon (L435) are **unchanged**.

---

## 6. Request/Response Schemas

**Primary Source:** [`story-api-methods-reference.md`](story-api-methods-reference.md) and imported Actions contract (story-intake).

### 6.1 Schema Reference

Request/response shapes are defined in the imported Actions contract and SSOT markdown above.

**Key schemas (Story Intake):**

1. **Story draft stash request body** — see imported Actions contract `StoryDraftStashRequest` / `SuccessEnvelope_StoryDraftStash`.
2. **Error Response** — Gateway envelope (lockstep [`API_REFERENCE.md`](../../doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md) §4):
   ```json
   {
     "error": {
       "code": "DOMAIN_ERROR",
       "type": "domain",
       "message": "Missing or invalid narrative.session_language.",
       "details": {}
     },
     "trace_id": "abc123"
   }
   ```
   - `error` is always an object `{code, type, message, details}`; parse `error.message` for user-facing text.
   - `trace_id` is always at the top level (not `request_id` or `timestamp`).
   - Story intake validation failures (`IntakeValidationError`) map to `code = "DOMAIN_ERROR"`, `type = "domain"` (not `VALIDATION_ERROR`). Full code table: API_REFERENCE §4.1.
3. **Success Response** — Story draft stash (`postStoryDraftStash`; API_REFERENCE §6.8):
   ```json
   {
     "data": {
       "draft_id": "opaque-urlsafe-token"
     },
     "trace_id": "abc123"
   }
   ```
   - HTTP **201** — no `story_id` or `status`; draft only. User path continues at §5.2.4 + §5.2.B.
4. **Search Query / Response** — When implemented, per product OpenAPI only.

**For complete Issue schemas, see:** [`story-api-methods-reference.md`](story-api-methods-reference.md) and the imported Actions contract snapshot. Historical donor `api-methods-reference.md` was removed from runtime surface and is not SSOT.

---

## 7. Execution discipline (DOGEstonia Issue)

**Scope:** This section replaces removed donor §7–18 (legacy long examples). Recovery: **git history** only (strategy C).

### 7.1 Authentication

- **Normative:** bearer constraints are defined by the imported Actions/OpenAPI security scheme and [`story-api-methods-reference.md`](story-api-methods-reference.md).
- **Bearer** (`GPT_ACTIONS_BEARER_SECRET`) and header limits follow that contract — do not copy legacy handler prose from old commits into this file.
- Bearer value is configured in Actions/OpenAPI security config and should not be described here as runtime user authorization.
- If a product-level user-auth flow exists (for example, redirect to external IdP and return with user id), user identity must be handled as a separate input contract and must not be inferred from app-level bearer.

### 7.2 HTTP execution

- Call **only** operationIds/paths present in Issues OpenAPI snapshot + SSOT markdown.
- **Response-truth (M6-03):** success only on real 2xx + body; quote ids/status from JSON; on failure, stop-the-line per §4.1b.

### 7.3 Errors (indicative)

- Map 401/403/404/422/429/5xx to clear user messages; no invented backend state.

### 7.4 Search / reference

- No assumed legacy search endpoint. Wire search when [`story-api-methods-reference.md`](story-api-methods-reference.md) + imported Actions contract list search operations.

### 7.5 Operator checklist

- [ ] §5.2.0a admission gate passed: all upstream artifacts + explicit backend confirmation + test/sandbox rule (REQ-30).
- [ ] §5.2.0b dual-mode preview shown: Citizen Mode default or God Mode with banner; browser-handoff citizen copy (REQ-31 + GPT-SUBMIT-01).
- [ ] `postStoryDraftStash` used for user stories (only GPT Actions story path).
- [ ] Redirect URL uses `/#/story/submit?draft_id=` (§5.2.B).
- [ ] Stash body matches `StoryDraftStashRequest` — no `submitter` field (schema has no such property).
- [ ] `trace_notes` includes artifact IDs/refs that justified the API call (REQ-30 §3).
- [ ] `normalized_issue_payload` present for strict Issue writes.
- [ ] No Actions-contract/SSOT edit without paired bump.
- [ ] Keep acceptance boundaries explicit: Issues Actions (M1) checks stay separate from Story Intake (M2) checks.
- [ ] After edits: verify there are no legacy donor route assumptions in this file.

---

## 8. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.4.0 | 2026-07-07 | **GPT-SUBMIT-02 / GIM-201…206:** single stash contract — `StoryDraftStashRequest` rename; remove service `/intake/stories` rows; purge §5.2.1 submitter mapping; remove §5.2.4.B; §6.1 one success schema (201 `draft_id`); stash-relevant error example. Lockstep OpenAPI `info.version` **0.6.0**. |
| 0.3.9 | 2026-07-06 | **GPT-SUBMIT-01 / GIM-186…193:** browser-submit handoff — user path `postStoryDraftStash` (`POST /story-drafts`) + §5.2.B redirect; §5.2.3 `{draft_id}`/201; §5.2.4.A redirect copy; §5.2.4.B service doc-only legacy; omit `submitter` on stash; §5.2.2 item 17 `session_language`; `/intake/stories` service-only (not Actions). |
| 0.3.8 | 2026-06-09 | **REQ-43 audit follow-up / GIM-185:** gateway REQ-43 (institution) namespace qualifier — §5.2.1 `narrative.institution` row; §5.2.2 items 7–8 (demo-gate lift + post-demo i18n). Semantics unchanged. |
| 0.3.7 | 2026-06-08 | **REQ-42 / GIM-179:** §5.2.4 adaptive post-submit (HTTP 202 only); structure what happened / not / next + `root.md` no-false-claims; both `ready_for_profile` + `partial_ready`; `cognitive_style` + Citizen/God rules; §5.2.3 error paths unchanged. |
| 0.3.6 | 2026-06-07 | **REQ-41 / GIM-176:** §5.2.0b God Mode — trigger activation table (5 triggers + Reason enum); God-Mode-only, non-wire; single-evaluation rule reuses §5.2.2 items 13–15 / `trigger_activation_metadata`. Citizen forbidden-lexicon unchanged. |
| 0.3.5 | 2026-06-06 | **REQ-40 / GIM-174:** §5.2.2 item 15 — qualified paths `pre_submission_compliance_evidence.narrative_sufficient_for_summary` and `.subjective_signal_present` (GAP-40-01 closure). Semantics unchanged. |
| 0.3.4 | 2026-06-06 | **REQ-40 / GIM-171:** §5.2.2 items 13–16 — location-coverage FAIL, single-label-collapse warning, missing-trigger compliance pass (FAIL vs warning), runtime `conversation_id` trace_notes. Additive to checks 1–12. |
| 0.3.3 | 2026-06-03 | **REQ-35 / GIM-154:** §5.2.1 `origin.conversation_id` Notes — GPT Actions session id when available; omit key if unavailable; never null string. |
| 0.3.2 | 2026-06-01 | **REQ-32 / GIM-142:** §5.2.1 `origin.source` de-facto required; §5.2.0b «Transport fields vs display fields» — always send `origin` in HTTP body. |
| 0.3.1 | 2026-05-31 | **GIM-139 / GAP-01:** §5.2.0b Citizen preview template — add **Destination: DOGEstonia** after Location (REQ-31 §4 AC #1). |
| 0.3 | 2026-05-31 | **REQ-31 / GIM-136:** §5.2.0b «Dual-mode pre-submit preview» — Citizen Mode (human preview, forbidden lexicon, «Submit Story», transport abstraction) + God Mode (operator activation phrase, session-scoped `debug_mode`, `DEBUG MODE ACTIVE`, full JSON payload). Runs after §5.2.0a/§5.2.0, before §5.2.2/HTTP. §7.5 checklist extended. |
| 0.2 | 2026-05-29 | **GIM-135 / FINDING-01:** §5.2 execution order — §5.2.0a admission gate now runs **before** §5.2.0 PII pre-send (after draft mapping build); cross-refs updated. Closes [`req30-code-audit-report-2026-05-29.md`](../docs/analysis/req30-code-audit-report-2026-05-29.md) FINDING-01. Gate substance unchanged (GIM-133). |
| 0.1 | 2026-05-29 | **REQ-30 / GIM-133:** §5.2.0a «Admission gate — strict-chain package» — enforceable upstream artifact checks (`ingest_validation_report`, `safety_compliance_report`, `policy_gate_result`, `normalized_issue_payload`, explicit backend confirmation); test/sandbox production block with et/ru/en refusal templates; audit trace in `trace_notes`; default STOP message «Локальная валидация FAIL: данных недостаточно для отправки в API.»; §7.5 operator checklist extended. Additive to §5.2.2 checks 1–12 (GIM-28). |

---

