# Pack locale-jurisdiction — `tallinn_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Declare **working languages**, **script policy** for place strings, **jurisdiction / demo-zone framing**, and **resident-facing STOP/422 copy language** for this civic instance. **Not** wire i18n slot-fill process. **Not** English taxonomy wire tokens.

| Field | Value |
|-------|--------|
| **Version** | 1.0 |
| **Date** | 2026-09-03 |
| **Traceability** | GPT-SSR-12 / REQ-45a §4 · §6; GPT-SSR-11 flat layout |
| **Binding pair** | `schema_id=tallinn_civic`, `schema_version=v1` |

**SEC-001:** Gateway Schema Runtime validates JSON only. This file is **non-executable** GPT prose. Do not call APIs from this file.

## 1. Working languages (this node)

**Declared `working_languages`:** `et`, `ru`, `en` (BCP-47-style short codes matching core wire slots).

| Code | Role for this demo node |
|------|-------------------------|
| `et` | Estonian — primary civic demo language |
| `ru` | Russian — supported resident language |
| `en` | English — supported resident / operator language |

**Interview:** the GPT MAY converse in languages beyond this set when the resident uses them, but **stash pre-flight** for demo M2 Story Intake MUST keep `session_language` ∈ `{ et, ru, en }` until product expands the allowlist.

**SSOT contract:** this pack **declares** the instance allowlist. Core [`story-i18n-policy.md`](story-i18n-policy.md) §6.2 and [`api-orchestrator.md`](api-orchestrator.md) §5.2.2 **enforce** the algorithm — full core pointer retarget to read this pack may complete in **SSR-13**; do not delete core weld text until that retarget is verified.

**Bootstrap `/lang`:** session detect stays in [`bootstrap.md`](bootstrap.md) — not redefined here.

## 2. Scripts for place strings

When the resident supplies a **place string** (street, district, city, landmark):

- **Prefer Latin script** for canonical `location_query` / address fields: freeform `<street/place>, <city>` in Latin (e.g. `Tallinn`, `Kalamaja, Tallinn`, `Tartu mnt 80, Tallinn`).
- **Cyrillic is acceptable** when the resident used it and Latin transliteration would distort meaning — do not force a misleading transliteration.
- **Do not invent** EHAK codes, `admin_id`, or bbox values — read machine scope from same-pack [`schema-packs.tallinn_civic.v1.pack.json`](schema-packs.tallinn_civic.v1.pack.json) only.

[`schema-packs.tallinn_civic.v1.interview-overlay.md`](schema-packs.tallinn_civic.v1.interview-overlay.md) applies this policy when forming geo evidence; overlay does **not** duplicate the full script rules.

## 3. Jurisdiction / demo zone framing

**Demo civic zone (resident-facing meaning):** Estonia / **Tallinn** settlement — aligned with `schema-packs.tallinn_civic.v1.pack.json` → `node_clustering.civic.geo_scope` = `settlement:tallinn` and instance territory rules in inbound §6.

**City-level canonicalization** (when confirmed location is **city-level only** — no street, district, or landmark):

| Resident input (examples) | Canonical Latin form |
|---------------------------|----------------------|
| `Tallinn` | `Tallinn, Estonia` |
| `Tallinn, Estonia` | `Tallinn, Estonia` (unchanged) |
| `Tallinna linn` | `Tallinn, Estonia` |

When the string includes district, street, or landmark detail, **preserve** that detail — do not collapse to city-only. Do not append `, Estonia` when it would simplify a more specific address the resident confirmed.

**Machine scope tokens** (`geo_scope`, `gpt_instance_territory`) remain JSON SSOT in pack.json — this section explains resident-facing framing only.

## 4. Resident-facing STOP / 422 copy languages

**Preferred copy language:** match `session_language` when known (`et` \| `ru` \| `en`); otherwise match `ui_lang` if in `working_languages`; else use **English** as fallback for short STOP/422 clarification.

### GEO_SCOPE_MISMATCH (orchestrator 422 path)

When core handles HTTP 422 `GEO_SCOPE_MISMATCH`, use this **zone framing** (adapt wording to `session_language`):

- Demo geo scope is **Estonia / Tallinn** civic zone.
- Ask the resident to clarify a place **inside** that region.
- Do not invent an in-zone address.

[`schema-packs.tallinn_civic.v1.inbound-validation.md`](schema-packs.tallinn_civic.v1.inbound-validation.md) §5 keeps the STOP **when**; this section owns **zone name framing** and copy language preference.

### Instance territory STOP (GPT pre-flight)

When place is outside instance `gpt_instance_territory` rules (see inbound §6 table):

- **STOP** — clarify; **do not stash**.
- Resident copy (English baseline — translate per session language): *Ask for a place inside this GPT's territory (demo: Tallinn / Estonia civic zone). Do not invent an in-zone address.*
- Bbox-only rule present: *Ask to confirm a place inside the map zone for this GPT.*

Admission **action** tables stay in inbound §6; wording SSOT for jurisdiction framing lives here.

## 5. Relation to wire i18n (core process — not swapped)

This file does **not** redefine:

- Trilingual object shape `{ et, ru, en }` on wire ([`story-data-model.md`](story-data-model.md) §4.1).
- Slot-fill / translation strategy ([`story-i18n-policy.md`](story-i18n-policy.md) §1–§5).
- `normalization_metadata.session_language` emit rules (§6.1).
- Phase 7 title/summary in `session_language` ([`story-interview-flow.md`](story-interview-flow.md) §7.2).

**Wire immutability (AC-GPT-REQ45a-06):** node swap must not invent new intake field names via this pack.

## 6. Relation to interview-overlay

[`schema-packs.tallinn_civic.v1.interview-overlay.md`](schema-packs.tallinn_civic.v1.interview-overlay.md) keeps:

- Phase→axes evidence tables
- Ecosystem cue examples
- Precision / `detail_level` inference
- `mirror_to_payload` sidecar sync

Overlay **reads** this file §2–§3 for script policy and city-level jurisdiction canon; overlay §geo formation cites locale instead of duplicating Latin/Cyrillic and Tallinn→Estonia examples.

## 7. Out of scope here

- **GPT-TAX-02** English snake_case taxonomy labels on wire — core normalizer/orchestrator/ingest ([`ingest-validation.md`](ingest-validation.md)).
- OpenAPI / Actions request bodies — gateway + [`api-orchestrator.md`](api-orchestrator.md).
- Gateway JSON validate — SEC-001; read [`schema-packs.tallinn_civic.v1.pack.json`](schema-packs.tallinn_civic.v1.pack.json) only.
- Pack Builder elicitation prompts — REQ-46.
- Orchestrator junk-STOP tri-lang boilerplate — core GUARD templates unless a future pack declares a full override set.
