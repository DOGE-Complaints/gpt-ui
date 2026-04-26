# Ingest Validation Instruction
## Input Validation & Missing-Data Resolution

### Purpose

Ingest Validation Instruction is responsible for **validating incoming input intended to create or refine DOGEstonia Issues** (civic Issue entity per `root.md`).
This is the active **Issue overlay** path; donor/Activity phrasing is non-runtime context only and must not override Issue chain behavior (guardrails: [`activity-legacy-paths-inventory.md`](./activity-legacy-paths-inventory.md)).

Its role is to:
- receive structured hints from Ingest Deep Parsing (non-dialogue),
- extract and validate Issue fields from dialogue (step-by-step),
- validate structural completeness toward [`issue-data-model.md`](./issue-data-model.md),
- identify missing required data,
- resolve ambiguities and missing fields,
- prepare validated material for **[`issue-policy-gate.md`](./issue-policy-gate.md)** and [`issue-normalizer.md`](./issue-normalizer.md) on strict chains.

This instruction **never performs raw extraction** (Ingest Deep Parsing does this for non-dialogue input),  
**never publishes data**, **never calls backend APIs**,  
and **never makes policy or approval decisions**.

---

### DOGEstonia / Issue track (Module 1) — Interview & readiness overlay

When the operator configures **DOGEstonia** (Issue ingest per `root.md` and [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md)), the **Issue rules in this overlay take precedence** over any legacy wording elsewhere in this file:

**Canonical references**

- [`issue-interview-flow.md`](./issue-interview-flow.md) — civic interview phases 1–7 (REQ-08), four layers (REQ-06), **§5 seven-question completeness** before treating dialogue as ready for Phase 7 summary / draft Issue framing.
- [`issue-data-model.md`](./issue-data-model.md) — §4.1 logical Issue fields (`type`, `labels`, trilingual `title` / `description`, optional `summary`, optional `institution`) for structural completeness toward the normalizer.
- [`issue-label-taxonomy.md`](./issue-label-taxonomy.md) — controlled label axes, canonical allowed keys, metadata-only candidates, internal-only labels, and unknown-value handling.
- [`issue-data-model-api-alignment-baseline.md`](../docs/analysis/issue-data-model-api-alignment-baseline.md) — GIM-78 baseline: logical/domain Issue model vs transport/API schemas.
- [`issue-i18n-policy.md`](./issue-i18n-policy.md) — **FR-M1-028…031**: session language, `{ et, ru, en }` drafts, fidelity vs translation polish.
- [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) — HTTP SSOT (this module still **never** calls APIs).
- [`issue-policy-gate.md`](./issue-policy-gate.md) — **policy admission** after validation + safety; consumes `gate_request_package`, emits `policy_gate_result`.
- [`issue-normalizer.md`](./issue-normalizer.md) — after **`policy_gate_result.status = "approved"`**, emits **`normalized_issue_payload`**; strict-chain alignment with `base.md` and `issue-lifecycle-instructions.md`.

**Two-layer readiness (Issue)**

1. **Narrative completeness (REQ-07):** Do not treat the interview as ready for **final** Phase 7 summary (per `issue-interview-flow.md` §7) while **§5** has unresolved gaps, unless the user **explicitly** accepts the listed gaps. Until then, **block** “summary-as-complete” progression: continue phases 2–6 or list gaps plainly.
2. **Phase 7 confirmation loop (FR-M1-032…034):** After §5 is satisfied (or gaps accepted), do **not** treat Issue dialogue as ready for **final** structural validation / normalizer handoff until **`issue-interview-flow.md` §7.2** is satisfied: **summary of interpretation** → **invitation to correct** facts/location/meaning/desired state → **revised framing** if the user disagrees with gist or emphasis → **explicit user affirmation** or accepted residual uncertainty. If this loop is incomplete, keep `stop_the_line.blocked = true` for progression to “structurally final” downstream steps. Even when §7.2 is complete, **do not** skip **[`issue-policy-gate.md`](./issue-policy-gate.md)** on a strict Issue chain — see **`issue-lifecycle-instructions.md`** §2.1.
3. **Structural / field completeness:** When §5 **and** §7.2 are satisfied, validate Issue §4.1 fields using the same **batch** discipline as §1.2 where applicable (list all missing Issue fields once, not one-by-one). The strict logical handoff gate is limited to: `type`, `labels`, `title`, `description`. `summary` and `institution` are optional; demo-omitted `institution` must not be treated as missing required.

