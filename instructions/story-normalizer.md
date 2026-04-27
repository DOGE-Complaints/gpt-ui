# Issue Normalizer — canonical Issue JSON (instruction layer)

**Product:** DOGEstonia — Module 1 (GPT Interview → Issue)  
**Purpose:** Turn **policy-approved**, validated Issue material into a single **deterministic** logical package — **`normalized_issue_payload`** — for downstream [`api-orchestrator.md`](./api-orchestrator.md) (HTTP only there). This module is **structural**: it does not re-run validation, safety, or policy admission.

| Document field | Value |
|----------------|--------|
| **Version** | 0.1.8 |
| **Date** | 2026-04-26 |
| **Traceability** | [REQ-09](../docs/requirements/REQ-09-functional-requirements.md) FR-M1-035–037; [REQ-10](../docs/requirements/REQ-10-output-content-model.md) §10.5; [REQ-20](../docs/requirements/REQ-20-label-taxonomy-and-extraction-axes.md); [`story-data-model.md`](./story-data-model.md) §4.1; [`story-label-taxonomy.md`](./story-label-taxonomy.md); [`story-lifecycle-instructions.md`](./story-lifecycle-instructions.md) §2.1; [`story-policy-gate.md`](./story-policy-gate.md); [technical-architecture.md](../docs/technical-architecture.md) §3.2, §7.2–7.4; strict-chain alignment with `base` / `ingest-validation` / `safety-compliance` |

---

## 1. Custom GPT path (classification)

**Plain Custom GPT** — this module is **instructions only**: **no GPT Actions**, **no HTTP**, **no backend calls**.  
The model emits a logical JSON-shaped artifact for the next module (`api-orchestrator.md`) to consume per OpenAPI when Actions are enabled.

---

## 2. Role and boundaries

### 2.1 This instruction MUST

- Run **only after** [`story-policy-gate.md`](./story-policy-gate.md) has produced `policy_gate_result.status = "approved"` for the same ingest handoff (see [`story-lifecycle-instructions.md`](./story-lifecycle-instructions.md) **§2.1**).
- Emit **`normalized_issue_payload`** with:
  - **`canonical_payload`** — Issue fields aligned with [`story-data-model.md`](./story-data-model.md) **§4.1** (`type`, `labels`, `title`, `description`, optional `summary`, optional `institution`; trilingual objects `{ et, ru, en }` per that file and [`story-i18n-policy.md`](./story-i18n-policy.md)).
  - **`normalization_metadata`** — **references** to upstream strict-chain artifacts (see §6), plus optional label extraction metadata, not a full duplicate of raw interview text or multimodal sources.
- Preserve **conservative** typing: enums and label keys must match **Issue** SoT ([`story-data-model.md`](./story-data-model.md) §4–5 and [`story-label-taxonomy.md`](./story-label-taxonomy.md)).

### 2.2 This instruction MUST NOT

- **Call APIs or GPT Actions** — ever; orchestrator owns HTTP.
- **Create transport request bodies** such as `IssueDraftCreateRequest`, `StoryIntakeRequest`, or `IssueCreateRequest`. This module emits only `normalized_issue_payload`; transport shaping belongs to [`api-orchestrator.md`](./api-orchestrator.md) or a runtime bridge.
- **Ask the user** clarification or follow-up questions (same separation as the legacy normalizer reference: normalization is not a dialogue step). Missing data must have been resolved **upstream** (`ingest-validation`, interview flow, or gate **`needs_clarification`** loop), not here.
- **Parse raw** multimodal input — belongs to ingest deep parsing / validation.
- **Re-evaluate** structural completeness — belongs to [`ingest-validation.md`](./ingest-validation.md).
- **Re-run** safety or policy — belongs to [`safety-compliance.md`](./safety-compliance.md) and [`story-policy-gate.md`](./story-policy-gate.md).
- **Invent** `id`, `status`, `created_at`, `arweave_txid`, `image_txid`, `image_hash`, or any backend-issued field ([`story-data-model.md`](./story-data-model.md) §4.4). Omit them or set explicit placeholders only if the orchestrator contract requires keys (then mark as `null` / `"pending_backend"` per **M1-06** alignment — do not fabricate values).

### 2.3 Source of truth for Issue shape

- **Authoritative for Module 1 Issue normalization:** [`story-data-model.md`](./story-data-model.md).  
- **Do not** use removed donor-era paths or donor schema names as SoT; this file + [`story-data-model.md`](./story-data-model.md) are authoritative for Issue normalization.

---

## 3. Inputs (logical)

