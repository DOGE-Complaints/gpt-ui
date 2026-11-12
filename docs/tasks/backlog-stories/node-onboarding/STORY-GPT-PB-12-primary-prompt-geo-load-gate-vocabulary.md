# STORY-GPT-PB-12 — Primary prompt: geo load-gate vocabulary (uus v2 false emit)

## Meta
- **Key:** `STORY-GPT-PB-12-primary-prompt-geo-load-gate-vocabulary`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) · P3 Layer 1+2 analysis 2026-09-06T13:43:19Z · pkg-000061
- **Приоритет:** P1 — не повторять uus v2 geo false-admission на новых нодах
- **Тип:** product / Pack Builder **primary prompt** — постановка + Layer 2 architecture (body P3 later; **не** в этой волне)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.1
- **Incident analysis (gateway):** [`uus-veerenni-civic-v2-active-volume-smoke-2026-09-06.md`](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v2-active-volume-smoke-2026-09-06.md)
- **Sibling scripts:** [`STORY-SCR-NODEONB-02`](../../../../../scripts/docs/tasks/backlog-stories/node-onboarding/STORY-SCR-NODEONB-02-validator-geo-load-gate-hard-fail.md) — validator must reject the same class
- **Depends:** GPT-PB-01 / PB-10 / PB-11 (primary elicit/enforce waves exist); soft PB-07 (geo_intake modes already enum-locked — **not** this gap)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Delivery class:** **Layer 1 + Layer 2 analysis only** — не править `pack-builder-primary-prompt.md` / meta / uus pack в этой волне

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

Pack Builder (primary) для ноды **Uus Veerenni civic v2** выдал `pack.json`, который **прошёл** Pack Builder / скриптовую валидацию как «хорошо», но **не загружается** на gateway при активации пары `uus_veerenni_civic` / `v2`.

Проблема не в «формате файлов» после rename и не в taxonomy axes. Primary **придумал / принял от оператора продуктовые строки**, которых runtime **не знает** в трёх geo-полях civic / geo_model. Оператор думает, что пак готов; нода не стартует.

Нужна коррекция **discovery + emit** в primary: до финального emit собирать и записывать **только допустимый словарь** этих полей (или явно STOP / уточнять, если оператор хочет «community» / «adaptive» вне словаря).

## Что было неправильно (инцидент uus v2)

Источник кандидата: flat Pack Builder emit / gateway copy  
`schema-packs.uus_veerenni_civic.v2.pack.json` ↔ `schema-packs/uus_veerenni_civic/v2/pack.json`.

| Поле в эмите | Что primary / пак записал | Почему это неправильно для runtime |
|--------------|---------------------------|-------------------------------------|
| `node_clustering.civic.geo_filter` | `adaptive_by_signal_type` | Допустимы только уровни admin-фильтра: `country` \| `region` \| `settlement` \| `district` (legacy `any` → country). Свободная «политика» сюда **не** пишется. |
| `node_clustering.civic.geo_scope` | `community:uus_veerenni_with_direct_impact_routes_and_services` | Формат `<level>:<value>`, где **level** из того же набора четырёх. Токен `community:…` — продуктовая метафора района, **не** wire/runtime scope. |
| `geo_model.precision_levels` | включает `neighborhood`, `local_zone`, `landmark`, `multi_street` (+ валидные) | Список уровней precision **закрытый**: `region`, `settlement`, `district`, `street`, `house`, `house_range`, `coordinates`. Лишние имена → отказ загрузки пака. |

После rename файлов в nested layout load всё равно падает на этой цепочке (smoke analysis 2026-09-06). Temp-патч трёх полей к словарю → load SUCCESS — значит класс ошибки именно **словарь geo knobs**, не «pack builder сломан целиком».

**Отдельно (не путать):** `geo_intake.mode` / merge / mirror уже закрывались PB-07; uus v2 здесь был ок (`optional`). Этот инцидент — **civic geo_filter / geo_scope + geo_model.precision_levels**.

## Scope 

Дополнить primary (elicit + enforce emit):

1. Явно спросить / зафиксировать **geo_filter** только из четырёх уровней (или null-политики ноды без зоны — по продукту tallinn-like).
2. Явно спросить **geo_scope**: либо `null`, либо `<level>:<value>` с level ∈ тех же четырёх; **не** эмитить `community:` / свободные namespace.
3. Явно спросить **precision_levels** только из закрытого списка семи; районные/landmark смыслы — в prose overlays / signals, не в wire enum precision.
4. MUST NOT emit финальный pack с значениями вне словаря; при нужде оператора в «гибкой community-зоне» — объяснить, что runtime пока не принимает, предложить ближайший допустимый mapping (напр. `settlement:tallinn` + district в payload/signals).
5. Связать с checklist / sibling validator hard gates (SCR-NODEONB-02).