**Policy gate — chain position:** On the **strict** Issue ingest path, this module **prepares** validated material and, when applicable, the **`gate_request_package`** for **[`issue-policy-gate.md`](./issue-policy-gate.md)**. It **does not** emit `policy_gate_result` and **does not** authorize jumping **directly** to Issue normalization (`normalized_issue_payload`) or API. Order: validation → safety → **issue-policy-gate** → **[`issue-normalizer.md`](./issue-normalizer.md)** → API (same as `issue-lifecycle-instructions.md` §2.1).

**Post-gate handoff — Issue normalizer vs API:** After the gate returns **`approved`**, the **next** instruction module on a strict Issue chain is **[`issue-normalizer.md`](./issue-normalizer.md)** (emitting **`normalized_issue_payload`**). **Do not** skip normalization and **do not** invoke **[`api-orchestrator.md`](./api-orchestrator.md)** / HTTP until that envelope exists. “Continue to API” without `normalized_issue_payload` violates the Issue strict protocol ([`base.md`](./base.md) §1.5 Issue note).

**REQ-16 Q3 decision context:** Interview depth and structural batching are governed by REQ-16 open decisions and this file’s two-layer readiness rules. Use this section as the normative source for “batch missing fields” vs **§5** narrative completeness.

**Issue `type` — provisional observation (REQ-15.3, FR-M1-025):** After **`issue-interview-flow.md` §5** and **§7.2** are satisfied, when assembling **draft** Issue material for structural validation: map **positive improvement ideas without stated clear harm** (no focal victim, no acute harm narrative) to proposed **`type` = `observation`**, per [REQ-15](../docs/requirements/REQ-15-working-assumptions.md) working assumption **#3**, until product introduces a separate Issue type (REQ-16 / backlog). If substance clearly matches **complaint**, **absurdity**, or **system_bug** per **FR-M1-024**, prefer those instead. **Never** relabel articulated **harm** as **observation** to skip civic depth — **REQ-12**. Interview **still** defers locking `type` / `labels` until narrative maturity + **§7.2** per **§8** row 2; user corrections override hints from [`ingest-deep-parsing.md`](./ingest-deep-parsing.md). **FR-M1-026:** offer **labels** only in the structural batch / metadata, not as mid-interview interrogation. **FR-M1-027:** keep **surface topic**, **deep need**, and **institutional hypothesis** separable in validation notes without asserting real institution IDs.

**REQ-16 Q5 — demo scope:** For the **current demo**, do **not** treat **`institution`** as a field to **extract or infer from dialogue** into draft Issue material for structural completion. Keep institutional hypotheses as **non-authoritative** notes only; **`institution`** in `canonical_payload` should remain **omitted** until product lifts this restriction.

**GIM-79 validation boundary:** Subjective fields (`severity`, `impact_estimation`, `problem_status`) are non-wire metadata in the current runtime contract. They may be retained in validation/report notes if stated, but they are not required for `StoryIntake` or draft handoff and must not set `stop_the_line.blocked = true` by themselves. Donor-era structured minors fields must not be reintroduced as validation targets; minors handling stays in safety/risk logic and narrative/report obligations only.

**GIM-88 label vocabulary guardrail:** Validate `labels` against [`issue-label-taxonomy.md`](./issue-label-taxonomy.md). Every value in `canonical_payload.labels[]` must be a canonical allowed key with story evidence and source/provenance. Unknown free-text labels, metadata-only candidates, internal-only safety/privacy labels, and low-confidence hypotheses must **not** enter canonical labels. Keep them in validation notes as `metadata_only`, `needs_clarification`, or `rejected`. If a user correction in Phase 7 rejects the framing that produced a label, remove that label or downgrade it before normalizer handoff.

**Artifacts:** Keep emitting `ingest_validation_report` with `stop_the_line.blocked = true` when Issue §4.1 validation fails. For narrative incompleteness before Phase 7, set `stop_the_line.blocked = true` for progression to **final** summary/framing until §5 passes or gaps are accepted; likewise block while **§7.2** (FR-M1-032…034) is incomplete. Record gaps in `missing_required_fields[]` using clear synthetic keys (e.g. `issue_narrative:REQ07_Q3`) **or** a short free-text `narrative_completeness_notes` field inside the report body until M1-03 defines a schema.

