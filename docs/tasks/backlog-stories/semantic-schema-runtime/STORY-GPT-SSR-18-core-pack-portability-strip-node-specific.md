# STORY-GPT-SSR-18 — Core vs pack portability (strip node-specific from shared instructions)

## Meta
- **Key:** `STORY-GPT-SSR-18-core-pack-portability-strip-node-specific`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md) (Wave 5 — portability / AC-016 hygiene)
- **Status:** Todo
- **Приоритет:** P0 — mass Estonia node deploy (районы / деревни / города) без правок Module-1 core
- **Тип:** product / shared Instruction surface vs instance pack split
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) AC-016 · [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md)
- **Siblings:** [GPT-SSR-01](./STORY-GPT-SSR-01-core-vs-swappable-packs.md) · [GPT-SSR-13](./STORY-GPT-SSR-13-core-retarget-flat-pack-paths.md) · soft [GPT-SSR-17](./STORY-GPT-SSR-17-retire-activity-legacy-paths-inventory.md)
- **Trigger:** operator — `instruction-modules-index.md` L60 still hardcodes `tallinn_civic`/`v1`; need full core audit before multi-node rollout; Responses API migration later (consumer of clean split)
- **Skill declared (future P3):** `openai-custom-gpt-builder`
- **Depends:** SSR-11…13 Done (flat six-file + README active pair)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Delivery class (эта волна):** **Layer 1 + Layer 2 story only** — не править `instructions/**` здесь

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

После первого пилота нужно **массово** поднимать ноды по Эстонии (район / деревня / город): меняется только **schema pack** (README + flat six-file), а shared Module-1 instructions остаются общими.

Сейчас в **общем** индексе модулей всё ещё зашита пара **`tallinn_civic` / `v1`** и формулировка «swap two files» — при живом active pair **`uus_veerenni_civic` / `v3`**. Это ломает доверие к «swap only pack» и провоцирует копировать node field lists в core.

Нужна постановка: **вынести / обезвредить** node-specific из shared instructions, **проверить все связи**, и зафиксировать правило: **в core module index — никаких каталогов полей ноды**.

## Operator constraints (intent lock)

1. Fix / redesign row around [`instruction-modules-index.md`](../../../../instructions/instruction-modules-index.md) L60 (stale tallinn pair + outdated swap wording).
2. **MUST NOT** add node-specific field lists (`civic_domain`, `failure_pattern`, …) to the core module index.
3. Audit **all** shared Module-1 instruction files for node leaks so production multi-node deploy is not a bug hunt.

## Target state (after P3)

```text
Shared core: process only (phases, STOP, envelope mapping, HTTP orchestration)
Instance:    schema-packs.README.md active pair + schema-packs.<id>.<ver>.* ×6

Swap node N → N':
  1) replace README active pair + six flat files
  2) operator SPA_BASE / Actions base URL if host changes
  3) NO edits to story-* / ingest-* / index for domain fields
```

## Out of scope

- P3 body edits to instructions (this wave = story only).
- Responses API / instruction-fetch Actions (future material).
- Pack Builder `node-onboarding/` exemplars (tooling, not Interview GPT upload).
- Gateway Volume / enum expansion / compressing pack JSON.
- SSR-17 product file move (orthogonal upload hygiene).

## Target / Acceptance Criteria (постановка)

- [ ] **AC-GPT-SSR18-01:** Core `instruction-modules-index` describes active pack **only** via README pointer; no hardcoded `tallinn_civic`/`v1` (or any fixed pair).
- [ ] **AC-GPT-SSR18-02:** Index swap wording = flat **six-file** + README (REQ-45a); **no** node signal/field catalogs in index.
- [ ] **AC-GPT-SSR18-03:** Core API sample binding uses placeholders / README — not welded tallinn + civic signal names as normative.
- [ ] **AC-GPT-SSR18-04:** Grep gate on shared Module-1 files: no normative `tallinn_civic` active-pair claims; ILLUSTRATIVE examples labeled.
- [ ] **AC-GPT-SSR18-05:** Documented regression: new village/city pack deploy = pack swap only (+ operator URL config).

