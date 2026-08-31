# Issue Normalizer — canonical Issue JSON (instruction layer)

**Product:** DOGEstonia — Module 1 (GPT Interview → Issue)  
**Purpose:** Turn **policy-approved**, validated Issue material into a single **deterministic** logical package — **`normalized_issue_payload`** — for downstream [`api-orchestrator.md`](api-orchestrator.md) (HTTP only there). This module is **structural**: it does not re-run validation, safety, or policy admission.

| Document field | Value |
|----------------|--------|
| **Version** | 0.2.22 |
| **Date** | 2026-08-31 |
| **Traceability** | FR-M1-035–037; REQ-33; REQ-34; REQ-35; REQ-36; REQ-38; REQ-39; REQ-40; REQ-41; GPT-TAX-01; GPT-TAX-02 / GIM-227; GPT-PII-01 / GIM-241; GPT-SSR-01 / GIM-251 / GIM-253 / GIM-254; GPT-SSR-03 / GIM-268; [`story-data-model.md`](story-data-model.md) §4.1; [`story-label-taxonomy.md`](story-label-taxonomy.md); [`schema-packs/README.md`](schema-packs/README.md); [`story-lifecycle-instructions.md`](story-lifecycle-instructions.md) §2.1; [`story-policy-gate.md`](story-policy-gate.md); strict-chain alignment with `base` / `ingest-validation` / `safety-compliance` |

---

## 1. Custom GPT path (classification)

**Plain Custom GPT** — this module is **instructions only**: **no GPT Actions**, **no HTTP**, **no backend calls**.  
The model emits a logical JSON-shaped artifact for the next module (`api-orchestrator.md`) to consume per OpenAPI when Actions are enabled.

---

## 2. Role and boundaries

### 2.1 This instruction MUST

- Run **only after** [`story-policy-gate.md`](story-policy-gate.md) has produced `policy_gate_result.status = "approved"` for the same ingest handoff (see [`story-lifecycle-instructions.md`](story-lifecycle-instructions.md) **§2.1**).
- Emit **`normalized_issue_payload`** with:
  - **`canonical_payload`** — Issue fields aligned with [`story-data-model.md`](story-data-model.md) **§4.1** (`type`, `labels`, `title`, `description`, optional `summary`, optional `institution`; trilingual objects `{ et, ru, en }` per that file and [`story-i18n-policy.md`](story-i18n-policy.md)).
  - **`canonical_payload.taxonomy`** (GPT-TAX-01) — per-axis object `{ axis: [{ label, disposition }] }` for all retained candidates (including `internal` / `metadata_only`); do **not** collapse axes. Flat `labels[]` remains derived (canonical only).
  - **`normalization_metadata`** — **references** to upstream strict-chain artifacts (see §6), plus optional label extraction metadata, not a full duplicate of raw interview text or multimodal sources.
- **Label extraction:** apply **all** taxonomy keys that genuinely apply to the story — one or more keys per applicable axis (`topic_domain`, `failure_mode`, `civic_signal`, `issue_archetype_support`) where evidence exists in the validated narrative. **Conservative** means: do not invent labels or apply low-confidence keys — it does **not** mean use only one label total.
- Preserve **conservative** typing for enums and label keys: values must match **Issue** SoT ([`story-data-model.md`](story-data-model.md) §4–5 and [`story-label-taxonomy.md`](story-label-taxonomy.md)). Civic payload field names / geo canon for the **active node** come from the active [`schema-packs/`](schema-packs/README.md) data-model pack — not as the sole core model.

### 2.2 This instruction MUST NOT

- **Call APIs or GPT Actions** — ever; orchestrator owns HTTP.
- **Create transport request bodies** such as `IssueDraftCreateRequest`, `StoryDraftStashRequest`, or `IssueCreateRequest`. This module emits only `normalized_issue_payload`; transport shaping belongs to [`api-orchestrator.md`](api-orchestrator.md) (orchestrator builds `StoryDraftStashRequest` per §5.2.1).
- **Ask the user** clarification or follow-up questions (same separation as the legacy normalizer reference: normalization is not a dialogue step). Missing data must have been resolved **upstream** (`ingest-validation`, interview flow, or gate **`needs_clarification`** loop), not here.
- **Parse raw** multimodal input — belongs to ingest deep parsing / validation.
- **Re-evaluate** structural completeness — belongs to [`ingest-validation.md`](ingest-validation.md).
- **Re-run** safety or policy — belongs to [`safety-compliance.md`](safety-compliance.md) and [`story-policy-gate.md`](story-policy-gate.md).
- **Invent** `id`, `status`, `created_at`, `arweave_txid`, `image_txid`, `image_hash`, or any backend-issued field ([`story-data-model.md`](story-data-model.md) §4.4). Omit them or set explicit placeholders only if the orchestrator contract requires keys (then mark as `null` / `"pending_backend"` per **M1-06** alignment — do not fabricate values).

### 2.3 Source of truth for Issue shape

