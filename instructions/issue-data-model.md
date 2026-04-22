# Issue — canonical data model (GPT instruction layer)

**Product:** DOGEstonia — Module 1 (GPT Interview / ingest → Issue)  
**Purpose:** Logical **Issue** schema for instruction modules (validation, normalization, orchestrator). This is **not** the transport/OpenAPI schema for the node; if it diverges from HTTP contract, **OpenAPI wins after publication**, and gaps are recorded in `REQ-16` / `docs/DOGEstonia.md` §2.3.

| Document field | Value |
|----------------|--------|
| **Version** | 0.7 |
| **Date** | 2026-04-20 |
| **Traceability** | [REQ-10](../docs/requirements/REQ-10-output-content-model.md) §10.5–10.6, [REQ-15](../docs/requirements/REQ-15-working-assumptions.md) §4–6; [`issue-i18n-policy.md`](./issue-i18n-policy.md) (FR-M1-028…031); [`issue-normalizer.md`](./issue-normalizer.md); SPA: [`spa-app/docs/mock-layer-issues-guide.md`](../../spa-app/docs/mock-layer-issues-guide.md), [`spa-app/src/domain/types.js`](../../spa-app/src/domain/types.js) |

---

## 1. Role in Custom GPT (separation of concerns)

Per Custom GPT methodology:

- **Instructions** encode behavior and workflow; they **do not** duplicate long API specs.
- This file is a **domain module reference**: single place for Issue fields during extraction, validation, and normalization **before** Actions calls (same structural role the repo formerly used for a separate entity schema — that schema file is now a **stub**; use **git history** if needed).

HTTP calls remain only in `api-orchestrator.md` per OpenAPI contract (epic M1-06).

---

## 2. Logical package vs transport

| Layer | Description |
|-------|-------------|
| **Logical content** | Narrative Core → Issue Projection ([REQ-10](../docs/requirements/REQ-10-output-content-model.md) §10.1–10.5): what was extracted and agreed with the resident. |
| **Card/dashboard fields** | `type`, `labels`, trilingual text, optional `institution` — see §4. |
| **Transport (node / JSON Schema)** | Wire field names and requiredness come from node OpenAPI; GPT **must not** treat model output as truth for publication status. |

---

## 3. REQ-10 → Issue fields mapping

| REQ-10 layer | Role for Issue |
|--------------|----------------|
| §10.1 Narrative Core | Raw material for `summary` / `description` (via interview). |
| §10.2 Meaning Layer | Depth refinements; not separate mandatory UI fields in MVP SPA contract. |
| §10.3 Civic Framing | Influences `labels`, `type` wording (civic genre vs observation). |
| §10.4 Desired State | `description` / resident-facing summary content. |
| §10.5 Issue Projection | **Direct source** of draft `type`, `labels`, `title`, `summary`, `description`, optional `institution`. |
| §10.6 Confidence / Completeness | Report artifacts in ingest (`ingest_validation_report`, etc.) — not duplicated in this file. |

---

## 4. Downstream fields (SPA alignment)

Below is the **target display contract** for `spa-app` (mocks and UI). Enum types for `type` and `status` must match [`spa-app/src/domain/types.js`](../../spa-app/src/domain/types.js) (`ISSUE_TYPE`, `ISSUE_STATUS`).

### 4.1 Required for logical Issue after interview (before API)

| Field | Type (logical) | Rule |
|-------|----------------|------|
| `type` | string enum | One of `ISSUE_TYPE`: `complaint`, `observation`, `absurdity`, `system_bug`. |
| `labels` | `string[]` | Tag keys (as in mocks). New keys — per UI rules (`AVAILABLE_LABELS` and dictionaries); see mock-guide. |
| `title` | `{ et: string, ru: string, en: string }` | Draft title; all three keys filled meaningfully or explicitly flagged “needs translation” in validator report — see [`issue-i18n-policy.md`](./issue-i18n-policy.md). |
| `description` | `{ et, ru, en }` | Full detail-page text. |
| `summary` | `{ et, ru, en }` (optional) | Short card text; if absent, UI may use `title` ([mock-layer-issues-guide](../../spa-app/docs/mock-layer-issues-guide.md)). |

### 4.2 Optional