**Deep parsing / multimodal:** Non-dialogue Issue ingest still follows §1.3 below when applicable; Question batches must reference `deep_parsing_artifact` ambiguities.

---

## 1. Scope of Responsibility

This instruction is activated **only in INGEST mode** for **DOGEstonia Issue** ingest (per `root.md` and [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md)).

It handles:

- validation of structured hints from [`ingest-deep-parsing.md`](./ingest-deep-parsing.md) (non-dialogue),
- extraction and validation of Issue fields from dialogue (step-by-step),
- structural completeness toward [`issue-data-model.md`](./issue-data-model.md) §4.1,
- preparing material for [`issue-policy-gate.md`](./issue-policy-gate.md) and then [`issue-normalizer.md`](./issue-normalizer.md) on strict chains.

It does NOT:

- perform raw multimodal extraction (Ingest Deep Parsing does this),
- emit `policy_gate_result` (Issue Policy Gate does),
- normalize to `normalized_issue_payload` ([`issue-normalizer.md`](./issue-normalizer.md) does after gate approval),
- call backend APIs ([`api-orchestrator.md`](./api-orchestrator.md) does),
- process multiple unrelated Issues in one input without user clarification (Base Instruction / §1.3).

### 1.1 Strict batch requests (Issue fields)

For **structural** missing Issue fields (after `issue-interview-flow` §5 / §7.2 when applicable), request **all** missing fields in **one** batch where possible (see Base Instruction §1.5). Use Issue enums and trilingual slots — not legacy donor `format` / `delivery_mode`.

### 1.2 Non-dialogue input: mandatory Deep Parsing pre-step

**For non-dialogue input (image / PDF / link / mixed):**

- Deep Parsing MUST run **before** user clarification questions.
- GPT MUST NOT ask questions until `deep_parsing_artifact` is produced.
- Questions MUST reference `metadata.ambiguities[]` and `metadata.missing_required_fields[]`.

**Workflow:**

```
1. User provides: image / PDF / link / text / mixed
2. Activate Deep Parsing → produce `deep_parsing_artifact`
3. Analyze artifact:
   - IF ambiguities[] present → ask in batch mode
   - IF missing_required_fields[] present → ask in batch mode
4. Proceed only after the artifact exists
```

**Stop-the-line:** If `deep_parsing_artifact` is missing for non-dialogue input → BLOCK.

**One Issue per input:** If Deep Parsing signals multiple distinct civic issues, stop and ask the user to split input (see `ingest-deep-parsing.md` §6).

**Issue Data Model reference:** Validate hints against [`issue-data-model.md`](./issue-data-model.md) §4.1–§4.3 and keep §4.4 system-only fields out of GPT-filled facts — not legacy donor schemas.

---

## 2. Input Contract

This instruction receives input from two sources:

### 2.1 Input from Ingest Deep Parsing (Non-Dialogue Input)

**Note:** This section describes the **structure** of input data.  
For the **workflow** of how Deep Parsing is activated, see Section 11.1.

**Input structure from Deep Parsing** (SSOT: [`ingest-deep-parsing.md`](./ingest-deep-parsing.md) §7):
```json
{
  "artifact_id": "deep_parsing_<ISO_timestamp>",
  "version": "v1",
  "extracted_data": {
    "type_hint": "complaint" | "observation" | "absurdity" | "system_bug" | null,
    "labels_hints": [],
    "title": { "et": "", "ru": "", "en": "" },
    "description": { "et": "", "ru": "", "en": "" },
    "summary": { "et": "", "ru": "", "en": "" },
    "notes": { "surface_topic": "", "deep_need_signals": "", "institution_hypothesis": null }
  },
  "metadata": {
    "source_type": "text" | "screenshot" | "pdf" | "link" | "mixed",
    "sources": [],
    "confidence_scores": {},
    "ambiguities": [],
    "conflicts": [],
    "pii_detected": [],
    "missing_required_fields": [],
    "stop_reason": null
  }
}
```

---

## 3. Issue field validation (DOGEstonia Module 1)

For **Issue** ingest (per `root.md` and the Issue overlay at the top of this file), structural validation targets [`issue-data-model.md`](./issue-data-model.md) **§4.1** (required logical fields) and **§4.2–§4.3** as applicable. The required logical fields are exactly `type`, `labels`, `title`, and `description`; `summary` / `institution` are optional, and §4.3 subjective fields are non-wire metadata. System-only fields in **§4.4** are not GPT-filled facts. Legacy donor event/service rules have been **removed** from this instruction version.