- **Authoritative for Module 1 Issue normalization:** [`story-data-model.md`](story-data-model.md).  
- **Do not** use removed donor-era paths or donor schema names as SoT; this file + [`story-data-model.md`](story-data-model.md) are authoritative for Issue normalization.

---

## 3. Inputs (logical)

The normalizer consumes a **handoff package** produced by upstream modules (names may match `gate_request_package` / validated context from the gate module). Minimum logical content:

| Input | Description |
|-------|-------------|
| **Validated Issue fields** | Draft values for §4.1 fields (from interview + validation), already meeting ingest rules for the current step. |
| **`policy_gate_result`** | Must be **`approved`**; include `policy_ref`, `rulebook_version` for traceability in metadata. |
| **Pointers to upstream artifacts** | Stable references (e.g. logical ids, timestamps, or one-line summaries) to **`ingest_validation_report`**, relevant **`safety_compliance_report`** checkpoints, and the **gate** result — for `normalization_metadata` (§6). |
| **`summary_draft` (optional)** | From Phase 7 Step 5b in [`story-interview-flow.md`](story-interview-flow.md): 1–2 sentences in `session_language`; use as primary source for `canonical_payload.summary[session_language]` before trilingual expansion. |

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
      "detected_input_language": "et",
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
    },
    "location_query": "Tartu mnt 80, Tallinn"
  }
}
```

Populate `canonical_payload.summary` per §4.1 **`summary` generation rule** (do not omit solely because upstream did not supply short text). Omit `institution` from `canonical_payload` always in demo scope (REQ-28 — see §4.1 constraint below); post-demo: omit unless all three slots `{et, ru, en}` are non-empty (REQ-23 §2.5). Omit `non_wire_metadata` entirely when subjective fields were not stated or confirmed upstream. Omit `live_story_context` when no narrative contradiction was detected (REQ-23 §3). Omit top-level `location_query` when location was not confirmed or string would be empty (REQ-26 §4.6).

### 4.1 `canonical_payload`

Must conform to [`story-data-model.md`](story-data-model.md) **§4.1** (required logical fields) and **§4.2** (optional logical fields), using the **Issue** enums and trilingual rules documented there. System-only fields in **§4.4** are not filled by GPT as facts.

| Field | Rule |
|-------|------|
| `type` | One of `ISSUE_TYPE` values per [`story-data-model.md`](story-data-model.md) §5 (display enum). Pack `signals.canonical_type` / payload field shapes: read the **active data-model pack**. Apply **observation vs complaint decision rule** below. |
| `taxonomy` | **GPT-TAX-01 SoT:** object keyed by taxonomy axis (13 axes in [`story-label-taxonomy.md`](story-label-taxonomy.md) §3 / gateway `TAXONOMY_AXIS_VALUES`). Each axis → array of `{ "label", "disposition" }`. Group candidates from `label_extraction_metadata` **by axis** — never flatten away the axis. Include dispositions `canonical`, `metadata_only`, `needs_clarification`, `rejected`, and **`internal`** (for §6 internal-only keys). Omit empty axes. Orchestrator maps to `narrative.taxonomy`. |
| `labels` | **Derived / compat:** string array of keys with `disposition = "canonical"` only (from `taxonomy` when present). Do not include metadata-only, internal-only, unknown, or low-confidence candidates. Apply **multi-axis** extraction per §2.1 — see rule below. |
| `title`, `description` | `{ et, ru, en }` per §4.1 and i18n policy. |
| `summary` | Optional `{ et, ru, en }` when content is minimal (see **`summary` generation rule** below); orchestrator/UI use summary for card preview when present. |
| `institution` | Optional `{ et, ru, en }` when interview/validation produced full trilingual institution text; **omit** if any slot is empty (REQ-23 §2.5). |

Do not add donor-era, legacy-only, Search-only, or backend-issued fields to `canonical_payload`.

**`summary` generation rule (REQ-34):** If `canonical_payload.description` and the interview narrative contain sufficient content (more than a bare factual incident — i.e., there is meaning, desired state, or civic angle beyond a bare description), the normalizer **MUST** generate a concise summary:

- 1–2 sentences in `session_language` capturing: what the problem is + what the desired state is (or civic angle).
- Prefer Phase 7 **`summary_draft[session_language]`** when present; otherwise derive from validated description and narrative.
- Translate to the other two languages per [`story-i18n-policy.md`](story-i18n-policy.md).
- Include as `canonical_payload.summary = { et, ru, en }`.

**Omit** `summary` only when: the story is so minimal that a meaningful summary would merely repeat the title. Do **not** omit because summary was not produced upstream — generate it here.

**observation vs complaint decision rule (REQ-34):** Use `complaint` when the story describes an **absent or malfunctioning condition** that the resident experiences as a problem — even if framed gently or as a wish. Key signal: the resident implies that something **should** exist or work, but **does not**.

Use `observation` only when: the story is a positive description of desired improvement **without** implied absence or harm — i.e., the resident explicitly frames it as a preference or idea, not a problem.

Examples:

- "There's no cultural space for children in our district" → `complaint` (absence = problem; improvement wish + implied harm).
- "It would be nice to have more parks" → `observation` (pure improvement wish, no implied harm or absence).
- "The kindergarten is overcrowded and there's nowhere to go after school" → `complaint` (concrete harm + absence).

When in doubt: if the user would answer "yes" to "Does this bother you or cause a problem?" → use `complaint`, not `observation`. Align with [`story-interview-flow.md`](story-interview-flow.md) §8 row 9 (FR-M1-025 improvement-without-harm baseline).

**Type vs axis classification (REQ-38):** Choosing `type` (`complaint` / `observation`) is **orthogonal** to multi-axis label extraction. An ecosystem-deficit narrative with implied absence → `complaint` per REQ-34 **and** `ecosystem_signal` + `topic_domain` labels when evidence supports both. Do not skip `ecosystem_signal` because `type` is already set.

**Multi-axis label rule:** A single story commonly maps to labels from multiple taxonomy axes:

- **topic_domain** (what civic area): `education`, `public_space`, `transport`, etc.
- **civic_signal** (what pattern): `city_for_people`, `equity_access`, `systemic_pattern`, etc.
- **issue_archetype_support** (story form): `improvement_wish`, `harm_reported`, `positive_observation`, etc.

Extract from each axis **independently**. A story about wanting better public spaces for children should produce: `["education", "public_space", "city_for_people", "improvement_wish"]` — not just `["education"]`.

Do **not** omit `civic_signal` labels because they seem "less obvious" — these are among the most valuable for clustering and analytics.

#### 4.1a Label extraction hints (commonly under-applied keys)

**Process stays in this core file.** Axis / key lists for the **active node** live in the pack — read [`schema-packs/README.md`](schema-packs/README.md), then the active pair’s data-model pack §5, plus [`story-label-taxonomy.md`](story-label-taxonomy.md) (pack include, Residual). Do **not** copy civic overlay tables here.

Use [`story-label-taxonomy.md`](story-label-taxonomy.md) as SSOT. When narrative evidence matches, include the key even if another topic_domain label is already present.

#### 4.1a.1 Ecosystem-deficit classification preference (REQ-38)

When narrative evidence matches **absence of environment / missing ecosystem** (see [`story-interview-flow.md`](story-interview-flow.md) §7.4) — not a single malfunctioning object:

1. **Preferentially** include one or more `ecosystem_signal` keys (`ecosystem_gap`, `institutional_decline`, `mentor_shortage`, `loss_of_continuity`, `community_fragmentation`, `replicable_model_needed`, `underused_resources`) **in addition to** applicable `topic_domain` keys.
2. **Anti-collapse:** do **not** emit only `["education"]` (or any single `topic_domain` key) when ecosystem evidence is present — mirror REQ-36 multi-axis rule below.
3. **Single-object failures** (one broken swing, one pothole, one portal error) → use `failure_mode` / `service_object` axes; **do not** force `ecosystem_signal` unless the resident frames systemic absence.

**Regression guard:** a broken-road **complaint** about potholes or repair backlog must **not** receive `city_for_people` — that civic_signal applies to desired human-centered city improvement, not infrastructure failure reports.

**Multi-axis rule (REQ-36):** cultural/science/youth stories often need `culture`, `youth_development`, or `science_and_research` **in addition to** `education` — do not collapse civic narratives to a single `education` label when other topic_domain keys match.

**`institution` — demo-scope constraint (M1 demo, REQ-28):**

Do **not** populate `canonical_payload.institution` in the current demo scope, regardless of what the GPT infers from the interview. The REQ-23 §2.5 i18n-omit rule (omit if any of `{et, ru, en}` is empty) remains in force as the **secondary** post-demo directive — but in the demo scope this gate has priority and applies even when all three i18n slots are present.

If interview reasoning produced an institution candidate (even with all three i18n slots populated):

- Record the candidate object in `non_wire_metadata.institution_candidate` (§4.3) — informational only, never wired.
- Leave `canonical_payload.institution` **absent** / not emitted.
- Add a one-line entry to `normalization_metadata.trace_notes`, e.g. `"demo scope: institution candidate parked in non_wire_metadata (REQ-28)"`.

This gate is lifted when backend integration for institution-routing matures — tracked as gateway REQ-43 (institution) ([`43-institution-json-story-column.md`](../../doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md)). The orchestrator pre-flight in [`api-orchestrator.md`](api-orchestrator.md) §5.2 enforces the same constraint as defense-in-depth.

### 4.2 `normalization_metadata` (required keys — instruction-layer scaffold)

Stable **references** to upstream work (opaque strings or objects — align with **M1-06** when JSON Schema is published):

| Key | Purpose |
|-----|---------|
| `session_language` | **Required** for story-intake handoff: `et` \| `ru` \| `en` — MUST match the primary interview language from [`bootstrap.md`](bootstrap.md) **`comm_context.ui_lang`** (see [`story-i18n-policy.md`](story-i18n-policy.md) §1–2). Maps to `StoryDraftStashRequest.narrative.session_language` (orchestrator §5.2.1). |
| `detected_input_language` | **Required** for wire v2: `et` \| `ru` \| `en` — auto-detected language of the user’s narrative text from deep parsing / validation (may differ from `session_language`). Maps to `StoryDraftStashRequest.narrative.language` per REQ-22. Do not substitute `session_language` when the detected language differs. |
| `contains_pii` | **Required** for REQ-23 / GPT-PII-01 handoff: boolean — conservative PII scan of the §4.4 scan set (all narrative wire text + sidecars). Read by [`api-orchestrator.md`](api-orchestrator.md) §5.2.0. |
| `ingest_validation_report_ref` | Reference to the validation artifact used (id, hash, or short summary line). |
| `safety_compliance_report_ref` | Reference to relevant safety checkpoint output for this handoff. |
| `policy_gate_ref` | At minimum: `policy_ref`, `rulebook_version`, and `policy_gate_result.status` copy or stable id. |
| `normalizer_module` | e.g. `issue-normalizer` + **version** of this instruction file (from document header). |
| `trace_notes` | Optional: free-text **internal** consistency notes (not for end-user display). |
| `label_extraction_metadata` | Optional label candidate metadata from validation; stores label, axis, source, confidence, and disposition. It is not copied to transport unless schema changes in lockstep. |
| `location_extraction_metadata` | Optional location confidence metadata from §4.6 processing; stores `location_detected`, `location_source`, and `confidence`. Internal diagnostic sidecar only (REQ-39) — not copied to transport. |
| `trigger_activation_metadata` | Optional per-trigger activation audit (REQ-41) — see §4.2.3. Internal diagnostic only; source for God Mode table in [`api-orchestrator.md`](api-orchestrator.md) §5.2.0b. |

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

Only candidates with `disposition = "canonical"` and keys allowed by [`story-label-taxonomy.md`](story-label-taxonomy.md) may appear in `canonical_payload.labels[]`. Candidates with `metadata_only`, `needs_clarification`, `rejected`, or **`internal`** disposition remain out of flat `labels[]` but **MUST** be grouped into `canonical_payload.taxonomy[axis]` with their disposition preserved (GPT-TAX-01 — backend filters public surfaces).

**Taxonomy emit rule (GPT-TAX-01):** Build `canonical_payload.taxonomy` from `label_extraction_metadata.candidates` (and equivalent confirmed keys) by grouping on `axis`. Example:

```json
{
  "topic_domain": [{ "label": "transport", "disposition": "canonical" }],
  "deep_need": [{ "label": "predictability", "disposition": "metadata_only" }],
  "risk_privacy_safety": [{ "label": "pii_present", "disposition": "internal" }]
}
```

Then derive `canonical_payload.labels[]` = all `label` values under any axis with `disposition = "canonical"` (order: axis table order in taxonomy §3, then label order within axis).

**English token rule (GPT-TAX-02 / GIM-227):** Every `label` written into `canonical_payload.taxonomy[*][].label` and into derived `canonical_payload.labels[]` MUST be an **English** token (snake_case / [`story-label-taxonomy.md`](story-label-taxonomy.md) §4 style). If a candidate is Estonian or Russian prose, **remap** to the closest listed English key or omit / `needs_clarification` — do **not** leave et/ru prose on the wire. This is GPT-side correction, **not** HTTP 422 / server vocabulary reject.

#### 4.2.2 `location_extraction_metadata`

When location is processed per §4.6, record extraction confidence under `normalization_metadata.location_extraction_metadata`:

```json
{
  "location_detected": true,
  "location_source": "explicit",
  "confidence": "HIGH"
}
```

| Field | Values | Rule |
|-------|--------|------|
| `location_detected` | boolean | `true` when §4.6 formed `location_query`; `false` when location omitted per rules 3–5 |
| `location_source` | `explicit` \| `inferred` | `explicit` only when the resident **named or confirmed** the place in interview material (REQ-35 privacy baseline — not invented from context alone); `inferred` when derived from narrative context without direct location confirmation |
| `confidence` | `LOW` \| `MEDIUM` \| `HIGH` | `HIGH` = explicit confirmation + unambiguous string; `MEDIUM` = confirmed but ambiguous or partial; `LOW` = inferred or weak signal |

**Non-wire rule:** This object is **internal-only** — do **not** copy to `StoryDraftStashRequest`, `canonical_payload`, or any top-level wire field. [`api-orchestrator.md`](api-orchestrator.md) does not consume it. Omit `location_extraction_metadata` entirely when `location_detected = false`.

#### 4.2.3 `trigger_activation_metadata` (REQ-41 / GIM-175)

When `ingest_validation_report.pre_submission_compliance_evidence` is available, **MUST** record trigger activation under `normalization_metadata.trigger_activation_metadata` using **the same trigger definitions** as [`api-orchestrator.md`](api-orchestrator.md) §5.2.2 items 13–15 (REQ-40 §2.3). This is the normalizer-side audit source for God Mode diagnostics — **not** enforcement (REQ-40 owns FAIL/warning).

```json
{
  "triggers": [
    { "trigger": "location_trigger", "activated": true, "reason": null },
    { "trigger": "origin_trigger", "activated": true, "reason": null },
    { "trigger": "summary_generation_trigger", "activated": false, "reason": "omitted-by-rule" },
    { "trigger": "multi_axis_labels_trigger", "activated": true, "reason": null },
    { "trigger": "gpt_signals_trigger", "activated": false, "reason": "no-evidence" }
  ],
  "evaluation_ref": "ingest_validation_report.pre_submission_compliance_evidence + normalized handoff (REQ-40 §2.3)"
}
```

| Trigger | `activated = true` when | Typical `reason` when `activated = false` |
|---------|-------------------------|-------------------------------------------|
| `location_trigger` | `pre_submission_compliance_evidence.location.confirmed = true` **and** §4.6 emitted non-empty `location_query` on the narrative handoff | `no-evidence` — location not confirmed; `omitted-by-rule` — confirmed but §4.6 omitted `location_query` |
| `origin_trigger` | Draft will send `origin.source = "openai_gpt_action"` (REQ-32) **and** `origin.conversation_id` present when Actions runtime exposed a session id | `runtime-unavailable` — `conversation_id` omitted because runtime did not expose session id (REQ-35); `no-evidence` — `origin.source` would be missing (should not reach HTTP) |
| `summary_generation_trigger` | `pre_submission_compliance_evidence.narrative_sufficient_for_summary = true` **and** `canonical_payload.summary` populated per §4.1 (REQ-34) | `no-evidence` — content threshold not met; `omitted-by-rule` — sufficient material but summary omitted |
| `multi_axis_labels_trigger` | `multi_axis_evidence.domain_count ≥ 2` (or `axes_detected` length ≥ 2) **and** `canonical_payload.labels` reflects multi-axis evidence (≥ 2 distinct axis keys when evidence supports it) | `no-evidence` — single-domain narrative; `omitted-by-rule` — multi-axis evidence but label collapse to single key (align with orchestrator item 14 warning) |
| `gpt_signals_trigger` | `pre_submission_compliance_evidence.subjective_signal_present = true` **and** applicable `non_wire_metadata` / wire `gpt_signals` populated per §4.3 (respecting `severity_confidence = LOW → omit`) | `no-evidence` — no subjective signal in evidence; `omitted-by-rule` — signal present but omitted by §4.3 confidence / omit rules |

**Reason enum (when `activated = false`):** `no-evidence` \| `runtime-unavailable` \| `omitted-by-rule`. Use `null` (or omit `reason`) when `activated = true`.

**Non-wire rule:** Internal-only — do **not** copy to `StoryDraftStashRequest` or wire envelope. Optionally mirror one-line summaries in `trace_notes` (e.g. `"trigger_audit: summary omitted-by-rule (REQ-41)"`) — free-text supplements only; structured truth lives in `trigger_activation_metadata`.

### 4.3 `non_wire_metadata` (optional sidecar)

`severity`, `impact_estimation`, and `problem_status` are resident-perceived subjective fields from [`story-data-model.md`](story-data-model.md) **§4.3**.

**Subjective signals determination rule (REQ-35):** When the interview narrative contains sufficient context to infer resident-perceived seriousness, scope, or problem status — from what the resident **already stated**, without leading questions or new interview prompts — **actively determine** and place all applicable fields in `non_wire_metadata`, including `severity` when a seriousness signal is present (do not omit `severity` merely because it was not pre-labeled upstream).

**Severity confidence (REQ-40 / GIM-172):** When inferring `severity`, assign internal `severity_confidence` as `HIGH`, `MEDIUM`, or `LOW`:

| `severity_confidence` | When |
|-----------------------|------|
| `HIGH` | Resident explicitly stated seriousness or unambiguous harm language |
| `MEDIUM` | Defensible inference from stated context without explicit severity wording |
| `LOW` | Weak, ambiguous, or single-token signal without supporting context |

**`severity_confidence = LOW` → omit `severity`** from `non_wire_metadata` (and thus from wire `gpt_signals.severity`). Do **not** weaken REQ-35 active determination above: when context supports a defensible `MEDIUM` or `HIGH` mapping, populate `severity` as today.

- Collect values **only** from stated resident material per [`story-data-model.md`](story-data-model.md) §4.3 («without leading the user»).
- When context supports all three dimensions, populate `severity`, `impact_estimation`, and `problem_status` together when each has a defensible mapping.
- **Omit** a field (or the entire sidecar) when no defensible signal exists — do **not** invent values.
- For `impact_estimation`: if unsure, **omit the field** — do **not** use `UNKNOWN` as a fallback (wire enum has no `UNKNOWN` for this field).
- For `problem_status`: if unsure, prefer `UNKNOWN` or omit that field only.

If nothing was collected or confirmed upstream **and** the narrative lacks sufficient subjective context, omit `non_wire_metadata`.

`non_wire_metadata` is an **internal** sidecar only. [`api-orchestrator.md`](api-orchestrator.md) maps these three fields to wire `gpt_signals` (REQ-23 §2). Do **not** copy the `non_wire_metadata` object into `StoryDraftStashRequest`.

Allowed enum values (must match **gateway REQ-42 (gpt_signals)** when mapped to wire — [`42-gpt-signals-story-intake-extension.md`](../../doge-complaints-gateway/docs/requirements/42-gpt-signals-story-intake-extension.md); not GPT-UI REQ-42 adaptive post-submit):

| Field | Values |
|-------|--------|
| `severity` | `LOW`, `MEDIUM`, `HIGH`, `CRITICAL` |
| `impact_estimation` | `LOCAL`, `DISTRICT`, `CITY`, `NATIONAL` |
| `problem_status` | `ONGOING`, `RESOLVED`, `RECURRING`, `UNKNOWN` |
| `institution_candidate` | Optional `{ et, ru, en }` (REQ-28): informational fallback for an institution candidate suppressed by the §4.1 demo-scope constraint. Never copied to wire; lift gate when gateway REQ-43 (institution) ([`43-institution-json-story-column.md`](../../doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md)) integration matures. |

If unsure about a value, prefer `UNKNOWN` for `problem_status` or omit that field from `non_wire_metadata`. `institution_candidate` is **only** populated when §4.1 demo-constraint suppressed a candidate from `canonical_payload.institution` — otherwise omit it.

### 4.4 PII detection (REQ-23 §1.2 / GPT-PII-01)

Scan the **full scan set** on this draft for personally identifiable information — not only `original_text` / `description.*`:

- narrative wire text: `original_text`; `canonical_payload.title.{et,ru,en}`; `canonical_payload.description.{et,ru,en}`; `canonical_payload.summary.{et,ru,en}` when present
- text sidecars: `live_story_context.consistency_notes`; `location_query`; institution strings; free-text labels (non-enum prose)

Do **not** treat opaque submitter ids, HMAC tokens, or identity phone hashes as narrative PII (GPT-PII-01 AC #5).

| PII type | Examples |
|----------|----------|
| Person name | «Иван Петров», «Mari Tamm» |
| Address | «Liivalaia 10-5», street + number |
| Phone | `+372…`, national formats, digits with spaces (`5 55 12 34`) |
| Email | `name@domain`, obfuscated (`name собака domain точка ee`) |
| Personal identifier | ID-card, passport, national ID numbers |

**Obfuscation (V8):** Treat spaced phones, «собака» / «at» / «точка» email disguises, and similar identifier hiding as PII.

**Rules:**

- If **high confidence** that any type is present → set `normalization_metadata.contains_pii = true`.
- If **low confidence** → still set `contains_pii = true` (conservative).
- If no PII detected in the scan set → `contains_pii = false`.

Do **not** perform the user interaction here; [`api-orchestrator.md`](api-orchestrator.md) §5.2.0 runs edit-or-STOP before HTTP. Decline redact is **STOP** (not send-with-flags).

### 4.5 Consistency notes (REQ-23 §3.2)

When the narrative contains contradictions worth documenting for operators, add a top-level sidecar on `normalized_issue_payload` (sibling to `canonical_payload`):

```json
"live_story_context": {
  "consistency_notes": "One or two sentences describing the contradiction only."
}
```

Set `consistency_notes` when any of these apply:

| Situation | Example |
|-----------|---------|
| Temporal contradiction | Problem stated as years ago vs «last week» |
| Geographic contradiction | Two different districts/addresses for one issue |
| Responsibility unclear | City vs state agency ambiguous |
| Problem status ambiguous | Both «resolved» and «still happening» |

**Format:** free text, 1–2 sentences, language = `session_language` or `en`; describe the **contradiction**, not the complaint summary.

**Omit rule:** if no contradiction → do **not** add `live_story_context`; never emit empty string.

### 4.6 `location_query` (REQ-26)

Optional **top-level** string on `normalized_issue_payload` (sibling to `canonical_payload`, not inside it). Maps to `StoryDraftStashRequest.narrative.location_query` via [`api-orchestrator.md`](api-orchestrator.md) §5.2.1. Server resolves non-empty values to geo ([`API_REFERENCE.md`](../../doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md) §6.3).

**Source data:** `location.freeform` from the ingest validation artifact ([`ingest-validation.md`](ingest-validation.md) report shape) after the user **confirmed** location in interview Phases 2–3 and [`story-interview-flow.md`](story-interview-flow.md) §7.2 affirmation.

**Formation rules (apply pack geo formation):**

1. **Delegate to pack `geo_intake.mode`:** Read [`schema-packs/README.md`](schema-packs/README.md), then that pack’s data-model §3–§4. If mode is **`optional`** (civic instance current value): **MAY** form `location_query` when the resident confirmed a place; **do not MUST** force it. If mode is `require_location_or_detail`, form `location_query` **or** leave geo to orchestrator `geo_detail` (one of the two must be non-empty). If mode is `require_detail`, `location_query` remains optional; structured `geo_detail` is orchestrator’s SHALL. The only grounds to omit a formed string are rules **3–5** below. Do **not** keep a civic city table in this core file.
2. **Format — prefer Latin script (REQ-35):** freeform string, preferably `<street/place>, <city>` in **Latin script**. Apply the **active data-model pack** geo-formation rules (pack data-model §4). The backend applies `normalize_location_query()` (lowercase) before geo-resolve ([`doge-complaints-gateway/src/core/geo/normalize.py`](../../doge-complaints-gateway/src/core/geo/normalize.py)). Do **not** invent address detail beyond what the resident stated.
   - **City-level canonicalization (REQ-39):** When the confirmed location is **city-level only**, apply the pack’s city canon. When the string includes **district, street, or landmark** detail, **preserve** that detail — do **not** collapse to city-only.
3. If the user mentioned **multiple** locations without a single clear primary — pick the most specific / complaint-relevant one (D-02). Reflect ambiguity in `live_story_context.consistency_notes` (§4.5) when useful.
4. If location was **not** confirmed or remains ambiguous after §7.2 — **omit** `location_query` (do not send null or `""`).
5. Never emit a blank or whitespace-only string.

**Do not** place `location_query` inside `canonical_payload` or `normalization_metadata`.

---

## 5. Narrative layers alignment (brief)

Narrative layers described in [`story-data-model.md`](story-data-model.md) §3 feed **content** inside `title` / `summary` / `description` / `labels` / `type`. This module **projects** already validated material into the §4.1 card shape; it does not re-author the interview.

### 5.1 Layer projection → Issue card fields

| Narrative layer | Projection target in `canonical_payload` |
|-----------------|------------------------------------------|
| Narrative core | factual base for `summary` / `description` |
| Meaning layer | depth and framing in `description`; influences `labels` |
| Civic framing | `type` + civic taxonomy `labels` |
| Desired state | action-oriented `description`; optional `institution` hypothesis (FR-M1-027, conservative) |
| Issue-shaped output | final `title` / `summary` / `description` + optional `institution` |

---

## 6. Relationship to other modules

| Module | Relationship |
|--------|----------------|
| [`ingest-validation.md`](ingest-validation.md) | Upstream — completeness / batch rules. |
| [`safety-compliance.md`](safety-compliance.md) | Upstream — safety checkpoints before or beside gate per lifecycle. |
| [`story-policy-gate.md`](story-policy-gate.md) | Immediate upstream — **approval** required. |
| [`api-orchestrator.md`](api-orchestrator.md) | Downstream — **only** module that performs HTTP and consumes `normalized_issue_payload`. |
| [`base.md`](base.md) | §1.5 — Issue strict chain uses **`normalized_issue_payload`** as the canonical normalization artifact. |

---

## 7. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.2.22 | 2026-08-31 | **GPT-SSR-03 / GIM-268:** §4.6 = apply pack geo formation; MUST-when-confirmed reconciled with pack `geo_intake.mode=optional` (MAY emit; do not force). Omit/blank core rules 3–5 kept. |
| 0.2.21 | 2026-08-31 | **GPT-SSR-01 / GIM-254:** §4.1a civic overlay table removed — lists live in pack / taxonomy include. |
| 0.2.20 | 2026-08-31 | **GPT-SSR-01 / GIM-253:** §4.1a / §4.6 pack pointers README-only (no hardcoded civic pack path). |
| 0.2.19 | 2026-08-31 | **GPT-SSR-01 / GIM-251:** §4.6 city canon + Latin preference → active data-model pack; §4.1 type/taxonomy weld points at pack payload fields; PII §4.4 unchanged. |
| 0.2.18 | 2026-08-21 | **GPT-PII-01 / GIM-241:** §4.4 scan set = all narrative i18n wire text + sidecars; obfuscated identifiers = PII; opaque submitter/HMAC ids not narrative PII; decline interaction is orchestrator STOP. |
| 0.2.17 | 2026-08-21 | **GPT-TAX-02 / GIM-227:** English token emit for `taxonomy[].label` and derived `labels[]`; et/ru prose remap or omit — not HTTP 422. |
| 0.1 | 2026-04-10 | Initial scaffold: `normalized_issue_payload`, `canonical_payload`, `normalization_metadata`, Plain GPT, no API, no user questions; Issue SoT = `story-data-model.md` §4.1. |
| 0.1.1 | 2026-04-10 | Added §6 cross-link to `base.md` §1.5 Issue artifact alignment. |
| 0.1.2 | 2026-04-10 | Added §5.1 concise narrative-layer projection table for `title` / `summary` / `description` / optional `institution`. |
| 0.1.3 | 2026-04-20 | Added required `normalization_metadata.session_language`; optional subjective intake fields in `canonical_payload`; demo `institution` omit. |
| 0.1.4 | 2026-04-22 | Added donor-era minors metadata per `issue-data-model` §4.4 (GIM-65). |
| 0.1.5 | 2026-04-25 | Removed donor-era minors metadata from `canonical_payload`; system-only reference restored to §4.4 (GIM-72). |
| 0.1.6 | 2026-04-25 | Clarified subjective intake fields as non-wire metadata for current runtime contract (GIM-77). |
| 0.1.7 | 2026-04-26 | Locked final `normalized_issue_payload` shape, optional `non_wire_metadata` sidecar, and no-direct-transport rule (GIM-80). |
| 0.1.8 | 2026-04-26 | Added label extraction metadata and canonical label disposition rules (GIM-89). |
| 0.1.9 | 2026-05-22 | **REQ-22 / GIM-104:** required `normalization_metadata.detected_input_language` for `narrative.language` wire mapping. |
| 0.2.1 | 2026-05-24 | **REQ-26 / GIM-119:** §4.6 top-level `location_query` from confirmed `location.freeform`; omit when unconfirmed or empty. |
| 0.2.0 | 2026-05-22 | **REQ-23 / GIM-108–111:** §4.4 PII (`contains_pii`); §4.5 `live_story_context.consistency_notes`; §4.3 → `gpt_signals` wire via orchestrator; institution emit when full i18n. |
| 0.2.2 | 2026-05-25 | **REQ-28 / GIM-125:** §4.1 explicit demo-scope constraint for `canonical_payload.institution` (do-not-populate + `trace_notes` entry); §4.3 `institution_candidate` sidecar fallback row; REQ-23 §2.5 i18n-omit retained as secondary post-demo directive; lift gate tied to REQ-43. |
| 0.2.3 | 2026-06-01 | **REQ-33 / GIM-145:** §2.1 multi-axis label extraction + conservative clarification; §4.1 multi-axis rule + example; §4.1a extraction hints (`public_space`, `city_for_people`, `improvement_wish`, `equity_access`) + broken-road regression guard. |
| 0.2.4 | 2026-06-02 | **REQ-33 audit follow-up / GIM-147…148:** §2.1 changed to `one or more keys per applicable axis`; header traceability explicitly includes `REQ-33` (GAP-01/02 closure). |
| 0.2.5 | 2026-06-02 | **REQ-34 / GIM-149…150:** §4.1 `summary` generation rule (mandatory when content sufficient); §4.1 `type` observation vs complaint boundary with examples; handoff input `summary_draft` (§3). |
| 0.2.6 | 2026-06-03 | **REQ-35 / GIM-152…153:** §4.6 `location_query` MUST when place confirmed + Latin/format + `normalize_location_query()` ref; §4.3 active subjective signals determination (`severity` + peers, no leading questions). |
| 0.2.7 | 2026-06-05 | **REQ-36 / GIM-159:** §4.1a extraction hints for `culture`, `youth_development`, `science_and_research`, `ecosystem_gap`, `cooperative_model`; multi-axis rule against `education`-only collapse for civic narratives. |
| 0.2.8 | 2026-06-05 | **REQ-38 / GIM-164:** §4.1 type-vs-axis orthogonality; §4.1a.1 ecosystem-deficit preferential classification + anti-collapse; expanded `ecosystem_signal` hints (`institutional_decline`, `mentor_shortage`, `community_fragmentation`, `replicable_model_needed`). |
| 0.2.9 | 2026-06-05 | **REQ-38 audit follow-up / GIM-166:** §4.1a.1 GAP-38-01 — removed duplicate «Ecosystem anti-collapse» (L205); canonical anti-collapse = item 2 + REQ-36 multi-axis rule below. |
| 0.2.10 | 2026-06-05 | **REQ-39 / GIM-167:** §4.6 rule 2 city-level canonicalization to `<City>, Estonia` (`Tallinn`, `Tallinna linn` examples); district/street strings preserved (`Kalamaja, Tallinn`). |
| 0.2.16 | 2026-07-23 | **GPT-TAX-01 / GIM-211:** emit `canonical_payload.taxonomy` per-axis (incl. `internal`); flat `labels[]` derived; disposition table lockstep. |
| 0.2.15 | 2026-07-07 | **GPT-SUBMIT-02 reaudit / GIM-207:** N1 rename propagation — `StoryDraftStashRequest` in mapping/non-wire prose (lockstep OpenAPI v0.6.0 + orchestrator §5.2.1). |
| 0.2.14 | 2026-06-09 | **REQ-43 audit follow-up / GIM-185:** gateway REQ-43 (institution) namespace qualifier — §4.1 lift-gate note; §4.3 `institution_candidate` row. Semantics unchanged. |
| 0.2.13 | 2026-06-07 | **REQ-41 / GIM-175:** §4.2.3 `trigger_activation_metadata` — five-trigger activation audit (`location`/`origin`/`summary`/`multi_axis_labels`/`gpt_signals`) with Reason enum; reuses REQ-40 evidence definitions; non-wire diagnostic source for God Mode. |
| 0.2.12 | 2026-06-06 | **REQ-40 / GIM-172:** §4.3 explicit `severity_confidence` (`HIGH`/`MEDIUM`/`LOW`); `LOW` → omit `severity`; REQ-35 active determination preserved. |
| 0.2.11 | 2026-06-05 | **REQ-39 / GIM-168:** §4.2.2 `location_extraction_metadata` sidecar (`location_detected`, `location_source`, `confidence`); non-wire; explicit source from resident-stated material only. |
