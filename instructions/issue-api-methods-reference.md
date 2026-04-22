# Issue API — SSOT reference (DOGEstonia / GPT Actions)

**Version:** 0.5 · 2026-04-21  
**Scope:** Module 1 Issue API SSOT  
**OpenAPI build/import artifact:** [`../docs/custom-gpt-issues-reference.openapi.yaml`](../docs/custom-gpt-issues-reference.openapi.yaml)  
**Auth / Actions constraints:** [`../docs/gpt-actions-bot-api-auth-mapping.md`](../docs/gpt-actions-bot-api-auth-mapping.md)  
**SPA field contract:** [`../../spa-app/docs/mock-layer-issues-guide.md`](../../spa-app/docs/mock-layer-issues-guide.md)

This file is the **HTTP source of truth for the Issue track** in instructions. Legacy donor route references are removed from active runtime surface. Until the node publishes canonical OpenAPI, paths below are **candidates**; before production, reconcile with `GET /openapi.json` on the deployed API.

**lock source:** no `openapi.json` snapshot is currently committed in this repo, so parity is locked to the checked-in import artifact + Actions runtime configuration (`info.version: 0.1.3`).

---

## 1. Operations (parity matrix)

| operationId | Method | Path | Request contract | Success envelope | Purpose |
|-------------|--------|------|------------------|------------------|---------|
| `createIssueDraft` | POST | `/issues/draft` | `IssueDraftCreateRequest` (required: `type`, `labels`, `title`, `description`) | `IssueEnvelope` (`201`) | Create Issue draft after normalizer. |
| `getIssueById` | GET | `/issues/{issue_id}` | path param `issue_id` (required) | `IssueEnvelope` (`200`) | Read Issue by `id` from API response. |
| `updateIssueDraft` | PUT | `/issues/{issue_id}` | path param `issue_id` + `IssueDraftCreateRequest` body | `IssueEnvelope` (`200`) | Update draft payload. |

Additional operations (submit, search, reference) — add when node contract exists; mirror in this table and in YAML with one-to-one method/path lock.

---

## 2. Authorization

- **GPT Actions:** `security: GptActionsBearer` in YAML; UI value = **`GPT_ACTIONS_BEARER_SECRET`** on server.
- Do not assume arbitrary headers from ChatGPT; user identity per node policy / `mock_user` (see mapping doc).
- **Current identity model (M6-04):** app-level bearer integration key; no per-user OAuth claims are assumed in this contract.
- Any per-user identity model requires explicit backend/proxy design and must not be implied by this SSOT.

---

## 2.1 Actions runtime discipline (no filename dependency)

- Runtime behavior must depend on the imported **Actions contract** (`operationId`, method/path, request/response schema, security), not on a local filename string.
- Local YAML path in this repository is a **build/import artifact** for operator workflow only.
- Before release, verify parity across:
  1. Imported Actions schema (UI),
  2. this SSOT table,
  3. deployed API `GET /openapi.json` (when available).

---

## 3. Requirements traceability

| Source | Note |
|--------|------|
| REQ-09 FR-M1-035–038 | Ingest exit → API; do not invent `id` / status. |
| REQ-15 §6 | Status outside GPT; API response is truth. |
| `api-orchestrator.md` | Only module that initiates HTTP; DOGEstonia overlay at top of file. |

---

## 4. OpenAPI version lockstep policy

When contract fields change, update **both** artifacts in one change set:

1. OpenAPI import artifact (`custom-gpt-issues-reference.openapi.yaml`) → `info.version`
2. this file (`issue-api-methods-reference.md`) → `Version`
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
| 0.1 | 2026-04-10 | First draft + YAML 0.1.0 (STORY-M6-01). |
| 0.2 | 2026-04-10 | **English-only** instruction text (repo policy). |
| 0.3 | 2026-04-10 | **M6-02:** one-to-one parity matrix (operationId/method/path/request/response) + lock source note (YAML snapshot 0.1.1, no committed node `/openapi.json`). |
| 0.4 | 2026-04-10 | **M6-04:** explicit app-level bearer identity assumption + version lockstep policy for OpenAPI/SSOT updates. |
| 0.5 | 2026-04-21 | Runtime decoupled from local YAML filename; lock source/version updated to 0.1.3 and Actions contract parity rule clarified. |