The normalizer consumes a **handoff package** produced by upstream modules (names may match `gate_request_package` / validated context from the gate module). Minimum logical content:

| Input | Description |
|-------|-------------|
| **Validated Issue fields** | Draft values for §4.1 fields (from interview + validation), already meeting ingest rules for the current step. |
| **`policy_gate_result`** | Must be **`approved`**; include `policy_ref`, `rulebook_version` for traceability in metadata. |
| **Pointers to upstream artifacts** | Stable references (e.g. logical ids, timestamps, or one-line summaries) to **`ingest_validation_report`**, relevant **`safety_compliance_report`** checkpoints, and the **gate** result — for `normalization_metadata` (§6). |

If `policy_gate_result.status` is not **`approved`**: **do not** emit `normalized_issue_payload` for API-bound ingest; return control to lifecycle / gate (same pattern as legacy reference normalizer after rejection).

---

## 4. Output — `normalized_issue_payload` (top level)

Single final envelope:

```json
{
  "normalized_issue_payload": {
    "canonical_payload": {
      "type": "complaint",
      "labels": ["transport", "accessibility"],
      "title": {
        "et": "Primary-language or translated title",
        "ru": "Primary-language or translated title",
        "en": "Primary-language or translated title"
      },
      "description": {
        "et": "Primary-language or translated description",
        "ru": "Primary-language or translated description",
        "en": "Primary-language or translated description"
      },
      "summary": {
        "et": "Optional short card text",
        "ru": "Optional short card text",
        "en": "Optional short card text"
      }
    },
    "normalization_metadata": {
      "session_language": "et",
      "ingest_validation_report_ref": "validation_<id>",
      "safety_compliance_report_ref": "safety_<id>",
      "policy_gate_ref": {
        "policy_ref": "policy_<id>",
        "rulebook_version": "v1",
        "status": "approved"
      },
      "label_extraction_metadata": {
        "candidates": [
          {
            "label": "transport",
            "axis": "topic_domain",
            "source": "ingest_validation_report",
            "confidence": "high",
            "disposition": "canonical"
          }
        ]
      },
      "normalizer_module": "issue-normalizer@0.1.8",
      "trace_notes": []
    },
    "non_wire_metadata": {
      "severity": null,
      "impact_estimation": null,
      "problem_status": null
    }
  }
}
```

Omit `summary` if no validated short text exists. Omit `institution` from `canonical_payload` in demo scope unless product explicitly lifts REQ-16 Q5. Omit `non_wire_metadata` entirely when subjective fields were not stated or confirmed upstream.

### 4.1 `canonical_payload`

Must conform to [`story-data-model.md`](./story-data-model.md) **§4.1** (required logical fields) and **§4.2** (optional logical fields), using the **Issue** enums and trilingual rules documented there. System-only fields in **§4.4** are not filled by GPT as facts.

| Field | Rule |
|-------|------|
| `type` | One of `ISSUE_TYPE` values per issue-data-model / SPA SoT. |
| `labels` | String array; keys must be `canonical` labels allowed by [`story-label-taxonomy.md`](./story-label-taxonomy.md). Do not include metadata-only, internal-only, unknown, or low-confidence candidates. |
| `title`, `description` | `{ et, ru, en }` per §4.1 and i18n policy. |
| `summary` | Optional `{ et, ru, en }`; if omitted, orchestrator/UI may fall back per REQ-10 / mock guide. |
| `institution` | Optional `{ et, ru, en }` **only** when product scope allows; **demo default:** omit (REQ-16 Q5). |

Do not add donor-era, legacy-only, Search-only, or backend-issued fields to `canonical_payload`.

### 4.2 `normalization_metadata` (required keys — instruction-layer scaffold)

Stable **references** to upstream work (opaque strings or objects — align with **M1-06** when JSON Schema is published):

| Key | Purpose |
|-----|---------|
| `session_language` | **Required** for Issue ingest: `et` \| `ru` \| `en` — MUST match the primary interview language from [`bootstrap.md`](./bootstrap.md) **`comm_context.ui_lang`** (see [`story-i18n-policy.md`](./story-i18n-policy.md) §1–2). Backend readers use this to know which `{ et, ru, en }` slot was primary-authored. |
| `ingest_validation_report_ref` | Reference to the validation artifact used (id, hash, or short summary line). |
| `safety_compliance_report_ref` | Reference to relevant safety checkpoint output for this handoff. |
| `policy_gate_ref` | At minimum: `policy_ref`, `rulebook_version`, and `policy_gate_result.status` copy or stable id. |
| `normalizer_module` | e.g. `issue-normalizer` + **version** of this instruction file (from document header). |
| `trace_notes` | Optional: free-text **internal** consistency notes (not for end-user display). |
| `label_extraction_metadata` | Optional label candidate metadata from validation; stores label, axis, source, confidence, and disposition. It is not copied to transport unless schema changes in lockstep. |