## Out of scope

- Правка текста primary / meta / uus pack в этой волне (только Layer 1+2 docs).
- Repair instance uus v2 pack.json (operator / gateway pack repair wave).
- Расширение gateway enums под `community` / `neighborhood` (отдельное product-решение runtime).
- Module-1 interview; OpenAPI; Railway Volume docs.
- JS validator body (→ SCR-NODEONB-02).

## Target / Acceptance Criteria

See **Layer 2** refined ACs (AC-GPT-PB12-01…07). Layer 1 bullets remain intent lock.

## Dependencies

- Soft: SCR-NODEONB-02 (механический hard fail тех же полей).
- Soft: meta harden для тех же enum (может быть часть SCR-NODEONB-02 или отдельная PB-meta стори — не блокирует постановку PB-12).
- Soft: PB-10/11 — load-gate elicit/enforce; **this story adds geo vocabulary** they did not cover.

## Related

| Artifact | Role |
|----------|------|
| Gateway smoke analysis 2026-09-06 | Incident evidence (Volume activate fail chain) |
| `pack-builder-primary-prompt.md` | Future P3 target (§2.3 cues a/e weak) |
| `pack-builder-pack.schema.json` | **Post-SCR-02:** enum-locks geo_filter/scope/precision (`$comment NODEONB-02`). **Incident-time:** free strings (historical FALSE ADMISSION) |
| SCR-NODEONB-02 | Validator + meta hard-fail sibling (Done P3 2026-09-06) |
| REQ4 §5.3 | Volume / NODE_SCHEMA_* product model (context only) |

## Claims hygiene

| Claim | Evidence |
|-------|----------|
| uus v2 pack contains invalid geo_filter / geo_scope / precision extras | gateway `schema-packs/uus_veerenni_civic/v2/pack.json` L47–59, L127–128 |
| Load fails until those three are aligned | smoke analysis §3 |
| **Incident-time:** primary+meta allowed free strings for these | historical: pack schema free string; primary §2.3 (a)(e) vague |
| **As-of disk now:** meta enum-locks filter/scope/precision; primary still vague | `pack-builder-pack.schema.json` L96–112, L281–296 `$comment NODEONB-02`; primary §2.3 unchanged |
| Runtime dictionaries | `scope.py` GEO_FILTER_LEVELS; `contracts.py` GEO_PRECISION_LEVELS |

No primary body / meta / instance pack edit in this постановка+Layer2 wave.

---

# Layer 2 — Техническая архитектура и субтаски (P3 later; **не выполнять здесь**)

## 0. Contours — почему «validate OK / boot FAIL» (incident-time)

```text
Operator NL + Pack Builder PRIMARY
        │ emit pack.json
        ▼
┌──────────────────────────────────────────────┐
│ Meta + SCR validate                          │
│ incident-time: free strings → FALSE ADMISSION│
│ as-of disk (post SCR-02): enum-lock → REJECT │
└──────────────┬───────────────────────────────┘
               │ activate NODE_SCHEMA_* + SCHEMA_PACKS_ROOT (REQ4 §5.3)
               ▼
┌──────────────────────────────┐
│ gateway resolve_pack         │  ← parse_cluster_geo_* + GEO_PRECISION_LEVELS
│ FAIL until vocab aligned     │
└──────────────────────────────┘
```

| Contour | Role | uus v2 as-is / now |
|---------|------|--------------------|
| Primary discovery §2.3 | Elicit geo | Vague «zone / depth» — **no closed vocab** (still true) |
| Pack meta (PB-06…09) | Hard reject | **Incident-time:** free strings missed class. **Now (SCR-02):** enum-locks geo_filter/scope/precision |
| SCR-NODEONB-01 | CLI vs meta | Inherits meta (now rejects illegal trio) |
| Gateway load | Authoritative | Fail chain smoke §3.2 until pack repaired |
| SCR-NODEONB-02 | Hard fail same vocab + meta Write | **Done** — closed meta false-admission class |

**Thesis:** PB-07 closed `geo_intake.mode`. PB-10/11 closed exact/civic readiness + ClusterLens. SCR-02 closed **meta** geo vocab hard-fail. **PB-12** still needed for **primary** elicit+emit vocabulary (body P3 later). Volume/rename are orthogonal (smoke §2–§3).

