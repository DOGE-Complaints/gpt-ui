# STORY-GPT-SSR-12 — Locale/jurisdiction pack identity (flat path)

## Meta
- **Key:** `STORY-GPT-SSR-12-locale-jurisdiction-pack-identity`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Приоритет:** P1 — Wave 4 (REQ-45a §4 · §6)
- **Тип:** product / architecture + instruction overlay (doc-only P3; SEC-001)
- **Parent REQ:** [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md) · parent [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** upstream [GPT-SSR-11](./STORY-GPT-SSR-11-flat-schema-packs-prefix-layout.md); downstream [GPT-SSR-13](./STORY-GPT-SSR-13-core-retarget-flat-pack-paths.md); related [GPT-SSR-06](./STORY-GPT-SSR-06-inbound-validation-post-ssr-delta.md) dual contour
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** **GPT-SSR-11** (flat layout Done — target paths exist)
- **Blocks:** GPT-SSR-13 (complete three-prose set for retarget)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — claims with path:line

## Зачем простыми словами

Сейчас «язык» и «юрисдикция» **размазаны**: часть в core i18n (слоты wire), часть в bootstrap (детект языка), часть в overlay (Latin/Cyrillic + Tallinn canon), часть в inbound (copy Estonia/Tallinn), часть в orchestrator (allowlist et|ru|en + tri-lang STOP copy). Swap ноды **не** может безопасно сменить юрисдикцию/скрипт/рабочие языки, не трогая core.

REQ-45a требует отдельный pack artifact **`locale-jurisdiction`**. Эта story: (1) зафиксировать as-built sprawl; (2) зафиксировать **безопасные принципы** выноса; (3) создать flat файл; (4) точечно **переложить** instance locale/script/jurisdiction из overlay/inbound pointers — **без** поломки wire process и GPT-TAX-02.

---

## Архитектура: три prose identities + core process

```text
                    ┌─────────────────────────────────────┐
                    │ Module-1 CORE (process — not swapped)│
                    │ bootstrap ui_lang detect             │
                    │ story-i18n-policy: slot-fill rules   │
                    │ interview-flow: phases / Phase 7    │
                    │ normalizer/orchestrator: wire map   │
                    │ GPT-TAX-02: English label tokens    │
                    └──────────────┬──────────────────────┘
                                   │ reads active pack
          ┌────────────────────────┼────────────────────────┐
          ▼                        ▼                        ▼
┌──────────────────┐  ┌──────────────────────┐  ┌─────────────────────────┐
│ inbound-validation│  │ interview-overlay    │  │ locale-jurisdiction     │
│ Contour B         │  │ interview guidance   │  │ language / jurisdiction │
│ admission lists   │  │ phase→axes           │  │ working langs/scripts   │
│ completeness      │  │ ecosystem cues       │  │ jurisdiction frames     │
│ GEO_SCOPE STOP    │  │ precision tables     │  │ STOP/422 *language(s)*  │
│ territory STOP    │  │ geo formation *canon*│  │ geo *script* policy     │
└──────────────────┘  └──────────────────────┘  └─────────────────────────┘
          │                        │                        │
          └────────────────────────┴────────────────────────┘
                                   │
                                   ▼
                         Gateway validate JSON only (SEC-001)
```

| Contour | Owns | Must NOT own |
|---------|------|--------------|
| **Core process** | When to detect language; how to fill `{et,ru,en}` slots; English taxonomy **wire** tokens; HTTP pre-flight algorithm | Instance city lists; “prefer Latin for Tallinn”; demo-zone marketing copy |
| **inbound** | Admission BLOCK lists; GEO_SCOPE / territory **STOP action** + short resident copy | Phase→axes tables; Latin/Cyrillic policy as primary home |
| **interview-overlay** | What evidence to capture per phase; ecosystem cue examples; precision inference; geo formation **structure** (when optional) | Wire allowlist; FR-M1-028 algorithm; invent OpenAPI fields |
| **locale-jurisdiction** | **Which** languages/scripts this node supports; jurisdiction/demo-zone framing; **which language(s)** to speak STOP/422 copy in; geo **script** preference for place strings | Phase process; taxonomy English wire rule; JSON Schema validate |

**Critical split (safe for system):**

| Concern | Home after SSR-12 | Why |
|---------|-------------------|-----|
| Wire shape `{ et, ru, en }` + `session_language` emit | **core** `story-i18n-policy.md` | OpenAPI / REQ-22 contract — node swap must not invent wire keys |
| Detect `ui_lang` / `/lang` | **core** `bootstrap.md` | Session machinery |
| English snake_case taxonomy labels on wire | **core** GPT-TAX-02 (normalizer/orchestrator/ingest) | Gateway/embeddings contract — **not** “locale” |
| Working language allowlist for **this node** (may equal et\|ru\|en) | **locale-jurisdiction** (declare) + core **reads** pack | Today welded in i18n §6.2 + orchestrator L611 — unsafe for non-civic nodes |
| Prefer Latin / accept Cyrillic for place strings | **locale-jurisdiction** (policy) ← migrate from overlay L19 | Instance script policy, not interview phase logic |
| `<City>, Estonia` city canon examples | **locale-jurisdiction** (jurisdiction frame) + overlay may **pointer** | Jurisdiction framing; formation *process* stays normalizer §4.6 |
| GEO_SCOPE / territory STOP **action** | **inbound** §5–§6 | Admission contour |
| GEO_SCOPE / territory **wording language + zone name framing** | **locale-jurisdiction** supplies language/jurisdiction text; inbound cites it | Avoid duplicating Tallinn prose in three files |
| Phase→axes / ecosystem RU cue examples | **interview-overlay** | Interview guidance (resident evidence), not wire locale |
| Orchestrator junk-STOP tri-lang boilerplate (et/ru/en) | **core process** templates; optional pack override later | Fail-closed GUARD — do not move in this story unless pack declares full set |

---

## As-built sprawl inventory (verified)

### A. Language / i18n (process vs instance)

| Location | What lives there | Class today | Target after SSR-12 |
|----------|------------------|-------------|---------------------|
| [`bootstrap.md`](../../../../instructions/bootstrap.md) L77–110, L334 | `ui_lang` detect; `/lang ru\|et\|en` | **process** | stay core |
| [`story-i18n-policy.md`](../../../../instructions/story-i18n-policy.md) §1–§5 | Surface language; fidelity; no distortion | **process** | stay core |
| same §2 | Fill `{et,ru,en}` strategy | **process** (wire shape) | stay core |
| same §6–§6.1 | `session_language` artifact + wire map | **process** | stay core |
| same §6.2 L79–87 | Demo allowlist `et\|ru\|en` for stash | **instance welded as process** | **declare in locale**; core pre-flight **reads pack allowlist** (SSR-13 may finish pointers; this story defines pack field) |
| [`story-interview-flow.md`](../../../../instructions/story-interview-flow.md) L153–162 | Phase 7 title/summary in `session_language`; translation transparency note + et/ru/en examples | **process** + fixed three-lang disclosure | process stays; disclosure **language set** may cite pack working languages |
| [`story-normalizer.md`](../../../../instructions/story-normalizer.md) L212–213, L263 | session/detected language; English taxonomy tokens | **process** + **wire** | stay core |
| [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) L611–667, L446–450 | stash allowlist; English labels STOP; junk STOP tri-lang copy | **process** + **welded allowlist** | allowlist → read pack locale; tri-lang GUARD copy stay core unless pack overrides |
| [`story-data-model.md`](../../../../instructions/story-data-model.md) L60–62, L77 | trilingual card fields; `impact_estimation` “Estonia scale” | wire shape + **jurisdiction-tinged enum prose** | wire stay; “Estonia scale” note → locale jurisdiction framing (pointer) |

### B. Jurisdiction / geo script (overlay + inbound)

| Location | What lives there | Class today | Target after SSR-12 |
|----------|------------------|-------------|---------------------|
| [`interview-overlay.md`](../../../../instructions/schema-packs/tallinn_civic/v1/interview-overlay.md) L19 | Prefer Latin; Cyrillic OK if resident used it | **locale/script in overlay** | **move to locale**; overlay → “read locale-jurisdiction geo script policy” |
| same L21–27 | Tallinn → `Tallinn, Estonia` city canon | **jurisdiction + formation** | **jurisdiction frame + examples → locale**; formation *when/optional* stays overlay/normalizer |
| same L31–55 | mirror_to_payload; detail_level; territory pointer | **interview guidance** | stay overlay |
| same L57–77 | phase→axes; ecosystem cues (incl. RU examples) | **interview guidance** | stay overlay |
| [`inbound-validation.md`](../../../../instructions/schema-packs/tallinn_civic/v1/inbound-validation.md) L57–59 | GEO_SCOPE copy “Estonia / Tallinn” | **admission copy** | STOP action stay inbound; **zone framing + copy language** cite locale |
| same L61–74 | territory STOP + Tallinn/Estonia resident copy | **admission** | same split |
| [`pack.json`](../../../../instructions/schema-packs/tallinn_civic/v1/pack.json) `geo_scope` / `gpt_instance_territory` | Machine scope tokens | **JSON SSOT** | stay JSON; locale explains resident-facing meaning |

### C. Absent (Historical pre-P3 — superseded by Verified as-built)

| Fact | Evidence (at story authoring) |
|------|----------|
| `locale-jurisdiction.md` (any layout) | Was absent under `GPT UI/instructions/` — **now present** post-P3 (see Verified) |
| Flat `schema-packs.*` | Prerequisite SSR-11 was Todo — **now Done** (pkg-000048) |

### D. Sprawl risk summary

| Risk ID | Risk | If ignored |
|---------|------|------------|
| R-LOC-01 | Script policy only in overlay | Operator edits overlay for “Arabic node” but forgets inbound Tallinn copy |
| R-LOC-02 | Allowlist welded in i18n §6.2 + orchestrator | Non-et/ru/en node cannot stash without core edit (AC-016 break) |
| R-LOC-03 | Moving wire `{et,ru,en}` into pack | Breaks OpenAPI / spa preview / REQ-22 |
| R-LOC-04 | Moving GPT-TAX-02 English labels into “locale” | Confuses human language with wire tokens → embeddings drift |
| R-LOC-05 | Duplicating Tallinn strings in overlay+inbound+locale | Triple drift on swap |
| R-LOC-06 | Empty locale file “for completeness” | Pack Builder/REQ-46 thinks Done; runtime still core-welded |

---

## Safe extraction principles (locked for P3)

1. **Process vs instance:** algorithms stay core; lists/frames/scripts/working-langs stay pack.
2. **Wire immutability:** no new intake field names; no change to `{et,ru,en}` object shape in this story (AC-GPT-REQ45a-06).
3. **English taxonomy ≠ locale:** GPT-TAX-02 remains core wire rule; locale may note “resident interview languages” separately.
4. **Single home for script policy:** locale owns; overlay/inbound **pointer only** after migrate.
5. **Single home for jurisdiction framing:** locale owns demo-zone / country-city framing text; inbound owns STOP **when**; orchestrator owns HTTP 422 **handling**.
6. **Allowlist declare-then-read:** locale declares `working_languages[]` (tallinn seed: et, ru, en); core pre-flight continues to enforce — pointer update may complete in SSR-13 if too large for this story; **this story MUST document the contract**.
7. **No silent core delete:** do not remove i18n §6.2 text until SSR-13 retarget proves pack read path; prefer “SSOT = pack; core cites pack” in one P3 wave or explicit Residual.
8. **SEC-001:** locale MD is non-executable; gateway does not validate prose.
9. **Flat path only:** `schema-packs.tallinn_civic.v1.locale-jurisdiction.md` (SSR-11 Done).
10. **Incomplete pack rule:** active pair without locale file = incomplete GPT-facing pack (REQ-45a AC-05).

---

## Целевой путь + outline файла

```text
GPT UI/instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md
```

### Recommended sections (seed → P3 body)

| § | Title | Seed from (as-built) | Notes |
|---|-------|----------------------|-------|
| 0 | Meta / binding pair / SEC-001 | pack README | Non-executable |
| 1 | Working languages | i18n §6.2; bootstrap `/lang` set | Declare `et\|ru\|en` for tallinn; interview MAY use others but stash allowlist = this set |
| 2 | Scripts for place strings | overlay L19 | Prefer Latin; Cyrillic when resident-used |
| 3 | Jurisdiction / demo zone framing | overlay L21–27; inbound §5–§6; pack.json geo_scope | Estonia / Tallinn meaning for residents; not EHAK invent |
| 4 | Resident-facing STOP/422 copy languages | inbound §5–§6; orchestrator 422 path | Which `session_language` templates to prefer; point inbound to this § |
| 5 | Relation to wire i18n | story-i18n-policy | Explicit: slots/process stay core; this file does not redefine `{et,ru,en}` |
| 6 | Relation to interview-overlay | overlay | Overlay reads §2–§3 for script/canon; keeps phase→axes |
| 7 | Out of scope here | GPT-TAX-02; OpenAPI; gateway validate | Prevent conflation |

---

## Migration map (what moves on P3 of this story)

| Content | From | To | Overlay/inbound after |
|---------|------|-----|------------------------|
| Latin/Cyrillic prefer rules | overlay L19 | locale §2 | One-line pointer |
| City canon Tallinn→Estonia examples | overlay L21–27 | locale §3 | Pointer + “apply locale canon when city-level only” |
| “Estonia / Tallinn” zone framing in STOP copy | inbound §5–§6 (partial) | locale §3–§4 | Inbound keeps STOP table; copy text “see locale §4” or short cite |
| working_languages allowlist declaration | i18n §6.2 (welded) | locale §1 | Core keeps enforce algorithm; SSOT list → pack (pointer may finish SSR-13) |
| phase→axes, ecosystem, precision, mirror | overlay | **stay** | — |
| admission BLOCK lists | inbound | **stay** | — |

---

## Scope

- T01: Sprawl inventory freeze (tables above — expand path:line at P3 if drift).
- T02: Principles + duty split signed in story AC.
- T03: Author flat `locale-jurisdiction.md` per outline.
- T04: Pointer updates in **flat** overlay + inbound (post-SSR-11 paths) — script/canon/jurisdiction cites locale; no full core rewrite.
- T05: README active pair sixth artifact.
- T06: Explicit non-moves list (wire, TAX-02, bootstrap detect).

## Out of scope

- Flat migrate of existing five files (→ SSR-11).
- Full core retarget of all nested path strings (→ SSR-13).
- Changing OpenAPI / inventing wire fields.
- Pack Builder prompts (REQ-46).
- Softening GPT-TAX-02 English labels.
- Gateway `pack.json` pointer key (REQ-45a §9.3).

## Verified current state (short)

| Fact | Evidence |
|------|----------|
| locale file present (as-built post-P3) | [`schema-packs.tallinn_civic.v1.locale-jurisdiction.md`](../../../../instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md) v1.0 |
| Script policy + jurisdiction framing in locale | locale §2–§3; overlay/inbound pointer-only |
| GEO_SCOPE / territory STOP action in inbound; framing cites locale | flat inbound §5–§6 |
| Allowlist declared in locale; core weld Residual→SSR-13 | locale §1; `story-i18n-policy.md` §6.2 retained |
| Flat prerequisite | SSR-11 Done (pkg-000048) · pkg-000049 GIM-341…347 🟢 |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Sprawl inventory | Tables A–D still match disk at P3 start |
| **T02** | Principles lock | § Safe extraction principles in story; AC cites them |
| **T03** | Create locale MD | Flat path; §§0–7 outline filled for tallinn_civic |
| **T04** | Overlay/inbound pointers | Script/canon/jurisdiction framing cite locale; no triple full copy |
| **T05** | README | Sixth flat file listed |
| **T06** | Non-move checklist | Wire i18n + TAX-02 + bootstrap detect unchanged as process |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR12-01:** File at `GPT UI/instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md` (AC-GPT-REQ45a-05). Verified: path exists post-pkg-000049.
- [x] **AC-GPT-SSR12-02:** Three-identity duty split + architecture diagram reflected in pack docs (AC-GPT-REQ45a-02). Verified: locale §5–§7 + `schema-packs/README.md` artifact roles.
- [x] **AC-GPT-SSR12-03:** Sprawl inventory path-backed; no fantasy fields (analysis.mdc). Verified: inventory-confirm GIM-341.
- [x] **AC-GPT-SSR12-04:** Safe principles applied — wire `{et,ru,en}` and GPT-TAX-02 **not** moved into locale (AC-GPT-REQ45a-06). Verified: locale §5/§7; GIM-346 checklist.
- [x] **AC-GPT-SSR12-05:** Script policy + jurisdiction framing have **single pack home** (locale); overlay/inbound pointer-only for those bits (R-LOC-01/05 mitigated).
- [x] **AC-GPT-SSR12-06:** `working_languages` declared in locale for tallinn (`et`,`ru`,`en`); core allowlist weld documented as Residual→SSR-13 if not fully retargeted here (R-LOC-02).
- [x] **AC-GPT-SSR12-07:** Active-pair README lists locale flat name. Verified: `schema-packs.README.md` / `schema-packs/README.md` v2.1.
- [x] **AC-GPT-SSR12-08:** Depends — SSR-11 Done before P3. Verified: pkg-000048 Done; flat ×6 with locale additive.

## Точки в docs (P3)

| Action | Path |
|--------|------|
| CREATE | `GPT UI/instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md` |
| UPDATE | flat `…interview-overlay.md` — pointers §script/canon |
| UPDATE | flat `…inbound-validation.md` — cite locale for zone framing / copy language |
| UPDATE | `schema-packs/README.md` — sixth artifact + identity row |
| MAY UPDATE | `story-i18n-policy.md` §6.2 — “allowlist SSOT = active locale pack” (or defer SSR-13) |

## Dependencies

- **Hard depends:** GPT-SSR-11 Done.
- **Feeds:** GPT-SSR-13 (core allowlist/orchestrator pointers if deferred).
- **Consumer later:** REQ-46 Pack Builder must elicit locale §§.

## Residual

- Full orchestrator/i18n allowlist retarget if not completed in T04/T06 → **SSR-13**.
- Pack Builder checklist locale required → REQ-46 package.
- Non-tallinn packs (mobility/legal) — no GPT overlay yet (G-PAYLOAD-02); locale pattern applies when authored.
- Exact bilingual STOP phrasing — operator review at P3.
