# STORY-GPT-PB-13 — Align Pack Builder instructions with geo load-gate standards

## Meta
- **Key:** `STORY-GPT-PB-13-node-onboarding-standards-geo-gate-alignment`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) · P3 story постановка 2026-09-07T08:03:42Z · instruction bodies deferred
- **Приоритет:** P1 — CLI+meta уже hard-fail; prompts/checklist/wrapper отстают
- **Тип:** product / Pack Builder **instruction bodies** under `GPT UI/instructions/node-onboarding/`
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md)
- **Upstream Done (gates exist):** GPT-PB-06…09 (meta) · SCR-NODEONB-02 (CLI+meta geo enum) · GPT-PB-10…12 (intent / Layer2; **primary body still pending**)
- **Evidence:** live `npm run pack:validate -- --id uus_veerenni_civic --ver v2` → hard `geo_load_gate` FAIL; gap audit 2026-09-07
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Delivery class:** **story постановка** (эта волна) → future P3 edits **only** listed files in `instructions/node-onboarding/`

---

# Layer 1 — Постановка

## Зачем простыми словами

Валидатор и `pack-builder-pack.schema.json` уже **отсекают** незаконный geo-словарь (`geo_filter` / `geo_scope` / `precision_levels`).  
Но операторский контур Pack Builder **не догнан**:

1. **Primary** всё ещё говорит расплывчато про zone/depth — может снова эмитить uus-v2-shaped illegal values.
2. **Repair** не знает класс `geo_load_gate` и не даёт mapping «community / adaptive / neighborhood → wire».
3. **Checklist §5** всё ещё помечает load gates как advisory-only и **не** перечисляет trio, которое уже hard в CLI+meta.
4. **Wrapper** держит устаревший `standards_set_hash` после NODEONB-02 правки pack schema.

Нужна одна стори на **выравнивание instruction set** под уже работающие standards — без правки instance uus pack (это отдельный plan → v3).

## Verified gap map (2026-09-07)

| Artifact | As-built | Gap |
|----------|----------|-----|
| `pack-builder-pack.schema.json` | enum/pattern NODEONB-02 на filter/scope/precision | **OK** — не трогать без drift |
| `pack-builder-payload-schema.schema.json` / taxonomy meta | unrelated to geo trio | **OK** |
| `pack-builder-primary-prompt.md` | `pb-primary-v1.1-20260903`; §2.3 (a)/(e) vague; нет PB-10/11/12 body | **UPDATE** elicit+emit gates + geo vocab |
| `pack-builder-repair-debug-prompt.md` | v1.1; generic fail paste only | **UPDATE** `geo_load_gate` recipe + anti-patterns |
| `pack-builder-overlays.checklist.md` §5 | «Advisory only»; geo = `geo_intake.mode` only | **UPDATE** document hard geo vocab; clarify hard vs advisory |
| `pack-builder-wrapper.md` | `standards_set_hash` = `dccaebb…` | **UPDATE** recompute (disk now `c3f11ad8…`) |
| `pack-builder-README.md` | points to §5 advisory | Soft sync after checklist |

**Instance uus v2 FAIL** = expected until Part B / v3 plan — **out of this story**.

## Scope (future P3)

Только файлы под `GPT UI/instructions/node-onboarding/`:

1. **Primary** — внедрить intent PB-10 + PB-11 + Layer2 PB-12: discovery + MUST NOT emit вне словаря; bump `prompt_version_hash`.
2. **Repair** — слоты/чеклист для `geo_filter_invalid` / `geo_scope_invalid` / `geo_precision_invalid`; NL→wire mapping; bump hash.
3. **Overlays checklist §5** — явно: geo vocab trio = **hard** (meta + CLI), не advisory; оставить readiness/ClusterLens/contour2 как сейчас (advisory unless уже hard в meta).
4. **Wrapper** — пересчитать `standards_set_hash`; при смене primary — обновить `prompt_version_hash` pin.
5. **README** — одна строка про hard geo gates + ссылка на SCR-NODEONB-02 / PB-12.

