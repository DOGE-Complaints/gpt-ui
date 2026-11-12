# STORY-GPT-PB-10 — Primary prompt: elicit load gates before first validation

## Meta
- **Key:** `STORY-GPT-PB-10-primary-prompt-elicit-load-gates`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) · pkg-000059 GIM-408…413 🟢 P3 story-only 2026-09-05T12:27:59Z
- **Приоритет:** P1 — закрыть класс uus gaps на этапе discovery (до emit / первой валидации)
- **Тип:** product / Pack Builder primary prompt — постановка + архитектура реализации (Layer 1 + Layer 2 analysis; **no P3 body edits in this wave**)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.1 · prior GPT-PB-01
- **Parent analysis:** [`uus-veerenni-civic-v1-pack-validation-audit-2026-09-05.md`](../../../../../docs/analysis/uus-veerenni-civic-v1-pack-validation-audit-2026-09-05.md) · handoff Part A civic/exact rows
- **Skill declared (future P3):** `prompt-engineer` · `openai-custom-gpt-builder`
- **Depends:** GPT-PB-01 Done (primary on disk); GPT-PB-06…09 Done (meta + checklist §5 encode the gates)
- **Soft:** [`STORY-SCR-NODEONB-01`](../../../../../scripts/docs/tasks/backlog-stories/node-onboarding/STORY-SCR-NODEONB-01-node-pack-validation-script.md) — validator ловит fail; primary должен не порождать тот же класс
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Delivery class:** **design SSOT (Layer 1+2)** — primary body P3 = [`GPT-PB-11`](./STORY-GPT-PB-11-primary-prompt-enforce-load-gate-emit.md); this deepen does **not** edit `pack-builder-primary-prompt.md`

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

У `uus_veerenni_civic`/`v1` первая валидация (и gateway load) падает на `pack.json`: нет `readiness_policy` на exact lenses, нет `min_size_by_lens`, civic lens ids вне `ClusterLens`. Meta (PB-06…09) это уже отвергает; **repair** закрывает дыру реактивно.

Корневая причина для *новых* онбордингов: **primary discovery** (§2 topic checklist) слишком общий («clustering / geo») — оператор не доспрашивается knobs, которые meta и gateway требуют. Нужно, чтобы Pack Builder **в первой итерации интервью (до emit и до первой валидации)** собирал эти поля явно. Тогда класс uus-gaps на новых нодах не должен повторяться.

## Scope

Дополнить обязанности primary prompt (discovery **до emit**):

- **Exact lenses:** полный ExactLensBlock, включая nested `readiness_policy` (`min_readiness_score`, `min_stories`, `require_actionable_canonical_type`); non-empty `exact_lenses`.
- **Civic clustering:** `node_clustering.civic` со всеми `_CIVIC_REQUIRED_KEYS`, включая `min_size_by_lens`.
- **Lens ids:** `active_lenses` / `primary_lens` ∈ фиксированный набор ClusterLens (10); `primary_lens` ∈ `active_lenses`.
- **Geo intake:** `geo_intake.mode` только из gateway `GEO_INTAKE_MODES`; плюс `merge` / `mirror_to_payload`.
- **Field policy:** значения только из `FIELD_POLICY_STATES` (семь состояний).
- **Contour2:** оси pack-defined / free (`minItems` 1) — **MUST NOT** требовать equality к tallinn 13; связать с checklist §5 / PB-08.
- Явная связь discovery с advisory load gates checklist §5 (оператор понимает, зачем knobs).
- MUST NOT emit финальный `pack.json` без собранных load-gate knobs (для civic-нод с clustering).

## Out of scope

- Редактирование `pack-builder-primary-prompt.md` в этой deepen-волне (body → **GPT-PB-11**).
- Rewrite repair-debug prompt (достаточно явной ссылки: fail → repair; цель — не нуждаться в repair для этого класса).
- Изменение meta-schema bodies (PB-06…09 уже Done).
- Part B repair instance `uus_veerenni_civic` v1/v2 pack.json.
- JS validation script (SCR-NODEONB-01).
- Module-1 interview process / OpenAPI.
- Gateway Schema Runtime code.
- Ослабление meta, чтобы пропускать неполный pack.

