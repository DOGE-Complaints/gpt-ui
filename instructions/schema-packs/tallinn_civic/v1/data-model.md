# Pack data-model — `tallinn_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Non-executable **projection** of the gateway civic schema pack for interview clustering, payload field shapes, `geo_intake`, and geo formation. Not a validator.

| Field | Value |
|-------|--------|
| **Version** | 1.2 |
| **Date** | 2026-08-31 |
| **Traceability** | GPT-SSR-03 / GIM-267…270; GPT-SSR-01 / GIM-249 / GIM-254; REQ-45; gateway `schema-packs/tallinn_civic/v1/` |
| **Binding pair** | `schema_id=tallinn_civic`, `schema_version=v1` |
| **Gateway SSOT** | `doge-complaints-gateway/schema-packs/tallinn_civic/v1/pack.json` + `payload.schema.json` |

**SEC-001:** this file does **not** execute validate. Gateway Schema Runtime is authoritative. Do not invent OpenAPI names.

## 1. Binding pair

- `schema_id`: `tallinn_civic`
- `schema_version`: `v1` (semantic pack version — **not** envelope `m2.story_intake_envelope.v2`)

## 2. `structured_payload` shape (`signals.*` / `geo.*`)

Projection of gateway `payload.schema.json`. Root object requires `signals`. Additional properties forbidden on `signals` and `geo`. Required pack fields: `signals.civic_domain`, `signals.failure_pattern` (`field_policy`).

### 2.1 `signals` (required object)

| Field | Policy (`field_policy`) | Notes |
|-------|-------------------------|--------|
| `civic_domain` | **required** | JSON Schema required. |
| `failure_pattern` | **required** | JSON Schema required. |
| `civic_weight` | optional | |
| `desired_outcome` | optional | |
| `affected_group` | optional | |
| `service_object` | optional | |
| `need` | optional | |
| `ecosystem_signal` | optional | |
| `canonical_type` | optional | Pack payload field; card `ISSUE_TYPE` remains a display enum in core `story-data-model.md` until a node pack replaces it. |

### 2.2 `geo` (optional object)

All optional per `field_policy`: `district`, `settlement`, `region`, `country`, `street`, `house`, `house_range`, `houses` (array of strings).

Do not invent OpenAPI sidecar names here (`schema_binding` / `geo_detail` = GPT-SSR-02 wire tokens). Orchestrator **MUST** emit `schema_binding.structured_payload` using these keys (GPT-SSR-03). Required for this civic instance: `signals.civic_domain`, `signals.failure_pattern`.

## 3. `geo_intake` (from gateway `pack.json`)

| Key | Value |
|-----|--------|
| `mode` | `optional` |
| `merge` | `true` |
| `mirror_to_payload` | `true` |

`merge` and `mirror_to_payload` are **gateway-side** (authoritative merge / payload mirror). They are **not** OpenAPI / stash wire keys. GPT documents them here only.

### 3.1 Mode matrix (GPT emit)

| `geo_intake.mode` | GPT SHALL | Civic `tallinn_civic/v1` |
|------------------|-----------|---------------------------|
| `optional` | MAY omit `geo_detail` and `location_query` | **Current pack value** |
| `require_location_or_detail` | non-empty `location_query` **or** `geo_detail` | — |
| `require_detail` | non-empty `geo_detail` | — |

### 3.2 Decision table (confirmed place, no `geo_detail`, no `location_query`)

| Mode | Action |
|------|--------|
| `optional` | Proceed (MAY still emit if resident gave place) |
| `require_location_or_detail` | STOP — collect one of the two |
| `require_detail` | STOP if no `geo_detail` |

Core orchestrator **reads** `geo_intake.mode` and applies this table at pre-flight. HTTP 422 handling stays in orchestrator; **scope copy** for this instance is below.

### 3.3 GEO_SCOPE copy (this civic instance)

Demo geo scope: **Estonia / Tallinn**. When the gateway returns HTTP 422 `GEO_SCOPE_MISMATCH`, ask the resident to clarify a place **inside this pack’s scope** (Tallinn / Estonia). Core orchestrator must not hard-code this copy.

## 4. Geo formation (owned by this pack; core applies)

Process (when to form / omit `location_query` per **mode**) stays in [`story-normalizer.md`](../../../story-normalizer.md) §4.6 as «apply pack geo formation». **City canon and Latin preference for this pack** live here:

**Format — prefer Latin script:** freeform `<street/place>, <city>` in Latin (e.g. `Tallinn`, `Kalamaja, Tallinn`, `Tartu mnt 80, Tallinn`). Cyrillic is acceptable when the resident used it and Latin transliteration would distort meaning.

