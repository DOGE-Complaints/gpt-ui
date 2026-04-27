# Issue — canonical data model (GPT instruction layer)

**Product:** DOGEstonia — Module 1 (GPT Interview / ingest → Issue)  
**Purpose:** Logical **Issue** schema for instruction modules (validation, normalization, orchestrator). This is **not** the transport/OpenAPI schema for the node; if it diverges from HTTP contract, **OpenAPI wins after publication**, and gaps go to the product backlog.

| Document field | Value |
|----------------|--------|
| **Version** | 0.11 |
| **Date** | 2026-04-26 |
| **Traceability** | [`story-label-taxonomy.md`](story-label-taxonomy.md); [`story-i18n-policy.md`](story-i18n-policy.md) (FR-M1-028…031); [`story-normalizer.md`](story-normalizer.md) |

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
| **Logical content** | Narrative Core → Issue Projection: what was extracted and agreed with the resident. |
| **Card/dashboard fields** | `type`, `labels`, trilingual text, optional `institution` — see §4. |
| **Transport (node / JSON Schema)** | Wire field names and requiredness come from node OpenAPI; GPT **must not** treat model output as truth for publication status. |

---

## 3. Narrative layers → Issue fields mapping

| Layer | Role for Issue |
|--------------|----------------|
| Narrative Core | Raw material for `summary` / `description` (via interview). |
| Meaning Layer | Depth refinements; not separate mandatory UI fields in MVP SPA contract. |
| Civic Framing | Influences `labels`, `type` wording (civic genre vs observation). |
| Desired State | `description` / resident-facing summary content. |
| Issue Projection | **Direct source** of draft `type`, `labels`, `title`, `summary`, `description`, optional `institution`. |
| Confidence / Completeness | Report artifacts in ingest (`ingest_validation_report`, etc.) — not duplicated in this file. |

---

## 4. Downstream fields (UI alignment)

Below is the **target display contract** for the product UI (mocks and dashboards). Enum types for `type` and `status` must match the canonical Issue enums **`ISSUE_TYPE`** and **`ISSUE_STATUS`** (same symbol names as the SPA type module in the main product repo).

### 4.1 Required for logical Issue after interview (before API)

| Field | Type (logical) | Rule |
|-------|----------------|------|
| `type` | string enum | One of `ISSUE_TYPE`: `complaint`, `observation`, `absurdity`, `system_bug`. |
| `labels` | `string[]` | Controlled tag keys from [`story-label-taxonomy.md`](story-label-taxonomy.md) with disposition `canonical`. Unknown, free-text, internal-only, low-confidence, or metadata-only candidates must not enter `canonical_payload.labels[]`. |
| `title` | `{ et: string, ru: string, en: string }` | Draft title; all three keys filled meaningfully or explicitly flagged “needs translation” in validator report — see [`story-i18n-policy.md`](story-i18n-policy.md). |
| `description` | `{ et, ru, en }` | Full detail-page text. |
| `summary` | `{ et, ru, en }` (optional) | Short card text; if absent, UI may use `title`. |

### 4.2 Optional

| Field | Type | Rule |
|-------|------|------|
| `institution` | `{ et, ru, en }` | Agency / institution **if** product scope allows inferring it from interview. **Demo scope:** do **not** populate from dialogue; keep field **absent** / null in `canonical_payload` until integration matures. |

### 4.3 Subjective intake extensions (resident-perceived, non-wire)

Optional fields using subjective enums **`SEVERITY`**, **`IMPACT_ESTIMATION`**, **`PROBLEM_STATUS`**. Values are **subjective** (how the resident experiences impact), not an objective audit. These fields are **non-wire metadata** for the current runtime contract and must not be sent as `StoryIntakeRequest` fields until HTTP/OpenAPI explicitly includes them.

| Field | Type (logical) | Rule |
|-------|------------------|------|
| `severity` | `low` \| `medium` \| `high` \| `critical` | Resident-perceived seriousness; optional until interview collects it without leading the user. |
| `impact_estimation` | `personal` \| `city/town` \| `state` \| `country` \| `Earth` | Self-reported perceived scope of impact. |
| `problem_status` | `ongoing` \| `resolved` \| `worsened` | How the resident frames change over time, if stated. |

