# REQ-48 — Module-1 → role / archive / server mapping

| Field | Value |
|-------|--------|
| **UTC** | `2026-11-02T13:25:46Z` |
| **Story** | STORY-GPT-ROL-02-module1-role-mapping · AC-GPT-ROL02-01 (= AC-GPT-48-01) · AC-GPT-ROL02-02 |
| **Parent** | [REQ-48](../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) · parent AC-11 methodology preservation |
| **Baseline** | [`GPT UI/instructions-archive-module1-req48/`](../../instructions-archive-module1-req48/) (ROL-01 snapshot; prefer over drifting working tree) |
| **Arch** | [00-overview](../architecture/req-48-role-oriented-instruction-line/00-overview.md) · [02-export-roles-schema-and-packs](../architecture/req-48-role-oriented-instruction-line/02-export-roles-schema-and-packs.md) (**P1** packs) |
| **Wave** | pkg-000065 · GIM-447…451 |

## Legend (classification vocabulary)

| Class | Meaning |
|-------|---------|
| `shared` | Goes into sibling `core.md` (or shared prose reused by roles) |
| `role(s)` | One or more §3b role ids: `communication_setup` · `interviewer` · `extractor` · `normalizer` · `checker` |
| `archive-only` | Kept in snapshot / as-was; not authored into new roles line |
| `server-only` | Duty moves to aibridge/gateway code (not instruction prose) — peer REQ-11 for stash/SPA |

**Multi-class rows:** primary class in **Class**; secondary notes in **Notes** (no silent drop).

## C2 continuity

Working `GPT UI/instructions/` remains readable. This mapping does **not** require rewriting as-was Module-1 bodies (incl. `api-orchestrator.md`) for ROL-02 DoD. Snapshot is the classification baseline.

## Module-1 inventory rows (GIM-447 · T01)

Baseline = ROL-01 T01 inventory (24 files under snapshot). Every file has a class.

| # | File (snapshot) | Class | Role id(s) | Notes |
|---|-----------------|-------|------------|-------|
| 1 | `root.md` | `shared` | — | Hierarchy / activation → core |
| 2 | `bootstrap.md` | `role(s)` | `communication_setup` | Session context / presets bootstrap |
| 3 | `base.md` | `shared` | — | Global constitution → core |
| 4 | `communication-presets-reference.md` | `role(s)` | `communication_setup` | Preset definitions |
| 5 | `story-data-model.md` | `shared` | — | Canonical Issue fields → core |
| 6 | `story-lifecycle-instructions.md` | `shared` | — | Chain order reference → core |
| 7 | `story-interview-flow.md` | `role(s)` | `interviewer` | Dialogue intake |
| 8 | `story-i18n-policy.md` | `shared` | — | `{et,ru,en}` policy reused across roles |
| 9 | `story-policy-gate.md` | `role(s)` | `checker` | Admission decision |
| 10 | `story-normalizer.md` | `role(s)` | `normalizer` | `normalized_issue_payload` |
| 11 | `story-api-methods-reference.md` | `archive-only` | — | OpenAPI/HTTP reference retained in snapshot; live submit wire → server (REQ-11) |
| 12 | `story-label-taxonomy.md` | `archive-only` | — | SSR-10 redirect stub; enum SSOT is pack `taxonomy.json` |
| 13 | `ingest-validation.md` | `role(s)` | `extractor` | Ingest validation report |
| 14 | `ingest-deep-parsing.md` | `role(s)` | `extractor` | Non-dialogue deep parse |
| 15 | `safety-compliance.md` | `role(s)` | `checker` | Safety BLOCK/REQUEST |
| 16 | `api-orchestrator.md` | `server-only` | — | **§5.2 / §5.2.B stash + SPA redirect** → aibridge/gateway (REQ-11). As-was file stays in snapshot (C2); rewrite not required here |
| 17 | `instruction-modules-index.md` | `archive-only` | — | As-was navigation map; new line uses roles-schema manifest |
| 18 | `schema-packs.README.md` | `archive-only` | — | See pack table (P1) |
| 19 | `schema-packs.uus_veerenni_civic.v3.pack.json` | `archive-only` | — | See pack table (P1) |
| 20 | `schema-packs.uus_veerenni_civic.v3.payload.schema.json` | `archive-only` | — | See pack table (P1) |
| 21 | `schema-packs.uus_veerenni_civic.v3.taxonomy.json` | `archive-only` | — | See pack table (P1) |
| 22 | `schema-packs.uus_veerenni_civic.v3.inbound-validation.md` | `archive-only` | — | See pack table (P1) |
| 23 | `schema-packs.uus_veerenni_civic.v3.interview-overlay.md` | `archive-only` | — | See pack table (P1) |
| 24 | `schema-packs.uus_veerenni_civic.v3.locale-jurisdiction.md` | `archive-only` | — | See pack table (P1) |

