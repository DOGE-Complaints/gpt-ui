# STORY-GPT-PB-06 — Harden `exact_lenses` + civic `node_clustering` meta

## Meta
- **Key:** `STORY-GPT-PB-06-harden-exact-lenses-and-civic-clustering-meta`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) — P3 pkg-000055 · GIM-378…385 🟢 · 2026-09-04T19:15:31Z
- **Приоритет:** P1 — Part A load-blocker false-admission
- **Тип:** product / Pack Builder pack.json meta harden (doc+JSON Schema P3)
- **Parent analysis:** [handoff 2026-09-04 Part A](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) — `exact_lenses` · `node_clustering` rows
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) · prior GPT-PB-03
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-PB-03 Done (meta files on disk)
- **Blocks:** soft GPT-PB-09 (`standards_set_hash` after schema change)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

Pack Builder meta сейчас принимает `exact_lenses: []` и `node_clustering: {}` / lenses без `readiness_policy`. Gateway `resolve_pack` на таких кандидатах **падает** (`KeyError: readiness_policy`, missing civic keys, lenses ∉ `ClusterLens`). Нужно ужесточить [`pack-builder-pack.schema.json`](../../../../instructions/node-onboarding/pack-builder-pack.schema.json), чтобы Pack Builder **не** помечал VALID то, что gateway load отвергнет на exact/civic.

## Scope

- Harden **exact_lenses** items to gateway `ExactLensBlock` shape (incl. `readiness_policy`).
- Harden **node_clustering.civic** to `_CIVIC_REQUIRED_KEYS` + `active_lenses`/`primary_lens` ∈ `ClusterLens`.
- Keep tallinn-minimal exemplars valid under new rules (or update exemplar in-schema).
- Do **not** repair uus instance pack.json (Part B).

## Out of scope

- Part B uus pack repair / nested gateway copy.
- Taxonomy axes lockstep (→ PB-08 / invert SSR-15).
- `geo_intake` / `field_policy` enums (→ PB-07).
- Optional-keys drift / checklist (→ PB-09).
- Gateway code changes.

## Verified current state (high-level)

| Fact | Evidence |
|------|----------|
| Meta allows empty exact / empty clustering | `pack-builder-pack.schema.json` `exact_lenses` MAY `[]`; `node_clustering` MAY `{}` |
| Gateway requires non-empty exact + readiness | handoff Part A; `resolver.py` exact parse |
| Civic requires full key set + ClusterLens | `_CIVIC_REQUIRED_KEYS`; `ClusterLens` enum |
| uus false-admission path | meta VALID, gateway FAIL on readiness / civic (Part A note 2) |

## Target / Acceptance Criteria (stubs)

- [x] **AC-GPT-PB06-01:** Meta rejects exact lens objects missing `readiness_policy` (and other ExactLensBlock required fields).
- [x] **AC-GPT-PB06-02:** Meta rejects civic clustering missing any `_CIVIC_REQUIRED_KEYS` or lenses outside `ClusterLens`.
- [x] **AC-GPT-PB06-03:** Empty `exact_lenses: []` / empty `node_clustering: {}` no longer pass when gateway would require filled blocks (policy documented in story Layer 2).
- [x] **AC-GPT-PB06-04:** No Part B instance pack edits; no Module-1 process edits.

---

# Layer 2 — Реализация (P3; не в этой materialize-волне)

## Gateway contracts (cite — do not invent)