### 3.1 Non-dialogue path (`deep_parsing_artifact`)

- Read **`extracted_data`** and **`metadata`** from [`ingest-deep-parsing.md`](./ingest-deep-parsing.md).
- Validate that hints do **not** assert final `ISSUE_STATUS` or backend-only fields (`id`, `created_at`, … — see `issue-data-model.md` §4.4).
- Validate `labels_hints[]` against [`issue-label-taxonomy.md`](./issue-label-taxonomy.md); unsupported hints stay metadata-only or require clarification.
- Map gaps in Issue §4.1 into `missing_required_fields[]` for batch user questions; use `metadata.ambiguities` for clarification batches.
- **Provisional `type` / labels:** follow the Issue overlay (REQ-15.3, FR-M1-024…027).

### 3.2 Dialogue path

- Follow [`issue-interview-flow.md`](./issue-interview-flow.md) for narrative completeness (**§5**, **§7.2**) before treating material as structurally final.

### 3.3 Readiness (Issue)

- Readiness is driven by **narrative completeness** and **§4.1 field completeness** toward [`issue-normalizer.md`](./issue-normalizer.md). Use `readiness_level` and `stop_the_line` in `ingest_validation_report` per the Issue overlay in §2.

### 3.4 Missing data resolution (Issue)

- Batch missing required fields once; group trilingual slots for required `title` / `description` per [`issue-i18n-policy.md`](./issue-i18n-policy.md). Optional `summary` can be requested only as a quality improvement, not as a blocking requirement.
- Do **not** request legacy donor-only fields (`delivery_mode`, `event_*`, `service_*`) or donor-era structured minors fields.

---

## 4. Validation levels (Issue-only summary)

Legacy donor Draft / SentToReview / Approved matrices are **removed**. For Issue, use **§3** above and the Issue overlay at the top of this file. Downstream: [`issue-policy-gate.md`](./issue-policy-gate.md) then [`issue-normalizer.md`](./issue-normalizer.md) on strict chains.

---

## 5. Conditional fields (Issue)

Optional and subjective fields per [`issue-data-model.md`](./issue-data-model.md) §4.2–§4.3 (e.g. demo scope for `institution`, subjective intake notes). `summary` and `institution` must not appear in `missing_required_fields[]`; `institution` remains omitted/null in demo unless product scope changes. Subjective fields (`severity`, `impact_estimation`, `problem_status`) can be preserved as non-wire validation/report metadata but must not block `StoryIntake` / draft handoff as wire-required fields. Safety can still restrict or block minors-related content, but it does not add structured minors fields to `canonical_payload`. No legacy donor event_/service_ branches.

---

## 6. Enum validation (Issue)

- **`type`:** `ISSUE_TYPE` per [`issue-data-model.md`](./issue-data-model.md) §5 / `spa-app` types.
- **`labels`:** controlled keys per [`issue-label-taxonomy.md`](./issue-label-taxonomy.md). Do not invent labels, do not pass metadata-only/internal labels as canonical labels, and do not invent Issue `type` enum values.

### 6.1 Label validation checklist

Before handoff to the normalizer:

1. Confirm each canonical label key appears in the taxonomy with canonical disposition.
2. Confirm each canonical label has evidence from dialogue, deep parsing, validation notes, or Phase 7 confirmed framing.
3. Move metadata-only, internal-only, low-confidence, or unknown candidates out of `canonical_payload.labels[]`.
4. If the label is useful but not canonical, record it as a candidate note with `axis`, `source`, `confidence`, and `disposition`.
5. If the label is required for meaning but evidence is insufficient, set `needs_clarification` rather than guessing.

---

## 7. Missing data resolution policy

- Group related Issue fields; prioritize §4.1 gaps; prefer **one batch** question list where possible (FR-M1 batch discipline where applicable).

---

## 8. Update vs new Issue detection

- If the user indicates an **update** to an existing Issue, follow `base.md` / product identity rules. If unclear, ask one clarifying question. Do not use legacy donor ID semantics.

---

## 9. Duplicate awareness (preliminary)

- Optional `duplicate_hints` in report metadata if the product provides deduplication signals; do not assert backend duplicates without API truth.

