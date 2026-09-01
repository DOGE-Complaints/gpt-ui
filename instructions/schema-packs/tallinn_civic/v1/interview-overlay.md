# Pack interview-overlay — `tallinn_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Prose-only interview guidance: geo formation canon, phase→axes capture, ecosystem cues. **Not** JSON Schema shape (read `pack.json` + `payload.schema.json`). **Not** admission rules (read `inbound-validation.md`).

| Field | Value |
|-------|--------|
| **Version** | 1.1 |
| **Date** | 2026-09-01 |
| **Traceability** | GPT-SSR-08 / GIM-312–314; GPT-SSR-09 / GIM-308; migrated from archived `archive/data-model.v1.3.md` §4–§5 |
| **Binding pair** | `schema_id=tallinn_civic`, `schema_version=v1` |

**SEC-001:** Gateway Schema Runtime validates JSON. This file is GPT process overlay only.

## Geo formation (this civic instance)

Process (when to form / omit `location_query` per **mode**) stays in [`story-normalizer.md`](../../../story-normalizer.md) §4.6. Read `geo_intake.mode` from same-pack [`pack.json`](pack.json) (not prose here).

**Format — prefer Latin script:** freeform `<street/place>, <city>` in Latin (e.g. `Tallinn`, `Kalamaja, Tallinn`, `Tartu mnt 80, Tallinn`). Cyrillic is acceptable when the resident used it and Latin transliteration would distort meaning.

**City-level canonicalization:** when the confirmed location is **city-level only** (no street, district, or landmark), canonicalize to `<City>, Estonia` in Latin:

- `Tallinn` → `Tallinn, Estonia`
- `Tallinn, Estonia` → `Tallinn, Estonia` (already canonical)
- `Tallinna linn` → `Tallinn, Estonia`

When the string includes district, street, or landmark detail, **preserve** that detail — do not collapse to city-only. Do not append `, Estonia` when it would simplify a more specific address the resident confirmed.

**Reconcile with `mode=optional`:** this pack does **not** require `location_query` when a place is confirmed. Core MUST NOT force `location_query` for this instance. MAY still form it when the resident gave a place.

## Geo sidecar sync (mirror_to_payload)

When `pack.json` → `geo_intake.mirror_to_payload` is `true` (civic default): keep `geo_detail.address.*` consistent with `structured_payload.geo.*` before stash. Latitude/longitude live on `geo_detail` root only — **not** in payload schema.

## Precision index (`detail_level`)

Read `geo_model` from [`pack.json`](pack.json). Allowed levels = `precision_levels` (also `payload.schema.json` `geo.detail_level` enum). Inference default: `from_address_depth`.

| Deepest confirmed evidence | Emit `detail_level` |
|----------------------------|---------------------|
| region only | `region` |
| city / settlement | `settlement` |
| district | `district` |
| street (no house) | `street` |
| house | `house` |
| house range / houses[] | `house_range` |
| lat/lon confirmed | `coordinates` (keep house if house also confirmed) |

Do **not** invent street/house to deepen precision. God Mode: warn if coarse `detail_level` + coords without house.

## Instance territory (GPT pre-flight)

Read `gpt_instance_territory` from same-pack [`pack.json`](pack.json). Gateway **parse-only**; GPT **STOP / clarify** per [`inbound-validation.md`](inbound-validation.md) §6 when resident place is outside instance rules (may be stricter than node `geo_scope`).

**This pack:** `admin_token` settlement `tallinn`. Do not invent EHAK/`admin_id` or bbox rules unless present in pack JSON.

## Phase → axes + ecosystem cues

Phases 1–7 **process** stays in [`story-interview-flow.md`](../../../story-interview-flow.md). Field/axis lists for this civic instance:

| Phase | Evidence to capture | Label axes supported |
|-------|---------------------|----------------------|
| **2** Episode | What happened, object/service, place, friction | `topic_domain`, `service_object`, `location_context`, `failure_mode` |
| **3** Emotion | Unfair / draining / unsafe / confusing | `deep_need`, `failure_mode`, `civic_signal` |
| **4** Deeper need | Predictability, respect, dignity, agency, fairness | `deep_need` (metadata-only unless taxonomy promotes) |
| **5** Desired state | Better future / improvement shape | `desired_outcome`, `topic_domain`, `service_object`, `ecosystem_signal`, `governance_signal` |
| **6** Civic generalization | Recurrence, others affected, system pattern | `civic_signal`, `affected_scope`, `issue_archetype_support`, `ecosystem_signal` |
| **Safety** | PII, minors, health, violence | `risk_privacy_safety` internal-only |

**Ecosystem-deficit recognition:** when the resident describes absence of environment / institutional decline / community fragmentation / replicable-model desire — capture `ecosystem_signal` evidence from what they already said. Do **not** ask leading taxonomy questions.

| Class | Resident signals (examples) | Downstream axis hints |
|-------|----------------------------|------------------------|
| Absence of environment | «негде», «нет места для», «раньше было, теперь нет» | `ecosystem_gap`, `missing_infrastructure` |
| Institutional decline | Closing venues, programs ending | `institutional_decline`, `mentor_shortage`, `loss_of_continuity` |
| Community fragmentation | Weak ties, underused spaces | `community_fragmentation`, `underused_resources` |
| Replicable model desire | «Should work in other districts too» | `replicable_model_needed`, `governance_signal` |

Allowed canonical labels = [`taxonomy.json`](taxonomy.json) `canonical_keys` with disposition `canonical`. Narrative glossary: [`story-label-taxonomy.md`](../../../story-label-taxonomy.md).

## Civic stash examples (demo only)

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

Do not copy these tokens into core orchestrator as the only demo body.