**Count:** 24/24 inventory files classified · no silent drop.

## P1 pack rows — outside roles-schema volume (GIM-448 · T02 · AC-02)

Active pair: `uus_veerenni_civic` / `v3` (from snapshot `schema-packs.README.md`).

| File | Class | Volume note |
|------|-------|-------------|
| `schema-packs.README.md` | `archive-only` | Custom GPT continuity + active-pair index; **not** a roles-schema root child |
| `schema-packs.uus_veerenni_civic.v3.pack.json` | `archive-only` | Pack methodology retained in snapshot / future bridge; **outside** `instructions-roles-req15` export |
| `schema-packs.uus_veerenni_civic.v3.payload.schema.json` | `archive-only` | Same — outside roles volume (arch 02 P1) |
| `schema-packs.uus_veerenni_civic.v3.taxonomy.json` | `archive-only` | Enum SSOT retained; outside roles volume |
| `schema-packs.uus_veerenni_civic.v3.inbound-validation.md` | `archive-only` | Prose pack identity; outside roles volume |
| `schema-packs.uus_veerenni_civic.v3.interview-overlay.md` | `archive-only` | Prose pack identity; outside roles volume |
| `schema-packs.uus_veerenni_civic.v3.locale-jurisdiction.md` | `archive-only` | Prose pack identity; outside roles volume |

**P1 lock:** Do **not** extend roles.manifest with pack entries. Packs appear in this mapping (AC-01) but are **not** part of roles-schema export inventory (arch 02).

## api-orchestrator stash / SPA → server-only (GIM-449 · T03 · AC-02)

| Source | Duty | Class | Peer |
|--------|------|-------|------|
| `api-orchestrator.md` **§5.2** Story Intake browser-submit handoff (stash via `postStoryDraftStash` / `POST /story-drafts`) | Model-owned stash procedure in as-was prose | `server-only` | aibridge **REQ-11** |
| `api-orchestrator.md` **§5.2.B** Browser redirect URL to SPA | SPA redirect after `{draft_id}` | `server-only` | aibridge **REQ-11** / SPA |

**C2 note:** Rewriting working `GPT UI/instructions/api-orchestrator.md` is **not** required for ROL-02 DoD. Snapshot copy remains under `instructions-archive-module1-req48/api-orchestrator.md`. Strip/author of new roles line = ROL-03+.

## Methodology preservation (parent AC-11)

Every interview-relevant Module-1 file + active pack from ROL-01 inventory has an explicit `shared` | `role(s)` | `archive-only` | `server-only` decision. No inventory row left unclassified.

## Links

- Snapshot home: [`../../instructions-archive-module1-req48/README.md`](../../instructions-archive-module1-req48/README.md)
- Backlog story: [`../tasks/backlog-stories/req-48-role-oriented-instruction-line/STORY-GPT-ROL-02-module1-role-mapping.md`](../tasks/backlog-stories/req-48-role-oriented-instruction-line/STORY-GPT-ROL-02-module1-role-mapping.md)
- REQ-48: [`../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md`](../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md)
- Arch 02: [`../architecture/req-48-role-oriented-instruction-line/02-export-roles-schema-and-packs.md`](../architecture/req-48-role-oriented-instruction-line/02-export-roles-schema-and-packs.md)