## 1. Runtime dictionaries (cite — copy into primary at P3)

| Field | Allowed values | Parser |
|-------|----------------|--------|
| `node_clustering.civic.geo_filter` | `country` \| `region` \| `settlement` \| `district`; legacy `any` → `country` | [`scope.py`](../../../../../doge-complaints-gateway/src/core/geo/scope.py) `GEO_FILTER_LEVELS` L5–7; `parse_cluster_geo_filter` L43–52 |
| `node_clustering.civic.geo_scope` | `null` **or** `<level>:<value>` with `level` ∈ same four; value non-empty | `GEO_SCOPE_LEVELS` = filter set; `parse_cluster_geo_scope` L21–40 |
| `geo_model.precision_levels[]` | ⊆ `{region, settlement, district, street, house, house_range, coordinates}` non-empty | [`contracts.py`](../../../../../doge-complaints-gateway/src/core/schema/contracts.py) `GEO_PRECISION_LEVELS` L110–119; resolver `_parse_geo_model` L450–468 |

**Not in wire enums (put in prose / signals / community_focus soft keys):** `community`, `neighborhood`, `local_zone`, `landmark`, `multi_street`, `adaptive_by_signal_type`.

## 2. Incident field map (uus v2)

| Path | Emitted value | Runtime |
|------|---------------|---------|
| `civic.geo_filter` | `adaptive_by_signal_type` | REJECT |
| `civic.geo_scope` | `community:uus_veerenni_with_…` | REJECT (level) |
| `geo_model.precision_levels` | includes `neighborhood`, `local_zone`, `landmark`, `multi_street` | REJECT unknown tokens |
| `geo_intake` | `optional` + merge/mirror | OK (PB-07 class) |
| exact/civic ClusterLens + readiness | present on gateway copy | OK (post PB-11-class repair on this copy) |

Temp patch order that proved sufficiency (smoke §3.2): filter → scope → trim precision → **SUCCESS** (24 axes).

## 3. Gap in primary today

| Section | As-built | Problem |
|---------|----------|---------|
| §2.2 Geo mode | «Mode, mirror, depth» | Depth ≠ closed precision list; no geo_filter |
| §2.3 (a) | «Node zone / geo_scope» | No format `<level>:<value>`; invites `community:` |
| §2.3 (e) | «Depth levels / detail_level / geo_model» | No `GEO_PRECISION_LEVELS` enum |
| §4 / §6 | No geo vocab emit gate | Can emit invalid pack after PB-11 civic gates pass |
| Checklist §5 | geo_intake modes only | Does not list filter/scope/precision (SCR-02 / meta follow-up) |

## 4. Semantic split (teach operator)

| Product idea | Where it belongs | Where it MUST NOT go |
|--------------|------------------|----------------------|
| «Uus Veerenni community / district focus» | inbound/overlay/locale prose; optional soft civic keys; `geo.district` in payload; territory rules | `geo_scope` level `community`; free `geo_filter` policy strings |
| Admin clustering filter depth | `geo_filter` ∈ 4 levels | Adaptive policy phrases |
| How fine address precision is modeled | `precision_levels` ⊆ 7 | neighborhood / landmark / multi_street as precision tokens |
| Interview geo intake strictness | `geo_intake.mode` (PB-07) | Confusing with geo_filter |

**Suggested mapping when operator wants community zone (NL → legal pack):**  
`geo_filter=settlement` (or `district` if product chooses); `geo_scope=settlement:tallinn` (or `district:<token>` if district is real admin token); community narrative → overlays; do **not** invent new scope levels in primary.

## 5. Where to change primary (P3 file map)

Target: [`pack-builder-primary-prompt.md`](../../../../instructions/node-onboarding/pack-builder-primary-prompt.md).

| Section | Change |
|---------|--------|
| §2.2 | Add rows: Civic `geo_filter`; Civic `geo_scope`; `geo_model.precision_levels` |
| §2.3 (a) | Require format + four levels; forbid `community:` |
| §2.3 (e) | List seven `GEO_PRECISION_LEVELS`; forbid extras |
| New §2.5b or extend §2.5 | Geo vocabulary block (table Gate → Ask → pack path) next to PB-10/11 load gates |
| §4 Invariants | MUST emit only closed geo vocab; MUST NOT invent adaptive/community as filter/scope |
| §6 Readiness | Block emit if filter/scope/precision violate dictionaries |
| §8 Cross-links | smoke analysis 2026-09-06; `scope.py` / `GEO_PRECISION_LEVELS` as cite; SCR-NODEONB-02 |
| Header | bump `prompt_version_hash` + wrapper pin |