## Dependencies

- Hard: flat pack layout + `schema-packs.README.md` active pair (SSR-11…13).
- Soft: REQ-45 AC-016; D-SSR-11 / GPT-PB-08 free Contour2 axes (vs stale «13-axis» wording).

## Related

| Artifact | Role |
|----------|------|
| `instruction-modules-index.md` L60 | P0 leak |
| `schema-packs.README.md` | Live active pair SSOT (`uus_veerenni_civic`/`v3` as-built 2026-09) |
| `story-api-methods-reference.md` §1.2 | P1 sample weld |
| Future Responses API wave | Consumer of clean core/pack split — not implement here |

---

# Layer 2 — Техническая архитектура и субтаски (P3 later; **не выполнять здесь**)

## 0. Contours

```text
Shared Module-1 (upload once per product)
  root / base / story-* / ingest-* / orchestrator / index
        │
        │ MUST read pair + artifacts from
        ▼
schema-packs.README.md ──► schema-packs.<id>.<ver>.* ×6
        │
        └── instance-only: domain lists, locale copy, signal names, geo_scope
```

**Thesis:** Portability = **zero normative node identity in shared files**. Examples may keep Estonian street strings if marked ILLUSTRATIVE. Pilot `SPA_BASE` hostnames OK if marked operator-replace.

## 1. Verified leak inventory (2026-09-10 grep)

### P0 — must fix

| File | Loc | Leak | P3 fix |
|------|-----|------|--------|
| `instruction-modules-index.md` | L60 | `Active tallinn_civic/v1`; «swap **two** files» | Generic: active pair from [`schema-packs.README.md`](../../../../instructions/schema-packs.README.md); swap = six flat files + README; **no** field lists |
| same | L13, L29 | Link `schema-packs/README.md` (nested path) | Live SSOT = flat `schema-packs.README.md` (SSR-11/13) |

### P1 — normative / sample weld

| File | Loc | Leak | P3 fix |
|------|-----|------|--------|
| `story-api-methods-reference.md` | L55–66 | Sample binding `tallinn_civic`/`v1` + `civic_domain` / `failure_pattern` | Placeholders `<schema_id>` / `<schema_version>` / signals «from active pack.json field_policy» |
| same | L81 | «civic instance geo_intake.mode = optional» as universal | «per active pack `geo_intake.mode`» |
| `story-i18n-policy.md` | L82 | `(tallinn: et, ru, en)` in core SSOT line | Cite pack `locale-jurisdiction.md` via README only |
| `story-label-taxonomy.md` | L5 | Hard link `schema-packs.tallinn_civic.v1.taxonomy.json` | README → active taxonomy only |
| `api-orchestrator.md` | L667 | «Keep the **13-axis** GPT-TAX-01» | Axes ∈ active `taxonomy.json` (Contour2 free; D-SSR-11 / PB-08) |
| `story-normalizer.md` | ~L409 | «civic instance current value» next to `optional` | Mode from pack only; drop «civic instance» as if default |

### P2 — OK if labeled ILLUSTRATIVE / operator-replace

| File | Loc | Content | Rule |
|------|-----|---------|------|
| `api-orchestrator.md` | L270–276 | `SPA_BASE` …`tallinn-demo` | Keep; ensure **pilot default / replace after deploy** |
| `story-normalizer.md` / `ingest-validation.md` | samples | Tallinn / Pärnu mnt addresses | ILLUSTRATIVE geo strings — not active pair |
| Pack files `schema-packs.*` | — | uus / tallinn instance | **Correct** home for node specificity |
| `node-onboarding/*` meta examples | tallinn exemplar | Pack Builder tooling | Out of Interview GPT shared upload |

## 2. Link / path consistency map (P3 must preserve)

| From (shared) | Must resolve to |
|---------------|-----------------|
| Index / orchestrator / normalizer / i18n | `schema-packs.README.md` (flat) |
| README Locked paths | `schema-packs.<active_id>.<active_ver>.*` only |
| Relative `[taxonomy.json](schema-packs/README.md)` style | Prefer explicit flat README filename in prose |

