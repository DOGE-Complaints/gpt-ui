# Semantic schema runtime — backlog package

**Зона:** `GPT UI/docs/tasks/backlog-stories/semantic-schema-runtime/`  
**Тип:** backlog (постановка).  
**Метод:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — факты с paths.  
**Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) (Accepted — Wave 1–3 delivered) · continuation [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md) (Draft — Wave 4 flat + locale) · sibling gateway [`51-semantic-schema-runtime`](../../../../../doge-complaints-gateway/docs/requirements/51-semantic-schema-runtime.md)

## Architecture (one-liner)

**Wave 1 (Done):** Instruction core + two swappable pack projections → OpenAPI MUST `schema_binding` + optional `geo_detail`.

**Wave 2 (Done):** Literal gateway JSON + inbound + interview-overlay; core reads JSON; `data-model.md` archived.

**Wave 3 (Done):** Retire core `story-label-taxonomy.md` enum SSOT → pack `taxonomy.json` ([SSR-10](./STORY-GPT-SSR-10-core-taxonomy-ssot-retire-md.md)).

**Wave 4 (Done):** Flat `schema-packs.*` replace nested upload tree ([SSR-11](./STORY-GPT-SSR-11-flat-schema-packs-prefix-layout.md) Done) → `locale-jurisdiction` on flat path ([SSR-12](./STORY-GPT-SSR-12-locale-jurisdiction-pack-identity.md) Done · pkg-000049) → core/manuals retarget ([SSR-13](./STORY-GPT-SSR-13-core-retarget-flat-pack-paths.md) Done · pkg-000050). Parent: REQ-45a.

**Wave 5 (open):** Upload hygiene + **core/pack portability** — [SSR-17](./STORY-GPT-SSR-17-retire-activity-legacy-paths-inventory.md) (inventory); [SSR-18](./STORY-GPT-SSR-18-core-pack-portability-strip-node-specific.md) (strip node-specific from shared instructions); SSR-15 Cancelled (PB-08); SSR-16 Todo (meta optional).

**Target locked paths (wave 4):** see [SSR-11](./STORY-GPT-SSR-11-flat-schema-packs-prefix-layout.md) / [SSR-12](./STORY-GPT-SSR-12-locale-jurisdiction-pack-identity.md).

## Контекст одной фразой

Wave 1–3: core vs pack, JSON copy-paste, taxonomy SSOT. Wave 4: Custom GPT upload требует **flat** `schema-packs.*`; третья prose identity **locale-jurisdiction**; core links без nested segments. Wave 5: upload hygiene + **AC-016 portability** (shared core без hardcoded node pair / field lists) для массового деплоя нод.

## Stories

### Wave 1 — Done

| Order | Key | Story | Status | Depends | Subtasks |
|------|-----|-------|--------|---------|----------|
| 1 | GPT-SSR-01 | [Core vs two swappable pack artifacts](./STORY-GPT-SSR-01-core-vs-swappable-packs.md) | Done | — | T01–T07 · P6 GIM-253…255 🟢 |
| 2 | GPT-SSR-02 | [OpenAPI schema_binding + geo_detail](./STORY-GPT-SSR-02-openapi-schema-binding-geo-detail.md) | Done | GPT-SSR-01 | T01–T06 · pkg-000037 GIM-256…262 🟢 |
| 3 | GPT-SSR-03 | [Instructions emit binding + geo](./STORY-GPT-SSR-03-instructions-emit-binding-and-geo.md) | Done | GPT-SSR-01, GPT-SSR-02 | T01–T08 · pkg-000038 GIM-265…272 🟢 |

### Wave 2 — Done

| Order | Key | Story | Status | Depends | Gateway sibling |
|------|-----|-------|--------|---------|-----------------|
| 4 | GPT-SSR-04 | [Literal gateway JSON copy-paste](./STORY-GPT-SSR-04-literal-gateway-json-copy-paste.md) | Done | SSR-01…03 | pkg-000040 GIM-277…281 🟢 |
| 5 | GPT-SSR-05 | [Taxonomy JSON pack copy-paste](./STORY-GPT-SSR-05-taxonomy-json-pack-copy-paste.md) | Done | GPT-SSR-04 | GW-SSR-26/28/29 · pkg-000041 🟢 |
| 6 | GPT-SSR-06 | [inbound-validation post-SSR delta](./STORY-GPT-SSR-06-inbound-validation-post-ssr-delta.md) | Done | SSR-01…03 | pkg-000043 🟢 |
| 7 | GPT-SSR-07 | [Geo end-to-end inventory](./STORY-GPT-SSR-07-geo-end-to-end-inventory.md) | Done | GPT-SSR-03 | pkg-000044 🟢 |
| 8 | GPT-SSR-08 | [Geo precision + instance territory](./STORY-GPT-SSR-08-geo-precision-instance-territory.md) | Done | GPT-SSR-07 | GW-SSR-27 · pkg-000046 🟢 |
| 9 | GPT-SSR-09 | [Use-case audit + core consume JSON capstone](./STORY-GPT-SSR-09-use-case-audit-core-consume-json.md) | Done | SSR-04…08 | pkg-000045 🟢 |