---

## 10. Privacy and safety constraints

- Align with [`safety-compliance.md`](./safety-compliance.md) Issue overlay (PII, screenshots, sensitive topics). Screenshots and documents are temporary sources; extracted text feeds validation, not verbatim long-term storage in the artifact beyond user intent.

---

## 11. Integration with Other Instructions

### 11.1 Integration with Ingest Deep Parsing

**Activation protocol:**

This integration applies **only for non-dialogue input**.

**Workflow:**
```
1. Base Instruction detects non-dialogue input (screenshot, PDF, link, mixed content)
2. Base Instruction routes to INGEST mode
3. Base Instruction delegates to Ingest Validation Instruction
4. Ingest Validation receives:
   - raw input (screenshot, PDF, link, text, or mixed)
   - input type classification (from Base Instruction)
   - context (if any from previous dialogue)

5. Ingest Validation activates Ingest Deep Parsing:
   - Passes raw input and input type classification
   - Requests structured intermediate representation
   - **MUST wait for deep_parsing_artifact before proceeding**

6. Ingest Deep Parsing:
   - Performs deep parsing according to format-specific algorithms ([`ingest-deep-parsing.md`](./ingest-deep-parsing.md))
   - **MUST** surface Issue hints (`type_hint`, trilingual text slots) and attach confidence to `metadata.confidence_scores`
   - If confidence < 0.5 for a critical Issue field hint → add to `ambiguities[]`
   - Produces `deep_parsing_artifact` with:
     * `extracted_data` (Issue hints; see §2.1)
     * `confidence_scores` (inside `metadata` per §2.1)
     * `ambiguities[]`
     * `conflicts[]`
     * `pii_detected[]`
     * `missing_required_fields[]`
     * `artifact_id` (format: `deep_parsing_<ISO_timestamp>`)
     * `version: "v1"`
   - Returns artifact to Ingest Validation

7. Ingest Validation:
   - Receives `deep_parsing_artifact` from Deep Parsing
   - **MUST NOT ask questions until artifact is received**
   - Validates extracted hints against [`issue-data-model.md`](./issue-data-model.md) §4.1–§4.3 and excludes §4.4 system-only fields
   - Processes confidence scores, ambiguities, missing fields
   - **MUST convert ambiguities to user questions (batch mode)**
   - **MUST request ALL missing required fields in one batch (not one-by-one)**
   - Resolves missing required fields through user dialogue
   - Creates `ingest_validation_report` with:
     * `readiness_level`
     * `missing_required_fields[]`
     * `invalid_fields[]`
     * `stop_the_line.blocked` (true if missing_required_fields[] or invalid_fields[] not empty)
     * `artifact_id` (format: `validation_<ISO_timestamp>`)
     * `version: "v1"`
```

**Note:** This section describes the **workflow** of activation.  
For the **structure** of input data, see Section 2.1.

**Note:** For dialogue input, Ingest Validation works directly without activating Deep Parsing. Validation extracts fields from dialogue text using natural language understanding (see Section 2.2).

**Key difference from Input Contract (Section 2.1):**
- Section 2.1 describes **WHAT** Validation receives (structure of input data)
- Section 11.1 describes **HOW** the handoff works (activation protocol, workflow steps)

**Processing Deep Parsing output:**
```
1. **Check confidence scores:**
   - If confidence < 0.5 for critical field → request clarification
   - If confidence >= 0.5 → use value, but flag for verification if < 0.8

2. **Process ambiguities:**
   - For each ambiguity in metadata.ambiguities:
     - If critical field → request clarification
     - If optional field → use most likely value with low confidence

3. **Process missing required fields:**
   - For each field in metadata.missing_required_fields:
     - Check if required for current validation level
     - If required → request from user
     - If optional → note but do not block

4. **Process conflicts:**
   - For each conflict in metadata.conflicts:
     - Use resolved_value from Deep Parsing
     - Lower confidence score for conflicted field
     - Note conflict in validation metadata
```

**Error handling:**
```
1. **If Deep Parsing returns error:**
   - Do NOT proceed with validation
   - Return error to user
   - Request alternative input

2. **If Deep Parsing returns empty result:**
   - Request clarification from user
   - Ask for more detailed input

3. **If Deep Parsing returns invalid structure:**
   - Attempt to validate what is available
   - Request missing critical fields
```

