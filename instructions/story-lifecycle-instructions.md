# Issue — lifecycle in GPT instruction terms (DOGEstonia)

**Purpose:** Describe **ingest pipeline stages** from the instruction modules’ perspective: which artifacts and handoffs lead to backend-ready data, **without** equating them to the **`ISSUE_STATUS`** enum in UI/DB.

**Related files:** [`story-data-model.md`](story-data-model.md) (entity fields) · [`story-interview-flow.md`](story-interview-flow.md) (conversation phases 1–7) · [`story-policy-gate.md`](story-policy-gate.md) (strict-chain admission) · [`story-normalizer.md`](story-normalizer.md) · [`root.md`](root.md) (backend authority) · [`ingest-validation.md`](ingest-validation.md) · [`base.md`](base.md)

| Version | Date |
|---------|------|
| 0.7 | 2026-07-06 |
| 0.6 | 2026-04-10 |
| 0.5 | 2026-04-10 |
| 0.4 | 2026-04-10 |
| 0.3 | 2026-04-10 |

---

## 1. Role in Custom GPT

Per **openai-custom-gpt-builder**: behavior and **step order** live in instructions; this file is a **compact workflow reference** for modules (base, ingest, safety, gate, normalizer, orchestrator). Do not duplicate OpenAPI; do not assign product statuses without an API response body.

---

## 2. Stages “from the instructions’ viewpoint” (Issue track)

Names below are **logical dialogue/chain phases**, not JSON fields on the node. Order follows the **strict-chain** discipline described in this file and [`story-policy-gate.md`](story-policy-gate.md); for Issue the terminal artifact is **`normalized_issue_payload`** ([`story-normalizer.md`](story-normalizer.md) scaffold).

| # | Stage (instructions) | Meaning | Typical artifacts / output |
|---|----------------------|---------|----------------------------|
| 1 | **INGEST / routing** | Issue creation mode; delegation into functional modules (`base.md`). | Module decision, context. |
| 2 | **Deep parsing** (conditional) | Non-trivial input → extraction before validation. | `deep_parsing_artifact` (strict). |
| 3 | **Narrative collection → Issue projection** | Dialogue and field extraction per narrative→Issue projection rules; completeness follow-ups. | Draft fields from §4.1 [`story-data-model.md`](story-data-model.md). |
| 4 | **Ingest validation** | Required-field completeness, stop-the-line; **no API**. | `ingest_validation_report`; optional `gate_request_package` draft. |
| 5 | **Safety** | Checkpoints raw / extracted / validated / (later) normalized. | `safety_compliance_report`. |
| 6 | **Policy gate** (DOGEstonia) | Admission per operator rulebook; **no API**; not the same as “ticket status”. | `policy_gate_result` (reference pattern). |
| 7 | **Issue normalization** | Canonical JSON for orchestrator contract + metadata referencing steps 4–6. | [`story-normalizer.md`](story-normalizer.md) → **`normalized_issue_payload`** (`canonical_payload` + `normalization_metadata`). |
| 8 | **API orchestrator** | Only module that initiates HTTP (Actions). | OpenAPI operation call; **server response** is source of truth for `id`, `status`, errors. |

Between steps, **stop-the-line** from `base.md` §1.5 applies: on validation/safety/gate blocks, **do not** proceed to normalizer and **do not** call API.

### 2.1 Mandatory Issue strict-chain order

For **DOGEstonia / Issue** ingest when using the **strict** artifact discipline (gate + normalized payload before HTTP), the engineering modules **after** narrative + structural readiness (§2 rows 2–4) MUST follow this order — **no skipping**:

1. **Ingest validation** — `ingest_validation_report`; may prepare **`gate_request_package`** (must **not** contain a gate **decision**).
2. **Safety & compliance** — `safety_compliance_report` at product-required checkpoints (see [`safety-compliance.md`](safety-compliance.md)).
3. **[`story-policy-gate.md`](story-policy-gate.md)** — consumes `gate_request_package` (+ references to validation/safety artifacts); emits **`policy_gate_result`**; **no API**.
4. **Issue normalization** — [`story-normalizer.md`](story-normalizer.md) → **`normalized_issue_payload`**.
5. **API orchestrator** — HTTP only here.