#### 4.2.1 `label_extraction_metadata`

When validation supplies label reasoning, keep it under `normalization_metadata.label_extraction_metadata`:

```json
{
  "candidates": [
    {
      "label": "transport",
      "axis": "topic_domain",
      "source": "Phase 2 resident story / validation report",
      "confidence": "high",
      "disposition": "canonical"
    },
    {
      "label": "predictability",
      "axis": "deep_need",
      "source": "Phase 4 reframe accepted by user",
      "confidence": "medium",
      "disposition": "metadata_only"
    }
  ]
}
```

Only candidates with `disposition = "canonical"` and keys allowed by [`story-label-taxonomy.md`](./story-label-taxonomy.md) may appear in `canonical_payload.labels[]`. Candidates with `metadata_only`, `needs_clarification`, or `rejected` disposition remain in metadata only.

### 4.3 `non_wire_metadata` (optional sidecar)

`severity`, `impact_estimation`, and `problem_status` are resident-perceived subjective fields from [`story-data-model.md`](./story-data-model.md) **§4.3**. If collected and confirmed upstream, place them in optional `non_wire_metadata`; otherwise omit this sidecar.

`non_wire_metadata` is not a transport object. It must not be copied into current `StoryIntakeRequest`, `IssueCreateRequest`, or `IssueDraftCreateRequest` unless a separate OpenAPI/schema task explicitly adds those fields.

---

## 5. REQ-10 alignment (brief)

Narrative layers (REQ-10 §10.1–10.4) feed **content** inside `title` / `summary` / `description` / `labels` / `type` — see [`story-data-model.md`](./story-data-model.md) §3. This module **projects** already validated material into the §4.1 card shape; it does not re-author the interview.

### 5.1 REQ-10 projection → Issue card fields

| REQ-10 layer | Projection target in `canonical_payload` |
|--------------|------------------------------------------|
| §10.1 Narrative core | factual base for `summary` / `description` |
| §10.2 Meaning layer | depth and framing in `description`; influences `labels` |
| §10.3 Civic framing | `type` + civic taxonomy `labels` |
| §10.4 Desired state | action-oriented `description`; optional `institution` hypothesis (FR-M1-027, conservative) |
| §10.5 Issue projection | final `title` / `summary` / `description` + optional `institution` |

---

## 6. Relationship to other modules

| Module | Relationship |
|--------|----------------|
| [`ingest-validation.md`](./ingest-validation.md) | Upstream — completeness / batch rules. |
| [`safety-compliance.md`](./safety-compliance.md) | Upstream — safety checkpoints before or beside gate per lifecycle. |
| [`story-policy-gate.md`](./story-policy-gate.md) | Immediate upstream — **approval** required. |
| [`api-orchestrator.md`](./api-orchestrator.md) | Downstream — **only** module that performs HTTP and consumes `normalized_issue_payload`. |
| [`base.md`](./base.md) | §1.5 — Issue strict chain uses **`normalized_issue_payload`** as the canonical normalization artifact. |

---

## 7. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | Initial scaffold: `normalized_issue_payload`, `canonical_payload`, `normalization_metadata`, Plain GPT, no API, no user questions; Issue SoT = `story-data-model.md` §4.1. |
| 0.1.1 | 2026-04-10 | Added §6 cross-link to `base.md` §1.5 Issue artifact alignment. |
| 0.1.2 | 2026-04-10 | Added §5.1 concise REQ-10 projection table for `title` / `summary` / `description` / optional `institution`. |
| 0.1.3 | 2026-04-20 | Added required `normalization_metadata.session_language`; optional subjective intake fields in `canonical_payload`; demo `institution` omit. |
| 0.1.4 | 2026-04-22 | Added donor-era minors metadata per `issue-data-model` §4.4 (GIM-65). |
| 0.1.5 | 2026-04-25 | Removed donor-era minors metadata from `canonical_payload`; system-only reference restored to §4.4 (GIM-72). |
| 0.1.6 | 2026-04-25 | Clarified subjective intake fields as non-wire metadata for current runtime contract (GIM-77). |
| 0.1.7 | 2026-04-26 | Locked final `normalized_issue_payload` shape, optional `non_wire_metadata` sidecar, and no-direct-transport rule (GIM-80). |
| 0.1.8 | 2026-04-26 | Added label extraction metadata and canonical label disposition rules (GIM-89). |