Broken nested `schema-packs/README.md` links → Custom GPT / editors open wrong or missing path.

## 3. Regression checklist (mass deploy)

After P3, deploying node `foo_civic`/`v1` MUST require **only**:

1. Emit/copy flat six-file + update `schema-packs.README.md` active pair + Locked paths.
2. Operator Instructions: `SPA_BASE` (+ Actions server URL / Bearer) if environment differs.
3. **Zero** edits to `instruction-modules-index`, `story-api-methods-reference` sample pair, or core field catalogs.

Fail if operator must search-replace `tallinn_civic` inside shared Module-1 to go live.

## 4. Subtasks (Txx) — future P3 body

| ID | Subtask | Done-when |
|----|---------|-----------|
| T01 | Re-grep shared Module-1 for `tallinn_civic`, hardcoded pairs, civic signal catalogs in index | Inventory matches Layer 2 or updated |
| T02 | Rewrite index L60 + README path cites (L13/L29); bump index version/date | AC-01/02 |
| T03 | Placeholder `schema_binding` sample in `story-api-methods-reference.md` | AC-03 |
| T04 | `story-i18n-policy.md` + `story-label-taxonomy.md` README-only | No tallinn pin |
| T05 | Soften orchestrator 13-axis + normalizer «civic instance» wording | Pack-defined axes/mode |
| T06 | Label P2 examples ILLUSTRATIVE where needed | Grep gate AC-04 |
| T07 | Smoke narrative: uus↔tallinn or synthetic village swap without core domain edits | AC-05 |
| T08 | Out-of-scope guard | No Responses API; no pack content rewrite; no PB meta unless separate story |

## 5. Suggested index L60 replacement (copy at P3 — not applied now)

| Module | File | Primary output / constraint |
|--------|------|-----------------------------|
| Schema pack overlay | [`schema-packs.README.md`](schema-packs.README.md) | Active `schema_id` / `schema_version` + Locked flat six-file paths. Swap node = replace README pair + six artifacts; **do not** put node field/signal lists in this index (REQ-45 AC-016). |

## 6. Acceptance Criteria (refined)

- [ ] **AC-GPT-SSR18-01:** No hardcoded active pair in `instruction-modules-index` operational tables.
- [ ] **AC-GPT-SSR18-02:** Index documents six-file + README swap; forbids node field lists in index.
- [ ] **AC-GPT-SSR18-03:** API methods reference sample binding is placeholder / README-driven.
- [ ] **AC-GPT-SSR18-04:** Shared Module-1 `rg tallinn_civic` = empty **or** only ILLUSTRATIVE/changelog; no normative active-pair claims.
- [ ] **AC-GPT-SSR18-05:** Deploy checklist: new node = pack+README (+ ops URLs) only.
- [ ] **AC-GPT-SSR18-06:** Flat README path consistency in index (no nested `schema-packs/README.md` as live SSOT).

## 7. Anti-patterns

| Anti-pattern | Why fail |
|--------------|----------|
| Update L60 to `uus_veerenni_civic`/`v3` only | Still hardcodes next pilot; use README |
| Paste pack `field_policy` keys into index «for clarity» | Breaks AC-016 / multi-node |
| Leave sample `tallinn_civic` as silent default | Model copies wrong `schema_binding` |
| Re-lock Contour2 to 13 axes in orchestrator | Fights D-SSR-11 / uus free axes |

## Claims hygiene

| Claim | Evidence |
|-------|----------|
| Index L60 = tallinn_civic/v1 + two-file swap | `instruction-modules-index.md` L60 |
| Live active pair = uus_veerenni_civic/v3 | `schema-packs.README.md` Active pair table |
| Live README is flat `schema-packs.README.md` | file on disk; nested path retired SSR-11 |
| API sample welds tallinn + civic signals | `story-api-methods-reference.md` L55–66 |
| i18n / taxonomy stub name tallinn | `story-i18n-policy.md` L82; `story-label-taxonomy.md` L5 |
| Orchestrator 13-axis wording | `api-orchestrator.md` L667 |

No instruction body edit in this постановка+Layer2 wave.

---

*End of STORY-GPT-SSR-18*