## Verified current state (high-level)

| Fact | Evidence |
|------|----------|
| Primary on disk; §2.2 clustering = one-liner | `pack-builder-primary-prompt.md` §2.2 «Clustering / card» |
| Primary body: нет строк readiness / ClusterLens / min_size_by_lens / FIELD_POLICY enum elicit | grep 2026-09-06 — zero matches |
| Meta rejects incomplete exact/civic | GPT-PB-06 Done; `pack-builder-pack.schema.json` ExactLensBlock + ClusterLens |
| Checklist §5 documents advisory load gates | `pack-builder-overlays.checklist.md` §5 |
| uus **v1** fail = pack.json gaps | audit 2026-09-05 |
| uus **v2** repeats same class after PB-10 story-only | `schema-packs.uus_veerenni_civic.v2.pack.json` — no `readiness_policy`; civic free lens ids; missing `min_size_by_lens` / `readiness_threshold`; `prompt_version_hash` still `pb-primary-v1.1-20260903` |
| PB-10 Done without primary body P3 | INDEX · Delivery class story-only · pkg-000059 |
| Execute vehicle for body | [`STORY-GPT-PB-11`](./STORY-GPT-PB-11-primary-prompt-enforce-load-gate-emit.md) Todo |

## Target / Acceptance Criteria

See **Layer 2** refined ACs (same ids AC-GPT-PB10-01…09). Intent lock; **body implementation = GPT-PB-11**.

## Dependencies

- Hard: GPT-PB-01 (primary exists); GPT-PB-06…09 (gates encoded in meta + checklist).
- Soft: SCR-NODEONB-01 (механическая первая валидация теми же standards).
- Soft: GPT-PB-02 repair — residual fails only.
- **Downstream execute:** GPT-PB-11 — apply Layer 2 file map to `pack-builder-primary-prompt.md`.

## Related links

| Artifact | Role |
|----------|------|
| `pack-builder-primary-prompt.md` | target of **PB-11** P3 (not completed under PB-10) |
| [`STORY-GPT-PB-11`](./STORY-GPT-PB-11-primary-prompt-enforce-load-gate-emit.md) | enforce/emit body wave |
| `pack-builder-overlays.checklist.md` §5 | advisory load gates SSOT for discovery alignment |
| `pack-builder-pack.schema.json` | ExactLens + civic + geo/field enums (as-built after PB-06…09) |
| audit 2026-09-05 + uus v2 flat set | gap class to prevent on new nodes |
| SCR-NODEONB-01 | scripts validate after emit |

---

# Layer 2 — Техническая архитектура и субтаски (design SSOT; body P3 → **GPT-PB-11**)

## 0. Contours (who rejects what)

```text
Operator NL
    │
    ▼
┌───────────────────────────────────┐
│ Pack Builder PRIMARY (dialogue)   │  ← PB-10 design / PB-11 body
│ §2 discovery → §6 readiness emit  │
└───────────────┬───────────────────┘
                │ emit flat pack.json (+5 siblings)
                ▼
┌───────────────────────────────────┐
│ Meta + checklist §5               │  ← GPT-PB-06…09 Done
│ pack-builder-pack.schema.json     │     mechanical reject incomplete
│ overlays.checklist.md §5 advisory │
└───────────────┬───────────────────┘
                │ fail → repair (PB-02)  │ pass →
                ▼                       ▼
         repair re-emit          SCR-NODEONB-01 / app F2
                                        │
                                        ▼
                              gateway resolve_pack / load_pack
```

| Contour | Role | Status after PB-10 story-only |
|---------|------|-------------------------------|
| **Primary discovery** | Prevent incomplete knobs before first emit | **Still gap** — primary body unchanged (grep 2026-09-06) |
| **Meta pack schema** | Reject incomplete pack.json | **OK** — PB-06…09 |
| **Checklist §5** | Advisory same gates | **OK** |
| **Repair** | Reactive fill | Exists; uus v2 still relied on post-emit fail |
| **Gateway load** | Authoritative | Same chain if pack reaches load |