**Normative shorthand:** **validation → safety → policy gate → normalization → API**. Issue normalization **must not** run on a strict progression path **without** an explicit **`policy_gate_result`** (including outcomes under [`story-policy-gate.md`](story-policy-gate.md) §9 degraded mode), unless the operator documents a **narrower** exception in product config. [`base.md`](base.md) §1.5, [`ingest-validation.md`](ingest-validation.md) Issue overlay, and [`safety-compliance.md`](safety-compliance.md) Point **4** (Issue) all align on **`normalized_issue_payload`** / [`story-normalizer.md`](story-normalizer.md) — no API skip.

### 2.2 Boundary: Module 1 vs Module 2

This lifecycle describes **Module 1** execution only: strict Issue chain ending in `normalized_issue_payload` and Issues Actions/OpenAPI calls.

For **Module 2** Story Intake (`StoryIntakeRequest`, wire `m2.story_intake_envelope.v2`), follow [`api-orchestrator.md`](api-orchestrator.md) §5.2 and [`story-api-methods-reference.md`](story-api-methods-reference.md). **User runtime handoff** is **stash** (`postStoryDraftStash`) + browser redirect to SPA — not direct `POST /intake/stories` from GPT Actions. Service seed/sim may use `POST /intake/stories` (direct HTTP, not Actions). Do not treat M1 Issue API success as M2 story-intake success, and do not merge their acceptance checks into one step.

---

## 3. What this is NOT

| Confusion | Clarification |
|-----------|----------------|
| Phases 1–8 vs **`ISSUE_STATUS`** (`NEW`, `VERIFIED`, …) | Card status in SPA/node is owned by the **system** after persistence and business rules. GPT **must not** claim “ticket is VERIFIED” without API response body ([`root.md`](root.md)). |
| Stages vs **interview phases 1–7** | Question playbook: [`story-interview-flow.md`](story-interview-flow.md). Here — **engineering** module chain only. |
| Stages vs **narrative content layers** | Narrative layers are content; table §2 is **process** and artifacts. |

---

## 4. Mapping “instruction stage ↔ UI/node status”

Until the node publishes canonical Issue OpenAPI — **explicit TBD**. After schema is available, add one row per operation (e.g. draft create → which `ISSUE_STATUS` the API returns).

| Instruction stage (§2) | Expected system side (draft table) |
|--------------------------|-------------------------------------|
| Before step 8 | No guaranteed persisted Issue; only in-dialog artifacts. |
| After successful step 8 | `id`, `status`, etc. **only** from JSON response; surface to user per [`root.md`](root.md). |

---

## 5. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | First version aligned with strict-chain ingest discipline and Issue data model. |
| 0.2 | 2026-04-10 | Added link to `story-interview-flow.md`; clarified “stages vs interview phases” row. |
| 0.3 | 2026-04-10 | **English-only** instruction text (repo policy). |
| 0.4 | 2026-04-10 | Added §2.1 mandatory Issue strict order (validation → safety → `issue-policy-gate` → normalization → API); related-files link to `story-policy-gate.md`. |
| 0.5 | 2026-04-10 | Added pointers to [`story-normalizer.md`](story-normalizer.md) in §2 row 7, §2.1 step 4, intro; related-files. |
| 0.6 | 2026-04-10 | Added normative shorthand cross-links `base` §1.5 / `ingest-validation` / `safety-compliance` Point 4 (Issue) on **`normalized_issue_payload`**. |
| 0.7 | 2026-07-06 | **GIM-196 / GPT-SUBMIT-01 propagation:** §2.2 M2 user path = `postStoryDraftStash` + browser redirect; `/intake/stories` service seed/sim only. |
