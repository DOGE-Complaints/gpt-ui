# STORY-GPT-SSR-10 — Core taxonomy SSOT → pack `taxonomy.json`; retire `story-label-taxonomy.md`

## Meta
- **Key:** `STORY-GPT-SSR-10-core-taxonomy-ssot-retire-md`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — закрытие G-CORE-TAX / residual G-TAX-01 после wave 2
- **Тип:** product / instruction migration (doc-only; SEC-001)
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) · [`REQ-20`](../../../requirements/REQ-20-label-taxonomy-and-extraction-axes.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-05 Done · GPT-SSR-06 Done · GPT-SSR-09 Done · GW-SSR-26/28/29 Done
- **Analysis:** [`audit-ingest-validation-vs-pack-taxonomy-drift-2026-09-01.md`](../../../../../docs/analysis/audit-ingest-validation-vs-pack-taxonomy-drift-2026-09-01.md) · gap [`cross-project-node-model-portability-gap-2026-09-01.md`](../../../../../docs/analysis/cross-project-node-model-portability-gap-2026-09-01.md) G-TAX-01 / G-CORE-TAX-*
- **Gateway:** no code change required (parse-only Contour2 already Done)

## Зачем простыми словами

Wave 2 положил словарь меток в pack **`taxonomy.json`** (copy-paste gateway↔GPT). Pack `inbound-validation.md` §2 уже читает JSON. Но **core** Instructions (`ingest-validation`, часть `story-normalizer` / `api-orchestrator` / `story-data-model`) всё ещё говорят: «валидируй labels по **`story-label-taxonomy.md`**».

Это два «источника правды» на одной strict chain. При swap ноды оператор обновляет JSON — а ранний gate всё ещё смотрит в старый Markdown.

Операторский вопрос: **удалить ли старый MD из системы инструкций?** Ответ аудита: **не удалять «в ноль» без миграции**. В MD есть **process rules**, которых **нет** в `taxonomy.json`. Нужна полная история: inventory → миграция process → переадресация live-ссылок → archive → stub → обновление index/manual.

## Архитектура (что куда после Done)

```text
BEFORE (as-built drift)
  ingest-validation ──► story-label-taxonomy.md     ← enum SSOT (stale)
  pack inbound §2   ──► taxonomy.json               ← enum SSOT (correct)
  normalizer §4.7   ──► taxonomy.json               ← axis_to_signal_map OK
  normalizer L180   ──► story-label-taxonomy.md     ← dual SSOT

AFTER (this story)
  Enum / keys / axes / axis_to_signal_map
    → active pack taxonomy.json (+ inbound §2 disposition guidance)

  Process rules (do not invent keys, English tokens, Phase-7 lock, no HTTP 422 AC, …)
    → thin core home (see T02) — NOT duplicated as key tables

  story-label-taxonomy.md
    → ARCHIVE (full snapshot) + STUB redirect at old path
    → removed from active instruction-modules-index as live SSOT
```

Паттерн: тот же, что SSR-09 для `data-model.md` → `archive/` + stub/redirect + core pointers на JSON.

## Verified inventory (факты на диске)

### A. Что уникально в MD (нельзя потерять при delete)

| MD section | Content | In `taxonomy.json`? | Target home after retire |
|------------|---------|---------------------|--------------------------|
| §1 Role | instruction-layer role | no | stub / process home |
| §2 Non-negotiable rules (9) | no ask labels; Phase-7 lock; English tokens; no HTTP 422 vocab reject; … | **no** | **must migrate** → process home |
| §3 Axis model table | axis + disposition role prose | axes[] only; role prose partial | process home or interview-overlay |
| §4 Canonical key tables | ~172 keys | **yes** (`canonical_keys`) | **JSON only** |
| §5 Metadata-only candidates | disposition table | keys present; disposition only in `meaning` prose | inbound §2 + process home |
| §6 Internal-only labels | disposition + wire rule | keys + `internal_axes[]` | inbound §2 + process home |
| §7 Candidate metadata shape + disposition enum | JSON shape + 5 dispositions | dispositions[] list only | **must migrate** → process home |
| §8 Version history | changelog | no | archive only |

### B. Live instruction refs (MUST retarget) — runtime Instructions

| File | Refs (approx) | Current role of MD |
|------|---------------|--------------------|
| [`ingest-validation.md`](../../../../instructions/ingest-validation.md) | L31, L57, L160, L224 | **enum SSOT** (primary drift) |
| [`story-normalizer.md`](../../../../instructions/story-normalizer.md) | L10, L32, L135, L178–180, L249, L263 | dual: glossary + SSOT claims |
| [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) | L181 | flat `labels[]` from MD |
| [`story-data-model.md`](../../../../instructions/story-data-model.md) | L10, L59, L96, L114 | labels SoT pointer |
| [`story-interview-flow.md`](../../../../instructions/story-interview-flow.md) | L10, L174, L187 | evidence→axis map / ecosystem §4.9 |
| [`instruction-modules-index.md`](../../../../instructions/instruction-modules-index.md) | L13, L22 | **active module** listing |
| [`inbound-validation.md`](../../../../instructions/schema-packs/tallinn_civic/v1/inbound-validation.md) | L32 | already JSON SSOT; MD = glossary note |
| [`interview-overlay.md`](../../../../instructions/schema-packs/tallinn_civic/v1/interview-overlay.md) | L79 | glossary pointer |

### C. Operator / runtime docs (SHOULD retarget)

| File | Note |
|------|------|
| [`schema-pack-custom-model-operator-manual-ru.md`](../../../runtime-docs/schema-pack-custom-model-operator-manual-ru.md) | L194 Labels row → MD |
| [`story-validation-and-garbage-filtering-as-built.md`](../../../runtime-docs/story-validation-and-garbage-filtering-as-built.md) | L120, L127 |
| [`story-pii-processing-before-send-as-built.md`](../../../runtime-docs/story-pii-processing-before-send-as-built.md) | L73 internal keys |
| Gateway [`schema-packs-node-data-model-ru.md`](../../../../../doge-complaints-gateway/docs/runtime-docs/manuals/schema-packs-node-data-model-ru.md) | L161 deprecation already; align after stub |

### D. Historical / backlog / REQ (MAY leave + «superseded» note)

REQ-20/25/33/36/38/45; STORY-GPT-SSR-01/04/05/06; STORY-GPT-TAX-02; archives; analysis docs — **не блокируют** Instructions upload. Prefer stub path still resolve (redirect), not 404.

### E. Tooling dependency (MUST not break silently)

| Path | Dependency |
|------|------------|
| [`ssr28_gpt_taxonomy_extract.py`](../../../../../doge-complaints-gateway/tests/helpers/ssr28_gpt_taxonomy_extract.py) | Hard path to `GPT UI/instructions/story-label-taxonomy.md` for seed extract | After archive: point to archive snapshot **or** mark helper historical-only |

### F. Scale (why full STORY, not «простой таск»)

| Bucket | Count (order of magnitude) |
|--------|----------------------------|
| Live instruction files to edit | **8** |
| Operator/runtime docs | **3–4** |
| Unique process sections to migrate | **§2 + §7 (+ §5/§6 disposition prose)** |
| Key tables to stop treating as SSOT | **§4** (~full file body) |
| Repo-wide string matches | **~45 files** (many historical) |
| Gateway product code | **0** required |

## Decision: delete vs archive (product)

| Option | Verdict |
|--------|---------|
| **Hard-delete MD with zero migration** | **Reject** — loses §2/§7 process rules |
| **Keep MD forever as dual SSOT** | **Reject** — G-CORE-TAX remains |
| **Archive + stub redirect + migrate process** | **Accept** (SSR-09 data-model pattern) |

**Stub at old path** (`instructions/story-label-taxonomy.md`) after archive:

```markdown
# ARCHIVED / REDIRECT — story-label-taxonomy

**Status:** Retired as enum SSOT (GPT-SSR-10).

**Enum / keys / axes / axis_to_signal_map:** active pack
[`taxonomy.json`](schema-packs/README.md) (tallinn: `schema-packs/tallinn_civic/v1/taxonomy.json`).

**Disposition / admission lists:** pack [`inbound-validation.md`](schema-packs/README.md) §2.

**Process rules (former §2 / §7):** → <process home path from T02>.

**Full snapshot:** [`archive/story-label-taxonomy.v0.2.3.md`](archive/story-label-taxonomy.v0.2.3.md)
```

Do **not** leave key tables live in stub (recreates dual SSOT).

## Process home (T02 — choose one at P3; indicators)

| Option | Home | Pros | Cons |
|--------|------|------|------|
| **A** | Fold §2+§7 into `ingest-validation.md` (+ short note in `story-normalizer.md`) | Fewer new files; process already lives there | ingest grows |
| **B** | New thin `story-label-process.md` (process only, no key tables) | Clean separation | one more core file in Custom GPT upload |
| **C** | Split: interview rules → `interview-overlay`; wire rules → normalizer | Pack-local where possible | process scattered |

**Recommendation indicator:** **A** for MVP (minimal new artifacts); B if ingest size becomes a problem.

## Scope / Out of scope

### In

- T00 inventory freeze (this story tables A–F as gate input).
- T01 migrate process (§2, §7, disposition guidance from §5/§6) to chosen home.
- T02 retarget **all live Instructions** refs (table B) → pack `taxonomy.json` + process home; remove «MD = SSOT» wording.
- T03 archive full MD → `instructions/archive/story-label-taxonomy.v0.2.3.md` (or `schema-packs/.../archive/` only if operator prefers pack-local — **default: instructions/archive** like shared glossary).
- T04 stub redirect at old path (table above).
- T05 update `instruction-modules-index.md`: remove MD as live SSOT module; list pack `taxonomy.json` + process home.
- T06 operator manual + as-built runtime docs (table C).
- T07 gateway helper path / historical note (`ssr28_gpt_taxonomy_extract.py`).
- T08 grep gate: zero live «validate against story-label-taxonomy» without redirect/glossary qualifier; `cmp` taxonomy.json still IDENTICAL.
- T09 INDEX progress + analysis G-CORE-TAX status closed.

### Out

- Gateway runtime enforce `canonical_keys` on intake (by design Residual).
- Add per-key `disposition` field to `taxonomy.json` schema (SSR-26 freeze; G-CORE-TAX-05).
- Rewrite historical REQ bodies end-to-end (optional superseded one-liners only).
- mobility/legal GPT overlays (separate).
- Automated CI sync script (optional residual).
- spa FE.

## Подзадачи

| ID | Задача | Не делать |
|----|--------|-----------|
| **T00** | Confirm inventory A–F still matches disk at P3 start | Invent new axes/keys |
| **T01** | Migrate §2 + §7 (+ disposition prose) to process home (A or B) | Copy §4 key tables into process home |
| **T02** | Retarget live Instructions (table B) | Leave ingest L57 on MD SSOT |
| **T03** | Archive full MD snapshot | Hard-delete without archive |
| **T04** | Stub redirect at old path | Empty 404 / silent delete |
| **T05** | `instruction-modules-index.md` + README if needed | Keep MD as «Controlled source of truth» |
| **T06** | Operator manual + as-built docs | Full REQ rewrite |
| **T07** | Helper / historical extract path | Break SSR-28 helper silently |
| **T08** | Grep + `cmp` gate | Assume without check |
| **T09** | INDEX + analysis closeout | Mark Done without T08 |

## Target / AC

- [x] **AC-GPT-SSR10-01:** Process rules from MD §2 and §7 live in process home; no key-table duplication.
- [x] **AC-GPT-SSR10-02:** `ingest-validation.md` enum checks point to active pack `taxonomy.json` (+ inbound §2); GIM-88 wording preserved.
- [x] **AC-GPT-SSR10-03:** `story-normalizer.md` / `api-orchestrator.md` / `story-data-model.md` / `story-interview-flow.md` have no «MD = enum SSOT» claims.
- [x] **AC-GPT-SSR10-04:** Full MD archived; stub redirect at old path; index no longer lists MD as live vocabulary SSOT.
- [x] **AC-GPT-SSR10-05:** Pack inbound §2 remains JSON SSOT; interview-overlay glossary points to stub or process home (not key tables).
- [x] **AC-GPT-SSR10-06:** `cmp` gateway↔GPT `taxonomy.json` IDENTICAL after edits.
- [x] **AC-GPT-SSR10-07:** Grep live Instructions: zero unqualified «validate labels against story-label-taxonomy.md».
- [x] **AC-GPT-SSR10-08:** Tooling path for SSR-28 extract updated or explicitly marked historical-only.

## Dependencies

- Wave 2 Done (SSR-05/06/09) + gateway taxonomy pack Done (26/28/29).
- Unblocks clean operator swap: edit JSON only for keys; process stays core.

## Residual

- Per-key machine disposition in JSON (G-CORE-TAX-05) — future gateway schema story if needed.
- Automated core↔pack taxonomy drift CI — optional.
- Historical REQ one-liners — optional superseded notes.
- Custom GPT **upload checklist**: remove old MD from active bundle if operator uploaded full tree; ensure stub + archive + taxonomy.json present.