### Wave 3 — Done

| Order | Key | Story | Status | Depends | Gateway sibling |
|------|-----|-------|--------|---------|-----------------|
| 10 | GPT-SSR-10 | [Core taxonomy SSOT retire MD](./STORY-GPT-SSR-10-core-taxonomy-ssot-retire-md.md) | Done | SSR-05, SSR-06, SSR-09 | none · pkg-000047 |

### Wave 4 — Done (REQ-45a)

| Order | Key | Story | Status | Depends | Notes |
|------|-----|-------|--------|---------|-------|
| 11 | GPT-SSR-11 | [Flat `schema-packs.*` replace nested](./STORY-GPT-SSR-11-flat-schema-packs-prefix-layout.md) | Done | SSR-09, SSR-10 | pkg-000048 GIM-333…340 🟢 |
| 12 | GPT-SSR-12 | [Locale/jurisdiction pack identity (flat path)](./STORY-GPT-SSR-12-locale-jurisdiction-pack-identity.md) | Done | **GPT-SSR-11** | pkg-000049 GIM-341…347 🟢 · `schema-packs.tallinn_civic.v1.locale-jurisdiction.md` |
| 13 | GPT-SSR-13 | [Core retarget flat pack paths](./STORY-GPT-SSR-13-core-retarget-flat-pack-paths.md) | Done | GPT-SSR-11, GPT-SSR-12 | pkg-000050 GIM-350…354 🟢 · Capstone pointers |

### Wave 5 — Upload hygiene / meta follow-ups

| Order | Key | Story | Status | Depends | Notes |
|------|-----|-------|--------|---------|-------|
| 14 | GPT-SSR-15 | [Taxonomy meta 13-axis lockstep](./STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep.md) | Cancelled (inverted by GPT-PB-08) | — | Do not execute |
| 15 | GPT-SSR-16 | [Meta optional symmetry + checklist gates](./STORY-GPT-SSR-16-meta-optional-symmetry-and-checklist-gates.md) | Todo | soft SSR-14 | Pack Builder meta; orthogonal to SSR-17 |
| 16 | GPT-SSR-17 | [Retire activity-legacy-paths-inventory from GPT upload](./STORY-GPT-SSR-17-retire-activity-legacy-paths-inventory.md) | Done (Awaiting Commits) | SSR-11…13 Done | pkg-000063 GIM-435…440 🟢 P3 Layer 1+2 2026-09-08T11:27:34Z · product retire deferred |
| 17 | GPT-SSR-18 | [Core vs pack portability — strip node-specific](./STORY-GPT-SSR-18-core-pack-portability-strip-node-specific.md) | Todo | SSR-11…13 Done | Layer 1+2 · index L60 + full shared-core leak inventory · P3 later |

## Progress

| Metric | Value |
|--------|-------|
| Stories total (Wave 1–4 + Wave 5 tracked) | 17 (SSR-01…13 + 15…18; SSR-14 absent) |
| Done (Wave 1–4) | 13 |
| Cancelled | 1 (SSR-15) |
| Todo (Wave 5) | 2 (SSR-16, SSR-18) |
| Done (Wave 5 Layer 1+2) | 1 (SSR-17) |
| In Progress (Wave 5) | 0 |
| **Wave 4 Progress** | **13/13 (100%)** |

## Recommended sequence (wave 4)

```text
SSR-11 (flat replace nested)
  → SSR-12 (locale at flat path)
  → SSR-13 (core/manuals retarget)
```

## Recommended sequence (wave 5)

```text
SSR-17 (retire editor inventory — Layer1+2 Done; product retire deferred)
  → SSR-18 (core/pack portability — Layer1+2; P3 strip leaks before multi-node)
  ∥ SSR-16 (Pack Builder meta optional — orthogonal)
SSR-15 Cancelled — skip
```

## Cross-project gateway indicators (not in this package)

| GPT story | Gateway follow-up |
|-----------|-------------------|
| SSR-05 | GW-SSR-26/28/29 |
| SSR-08 | GW-SSR-27 |
| SSR-11…13 | none required (GPT Instructions layout only); REQ-46 consumes flat emit later |

## Analysis input

- [`cross-project-node-model-portability-gap-2026-09-01.md`](../../../../../docs/analysis/cross-project-node-model-portability-gap-2026-09-01.md)
- [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md)
- [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md)
- SSR-18 leak inventory baked into story Layer 2 (index L60, API sample, i18n/taxonomy stubs, 13-axis wording)

## Граница

- **In package:** GPT UI instructions layout / pack prose identities / core path retarget / upload-set hygiene (SSR-17) / **core↔pack portability** (SSR-18).  
- **Out:** gateway Schema Runtime; Pack Builder bodies P3 (REQ-46 / SSR-16); spa; invent OpenAPI names; executing SSR-18 instruction edits in story-authoring wave; Responses API.
