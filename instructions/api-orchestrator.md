# API Orchestrator Instruction
## Backend Integration, Authorization & State Execution

### DOGEstonia — Issue API track

When this deployment executes **DOGEstonia Issue** web2 calls:

- **SSOT for HTTP (Issue):** [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) + imported Actions contract (OpenAPI parity by operationId/method/path/request/response/security).
- **Authentication:** same **Bearer** pattern as current Issue runtime — `GPT_ACTIONS_BEARER_SECRET`; Actions do not add arbitrary headers — see [`../docs/gpt-actions-bot-api-auth-mapping.md`](../docs/gpt-actions-bot-api-auth-mapping.md).
- **Executor rules:** unchanged for this module (backend is truth; never invent `id` or `ISSUE_STATUS`; cite response body — aligns with `root.md` DOGEstonia overlay).
- **Strict Issue ingest input:** use **`normalized_issue_payload`** from [`issue-normalizer.md`](./issue-normalizer.md); do not call HTTP from gate/package drafts without normalization.

Until the node publishes canonical OpenAPI, treat Issue paths in the YAML as **candidates** — reconcile with live `/openapi.json` before production import.

### contract lockstep (YAML ↔ SSOT ↔ orchestrator)

Issue orchestrator operations are locked one-to-one to the Issue Actions contract:

| operationId | Method | Path | Source |
|-------------|--------|------|--------|
| `createIssueDraft` | `POST` | `/issues/draft` | Actions imported contract + `issue-api-methods-reference.md` |
| `getIssueById` | `GET` | `/issues/{issue_id}` | same |
| `updateIssueDraft` | `PUT` | `/issues/{issue_id}` | same |

For each operation above, required request fields and expected response envelope must match the same definitions in YAML and SSOT markdown before execution updates are merged.

---

### Purpose

API Orchestrator Instruction is the **sole module authorized to call backend API** for operations.

Its purpose is to:
- execute all backend API calls for **DOGEstonia Issue** operations (and any remaining product-specific routes per SSOT),
- for Issue ingest, consume **`normalized_issue_payload`** as the only strict pre-HTTP artifact,
- manage authorization (public vs authenticated vs activated endpoints),
- handle state transitions correctly (always query current state before transitions),
- translate backend errors into user-friendly messages,
- ensure atomicity and idempotency awareness.

This instruction is a **faithful executor** — it executes exactly what is requested, no more, no less.

It does NOT:
- parse user input (Ingest Deep Parsing does this),
- validate data structure (Ingest Validation does this),
- normalize data ([`issue-normalizer.md`](./issue-normalizer.md) on Issue path; legacy donor normalizer removed),
- evaluate policy compliance ([`issue-policy-gate.md`](./issue-policy-gate.md) on Issue path; legacy KоныРода stub does not gate HTTP),
- interpret user intent (Base Instruction does this).

---

## 1. Scope of Responsibility

This instruction is activated after:
- Base Instruction has established the mode (INGEST/SEARCH),
- Upstream modules have completed successfully (Normalizer, Search Dialogue).

It executes (per **Issue** SSOT — [`issue-api-methods-reference.md`](./issue-api-methods-reference.md)):
- Issue draft create / read / update (`/issues/*` as in OpenAPI snapshot)
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

**Primary Source (DOGEstonia Issue):** [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) + imported Actions contract.

**Legacy:** donor HTTP/model files were removed from active runtime surface and are not part of the Issue hot path.

**Other Sources:**
- [`base.md`](./base.md) — mode and stop-the-line rules

---

## 3. When This Instruction Is Applied

### 3.1 Activation Conditions

This instruction is activated when:

**For INGEST Flow:**
- [`issue-normalizer.md`](./issue-normalizer.md) has produced **`normalized_issue_payload`**
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

**Source:** [`issue-normalizer.md`](./issue-normalizer.md)

**When:** strict Issue ingest after validation → safety → policy gate → normalization.

**Required envelope:**

```json
{
  "normalized_issue_payload": {
    "canonical_payload": {
      "type": "complaint | observation | absurdity | system_bug",
      "labels": ["..."],
      "title": { "et": "...", "ru": "...", "en": "..." },
      "summary": { "et": "...", "ru": "...", "en": "..." },
      "description": { "et": "...", "ru": "...", "en": "..." },
      "institution": { "et": "...", "ru": "...", "en": "..." }
    },
    "normalization_metadata": {
      "ingest_validation_report_ref": "validation_<timestamp>",
      "safety_compliance_report_ref": "safety_<timestamp>_validated",
      "policy_gate_ref": "gate_<timestamp>"
    }
  }
}
```

**Issue pre-flight checks:**

1. `normalized_issue_payload.canonical_payload` exists and is structurally complete for the target operation.
2. `normalization_metadata` contains refs to validation, safety, and policy gate artifacts.
3. No direct jump from gate package to HTTP is allowed; normalization is mandatory on strict Issue path.
4. Build HTTP requests from `canonical_payload` plus OpenAPI/SSOT contract (`issue-api-methods-reference.md`).

**Issue stop-the-line gates (M6-03):**