**Architecture thesis:** Meta+checklist close false-admission. PB-10 specifies moving the same gate set **upstream** into primary §2 / §6. **PB-11** owns applying that map to the primary file. Until PB-11 lands, new emits (e.g. uus v2) can still produce prose/payload/taxonomy PASS + pack.json INVALID.

### 0.1 Ownership split PB-10 vs PB-11

| Concern | GPT-PB-10 | GPT-PB-11 |
|---------|-----------|-----------|
| Gap analysis, file map, elicit matrices, AC map | **SSOT (this Layer 2)** | Consumes / implements |
| Edit `pack-builder-primary-prompt.md` | Out of scope (Done story-only) | **In scope at P3** |
| Emit MUST NOT without knobs | Specified (AC-06) | Enforced in §6 body text |
| Instance uus v1/v2 repair | Out | Out (Part B) |

## 1. Gap map — primary today vs required elicit

| Gate (checklist §5 / gateway / meta) | Primary as-built | Required elicit (PB-10 → PB-11) |
|--------------------------------------|------------------|----------------------------------|
| ExactLensBlock + nested `readiness_policy` (3 knobs) | Absent | MUST ask per lens before emit |
| `exact_lenses` non-empty (`minItems: 1`) | Not stated | MUST for civic clustering packs (meta as-built) |
| civic `_CIVIC_REQUIRED_KEYS` incl. `min_size_by_lens`, `readiness_threshold` | Absent («Clustering / card» only) | MUST |
| `active_lenses` / `primary_lens` ∈ ClusterLens (10) | Absent | MUST; forbid free civic ids |
| `primary_lens` ∈ `active_lenses` | Absent | MUST (document; gateway enforces) |
| `geo_intake.mode` ∈ GEO_INTAKE_MODES + merge/mirror | §2.3 vague | MUST name three modes |
| `field_policy` ∈ FIELD_POLICY_STATES (7) | «Required/optional» only | MUST name seven states |
| Contour2 free axes (not =13) | Taxonomy vague | MUST NOT lock to 13 |
| Link to checklist §5 | Absent in §8 | MUST |

### 1.1 Recurrence proof — uus v2 (2026-09-06)

From [`schema-packs.uus_veerenni_civic.v2.pack.json`](../../../../instructions/schema-packs.uus_veerenni_civic.v2.pack.json):

| Gap | Detail |
|-----|--------|
| `readiness_policy` | Absent on all 3 `exact_lenses[]` (other ExactLens fields present) |
| Civic lens ids | `service_object_local`, `facility_or_service_need_neighborhood`, `story_shape_fallback` used as **both** exact ids and civic `active_lenses` / `primary_lens` — civic path ∉ ClusterLens |
| `min_size_by_lens` | Absent under `node_clustering.civic` |
| `readiness_threshold` | Absent under civic |
| Soft-ignored extras | `semantic_merge_policy`, `preserve_distinct_variants`, `community_focus` present (harmless) |
| Primary hash | `prompt_version_hash`: `pb-primary-v1.1-20260903` — emit under pre–load-gate primary |

Geo/field_policy on v2 look aligned (mode `optional`; policy values ∈ seven) — fail class is **exact readiness + civic ClusterLens + min_size_by_lens/readiness_threshold**, same family as v1 audit.

## 2. Exact vs civic lens rule (do not conflate)

| Surface | `lens_id` rule | Evidence |
|---------|----------------|----------|
| **exact_lenses[]** | Custom string ids **OK** once full ExactLensBlock (incl. readiness) | meta required[] L137–146; resolver `_parse_lens` |
| **node_clustering.civic** | `active_lenses` / `primary_lens` **MUST** ∈ ClusterLens (10) | pack schema civic enum; `_KNOWN_LENS_IDS` |