**City-level canonicalization (this civic instance):** when the confirmed location is **city-level only** (no street, district, or landmark), canonicalize to `<City>, Estonia` in Latin:

- `Tallinn` → `Tallinn, Estonia`
- `Tallinn, Estonia` → `Tallinn, Estonia` (already canonical)
- `Tallinna linn` → `Tallinn, Estonia`

When the string includes district, street, or landmark detail, **preserve** that detail — do not collapse to city-only. Do not append `, Estonia` when it would simplify a more specific address the resident confirmed.

**Reconcile with `mode=optional`:** this pack does **not** require `location_query` when a place is confirmed. Core MUST NOT force `location_query` for this instance. MAY still form it when the resident gave a place.

## 5. Interview overlay notes (phase → axes + ecosystem)

Phases 1–7 **process** stays in [`story-interview-flow.md`](../../../story-interview-flow.md). Field/axis lists for this civic instance:

| Phase | Evidence to capture | Label axes supported (pack overlay) |
|-------|---------------------|-------------------------------------|
| **2** Episode | What happened, object/service, place, friction | `topic_domain`, `service_object`, `location_context`, `failure_mode` |
| **3** Emotion | Unfair / draining / unsafe / confusing | `deep_need`, `failure_mode`, `civic_signal` |
| **4** Deeper need | Predictability, respect, dignity, agency, fairness | `deep_need` (metadata-only unless taxonomy promotes) |
| **5** Desired state | Better future / improvement shape | `desired_outcome`, `topic_domain`, `service_object`, `ecosystem_signal`, `governance_signal` |
| **6** Civic generalization | Recurrence, others affected, system pattern | `civic_signal`, `affected_scope`, `issue_archetype_support`, `ecosystem_signal` |
| **Safety** | PII, minors, health, violence | `risk_privacy_safety` internal-only |

**Ecosystem-deficit recognition (pack overlay):** when the resident describes absence of environment / institutional decline / community fragmentation / replicable-model desire — capture `ecosystem_signal` evidence from what they already said. Do **not** ask leading taxonomy questions.

| Class | Resident signals (examples) | Downstream axis hints |
|-------|----------------------------|------------------------|
| **Absence of environment** | «негде», «нет места для», «раньше было, теперь нет», «нет среды», «nowhere for kids to…» | `ecosystem_gap`, `missing_infrastructure` |
| **Institutional decline / continuity loss** | Closing venues, programs ending, mentor/teacher shortage, «used to have…» | `institutional_decline`, `mentor_shortage`, `loss_of_continuity` |
| **Community fragmentation** | Weak ties, underused spaces, «people don’t gather anymore» | `community_fragmentation`, `underused_resources` |
| **Replicable model desire** | «Should work in other districts too», «model for other neighbourhoods» | `replicable_model_needed`, `governance_signal` when ownership model is named |

Under-applied label keys for this instance: follow [`story-label-taxonomy.md`](../../../story-label-taxonomy.md) include (§6). Do not copy that table into core.

Exact vs civic clustering: gateway pack `exact_lenses` (e.g. `civic_domain_micro`, `failure_pattern_micro`) inform how structured `signals.*` cluster; GPT does not execute those lenses.

## 6. Taxonomy pointer (Residual — P3 decision)

**Decision:** keep [`story-label-taxonomy.md`](../../../story-label-taxonomy.md) as SSOT for 13-axis label keys; this pack **includes** it by reference (do **not** migrate the whole file into the pack in this story). Pack-owned vocabulary may replace the include in a later story.

Allowed canonical labels for this instance = keys listed in `story-label-taxonomy.md` with disposition `canonical`.

## 7. Civic stash examples (moved from core orchestrator)

Core demo JSON uses generic placeholders. **This pack** owns civic instance examples:

```json
{
  "taxonomy": {
    "topic_domain": [{ "label": "transport", "disposition": "canonical" }],
    "civic_signal": [{ "label": "city_for_people", "disposition": "canonical" }],
    "risk_privacy_safety": [{ "label": "pii_present", "disposition": "internal" }]
  },
  "schema_binding": {
    "schema_id": "tallinn_civic",
    "schema_version": "v1",
    "structured_payload": {
      "signals": {
        "civic_domain": "example",
        "failure_pattern": "example"
      }
    }
  }
}
```

God Mode axis-summary examples for this instance may use the same labels. Do not copy these tokens into core orchestrator as the only demo body.