## Out of scope

- Instance pack repair / emit `uus_veerenni_civic` **v3** → отдельный plan.
- `scripts/lib/node-pack-validate/**` (SCR-NODEONB-02 Done).
- Module-1 interview Instructions; OpenAPI; gateway enums expand.
- Ослабление meta enums «чтобы uus v2 прошёл».

## Target / Acceptance Criteria

- [ ] **AC-GPT-PB13-01:** Primary содержит closed vocab + emit STOP для filter/scope/precision (PB-12 Layer2) **и** elicit/enforce readiness/civic knobs (PB-10/11).
- [ ] **AC-GPT-PB13-02:** Repair template имеет явный блок repair для `geo_load_gate` / трёх кодов FAIL + mapping guidance.
- [ ] **AC-GPT-PB13-03:** Checklist §5 не называет geo vocab trio «advisory-only»; tick-box отражает hard gate.
- [ ] **AC-GPT-PB13-04:** Wrapper `standards_set_hash` совпадает с sha256 concat текущих четырёх standards files; primary pin hash синхронен header.
- [ ] **AC-GPT-PB13-05:** Нет правок instance `schema-packs.uus_*` и нет Module-1 edits в этой story.

## Dependencies

- Hard: SCR-NODEONB-02 Done; pack meta geo locks on disk.
- Soft: PB-10 / PB-11 / PB-12 stories (intent SSOT for primary body).
- Parallel: uus v3 instance plan (не блокирует standards alignment).

## Related

| Artifact | Role |
|----------|------|
| Terminal validate uus v2 | Proof hard gates work; instance still dirty |
| PB-12 Layer2 | Primary file map / elicit table / emit tree |
| SCR-NODEONB-02 | CLI + meta SSOT for allowed values |
| `pack-builder-wrapper.md` §0 | Hash pin to refresh |

## Claims hygiene

| Claim | Evidence |
|-------|----------|
| CLI hard-fails uus v2 geo | `npm run pack:validate -- --id uus_veerenni_civic --ver v2` 2026-09-07 |
| Meta already enum-locked | `pack-builder-pack.schema.json` `$comment NODEONB-02` |
| Primary body unchanged since PB-05 pin | mtime/header `pb-primary-v1.1-20260903`; git log last `a8544cb` |
| Wrapper hash stale | claimed `dccaebb…` ≠ computed `c3f11ad8c55c28096e0b9b9f8142c0f3fb519c93096c48fc1414ece76cdafa01` |

---

# Layer 2 — P3 file map (не выполнять в этой волне постановки)

| ID | File | Change |
|----|------|--------|
| T01 | `pack-builder-primary-prompt.md` | §2.2–2.5 / §4 / §6 / §8 per PB-10+11+12; bump hash |
| T02 | `pack-builder-repair-debug-prompt.md` | §2.2 checkbox + §2.5 geo repair recipe; bump hash |
| T03 | `pack-builder-overlays.checklist.md` | §5 + §6 tick: hard geo vocab |
| T04 | `pack-builder-wrapper.md` | recompute `standards_set_hash`; sync primary pin |
| T05 | `pack-builder-README.md` | one-line hard geo + links |
| T06 | Smoke narrative | «uus v2-shaped emit» cannot pass primary self-check / would fail validate |

**Suggested repair mapping (copy into repair at P3):**

| Illegal | Legal wire | Soft place for product meaning |
|---------|------------|--------------------------------|
| `geo_filter=adaptive_by_signal_type` | `settlement` (or `district`) | policy prose / overlay |
| `geo_scope=community:…` | `settlement:tallinn` or `district:<token>` or `null` | `community_focus` / inbound STOP copy |
| precision `neighborhood` / `local_zone` / `landmark` / `multi_street` | drop from `geo_model.precision_levels`; keep ⊆ 7 | interview-overlay `detail_level` cues **as NL**, map to street/district/house |