**uus v2 anti-pattern:** reused exact custom ids as civic active/primary — conflates the two surfaces. Primary MUST teach: exact ids may be custom; civic ids MUST be chosen from ClusterLens list (map NL names → enum).

**ClusterLens (10):** `composite_primary_micro`, `civic_domain_micro`, `failure_pattern_micro`, `civic_weight_systemic`, `desired_outcome_local`, `affected_group_local`, `geographic_district_micro`, `service_object_micro`, `deep_need_local`, `ecosystem_signal_systemic`.

**GEO_INTAKE_MODES:** `optional` \| `require_location_or_detail` \| `require_detail`.  
**FIELD_POLICY_STATES (7):** `required`, `optional`, `disabled`, `forbidden`, `node_private`, `derived`, `computed_aggregate_only`.  
**ReadinessPolicy:** `min_readiness_score`, `min_stories`, `require_actionable_canonical_type`.  
**_CIVIC_REQUIRED_KEYS:** `active_lenses`, `primary_lens`, `min_size`, `min_size_by_lens`, `readiness_threshold`, `signal_source`, `id_algorithm`, `geo_filter`, `geo_scope`, `tie_breaker`, `type_resolution`.

## 3. Where to change primary (P3 file map → PB-11)

Target: [`pack-builder-primary-prompt.md`](../../../../instructions/node-onboarding/pack-builder-primary-prompt.md) (still v1.1 / `pb-primary-v1.1-20260903`).

| Primary section | Today | P3 structural change |
|-----------------|-------|----------------------|
| §1 Session inputs | Standards bind OK | Add: discovery MUST cover checklist §5 before emit |
| §2.2 Topic checklist | One row «Clustering / card» | Split: Exact lenses; Civic clustering; Field policy enum; Taxonomy free axes |
| §2.3 Geo cues | (c)(d) vague | Explicit three GEO_INTAKE_MODES + merge/mirror required |
| **New §2.5** | — | Load-gate discovery block (tables in §9–§10 below) |
| §4 Invariants | Meta align | Civic ClusterLens; exact readiness; no axes=13; no emit without §2.5 |
| §6 Readiness | 6 bullets | Load-gate knobs complete for civic clustering |
| §5 Repair | Reactive | uus-class prevent at §2.5; repair residual |
| §8 Cross-links | No §5 | → overlays checklist §5 |
| Header | hash v1.1 | Bump hash + wrapper pin |

**Non-clustering:** Prefer elicit filled ExactLens + civic for civic-family nodes (meta `exact_lenses` `minItems: 1`). Do not invent a silent empty-array path that fails meta.

## 4. Operator NL elicitation pattern

1. Exact lenses — count + per-lens ExactLensBlock fields (§10 matrix).
2. Civic — pick from ClusterLens (10); primary ∈ active; then remaining civic keys + `min_size_by_lens`.
3. Geo mode — one of three + merge/mirror.
4. Field policy — seven states per path.
5. Taxonomy axes — free list ≥1; never force 13.

Emit gate: civic clustering incomplete → **STOP**, do not emit final pack.json.

## 5. Subtasks (Txx) — execute under **GPT-PB-11**

| ID | Subtask | Detail / done-when | Owner |
|----|---------|-------------------|-------|
| T01 | Re-grep primary gaps | Zero hits readiness/ClusterLens/min_size_by_lens at P3 start | PB-11 |
| T02 | Insert §2.5 Load-gate discovery | Use §9 draft table; exact-vs-civic split | PB-11 |
| T03 | Rewrite §2.2 rows | Clustering/geo/policy/taxonomy | PB-11 |
| T04 | Tighten §2.3 (c)(d) | GEO_INTAKE_MODES + merge/mirror | PB-11 |
| T05 | Extend §4 Invariants | ClusterLens; readiness; no axes=13; emit gate | PB-11 |
| T06 | Extend §6 Readiness | Load-gate complete criterion | PB-11 |
| T07 | Wire §1 + §8 → checklist §5 | SSOT pointer | PB-11 |
| T08 | Clarify §5 Repair vs discovery | Residual only | PB-11 |
| T09 | Paste operator enum lists | ClusterLens / GEO / FIELD_POLICY / Readiness knobs | PB-11 |
| T10 | Bump hash + wrapper pin | New `prompt_version_hash` | PB-11 |
| T11 | Smoke narrative | Would v2 gaps be impossible if §2.5 followed? | PB-11 |
| T12 | Out-of-scope guard | No uus instance repair; no meta weaken; no Module-1 | PB-11 |
| T13 | Regression note | Document: v1 audit + v2 pack prove recurrence without body P3 | PB-11 / docs |

