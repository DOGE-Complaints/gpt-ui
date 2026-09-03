# Pack interview-overlay — `tallinn_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Prose-only interview guidance: geo formation canon, phase→axes capture, ecosystem cues. **Not** JSON Schema shape (read `schema-packs.tallinn_civic.v1.pack.json` + `schema-packs.tallinn_civic.v1.payload.schema.json`). **Not** admission rules (read `schema-packs.tallinn_civic.v1.inbound-validation.md`).

| Field | Value |
|-------|--------|
| **Version** | 1.2 |
| **Date** | 2026-09-03 |
| **Traceability** | GPT-SSR-12 / GIM-343–344; GPT-SSR-08 / GIM-312–314; GPT-SSR-09 / GIM-308; migrated from archived `archive/data-model.v1.3.md` §4–§5 |
| **Binding pair** | `schema_id=tallinn_civic`, `schema_version=v1` |

**SEC-001:** Gateway Schema Runtime validates JSON. This file is GPT process overlay only.

## Geo formation (this civic instance)

Process (when to form / omit `location_query` per **mode**) stays in [`story-normalizer.md`](story-normalizer.md) §4.6. Read `geo_intake.mode` from same-pack [`schema-packs.tallinn_civic.v1.pack.json`](schema-packs.tallinn_civic.v1.pack.json) (not prose here).

**Script policy and city-level jurisdiction canon** for place strings: read same-pack [`schema-packs.tallinn_civic.v1.locale-jurisdiction.md`](schema-packs.tallinn_civic.v1.locale-jurisdiction.md) §2–§3. Apply locale §3 city canon when forming `location_query` at city-level only; preserve street/district/landmark detail when the resident confirmed more.

**Reconcile with `mode=optional`:** this pack does **not** require `location_query` when a place is confirmed. Core MUST NOT force `location_query` for this instance. MAY still form it when the resident gave a place.

## Geo sidecar sync (mirror_to_payload)

When `schema-packs.tallinn_civic.v1.pack.json` → `geo_intake.mirror_to_payload` is `true` (civic default): keep `geo_detail.address.*` consistent with `structured_payload.geo.*` before stash. Latitude/longitude live on `geo_detail` root only — **not** in payload schema.

## Precision index (`detail_level`)

Read `geo_model` from [`schema-packs.tallinn_civic.v1.pack.json`](schema-packs.tallinn_civic.v1.pack.json). Allowed levels = `precision_levels` (also `schema-packs.tallinn_civic.v1.payload.schema.json` `geo.detail_level` enum). Inference default: `from_address_depth`.

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

Read `gpt_instance_territory` from same-pack [`schema-packs.tallinn_civic.v1.pack.json`](schema-packs.tallinn_civic.v1.pack.json). Gateway **parse-only**; GPT **STOP / clarify** per [`schema-packs.tallinn_civic.v1.inbound-validation.md`](schema-packs.tallinn_civic.v1.inbound-validation.md) §6 when resident place is outside instance rules (may be stricter than node `geo_scope`).

**This pack:** `admin_token` settlement `tallinn`. Do not invent EHAK/`admin_id` or bbox rules unless present in pack JSON.

## Phase → axes + ecosystem cues

Phases 1–7 **process** stays in [`story-interview-flow.md`](story-interview-flow.md). Field/axis lists for this civic instance:

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

Allowed canonical labels = [`schema-packs.tallinn_civic.v1.taxonomy.json`](schema-packs.tallinn_civic.v1.taxonomy.json) `canonical_keys` with disposition `canonical`. Process / redirect: [`story-label-taxonomy.md`](story-label-taxonomy.md) stub → [`ingest-validation.md`](ingest-validation.md) Label process rules.

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
