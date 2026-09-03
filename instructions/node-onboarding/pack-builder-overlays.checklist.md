# Pack Builder — Overlays checklist (three pack prose identities)

**Product:** DOGEstonia — Pack Builder validation standards  
**Role of this file:** Presence/shape checklist for the **three** GPT pack prose identities (REQ-45a §4). Used with the JSON meta-schemas to accept/reject a candidate flat pack.  
**Not** Module-1 Story Interview process. **Not** primary/repair/wrapper bodies.  
**Traceability:** REQ-46 §5.0 overlays checklist · AC-GPT-REQ46-09 · REQ-45a §4–§6 · GPT-PB-03 · AC-GPT-PB03-02/04  
**Version:** 1.0 · **Date:** 2026-09-03  

**Path:** `GPT UI/instructions/node-onboarding/pack-builder-overlays.checklist.md`

**Companion standards:**  
`pack-builder-pack.schema.json` · `pack-builder-payload-schema.schema.json` · `pack-builder-taxonomy.schema.json`

---

## 0. Protocol (gateway wins · no Module-1 edits)

| Rule | Requirement |
|------|-------------|
| **Semantic align** | Candidate pack SHOULD be semantically compatible with gateway `schema-packs/<id>/<ver>/` layout (`pack.json`, `payload.schema.json`, `taxonomy.json`). |
| **Gateway wins** | On **federation drift** (field names, geo semantics, taxonomy keys vs executable gateway pack / SchemaRuntime), **gateway is authoritative**. Pack Builder standards here are candidate authoring checks — they do **not** override gateway admission. |
| **MUST NOT** | Rewrite Module-1 interview process files, OpenAPI Actions, or core Instruction process. Emit **instance** flat `schema-packs.*` only. |
| **Tallinn exemplar** | `tallinn_civic` / `v1` is a **minimal** structural reference (flat GPT copies under `GPT UI/instructions/schema-packs.tallinn_civic.v1.*`). Do **not** require deep Contour2 enrichment to pass this checklist. |

---

## 1. Flat six-file set (presence)

For `schema_id` + `schema_version`, candidate **MUST** present these flat filenames (REQ-45a):

| # | Flat artifact | Class |
|---|---------------|--------|
| 1 | `schema-packs.<id>.<ver>.pack.json` | JSON model |
| 2 | `schema-packs.<id>.<ver>.payload.schema.json` | JSON model |
| 3 | `schema-packs.<id>.<ver>.taxonomy.json` | JSON model (when taxonomy required) |
| 4 | `schema-packs.<id>.<ver>.inbound-validation.md` | **Prose identity 1 — content / admission** |
| 5 | `schema-packs.<id>.<ver>.interview-overlay.md` | **Prose identity 2 — interview guidance** |
| 6 | `schema-packs.<id>.<ver>.locale-jurisdiction.md` | **Prose identity 3 — language / jurisdiction** |

**Tallinn minimal exemplar paths (read-only reference):**

- `GPT UI/instructions/schema-packs.tallinn_civic.v1.inbound-validation.md`
- `GPT UI/instructions/schema-packs.tallinn_civic.v1.interview-overlay.md`
- `GPT UI/instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md`

---

## 2. Three prose identities — MUST (AC-09 / AC-GPT-PB03-02)

Check **presence and shape**, not empty stubs and not “content lives only in core”.

### 2.1 Content / admission — `…inbound-validation.md`

| Check | Pass if |
|-------|---------|
| File present with flat name | Path matches §1 row 4 |
| Non-empty body | File length > trivial header-only stub |
| Duty shape | Mentions domain admission / BLOCK|clarify|ACCEPT **or** completeness / GEO_SCOPE|territory STOP guidance (pack-owned lists) |
| FAIL | Guidance deferred entirely to Module-1 core with empty pack file |

### 2.2 Interview guidance — `…interview-overlay.md`

| Check | Pass if |
|-------|---------|
| File present with flat name | Path matches §1 row 5 |
| Non-empty body | Not a one-line placeholder |
| Duty shape | Covers at least one of: geo formation cues, phase→axes / ecosystem, precision/`detail_level`, pack interview lists |
| FAIL | Interview lists left “in core only” with empty overlay |

### 2.3 Language / jurisdiction — `…locale-jurisdiction.md`

| Check | Pass if |
|-------|---------|
| File present with flat name | Path matches §1 row 6 |
| Non-empty body | Not a one-line placeholder |
| Duty shape | States working language(s)/script(s) **and/or** jurisdiction / demo-zone framing for STOP/422/standing copy |
| FAIL | Locale treated as optional; content/admission-only pack without this file |

**Incomplete pack:** missing any of the three prose identities → **FAIL** this checklist (REQ-45a AC-05).

---

## 3. Shape vs empty prose

| MUST | MUST NOT |
|------|----------|
| Each prose file has identifiable sections or tables carrying pack-owned content | Ship empty `# Title` stubs “so the path exists” |
| Flat `schema-packs.` prefix naming | Nested `schema-packs/<id>/<ver>/` as the GPT Instructions upload set |
| Keep process (phases, OpenAPI) in core | Copy Module-1 process into pack prose |

---

## 4. Operator tick-box (copy into repair / validate notes)

```text
schema_id / schema_version: [[PAIR]]
[ ] pack.json validates vs pack-builder-pack.schema.json
[ ] payload.schema.json validates vs pack-builder-payload-schema.schema.json
[ ] taxonomy.json validates vs pack-builder-taxonomy.schema.json (if required)
[ ] inbound-validation.md present + non-empty (identity 1)
[ ] interview-overlay.md present + non-empty (identity 2)
[ ] locale-jurisdiction.md present + non-empty (identity 3)
[ ] flat schema-packs.* names only
[ ] gateway-wins acknowledged on federation drift
[ ] no Module-1 process edits
```

**Pass:** all applicable boxes checked.  
**Next:** REQ4 / REQ-APP-04 app·gateway admission (authoritative) — Pack Builder standards do not replace them.