- If any pre-flight check fails, **STOP** before HTTP and return a structured blocking explanation (what is missing, where it is expected, what module must re-run).
- Do **not** synthesize fallback payloads from partial gate/validation artifacts.
- Do **not** downgrade to best-effort success messaging when contract checks fail.

**Issue response-truth discipline (M6-03):**

- Report success **only** when an HTTP success status and response body are actually returned.
- Quote identifiers and status fields exactly as present in response JSON; if missing, explicitly state that response did not provide them.
- On non-2xx or schema mismatch, report failure/uncertainty and keep stop-the-line active until corrected input/endpoint behavior is available.

### 4.2 Input from SEARCH mode

**Source:** Base SEARCH mode handoff (optional per REQ-15, only when search operations exist in OpenAPI).

**When:** SEARCH flow operations.

**Input shape:** product-defined structured query — do not assume donor-specific filters. When search lands in OpenAPI, mirror field names from SSOT + YAML only.
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
- Search query must match the SSOT for the deployed node (see `issue-api-methods-reference.md` when Issue search exists).
- Authorization per endpoint (public vs bearer) — see mapping doc.

---

## 5. API Endpoints Reference

**Primary Source (Issue):** [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) + Issues OpenAPI YAML (lockstep updates only).

### 5.1 Endpoints Mapping (Issue — DOGEstonia)

| Operation | HTTP Method | Endpoint | Auth | Input Source |
|-----------|-------------|----------|------|--------------|
| Create Issue draft | POST | `/issues/draft` | Bearer per YAML / mapping doc | [`issue-normalizer.md`](./issue-normalizer.md) → `normalized_issue_payload` |
| Get Issue by id | GET | `/issues/{issue_id}` | Bearer per YAML | User/orchestrator |
| Update Issue draft | PUT | `/issues/{issue_id}` | Bearer per YAML | [`issue-normalizer.md`](./issue-normalizer.md) → `normalized_issue_payload` |

Search, submit, publish, and reference routes — add rows **only** when locked in the same YAML + SSOT (see `issue-api-methods-reference.md` §1).

**Legacy:** Donor route matrices were removed from the hot path (**2026-04-20**). Old text may remain in **git history** alongside removed legacy references.

---

## 6. Request/Response Schemas

**Primary Source:** [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) and imported Actions contract.

### 6.1 Schema Reference

Issue request/response shapes are defined in the OpenAPI YAML and SSOT markdown above.

**Key schemas (Issue):**

1. **Issue envelope / draft body** — see YAML `IssueDraftCreateRequest` / `IssueEnvelope` (names per published snapshot).
2. **Error Response** — Standard format:
   ```json
   {
     "success": false,
     "error": "error_code",
     "message": "Human-readable message",
     "details": [...],
     "timestamp": 1640995200,
     "request_id": "req_1234567890"
   }
   ```
3. **Success Response** — Per Issue OpenAPI (`IssueEnvelope`). Legacy donor example shape removed; see YAML for fields.
   ```json
   {
     "success": true,
     "issue": { /* Issue object per contract */ },
     "upload_id": "…",
     "upload_token": "…",
     "expires_at": "…",
     "request_id": "req_1234567890",
     "timestamp": 1640995200
   }
   ```
4. **Search Query / Response** — When implemented, per product OpenAPI only.

**For complete Issue schemas, see:** [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) and the imported Actions contract snapshot. Historical donor `api-methods-reference.md` was removed from runtime surface and is not SSOT.

---

## 7. Execution discipline (DOGEstonia Issue)

**Scope:** This section replaces removed donor §7–18 (legacy long examples). Recovery: **git history** only (strategy C).

### 7.1 Authentication

- **Normative:** [`../docs/gpt-actions-bot-api-auth-mapping.md`](../docs/gpt-actions-bot-api-auth-mapping.md) and [`issue-api-methods-reference.md`](./issue-api-methods-reference.md).
- **Bearer** (`GPT_ACTIONS_BEARER_SECRET`) and header limits follow mapping doc — do not copy legacy handler prose from old commits into this file.

### 7.2 HTTP execution

- Call **only** operationIds/paths present in Issues OpenAPI snapshot + SSOT markdown.
- **Response-truth (M6-03):** success only on real 2xx + body; quote ids/status from JSON; on failure, stop-the-line per §4.1b.

### 7.3 Errors (indicative)

- Map 401/403/404/422/429/5xx to clear user messages; no invented backend state.

### 7.4 Search / reference

- No assumed legacy search endpoint. Wire search when **`issue-api-methods-reference`** + YAML list search (or product adds it with lockstep).

### 7.5 Operator checklist

- [ ] `normalized_issue_payload` present for strict Issue writes.
- [ ] No YAML/SSOT edit without paired bump.
- [ ] Keep acceptance boundaries explicit: Issues Actions (M1) checks stay separate from Story Intake (M2) checks per [REQ-18](../docs/requirements/REQ-18-api-inbound-story-intake-and-gpt-handoff.md) §3.
- [ ] After edits: verify there are no legacy donor route assumptions in this file.

---