## 6. Draft elicit table (copy at P3 — not applied now)

| Gate | Ask (NL) | Write |
|------|----------|-------|
| geo_filter | At which admin level should clustering filter stories? (country/region/settlement/district) | `node_clustering.civic.geo_filter` |
| geo_scope | Bound to a place? If yes: which level + token (e.g. settlement:tallinn). Else null. | `geo_scope` |
| Community / neighborhood story | Describe in overlays / signals — **not** as scope level | prose / `geo.district` |
| precision_levels | Which of the 7 precision tokens does this node use? | `geo_model.precision_levels` ⊆ 7 |
| STOP | Operator insists on community:/adaptive_by_signal_type/neighborhood as wire | Explain runtime reject; offer mapping |

## 7. Emit-gate decision tree (add to §6)

```text
After PB-11 civic/exact knobs OK:
  geo_filter ∈ {country,region,settlement,district} (or legacy any→country)?  --NO--> STOP
  geo_scope is null OR matches <level>:<value> with level ∈ four?             --NO--> STOP
  if geo_model present: every precision_levels[] ∈ GEO_PRECISION_LEVELS?      --NO--> STOP
  → MAY emit pack.json
```

## 8. Subtasks (Txx) — future P3 body

| ID | Subtask | Done-when |
|----|---------|-----------|
| T01 | Re-verify uus v2 pack + smoke §3 chain | Values still match Layer 1 table |
| T02 | Draft §2.5b / extend load-gate geo vocab | Table §6 in primary |
| T03 | Rewrite §2.3 (a)(e) | Closed dictionaries + format |
| T04 | Extend §2.2 topic rows | geo_filter / geo_scope / precision |
| T05 | §4 Invariants + §6 emit gate | MUST NOT emit illegal vocab |
| T06 | Operator mapping guidance | community → settlement:tallinn + prose |
| T07 | Cross-link smoke + SCR-02 | §8 |
| T08 | Align wording with PB-10/11 §2.5 | One discovery storyline, no contradiction |
| T09 | Bump prompt_version_hash / wrapper pin | After body edit |
| T10 | Smoke narrative | uus v2-shaped answers cannot pass emit gate |
| T11 | Out-of-scope guard | No gateway enum expansion; no uus pack repair; no SCR body; no meta Write unless separate story |

## 9. Точки в docs (P3 only)

| Action | Path |
|--------|------|
| UPDATE | `GPT UI/instructions/node-onboarding/pack-builder-primary-prompt.md` |
| UPDATE | soft wrapper pin / README |
| Soft parallel | SCR-NODEONB-02 + optional meta enum (not required to finish PB-12 постановка) |
| NO-OP here | uus v2 pack.json content; gateway `GEO_*` expansion; Railway |

## 10. Acceptance Criteria (refined)

- [ ] **AC-GPT-PB12-01:** Primary forbids emit `geo_filter` outside `{country, region, settlement, district}` (+ document legacy `any` → country).
- [ ] **AC-GPT-PB12-02:** Primary forbids emit `geo_scope` unless null or `<level>:<value>` with level ∈ those four.
- [ ] **AC-GPT-PB12-03:** Primary forbids emit `precision_levels` tokens outside the seven `GEO_PRECISION_LEVELS`.
- [ ] **AC-GPT-PB12-04:** Discovery explains community/neighborhood ≠ scope level ≠ precision enum.
- [ ] **AC-GPT-PB12-05:** uus v2-shaped illegal trio cannot pass «ready to emit» without STOP.
- [ ] **AC-GPT-PB12-06:** §2.3/(§2.5) + §6 updated per file map; hash bump when body lands.
- [ ] **AC-GPT-PB12-07:** No Module-1 / no gateway enum invent / no instance pack repair in this story’s P3.

## 11. Anti-patterns

| Anti-pattern | Why fail | Counter-rule |
|--------------|----------|--------------|
| `geo_filter=adaptive_by_signal_type` | Not in GEO_FILTER_LEVELS | Pick admin level only |
| `geo_scope=community:…` | Level not in GEO_SCOPE_LEVELS | Use settlement/district + prose |
| precision `neighborhood` / `landmark` / … | Not in GEO_PRECISION_LEVELS | Trim to 7; extras in overlay |
| Rely on Volume rename only | smoke: rename insufficient | Vocab must match runtime |
| Meta VALID ⇒ pack loadable | **Incident-time** true for this class; **now** meta rejects illegal trio (SCR-02) but gateway still needs aligned pack | Primary gate + SCR-02 + pack repair |
