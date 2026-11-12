# STORY-GPT-SSR-05 — Таксономия в JSON pack (copy-paste, не Residual include)

## Meta
- **Key:** `STORY-GPT-SSR-05-taxonomy-json-pack-copy-paste`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — устранение G-TAX-01 / G-PAYLOAD-01
- **Тип:** product / architecture (taxonomy artifact spec + cross-project dependency)
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) · [`REQ-20-label-taxonomy-and-extraction-axes.md`](../../../requirements/REQ-20-label-taxonomy-and-extraction-axes.md)
- **Siblings:** upstream [GPT-SSR-04](./STORY-GPT-SSR-04-literal-gateway-json-copy-paste.md); downstream [GPT-SSR-09](./STORY-GPT-SSR-09-use-case-audit-core-consume-json.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-04 (layout audit)
- **Gateway sibling (blocker indicator):** [GW-SSR-26](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-26-taxonomy-json-pack-contract.md) + [GW-SSR-28](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-28-tallinn-civic-taxonomy-json-seed.md) + [GW-SSR-29](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-29-taxonomy-docs-operator-copy-paste-seam.md) — loader + on-disk taxonomy in `schema-packs/` (**P3 on gateway side**)
- **Analysis input:** gap analysis G-TAX-01, G-PAYLOAD-01
- **Coherence analysis:** [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §7

## Coherence analysis (2026-09-01)

Verified against disk: 13 axes lockstep taxonomy.md §3 ↔ `axes.py` L4–19 — **zero discrepancies**. Minimum `axis_to_signal_map`: `topic_domain`→`civic_domain`, `failure_mode`→`failure_pattern` (data-model L51–54). data-model §6 Residual **superseded** by this story. P3 taxonomy path blocked until GW-SSR-26.

## Зачем простыми словами

GPT сейчас держит 13 осей и сотни canonical label keys в отдельном [`story-label-taxonomy.md`](../../../../instructions/story-label-taxonomy.md) (~347 строк). При смене ноды оператор правит **три несвязанных места**: gateway pack, GPT taxonomy, GPT data-model §2.3 (mapping осей в `signals.*`). Пользователь хочет **один JSON-артеfact** рядом с `pack.json`, который copy-paste из gateway — как для payload.

**Поправка к ожиданию:** таксономия **сегодня не лежит** в gateway `schema-packs/*.json` (grep verified 2026-09-01). SSR-01 сознательно оставил Residual include ([data-model §6](../../../../instructions/schema-packs/tallinn_civic/v1/data-model.md)). Wave 2 = закрыть этот Residual через новый JSON artifact **на обеих сторонах** (gateway SSOT + GPT copy).

## Архитектура решения (целевой artifact)

```text
schema-packs/<schema_id>/<schema_version>/
  pack.json
  payload.schema.json
  taxonomy.json          ← NEW (name TBD at spec; indicator: third copy-paste file)
```

**Минимальное содержание taxonomy JSON для copy-paste workflow:**

| Block | Purpose | Today |
|-------|---------|-------|
| `axes[]` | 13 axis keys lockstep gateway | `story-label-taxonomy.md` §3 + [`axes.py`](../../../../../doge-complaints-gateway/src/core/taxonomy/axes.py) |
| `canonical_keys` | per-axis allowed keys + disposition hints | taxonomy §4 |
| `axis_to_signal_map` | axis → `structured_payload.signals.*` | **only** data-model §2.3 |
| `wire` | reminder: per-axis on `narrative.taxonomy` (GPT-TAX-01) | separate from structured_payload |

```text
Resident story
  → GPT extracts per-axis labels (taxonomy JSON)
  → normalizer maps axis_to_signal_map → structured_payload.signals.*
  → orchestrator emits schema_binding + narrative.taxonomy
  → gateway persists both contours
```

### Option indicators (не финальный schema — выбор на deep analysis)

| Option | Pros | Cons |
|--------|------|------|
| **A:** standalone `taxonomy.json` | clean separation; third copy file | gateway loader extension |
| **B:** section inside `pack.json` | one file fewer | pack.json already large; loader change |

**Recommendation indicator:** Option A — symmetric with `payload.schema.json`; operator copies three files.

## Verified current state (As-of-Done P3 — mirrors pipeline)

| Fact | Evidence |
|------|----------|
| Taxonomy in gateway pack JSON body | **Absent** — `pack.json` L5 ref only |
| Gateway `taxonomy.json` | Present — GW-SSR-26/28 Done |
| GPT `taxonomy.json` | **Present** — `cmp` vs gateway = **IDENTICAL** |
| Axis lockstep | 13/13 — taxonomy §3 ↔ axes.py ↔ JSON `axes[]` |
| axis→signal minimum | JSON L779–782; data-model §6 superseded |
| Operator path | README v1.2 + manual §5 v2+SSR-05 third JSON step |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR05-01:** verified — taxonomy absent from current pack JSON (documented).
- [x] **AC-GPT-SSR05-02:** target artifact shape specified (Option A/B indicator + minimum blocks).
- [x] **AC-GPT-SSR05-03:** axis_to_signal_map minimum for tallinn_civic required fields documented.
- [x] **AC-GPT-SSR05-04:** gateway sibling dependency flagged (loader + on-disk file).
- [x] **AC-GPT-SSR05-05:** Residual data-model §6 superseded by this story spec.

## Inventory: story-label-taxonomy.md → JSON fields (T01 indicator)

| MD section | JSON field indicator |
|------------|---------------------|
| §3 Axis model | `axes[].key`, `axes[].role`, `axes[].disposition_default` |
| §4 Canonical keys | `canonical_keys.<axis>[]` with `key`, `meaning` |
| §2 Rules 1–9 | **partial** — disposition rules → inbound-validation; wire rules → core |
| §6 Under-applied keys | `under_applied_keys[]` per pack instance |
| axis→signal | `axis_to_signal_map.{axis}` → dotted path |

**Stays narrative-only (not in taxonomy JSON):** privacy interview process; «do not ask leading questions» → inbound-validation §4.

## Gap table (taxonomy JSON must express vs stays elsewhere)

| Concern | taxonomy JSON | inbound-validation MD | core |
|---------|---------------|-------------------------|------|
| Allowed label keys | yes | references JSON | normalizer process |
| axis→signal map | yes | — | normalizer §4.7 |
| Disposition rules | hints only | full checklists §2 | ingest process |
| English wire tokens | enum in JSON | — | GPT-TAX-02 |
| Internal-only axes | flag on axis | privacy §4 | filter on emit |

## Scope

- T01: inventory taxonomy.md sections → required JSON fields
- T02: compare with gateway axes.py + REQ-26 registries — lockstep table
- T03: gap table (above expanded) + axis_to_signal_map minimum for tallinn_civic
- T04: copy-paste operator path — gateway `taxonomy.json` → GPT same path; update README locked paths

## Out of scope

- Gateway loader implementation (GW-SSR-26 indicator only)
- OpenAPI NarrativeTaxonomy soften (Residual SSR-02)
- P3 file creation on disk
- Changing 13 axis names (REQ-36 OD-1 lock)

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | taxonomy.md → JSON field inventory | Section map table complete |
| **T02** | Lockstep with gateway axes | 13 axes match `TAXONOMY_AXIS_VALUES`; discrepancies listed |
| **T03** | Gap + axis_to_signal_map spec | Minimum map for civic required signals; gap rows for optional signals |
| **T04** | Copy-paste operator path | Step in swap v2: copy taxonomy.json; README lists third file |

## Cross-project dependency (gateway — satisfied P3)

| Need | Indicator story | P3 status |
|------|-----------------|-----------|
| On-disk `taxonomy.json` in schema-packs layout | [GW-SSR-26](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-26-taxonomy-json-pack-contract.md) | 🟢 Done |
| tallinn_civic seed data | [GW-SSR-28](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-28-tallinn-civic-taxonomy-json-seed.md) | 🟢 Done |
| Operator copy-paste + manual | [GW-SSR-29](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-29-taxonomy-docs-operator-copy-paste-seam.md) | 🟢 Done |
| Optional: validate axis keys against taxonomy on intake | gateway runtime (low priority — GPT-side omit sufficient per taxonomy §2 rule 8) |

## Dependencies

- GPT-SSR-04 (pack layout).
- Parallel with GPT-SSR-06 after T01.
- Blocks full GPT-SSR-09 capstone (core must read taxonomy JSON).

## Residual

- Migrate vs deprecate `story-label-taxonomy.md` — at P3: pack instance may symlink narrative glossary only; SSOT = JSON.
- mobility_observation / legal_process taxonomy overlays — out of tallinn scope; indicator in SSR-09 use-case matrix.
