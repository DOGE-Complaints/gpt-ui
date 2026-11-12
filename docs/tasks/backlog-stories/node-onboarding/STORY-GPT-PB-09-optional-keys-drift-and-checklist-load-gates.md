# STORY-GPT-PB-09 — Optional-keys drift + overlays checklist load gates

## Meta
- **Key:** `STORY-GPT-PB-09-optional-keys-drift-and-checklist-load-gates`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) · pkg-000058 GIM-401…407 🟢 P3 2026-09-04T19:44:48Z
- **Приоритет:** P2 — Part A semantic drift / cosmetic for load
- **Тип:** product / Pack Builder meta + overlays checklist
- **Parent analysis:** [handoff 2026-09-04 Part A](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) — geo_model / territory / taxonomy_schema · overlays · payload · repair recs 3–4
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** soft GPT-PB-06…08 (schema edits land first); GPT-PB-03 Done
- **Blocks:** —
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

Meta сейчас **требует** `geo_model`, `gpt_instance_territory`, `taxonomy_schema`, тогда как gateway трактует их как **optional** (typed if present). Checklist overlays проверяет prose MD (gateway `load_pack` их не читает) и не предупреждает о civic/exact/geo load gates. Нужно выровнять optional-keys, добавить checklist load-gate hints **без** «must equal 13 axes», кратко отметить payload meta drift, и после P3 схем пересчитать `standards_set_hash` в wrapper.

## Scope

- Make meta `geo_model` / `gpt_instance_territory` / `taxonomy_schema` **optional** like gateway (still validate shape when present).
- Extend overlays checklist with gateway load gates (readiness / civic ClusterLens / geo modes) — **without** 13-axis lockstep.
- Light note on payload meta vs load (JSON Schema object only).
- After PB-06…08 P3: recompute wrapper `standards_set_hash` (name as AC; not this materialize wave).

## Out of scope

- Implementing Part B; gateway code; primary prompt rewrites beyond hash pointer note.
- Changing three prose identity presence rules (REQ-45a).

## Verified current state (high-level)

| Fact | Evidence |
|------|----------|
| Meta required includes geo_model, territory, taxonomy_schema | pack schema `required[]` |
| Gateway optional typed-if-present | handoff Part A drift row |
| Overlays = prose presence | checklist; MD never read by load_pack |
| Payload meta requires properties.signals | handoff Part A payload row |

## Target / Acceptance Criteria (stubs)

- [x] **AC-GPT-PB09-01:** Named keys optional in pack meta required[]; shape still validated when present.
- [x] **AC-GPT-PB09-02:** Checklist documents load gates; explicitly **forbids** «axes must equal 13».
- [x] **AC-GPT-PB09-03:** Payload drift noted (not a false load gate).
- [x] **AC-GPT-PB09-04:** P3 path includes wrapper `standards_set_hash` recompute after schema edits.

---

# Layer 2 — Реализация (P3; не в этой materialize-волне)

## Drift inventory (Part A)

| Topic | Meta today | Gateway | Action |
|-------|------------|---------|--------|
| `taxonomy_schema` | in `required[]` | optional (typed if present) | remove from required; keep property schema |
| `geo_model` | in `required[]` | optional / typed if present | remove from required |
| `gpt_instance_territory` | in `required[]` | optional path (parser when present) | remove from required |
| Overlays checklist | prose MD only | MD never read by `load_pack` | add **advisory** load-gate section |
| Payload meta | requires `properties.signals` | load needs JSON Schema object | short `$comment` / checklist note — not a hard load gate |
| Axis lockstep | — | free Contour2 | checklist **MUST NOT** say «=13 axes» |

## Subtasks (Txx)

| ID | Subtask | Detail |
|----|---------|--------|
| T01 | Pack meta `required[]` | Drop `geo_model`, `gpt_instance_territory`, `taxonomy_schema` from required; keep property definitions for when present. Retain `payload_schema`, `field_policy`, `geo_intake`, `exact_lenses`, `node_clustering` as required unless PB-06 policy says otherwise. |
| T02 | Checklist § load gates | New section: readiness_policy on exact lenses; civic `_CIVIC_REQUIRED_KEYS` + ClusterLens; geo_intake modes; field_policy states — **advisory for gateway load**, complementary to MD prose. Explicit line: Contour2 axes are **node-defined** — do **not** require equality to tallinn 13. |
| T03 | Payload drift note | One paragraph in checklist or payload schema `$comment`: requiring `properties.signals` is Pack Builder authoring aid; gateway load only needs payload schema object. |
| T04 | `standards_set_hash` | After PB-06…08 (+ this) schema/checklist P3: recompute hash in [`pack-builder-wrapper.md`](../../../../instructions/node-onboarding/pack-builder-wrapper.md) §0 per existing recipe. |
| T05 | README one-liner | Point operators to checklist load-gate section + Part A handoff. |
| T06 | Smoke (P3) | Pack without `geo_model` still meta-VALID if other required present; checklist mentions ClusterLens / readiness. |

## Точки в docs (P3)

| Action | Path |
|--------|------|
| UPDATE | `pack-builder-pack.schema.json` (`required[]`) |
| UPDATE | `pack-builder-overlays.checklist.md` |
| UPDATE | soft `pack-builder-payload-schema.schema.json` `$comment` |
| UPDATE | `pack-builder-wrapper.md` `standards_set_hash` |
| UPDATE | soft `pack-builder-README.md` |

## Target / Acceptance Criteria (refined)

- [x] **AC-GPT-PB09-01:** `geo_model` / `gpt_instance_territory` / `taxonomy_schema` optional in pack meta `required[]`; shape validated when present.
- [x] **AC-GPT-PB09-02:** Checklist documents load gates; explicitly forbids «axes must equal 13».
- [x] **AC-GPT-PB09-03:** Payload drift noted (not a false load gate).
- [x] **AC-GPT-PB09-04:** Wrapper `standards_set_hash` recomputed after schema/checklist P3.

## Dependencies

- Soft after PB-06…08 schema P3.