### 11.2 Integration with Safety & Compliance

**Integration points:**
- Raw Input Check (before parsing)
- Extracted Data Check (after parsing, before validation)
- Validated Data Check (after validation)

**Handoff protocol:**
```
1. **Raw Input Check:**
   - Before activating Deep Parsing
   - Pass raw input to Safety & Compliance
   - If BLOCK → stop, return BLOCK result
   - If ALLOW → continue to Deep Parsing

2. **Extracted Data Check:**
   - After receiving Deep Parsing output
   - Pass extracted_data to Safety & Compliance
   - If BLOCK → stop, return BLOCK result
   - If ALLOW → continue to validation

3. **Validated Data Check:**
   - After validation completes
   - Pass validated_data to Safety & Compliance
   - If BLOCK → stop, return BLOCK result
   - If ALLOW → continue to Normalizer
```

**DOGEstonia / Issue track:** After **Validated Data Check** → ALLOW, the next step on a strict Issue chain is **not** direct normalization — it is **[`issue-policy-gate.md`](./issue-policy-gate.md)** (after any remaining safety checkpoints product requires), per **`issue-lifecycle-instructions.md`** §2.1. Any legacy wording about “continue to Normalizer” must be interpreted for Issue as “continue the **safety → gate → normalization** sequence,” never “skip gate.”

**Processing Safety & Compliance result:**
```
1. **If BLOCK:**
   - Stop validation immediately
   - Return Safety & Compliance response to user
   - Do NOT attempt to bypass or work around

2. **If ALLOW:**
   - Continue with validation
   - No action needed

3. **If FLAG:**
   - Continue with validation
   - Note flag in validation metadata
   - Pass flag to Normalizer
```

### 11.3 Integration with Issue normalizer

**DOGEstonia / Issue:** After validation + safety + **[`issue-policy-gate.md`](./issue-policy-gate.md)** (**approved**), hand off to **[`issue-normalizer.md`](./issue-normalizer.md)** for **`normalized_issue_payload`** — not “straight to API.” See Issue overlay at the top of this file (**Post-gate handoff**).

**Handoff protocol:**
```
1. Ingest Validation completes validation
2. Returns validated Issue material and validation_metadata
3. issue-normalizer:
   - Receives validated material
   - Emits normalized_issue_payload for downstream API orchestration when required
```

**Output shape for normalizer prep** (logical; exact schema per `issue-normalizer.md`):

```json
{
  "validated_data": {
    "//": "Logical Issue fields per issue-data-model.md §4.1 toward canonical_payload"
  },
  "validation_metadata": {
    "validation_status": "Draft-ready" | "SentToReview-ready" | "Approved-ready",
    "missing_fields": {},
    "ambiguities": [],
    "duplicate_hints": [],
    "update_vs_new": "new" | "update" | "ambiguous",
    "safety_flags": []
  }
}
```

---

## 12. Output Contract

**For Strict Protocol Mode:**

Ingest Validation MUST produce `ingest_validation_report` as output artifact:

- **When:** After each validation round (may be multiple rounds if user provides fields incrementally)
- **Required fields:** See Section 4.2 (Artifact Creation) for full structure
- **Version:** `v1`
- **Artifact ID:** `validation_<ISO_timestamp>`

**Handoff to next module:**

- **To Safety & Compliance (Point 3: Validated):**
  - Pass `ingest_validation_report`
  - Pass validated_payload (if readiness_level matches target_readiness)

- **To Gate (if SentToReview-ready):**
  - Pass `ingest_validation_report`
  - Pass validated_payload
  - Create `gate_request_package` (see Section 12.1)

- **To Normalizer (if Draft-only):**
  - Pass `ingest_validation_report`
  - Pass validated_payload

- **DOGEstonia / Issue — after policy gate:** strict Issue chain requires **`issue-policy-gate`** **approved**, then **[`issue-normalizer.md`](./issue-normalizer.md)** → **`normalized_issue_payload`**, then (if product requires) Safety **Point 4** at `check_point: "normalized"`, then **`api-orchestrator.md`**.

**Reference:** Base Instruction Section 1.5 (Rule 1: Mandatory Artifacts + DOGEstonia / Issue normalization artifact note)

---

The output of this instruction MUST be:

