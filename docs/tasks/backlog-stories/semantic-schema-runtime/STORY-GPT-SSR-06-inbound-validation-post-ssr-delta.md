# STORY-GPT-SSR-06 — inbound-validation.md: сверка post-SSR и delta при JSON-модели

## Meta
- **Key:** `STORY-GPT-SSR-06-inbound-validation-post-ssr-delta`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P2 — стабилизация dual contour (content MD + schema JSON)
- **Тип:** product / documentation (validation layer spec)
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) · REQ3-GPT-002
- **Siblings:** [GPT-SSR-04](./STORY-GPT-SSR-04-literal-gateway-json-copy-paste.md), [GPT-SSR-05](./STORY-GPT-SSR-05-taxonomy-json-pack-copy-paste.md), [GPT-SSR-08](./STORY-GPT-SSR-08-geo-precision-instance-territory.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-01, GPT-SSR-02, GPT-SSR-03 (Done)
- **Parallel with:** GPT-SSR-04, GPT-SSR-07
- **Coherence analysis:** [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §8

## Coherence analysis (2026-09-01)

Verified against disk: inbound **v1.3** closes SSR-01 T04 (analysis §8.1). Pointer delta **applied** P3: SSR-04→§3/§5, SSR-05→§2; §6 outline only (SSR-08 deferred). `interview-overlay.md` scope from data-model §4–§5 + geo sync rule (mirror_to_payload, pack.json L210) documented in analysis §8.3 — file create → SSR-09.

## Зачем простыми словами

Wave 1 разделил **форму данных** (data-model) и **правила содержания** (inbound-validation). Это сознательный **dual contour**: gateway проверяет JSON Schema; GPT проверяет смысл, mission, evidence labels **до** HTTP. Пользователь хочет, чтобы при переходе на literal JSON **inbound-validation остался Markdown** — и чтобы было ясно: что **уже готово** после SSR-01…03, что **точечно изменится**, и что **не трогаем**.

Переписывать inbound-validation с нуля **не нужно** — файл **v1.3** materialized: SSR-01 T04 outline закрыт; pointer delta SSR-04/05 **applied** P3.

## Архитектура dual contour (не меняется в wave 2)

```text
┌─────────────────────────────┐     ┌──────────────────────────────┐
│ Schema shape (JSON copy)    │     │ Content rules (MD inbound)   │
│ pack.json + payload.schema  │     │ inbound-validation.md        │
│ validated on GATEWAY        │     │ enforced in GPT interview    │
└─────────────────────────────┘     └──────────────────────────────┘
         SEC-001                              REQ3-GPT-002
```

**Process algorithms** остаются в core: [`story-policy-gate.md`](../../../../instructions/story-policy-gate.md), [`ingest-validation.md`](../../../../instructions/ingest-validation.md). Pack inbound даёт **списки и поля ноды**, core — **алгоритм решения**.

## Coverage matrix: SSR-01 T04 outline vs inbound v1.3 (verified post-P3)

| SSR-01 T04 outline item | inbound v1.3 § | Status |
|-------------------------|----------------|--------|
| Mission / IRRELEVANT_NON_CIVIC / standing | §1 | **Done** — domain lists from policy-gate §7.1 |
| Label enum / multi-axis evidence | §2 | **Done** — pointer to same-pack `taxonomy.json` |
| Completeness vs pack required fields | §3 | **Done** — `pack.json` `field_policy` literals (civic_domain + failure_pattern) |
| No leading taxonomy questions | §4 | **Done** — REQ-35 privacy baseline |
| GEO_SCOPE_MISMATCH resident copy | §5 | **Done** — Estonia/Tallinn copy; geo_scope cite `settlement:tallinn` |
| Instance territory (outline) | §6 | **Outline only** — SSR-08 deferred |
| HTTP Action schemas | — | **Correctly absent** — SEC-001 |

**Вывод:** SSR-01 inbound scope **закрыт** на v1.3 MD-модели. Wave 2 pointer delta **applied** (SSR-04/05); prose-only home → SSR-09.

## Expected delta при JSON-модели (SSR-04 / SSR-05)

| inbound § | Change | Trigger | Severity |
|-----------|--------|---------|----------|
| §2 Label enum | Pointer: `story-label-taxonomy.md` → `taxonomy.json` (same pack dir) | SSR-05 | **required** |
| §3 Completeness | Cite `pack.json` `field_policy` by path literal, not data-model prose | SSR-04 | **required** |
| §5 GEO_SCOPE | Unchanged text; optional: cite `node_clustering.civic.geo_scope` from pack.json | SSR-04 | optional clarity |
| **New §6** | Instance territory resident-facing STOP/copy rules | SSR-08 | **new section** when territory model exists |
| §? Interview overlay | Phase→axes + ecosystem cues from data-model §5 | SSR-04 migration | **new file** `interview-overlay.md` recommended |

## Prose-only content: home decision (from data-model.md eviction)

| Content | Not schema validation | Recommended home |
|---------|----------------------|------------------|
| §4 Geo formation / Tallinn city canon | yes — formatting rules | `interview-overlay.md` |
| §5 Phase→axes + ecosystem cues | yes — interview guidance | `interview-overlay.md` |
| §7 Stash examples | yes — demo only | `examples.md` or inbound appendix |
| §3.3 GEO_SCOPE copy | resident messaging | **stay** inbound §5 |
| §2.3 axis→signal | structural map | **taxonomy.json** (SSR-05), not inbound |

**Principle:** inbound-validation = **admission + evidence + completeness + resident copy for errors**. Interview overlay = **how to ask and capture**, not **what shape JSON has**.

## Explicit no-change list (do not rewrite)

- §1 Mission BLOCK/ACCEPT domain lists — unchanged unless product changes mission
- §4 No leading taxonomy questions — unchanged (privacy invariant)
- Core policy-gate / ingest-validation process modules — unchanged
- Dual contour operator manual §6 — unchanged philosophy
- Gateway validate authority — unchanged SEC-001

## Scope

- T01: coverage matrix SSR-01 T04 vs inbound v1.3 (table above — verified post-P3)
- T02: delta spec for SSR-04/05/08 pointer updates
- T03: prose-only migration map → `interview-overlay.md` vs inbound split
- T04: explicit no-change list signed off in story AC

## Out of scope

- Creating interview-overlay.md on disk (→ GPT-SSR-09 P3)
- Moving process algorithms from core policy-gate
- Gateway enforce_policy on intake
- Taxonomy JSON schema design (→ GPT-SSR-05)

## Verified current state (As-of-Done P3)

| Fact | Evidence |
|------|----------|
| inbound-validation **v1.3** | `GPT UI/instructions/schema-packs/tallinn_civic/v1/inbound-validation.md` |
| §2 → pack `taxonomy.json` | SSR-05/06 pointer (not MD include chain) |
| §3 → `pack.json` `field_policy` literals | civic_domain + failure_pattern required |
| §5 geo_scope cite | `settlement:tallinn` |
| interview-overlay.md | Absent — scope outlined; create → SSR-09 |
| Evidence | [`gpt-ssr-06-inbound-delta-evidence-2026-09-01.md`](../../../analysis/gpt-ssr-06-inbound-delta-evidence-2026-09-01.md) |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Coverage matrix | SSR-01 T04 rows vs inbound § — all Done or explained |
| **T02** | Delta spec | Table of § changes with trigger story IDs |
| **T03** | Prose-only home | interview-overlay.md scope outline; list sections to move from data-model |
| **T04** | No-change list | Explicit bullets; AC prevents unnecessary rewrite |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR06-01:** coverage matrix confirms inbound closes SSR-01 T04.
- [x] **AC-GPT-SSR06-02:** delta spec lists only pointer/home changes for JSON transition; required pointers applied.
- [x] **AC-GPT-SSR06-03:** interview-overlay.md scope defined (phase→axes, geo canon, ecosystem).
- [x] **AC-GPT-SSR06-04:** no-change list prevents scope creep into core process rewrite.

## Dependencies

- GPT-SSR-01…03 Done.
- Inputs from GPT-SSR-04 (migration map), GPT-SSR-05 (taxonomy path), GPT-SSR-08 (§6 territory).
- Feeds GPT-SSR-09 capstone file moves.

## Residual

- Single vs split MD files per pack instance — default three: JSON×2 + taxonomy JSON + inbound + interview-overlay (operator upload bundle).
