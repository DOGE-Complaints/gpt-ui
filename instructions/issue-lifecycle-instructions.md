# Issue — lifecycle in GPT instruction terms (DOGEstonia)

**Purpose:** Describe **ingest pipeline stages** from the instruction modules’ perspective: which artifacts and handoffs lead to backend-ready data, **without** equating them to the **`ISSUE_STATUS`** enum in UI/DB (`spa-app`).

**Related files:** [`issue-data-model.md`](./issue-data-model.md) (entity fields) · [`issue-interview-flow.md`](./issue-interview-flow.md) (conversation phases 1–7, REQ-08) · [`root.md`](./root.md) (backend authority) · [REQ-15](../docs/requirements/REQ-15-working-assumptions.md) · [technical-architecture.md](../docs/technical-architecture.md) §2–3.2

| Version | Date |
|---------|------|
| 0.3 | 2026-04-10 |

---

## 1. Role in Custom GPT

Per **openai-custom-gpt-builder**: behavior and **step order** live in instructions; this file is a **compact workflow reference** for modules (base, ingest, safety, gate, normalizer, orchestrator). Do not duplicate OpenAPI; do not assign product statuses without an API response body.

---

## 2. Stages “from the instructions’ viewpoint” (Issue track)

Names below are **logical dialogue/chain phases**, not JSON fields on the node. Order follows [technical-architecture.md](../docs/technical-architecture.md) §2 and the strict chain §3.2; for Issue the terminal artifact is **`normalized_issue_payload`** (planned — see epic M1-05).

| # | Stage (instructions) | Meaning | Typical artifacts / output |
|---|----------------------|---------|----------------------------|
| 1 | **INGEST / routing** | Issue creation mode; delegation into functional modules (`base.md`). | Module decision, context. |
| 2 | **Deep parsing** (conditional) | Non-trivial input → extraction before validation. | `deep_parsing_artifact` (strict). |
| 3 | **Narrative collection → Issue projection** | Dialogue and field extraction per [REQ-10](../docs/requirements/REQ-10-output-content-model.md) §10.5; completeness follow-ups. | Draft fields from §4.1 [`issue-data-model.md`](./issue-data-model.md). |
| 4 | **Ingest validation** | Required-field completeness, stop-the-line; **no API**. | `ingest_validation_report`; optional `gate_request_package` draft. |
| 5 | **Safety** | Checkpoints raw / extracted / validated / (later) normalized. | `safety_compliance_report`. |
| 6 | **Policy gate** (DOGEstonia) | Admission per operator rulebook; **no API**; not the same as “ticket status”. | `policy_gate_result` (reference pattern). |
| 7 | **Issue normalization** | Canonical JSON for orchestrator contract + metadata referencing steps 4–6. | `normalized_issue_payload` (target name; until module lands — align with `activity-normalizer` / `issue-normalizer`). |
| 8 | **API orchestrator** | Only module that initiates HTTP (Actions). | OpenAPI operation call; **server response** is source of truth for `id`, `status`, errors. |

Between steps, **stop-the-line** from `base.md` §1.5 applies: on validation/safety/gate blocks, **do not** proceed to normalizer and **do not** call API.

---

## 3. What this is NOT

| Confusion | Clarification |
|-----------|----------------|
| Phases 1–8 vs **`ISSUE_STATUS`** (`NEW`, `VERIFIED`, …) | Card status in SPA/node is owned by the **system** after persistence and business rules. GPT **must not** claim “ticket is VERIFIED” without API response body ([`root.md`](./root.md), [REQ-15](../docs/requirements/REQ-15-working-assumptions.md) §6). |
| Stages vs **interview phases 1–7** (REQ-08) | Question playbook: [`issue-interview-flow.md`](./issue-interview-flow.md) (**EPIC-M1-02**). Here — **engineering** module chain only. |
| Stages vs **Narrative layers REQ-10** | Layers §10.1–10.4 are content; table §2 is **process** and artifacts. |

---

## 4. Mapping “instruction stage ↔ UI/node status”

Until the node publishes canonical Issue OpenAPI — **explicit TBD**. After schema is available, add one row per operation (e.g. draft create → which `ISSUE_STATUS` the API returns).

| Instruction stage (§2) | Expected system side (draft table) |
|--------------------------|-------------------------------------|
| Before step 8 | No guaranteed persisted Issue; only in-dialog artifacts. |
| After successful step 8 | `id`, `status`, etc. **only** from JSON response; surface to user per [`root.md`](./root.md). |

---

## 5. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | STORY-GM1-02: first version aligned with technical-architecture §3.2 and issue-data-model. |
| 0.2 | 2026-04-10 | GM2-01: link to `issue-interview-flow.md`; clarified “stages vs REQ-08” row. |
| 0.3 | 2026-04-10 | **English-only** instruction text (repo policy). |