These should live in an explicit non-wire artifact (for example `ingest_validation_report`, mapping appendix, or a logical sidecar) until HTTP/OpenAPI lockstep explicitly includes them (imported Actions contract + SSOT updates in lockstep). They are not required for §4.1 completeness and must not block StoryIntake handoff.

### 4.4 Not filled by GPT as facts without backend

| Field | Why |
|-------|-----|
| `id` | Issued by node/store. |
| `status` | Values `NEW` / `VERIFIED` / `IN_REVIEW` / `ARCHIVED` — **only** from API response or backend rules; GPT must not assert final status verbally ([`root.md`](root.md)). |
| `created_at` | System timestamp. |
| `arweave_txid`, `image_txid`, `image_hash` | Only real integration values from API responses; **do not invent**. |

---

## 5. Source of truth for enums

- **Types and statuses:** product enums `ISSUE_TYPE`, `ISSUE_STATUS` — match §4 literal sets above for instruction-layer work.
- **Labels:** [`story-label-taxonomy.md`](story-label-taxonomy.md) — controlled axes, canonical allowed keys, metadata-only candidates, internal-only labels, and unknown-value handling.

If UI mocks disagree with this file on `title`/`summary` shape (string vs `{ et, ru, en }`), for **Module 1 GPT → UI** priority is **§4.1** (trilingual objects) unless the deployed OpenAPI explicitly locks a different wire shape.

---

## 6. PII and safety

Do not collect PII by default; do not store personal data in Issue content beyond operator policy.

---

## 7. Related instruction modules (planned)

| Module | Link to Issue |
|--------|----------------|
| [`story-lifecycle-instructions.md`](story-lifecycle-instructions.md) | Ingest phase order and strict-chain artifacts **in instruction terms** (not UI statuses). |
| [`story-policy-gate.md`](story-policy-gate.md) | Policy admission; `policy_gate_result` / `review_metadata` alignment; no API. |
| [`story-label-taxonomy.md`](story-label-taxonomy.md) | Controlled source of truth for `labels` and label extraction metadata boundaries. |
| `ingest-validation.md` | §4.1 completeness, batch follow-ups. |
| [`story-normalizer.md`](story-normalizer.md) | Canonical JSON for orchestrator / **`normalized_issue_payload`**. |
| `api-orchestrator.md` | Only HTTP entrypoint; response interpretation is authoritative. |
| `root.md` | Forbid false claims about backend/Gate/status. |

---

## 8. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | First draft; narrative-to-Issue mapping and UI mock alignment. |
| 0.2 | 2026-04-10 | Link to [`story-lifecycle-instructions.md`](story-lifecycle-instructions.md). |
| 0.3 | 2026-04-10 | **English-only** instruction text (repo policy). |
| 0.4 | 2026-04-10 | Added §4.1 `title` row link to [`story-i18n-policy.md`](story-i18n-policy.md). |
| 0.5 | 2026-04-10 | Added §7 link to [`story-policy-gate.md`](story-policy-gate.md) (`policy_gate_result`). |
| 0.6 | 2026-04-10 | Added §7 link to [`story-normalizer.md`](story-normalizer.md) (`normalized_issue_payload` scaffold). |
| 0.7 | 2026-04-20 | Added §4.3 subjective intake fields; added demo `institution` scope rule; aligned UI enum references. |
| 0.8 | 2026-04-22 | Added donor-era structured minors metadata and temporarily moved the system-only block (GIM-65). |
| 0.9 | 2026-04-25 | Removed donor-era minors metadata from active Issue model; restored system-only block to §4.4 (GIM-72). |
| 0.10 | 2026-04-25 | Clarified subjective intake fields as non-wire metadata for current runtime contract (GIM-77). |
| 0.11 | 2026-04-26 | Linked `labels` to controlled label taxonomy and forbade unknown/free-text/internal label values in canonical payload (GIM-86). |