| Field | Type | Rule |
|-------|------|------|
| `institution` | `{ et, ru, en }` | Agency / institution **if** product scope allows inferring it from interview. **Demo scope (REQ-16 Q5):** do **not** populate from dialogue; keep field **absent** / null in `canonical_payload` until integration matures. |

### 4.3 Subjective intake extensions (resident-perceived, REQ-16 Q4)

Optional fields aligned with `IssueIntakePayload` enums in [`spa-app/src/domain/types.js`](../../spa-app/src/domain/types.js) — **`SEVERITY`**, **`IMPACT_ESTIMATION`**, **`PROBLEM_STATUS`**. Values are **subjective** (how the resident experiences impact), not an objective audit.

| Field | Type (logical) | Rule |
|-------|------------------|------|
| `severity` | `low` \| `medium` \| `high` \| `critical` | Resident-perceived seriousness; optional until interview collects it without leading the user. |
| `impact_estimation` | `personal` \| `city/town` \| `state` \| `country` \| `Earth` | Self-reported perceived scope of impact. |
| `problem_status` | `ongoing` \| `resolved` \| `worsened` | How the resident frames change over time, if stated. |

These may live beside §4.1 in `canonical_payload` or in a sibling object per [`issue-normalizer.md`](./issue-normalizer.md) until HTTP/OpenAPI lockstep explicitly includes them (YAML/SSOT updates in lockstep).

### 4.4 Not filled by GPT as facts without backend

| Field | Why |
|-------|-----|
| `id` | Issued by node/store. |
| `status` | Values `NEW` / `VERIFIED` / `IN_REVIEW` / `ARCHIVED` — **only** from API response or backend rules; GPT must not assert final status verbally ([`root.md`](./root.md), [REQ-15](../docs/requirements/REQ-15-working-assumptions.md) §6). |
| `created_at` | System timestamp. |
| `arweave_txid`, `image_txid`, `image_hash` | Only real integration values; **do not invent** ([mock-layer-issues-guide](../../spa-app/docs/mock-layer-issues-guide.md) §5). |

---

## 5. Source of truth for enums

- **Types and statuses:** `spa-app/src/domain/types.js` — `ISSUE_TYPE`, `ISSUE_STATUS`.
- **Sample objects and i18n:** `spa-app/docs/mock-layer-issues-guide.md`.

If JSDoc in `types.js` disagrees with mock-guide on `title`/`summary` shape (string vs `{ et, ru, en }`), for **Module 1 and GPT→SPA** priority is **mock-guide** and §4.1 (trilingual objects).

---

## 6. PII and safety

Follow [REQ-15](../docs/requirements/REQ-15-working-assumptions.md) §5: do not collect PII by default; do not store personal data in Issue content beyond operator policy.

---

## 7. Related instruction modules (planned)

| Module | Link to Issue |
|--------|----------------|
| [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) | Ingest phase order and strict-chain artifacts **in instruction terms** (not UI statuses). |
| [`issue-policy-gate.md`](./issue-policy-gate.md) | Policy admission; `policy_gate_result` / `review_metadata` alignment; no API. |
| `ingest-validation.md` | §4.1 completeness, batch follow-ups. |
| [`issue-normalizer.md`](./issue-normalizer.md) | Canonical JSON for orchestrator / **`normalized_issue_payload`**. |
| `api-orchestrator.md` | Only HTTP entrypoint; response interpretation is authoritative. |
| `root.md` | Forbid false claims about backend/Gate/status. |

---

## 8. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | First draft; REQ-10/REQ-15 and SPA mock-guide alignment. |
| 0.2 | 2026-04-10 | Link to [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md). |
| 0.3 | 2026-04-10 | **English-only** instruction text (repo policy). |
| 0.4 | 2026-04-10 | Added §4.1 `title` row link to [`issue-i18n-policy.md`](./issue-i18n-policy.md). |
| 0.5 | 2026-04-10 | Added §7 link to [`issue-policy-gate.md`](./issue-policy-gate.md) (`policy_gate_result`). |
| 0.6 | 2026-04-10 | Added §7 link to [`issue-normalizer.md`](./issue-normalizer.md) (`normalized_issue_payload` scaffold). |
| 0.7 | 2026-04-20 | Added §4.3 subjective intake fields; added demo `institution` scope rule; kept `spa-app` path updates. |