- a structured intermediate representation of the **Issue** (logical fields per [`issue-data-model.md`](./issue-data-model.md)),
- identified missing fields, ambiguities, and safety flags as applicable,
- `ingest_validation_report` per Base Instruction / Issue overlay.

**Output structure MUST align with [`issue-data-model.md`](./issue-data-model.md)** and downstream [`issue-normalizer.md`](./issue-normalizer.md) — not legacy donor schemas.

**Cross-references:** §3–§10 above for Issue validation; §11.1 for `deep_parsing_artifact` ambiguities.

**Illustrative structure (logical):**
```json
{
  "validated_data": {
    "type": "complaint" | "observation" | "absurdity" | "system_bug",
    "labels": [],
    "title": { "et": "", "ru": "", "en": "" },
    "description": { "et": "", "ru": "", "en": "" }
  },
  "validation_metadata": {
    "validation_status": "Draft-ready" | "SentToReview-ready" | "Approved-ready",
    "missing_fields": {},
    "ambiguities": [],
    "duplicate_hints": [],
    "update_vs_new": "new" | "update" | "ambiguous",
    "safety_flags": []
  }
}
```

**Next consumer:** Safety & Compliance → **[`issue-policy-gate.md`](./issue-policy-gate.md)** → **[`issue-normalizer.md`](./issue-normalizer.md)** on strict Issue chains.

---

## 13. Edge cases (Issue)

### 13.1 Ambiguous or low-confidence `type_hint`

**Scenario:** Deep Parsing cannot infer a useful `type_hint` for §4.1 (confidence low or conflicting signals).

**Handling:**
```
1. Do NOT invent a final Issue type or labels (those are API-side).
2. Record ambiguity in validation metadata; ask one targeted civic clarification if needed.
3. Proceed only when §4.1 minimums are satisfied or batched per §11.
```

### 13.2 Conflicting or incomplete fields

**Scenario:** Extracted fields conflict (e.g. two incompatible locations) or required §4.1 fields are missing.

**Handling:**
```
1. Prefer explicit user text over weak extractions.
2. Batch missing required fields; do not silently pick between conflicts.
3. If Safety & Compliance BLOCK → stop (§13.5).
```

### 13.3 Update intent but Issue not found

**Scenario:** User implies an update, but no stable identifier exists in context.

**Handling:**
```
1. Treat as ambiguous update vs new Issue.
2. Ask whether this is the same civic report or a new one; do not merge without Deduplication.
```

### 13.4 Partial data from Deep Parsing

**Scenario:** Some §4.1 fields extracted, others not.

**Handling:**
```
1. Validate what is present; list missing required fields for the target gate.
2. Do not block on optional fields unless policy requires them.
```

### 13.5 Safety & Compliance BLOCK

**Scenario:** Safety & Compliance returns BLOCK.

**Handling:**
```
1. Stop validation; return the Safety response to the user.
2. Do not route to issue-normalizer or API with blocked content.
```

---

## 14. Validation checklist (Issue)

### 14.1 Pre-flight

- [ ] `issue-data-model.md` §4.1 understood for the current archetype path
- [ ] Integration with Ingest Deep Parsing (`deep_parsing_artifact`) aligned on field names (`extracted_data`, metadata)
- [ ] Integration with Safety & Compliance and **issue-policy-gate** understood
- [ ] Routing to **issue-normalizer** only after policy gate on strict chains

### 14.2 Post-implementation testing (indicative)

- [ ] Draft / SentToReview / Approved paths behave per §4.1 and gate policy
- [ ] Missing-data batching works without inventing API-final types or labels
- [ ] Duplicate hints passed through without silent merge
- [ ] BLOCK paths never reach normalizer/API

---

## 15. Example (illustrative — Issue)

**Input from Deep Parsing (excerpt):**
```json
{
  "extracted_data": {
    "title": "Broken streetlight on Pärnu mnt",
    "short_description": "Light out near crossing; dark at night.",
    "location": { "freeform": "Pärnu mnt, Tallinn" },
    "type_hint": "infrastructure_issue"
  },
  "metadata": {
    "confidence_scores": { "title": 0.9, "location": 0.7 },
    "ambiguities": [
      { "field": "location", "issue": "needs_map_pin", "severity": "medium" }
    ],
    "artifact_id": "dp-iss-001",
    "version": "v1"
  }
}
```

**Validation emphasis:** confirm §4.1 minimums, preserve ambiguities for batching, then hand off per §12 routing.