| Contract | Values / shape | Evidence |
|----------|----------------|----------|
| `ExactLensBlock` fields | `lens_id`, `source_fields`, `algorithm`, `scope`, `scale`, `missing_value_policy`, `min_size`, `readiness_policy`, `version` | [`contracts.py`](../../../../../doge-complaints-gateway/src/core/schema/contracts.py) L38–48; [`resolver.py`](../../../../../doge-complaints-gateway/src/core/schema/resolver.py) `_parse_lens` L605–617 |
| `ReadinessPolicy` | `min_readiness_score`, `min_stories`, `require_actionable_canonical_type` | `contracts.py` L29–35; `_parse_readiness` L412–417 |
| exact non-empty | `exact_lenses` must be non-empty array | `resolver.py` L650–652 |
| `_CIVIC_REQUIRED_KEYS` | `active_lenses`, `primary_lens`, `min_size`, `min_size_by_lens`, `readiness_threshold`, `signal_source`, `id_algorithm`, `geo_filter`, `geo_scope`, `tie_breaker`, `type_resolution` | `resolver.py` L35–47 |
| `ClusterLens` (10) | `composite_primary_micro`, `civic_domain_micro`, `failure_pattern_micro`, `civic_weight_systemic`, `desired_outcome_local`, `affected_group_local`, `geographic_district_micro`, `service_object_micro`, `deep_need_local`, `ecosystem_signal_systemic` | [`types.py`](../../../../../doge-complaints-gateway/src/core/cluster/types.py) L10–22; `_KNOWN_LENS_IDS` |
| civic required | `node_clustering` object with `civic` object | `resolver.py` L95–108 |

## Meta today (false-admission)

| Branch | Meta rule now | Problem |
|--------|---------------|---------|
| `exact_lenses` items | required only `lens_id`, `source_fields`, `algorithm` | Missing readiness → gateway KeyError |
| `exact_lenses` | MAY `[]` | Gateway: must be non-empty |
| `node_clustering` | MAY `{}`; additionalProperties true | Gateway requires `civic` + all keys + ClusterLens |

## Subtasks (Txx)

| ID | Subtask | Detail |
|----|---------|--------|
| T01 | Expand exact lens item schema | `required` = ExactLensBlock field set; nest `readiness_policy` object with three knobs (types: int/int/bool). |
| T02 | Non-empty exact_lenses policy | Set `minItems: 1` on `exact_lenses` **or** document exception for non-clustering nodes in `$comment` + checklist (PB-09). **Default for P3:** `minItems: 1` to match `resolver.py` L650–652 (gateway always requires non-empty today). |
| T03 | Civic object schema | When `node_clustering` present: require `civic`; require all `_CIVIC_REQUIRED_KEYS`; `active_lenses` non-empty array of `ClusterLens` enum; `primary_lens` enum ∈ ClusterLens and ∈ active_lenses (document; enforce via enum + description if JSON Schema can't express ∈ active). |
| T04 | `min_size_by_lens` | object with additionalProperties integer (or number); required key in civic. |
| T05 | Update pack schema `examples` | Replace `exact_lenses: []` / `node_clustering: {}` with tallinn-shaped minimal filled objects (copy shape from gateway `tallinn_civic/v1/pack.json` — not uus). |
| T06 | `$comment` / description | Trace GPT-PB-06 · handoff Part A · gateway wins. |
| T07 | Smoke (P3) | Validate tallinn flat pack.json vs new meta → VALID; validate intentionally broken fixture (no readiness) → INVALID. |

## Точки в docs (P3 only)

| Action | Path |
|--------|------|
| UPDATE | `GPT UI/instructions/node-onboarding/pack-builder-pack.schema.json` |
| Soft | wrapper `standards_set_hash` → PB-09 |

## Target / Acceptance Criteria (refined)

- [x] **AC-GPT-PB06-01:** Meta rejects exact lens objects missing any ExactLensBlock required field incl. nested `readiness_policy` three knobs.
- [x] **AC-GPT-PB06-02:** Meta rejects civic clustering missing any `_CIVIC_REQUIRED_KEYS` or `active_lenses`/`primary_lens` outside `ClusterLens`.
- [x] **AC-GPT-PB06-03:** `exact_lenses` `minItems: 1` (aligned resolver); empty `node_clustering: {}` fails (requires `civic`).
- [x] **AC-GPT-PB06-04:** No Part B instance pack edits; no Module-1 process edits; tallinn exemplar example in schema VALID.

## Dependencies

- Soft: PB-07 can parallel on same file — merge carefully.
- Soft: PB-09 recomputes wrapper `standards_set_hash` after P3.