## 6. Точки в docs (P3 via PB-11)

| Action | Path |
|--------|------|
| UPDATE | `GPT UI/instructions/node-onboarding/pack-builder-primary-prompt.md` |
| UPDATE | soft `pack-builder-wrapper.md` §0 pin / `prompt_version_hash` |
| UPDATE | soft `pack-builder-README.md` |
| NO-OP under PB-10 | meta / checklist / uus instance packs |

## 7. Target / Acceptance Criteria (refined — design; close via PB-11 body)

- [ ] **AC-GPT-PB10-01:** Discovery elicites ExactLensBlock + nested readiness (3 knobs) — §2.5 / T02/T09.
- [ ] **AC-GPT-PB10-02:** Discovery elicites civic `_CIVIC_REQUIRED_KEYS` incl. `min_size_by_lens` (+ `readiness_threshold`).
- [ ] **AC-GPT-PB10-03:** Civic lenses ∈ ClusterLens (10); `primary_lens` ∈ `active_lenses`; exact ids may stay custom.
- [ ] **AC-GPT-PB10-04:** Geo mode ∈ GEO_INTAKE_MODES (+ merge/mirror); field_policy ∈ seven states.
- [ ] **AC-GPT-PB10-05:** MUST NOT require Contour2 axes = 13.
- [ ] **AC-GPT-PB10-06:** MUST NOT emit final pack.json for civic clustering until §2.5 knobs gathered.
- [ ] **AC-GPT-PB10-07:** Primary links checklist §5 as discovery SSOT.
- [ ] **AC-GPT-PB10-08:** No Part B / Module-1 / meta weaken; repair residual.
- [ ] **AC-GPT-PB10-09:** Bump `prompt_version_hash` / wrapper pin when body lands (PB-11 T10).

*Note:* PB-10 story Status may remain Done (story-only); unchecked ACs above track **body** completion under PB-11.

## 8. Claims hygiene

| Claim | Evidence |
|-------|----------|
| Primary lacks load-gate strings | grep 2026-09-06 — no matches |
| §2.2 clustering one-liner | primary §2.2 |
| Checklist §5 | overlays L93–106 |
| Meta ExactLens required incl. readiness | pack schema L137–146 |
| uus v2 missing readiness / civic enum / min_size_by_lens | v2 pack.json L73–131 |
| PB-10 story-only Done | INDEX pkg-000059; Delivery class |
| Body execute = PB-11 | INDEX order 11 Todo |

---

## 9. Draft §2.5 table (copy into primary at PB-11 — not applied here)

| Gate | Ask operator (NL) | Write into `pack.json` |
|------|-------------------|------------------------|
| Exact lens set | How many exact lenses? Names? | `exact_lenses[]` (≥1) |
| ExactLensBlock | Per lens: sources, algorithm, scope, scale, missing policy, min_size, version | item fields |
| Readiness | Per lens: min score, min stories, require actionable type? | `exact_lenses[].readiness_policy` |
| Civic active | Which of the 10 ClusterLens ids? | `node_clustering.civic.active_lenses` |
| Civic primary | Which one is primary (must be in active)? | `primary_lens` |
| Civic sizes | Default min_size; per-lens mins | `min_size`, `min_size_by_lens` |
| Civic rest | signal_source, id_algorithm, geo_filter, geo_scope, tie_breaker, type_resolution, readiness_threshold | remaining `_CIVIC_REQUIRED_KEYS` |
| Geo intake | Mode among 3; merge?; mirror? | `geo_intake` |
| Field policy | Per path: one of 7 states | `field_policy` |
| Contour2 | Axes list for this node (any count ≥1) | `taxonomy.json` axes — **not** force 13 |

