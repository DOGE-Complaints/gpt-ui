# STORY-GPT-SSR-16 — Meta/gateway optional symmetry + checklist load gates

## Meta
- **Key:** `STORY-GPT-SSR-16-meta-optional-symmetry-and-checklist-gates`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md) (Wave 5)
- **Status:** Todo
- **Приоритет:** P1
- **Тип:** product / Pack Builder meta + overlays checklist harden (doc P3)
- **Parent analysis:** [handoff 2026-09-04](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) **Part A** (drift rows + notes 1–3 + repair 2,4,5)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) · GPT-PB-03
- **Gateway SSOT:** `GEO_PRECISION_LEVELS` · `GPT_INSTANCE_RULE_TYPES` · optional taxonomy / geo_model / territory on SchemaContext
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** **GPT-SSR-14** (pack required-list coherence); soft GPT-SSR-15
- **Blocks:** —
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

Meta сейчас **строже gateway** не там: всегда требует `geo_model`, `gpt_instance_territory`, `taxonomy_schema`. Gateway делает их **optional**. Это ломает «gateway wins» симметрию и заставляет Pack Builder выдумывать ветки (legal/mobility без Contour2 taxonomy).

Параллельно overlays checklist PASS по трём MD **не проверяет** load gates — вместе с слабой meta даёт false admission. Нужно: (1) optional alignment + enums when present; (2) checklist section «Gateway load gates»; (3) явный бан «meta VALID ≠ resolve_pack».

## Scope

- [`pack-builder-pack.schema.json`](../../../../instructions/node-onboarding/pack-builder-pack.schema.json): remove always-required for `geo_model`, `gpt_instance_territory`, `taxonomy_schema` (align gateway); when present → lock precision / rule types / reject unknown keys.
- [`pack-builder-overlays.checklist.md`](../../../../instructions/node-onboarding/pack-builder-overlays.checklist.md): new tick-box section for load gates (separate from prose identities §).
- Protocol / README one-liner: payload meta skeleton PASS and MD prose PASS are **not** substitutes for `resolve_pack` (Part A repair #5 — document only).

## Out of scope

- Re-doing SSR-14/15 field harden (already specified there).
- Part B uus repair; Railway; weakening resolver; changing payload meta to simulate load.

## Verified current state

| Topic | Meta | Gateway | Severity |
|-------|------|---------|----------|
| `geo_model` | **Required** top-level; loose extras | Optional; if present levels ⊆ `GEO_PRECISION_LEVELS` | **drift** + conditional blocker |
| `gpt_instance_territory` | **Required**; free rules | Optional; `enabled` bool; rule `type` ∈ {admin_id,admin_token,bbox} | **drift** + conditional blocker |
| `taxonomy_schema` | **Required** in pack meta | Optional → taxonomy None | **semantic drift** |
| Overlays checklist | Prose MD presence → PASS | Load reads JSON only | **cosmetic** for load; **false confidence** for operators |
| Payload meta | Requires signals/geo skeleton | Load needs object file only | **semantic drift** (document; do not use as load gate) |

## Target / Acceptance Criteria (P3)

- [ ] **AC-GPT-SSR16-01:** Pack meta `required` array does **not** force `geo_model` / `gpt_instance_territory` / `taxonomy_schema`.
- [ ] **AC-GPT-SSR16-02:** When `geo_model` present: `precision_levels` ⊆ `GEO_PRECISION_LEVELS`; unknown keys rejected.
- [ ] **AC-GPT-SSR16-03:** When `gpt_instance_territory` present: `enabled` bool; each rule `type` ∈ `GPT_INSTANCE_RULE_TYPES`; unknown keys rejected.
- [ ] **AC-GPT-SSR16-04:** Checklist has **Gateway load gates** section with ticks covering readiness_policy, civic keys, ClusterLens, axis lockstep, geo modes (points to SSR-14/15 contracts).
- [ ] **AC-GPT-SSR16-05:** Checklist + pack/README state: Pack Builder VALID + checklist PASS **≠** gateway admission / `resolve_pack` SUCCESS.
- [ ] **AC-GPT-SSR16-06:** Tallinn pack (has optional blocks present) still validates; a minimal pack without taxonomy/geo_model/territory can validate pack meta **if** load-critical SSR-14 fields OK (when such emit is in scope).

## Точки в docs (P3)

| Action | Path |
|--------|------|
| UPDATE | `pack-builder-pack.schema.json` (`required` + optional block shapes) |
| UPDATE | `pack-builder-overlays.checklist.md` |
| UPDATE | `pack-builder-README.md` and/or wrapper protocol line (false-admission) |
| UPDATE | `standards_set_hash` |

---

# Layer 2 — Глубокая проработка реализации

## Optional blocks — when present

### `GEO_PRECISION_LEVELS`

```text
region, settlement, district, street, house, house_range, coordinates
```

### `GPT_INSTANCE_RULE_TYPES`

```text
admin_id, admin_token, bbox
```

### Before → after (pack `required`)

```text
BEFORE required includes:
  taxonomy_schema, geo_model, gpt_instance_territory, …

AFTER required keeps load-critical identity + field_policy + geo_intake + exact_lenses + node_clustering
  (± payload_schema pointer as today);
  taxonomy_schema / geo_model / gpt_instance_territory → properties only (optional)
```

**Coherence with SSR-14:** do not undo civic/exact/geo_intake/field_policy harden; only change optionality of the three drift fields.

## Checklist — new section (draft ticks)

Keep existing prose-identity sections. Add:

```text
## Gateway load gates (authoring preflight — not admission)

[ ] exact_lenses non-empty; every lens has readiness_policy (3 knobs)
[ ] node_clustering.civic has all _CIVIC_REQUIRED_KEYS
[ ] active_lenses / primary_lens ∈ ClusterLens (10)
[ ] taxonomy axes lockstep = 13 Contour2 (when taxonomy emitted)
[ ] geo_intake.mode ∈ {optional, require_location_or_detail, require_detail}; merge+mirror present
[ ] field_policy values ∈ FIELD_POLICY_STATES
[ ] Operator acknowledges: checklist PASS + meta VALID ≠ resolve_pack / REQ4 F2
```

## False-admission protocol (must appear)

| Claim | Status after SSR-16 |
|-------|---------------------|
| Meta VALID ⇒ gateway load OK | **Forbidden** |
| Three MD PASS ⇒ load OK | **Forbidden** (MD unused by loader) |
| Payload meta skeleton PASS ⇒ load OK | **Forbidden** |
| `resolve_pack` SUCCESS | Authoritative load check (gateway / app) |

## Implementation notes (P3)

1. Update pack schema `$comment`: optional Contour2 branches; gateway wins.
2. Exemplars: keep tallinn with optional blocks present; add second minimal exemplar without taxonomy if useful.
3. Cross-link SSR-14/15 AC in checklist footnotes.
4. Recompute `standards_set_hash`.

## Dependencies

- Hard: SSR-14 Done (or same P3 wave after 14) so required-list edit doesn't fight empty clustering.
- Soft: SSR-15 for axis lockstep tick wording.