**MUST NOT emit** until rows Exact+Readiness+Civic (for civic clustering) are filled.

## 10. Field-level elicit matrices

### 10.1 ExactLensBlock (per item)

| Field | Type / note | Source |
|-------|-------------|--------|
| `lens_id` | non-empty string (custom OK) | meta |
| `source_fields` | string[] | meta |
| `algorithm` | string | meta |
| `scope` | string | meta |
| `scale` | string | meta |
| `missing_value_policy` | string | meta |
| `min_size` | number | meta |
| `version` | string | meta |
| `readiness_policy.min_readiness_score` | int | contracts ReadinessPolicy |
| `readiness_policy.min_stories` | int | same |
| `readiness_policy.require_actionable_canonical_type` | bool | same |

### 10.2 Civic block

| Field | Constraint |
|-------|------------|
| `active_lenses` | non-empty[]; each ∈ ClusterLens |
| `primary_lens` | ∈ ClusterLens ∧ ∈ active_lenses |
| `min_size` | number |
| `min_size_by_lens` | object; keys should cover active lenses |
| `readiness_threshold` | number |
| `signal_source` | string |
| `id_algorithm` | string (gateway known set includes `legacy_hash`, `sha256`) |
| `geo_filter` | string |
| `geo_scope` | string or structured per pack convention |
| `tie_breaker` | string |
| `type_resolution` | string |

## 11. Emit-gate decision tree (primary §6 logic)

```text
Is this a civic / clustering node?
  NO  → elicit minimal exact+civic OR STOP with question (do not emit empty exact_lenses)
  YES →
    For each exact lens: full ExactLensBlock incl. readiness?  --NO--> ask / STOP
    Civic active/primary ∈ ClusterLens and primary ∈ active?  --NO--> map ids / STOP
    All _CIVIC_REQUIRED_KEYS present (incl. min_size_by_lens)? --NO--> ask / STOP
    geo_intake mode ∈ 3 + merge/mirror bools?                 --NO--> ask / STOP
    field_policy values ∈ 7?                                  --NO--> ask / STOP
    taxonomy axes free (≥1), not forced to 13?                --OK-->
    → MAY emit flat six-file set
```

## 12. Proposed primary excerpt (illustrative — **do not Write to disk in this deepen**)

```markdown
### 2.5 Load-gate discovery (before emit)

You MUST complete overlays checklist §5 gates in dialogue before final pack.json emit.
- Exact lenses: full ExactLensBlock including readiness_policy (3 knobs); array non-empty.
- Civic: active_lenses/primary_lens ONLY from ClusterLens (10 ids); primary ∈ active;
  all civic required keys including min_size_by_lens and readiness_threshold.
- Geo: mode ∈ {optional, require_location_or_detail, require_detail}; merge + mirror_to_payload bools.
- field_policy values ∈ seven FIELD_POLICY_STATES.
- Contour2 axes: node-defined; MUST NOT require equality to tallinn 13.

MUST NOT emit final pack.json for a civic clustering node until the above are collected.
Exact lens_id may be custom; civic lens ids must not reuse free-form names without ClusterLens mapping.
```

## 13. Anti-patterns (from uus v1/v2)

| Anti-pattern | Why it fails | Primary counter-rule |
|--------------|--------------|----------------------|
| Exact lenses without `readiness_policy` | Meta + gateway KeyError | Elicit readiness per lens |
| Civic `active_lenses` = custom exact ids | ∉ ClusterLens | Separate civic enum pick |
| Missing `min_size_by_lens` / `readiness_threshold` | Meta + gateway missing keys | Walk full civic key list |
| Rely on repair after first validate | Operator pain; PB-10 intent | Emit gate §6 / §2.5 |
| Story-only without body P3 | v2 recurrence | Execute via PB-11 |
