# REQ-45a: Pack prose identities + flat `schema-packs.*` layout

Parent: [REQ-45-semantic-schema-intake-envelope-compatibility.md](./REQ-45-semantic-schema-intake-envelope-compatibility.md) (Accepted — Wave 1 core vs pack + wire `schema_binding` / `geo_detail`)  
Parent-id: req3-semantic-schema-runtime  
Related (product onboarding): [REQ4](../../../docs/requirements%20backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md) §5.1 Pack Builder  
Related (authoring tooling): [REQ-46](./REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) — Pack Builder **emits** the layout/identities defined here; does **not** own runtime SSOT  
Status: Draft / Proposed (requirements only — create/migrate of flat files = later explicit wave)  

Версия: 0.1 · 2026-09-03  
Проект: GPT UI  
Фокус: продолжение REQ-45 — **три** pack-owned GPT prose identities + **flat** Instructions filenames (`schema-packs.*`); без Pack Builder prompt bodies

---

## 1) Goal

REQ-45 закрыл: universal Module-1 **process** core + swappable **node** pack; emit gateway-named `schema_binding` / optional `geo_detail`; SEC-001 (gateway validates).

REQ-45a закрывает следующий слой той же логики:

1. Кроме JSON-модели (`pack.json`, `payload.schema.json`, `taxonomy.json`) и **content/admission** (`inbound-validation.md`) pack **MUST** владеть ещё двумя prose-идентичностями:
   - **interview guidance** → `interview-overlay.md`
   - **language / jurisdiction** → `locale-jurisdiction.md`
2. На стороне Custom GPT Instructions instance pack **MUST** жить **плоско** с префиксом `schema-packs.` (без субдиректорий) — иначе upload Custom GPT отбрасывает nested folders.
3. Module-1 core остаётся **process-only** (фазы, STOP HTTP, envelope mapping); instance lists / copy / locale frames — **только** в pack (REQ3-AC-016).

Это **runtime pack contract** для interview Custom GPT. Pack Builder (REQ-46) **потребляет** этот contract как emit target; не определяет его заново.

---

## 2) Scope / Out of scope

### In scope

- Target set of GPT-facing pack artifacts (JSON + three prose identities).
- Flat naming rules under `GPT UI/instructions/` (`schema-packs.<id>.<ver>.<artifact>`).
- Root Instruction retarget: links to instance pack **MUST** use flat names (no nested path segments).
- Duty split: content/admission vs interview guidance vs language/jurisdiction vs core process vs gateway executable JSON.
- Alignment with Wave 2 SSR as-built (nested tree) → target flat migrate.

### Out of scope

- Pack Builder wrapper / primary / repair prompts and meta-schemas → [REQ-46](./REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md).
- Creating or rewriting instruction file bodies in this REQ step (analysis.mdc: name ≠ create).
- Changing gateway on-disk nested layout `doge-complaints-gateway/schema-packs/<id>/<ver>/` (executable SSOT MAY stay nested).
- Inventing new intake wire field names beyond sibling 51 / REQ-45.
- Stripe / Arweave / EVM / app federation gate implementation.

---

## 3) Verified current state (analysis.mdc)

| Fact | Evidence |
|------|----------|
| REQ-45 Accepted; Wave 1 SSR-01…03 Done | [REQ-45](./REQ-45-semantic-schema-intake-envelope-compatibility.md) Status; backlog INDEX |
| Wave 2 nested pack on disk | `GPT UI/instructions/schema-packs/tallinn_civic/v1/` — `pack.json`, `payload.schema.json`, `taxonomy.json`, `inbound-validation.md`, `interview-overlay.md` |
| Active-pair index still nested | [`schema-packs/README.md`](../../instructions/schema-packs/README.md) L21–31 |
| Flat `schema-packs.*` filenames | **Not present** under `GPT UI/instructions/` |
| `locale-jurisdiction.md` (any layout) | **Not present** |
| Interview guidance partially pack-owned | [`interview-overlay.md`](../../instructions/schema-packs/tallinn_civic/v1/interview-overlay.md) v1.1 (geo canon, phase→axes, ecosystem, precision) |
| Locale / i18n process still core | [`story-i18n-policy.md`](../../instructions/story-i18n-policy.md) — wire `{ et, ru, en }` slots; no pack-owned jurisdiction artifact |
| Gateway executable packs nested | `doge-complaints-gateway/schema-packs/<id>/<ver>/` |
| Custom GPT upload constraint | Nested folders under Instructions are dropped on upload (operator constraint driving flat prefix) |

---

## 4) Three GPT pack prose identities (continuation of REQ-45)

REQ-45 named **two** pack artifacts (data-model projection + inbound-validation). Wave 2 replaced MD data-model with **literal JSON** + kept inbound + added interview-overlay. REQ-45a **locks** the full prose set:

| Identity | Pack artifact (logical name) | Duty | Core keeps |
|----------|------------------------------|------|------------|
| **Content / admission** | `inbound-validation.md` | Domain BLOCK/clarify/ACCEPT lists; completeness vs `field_policy`; label enum checklists; GEO_SCOPE / instance-territory **STOP** copy | Contour A process (`story-policy-gate`, `ingest-validation`, orchestrator gates) |
| **Interview guidance** | `interview-overlay.md` | Geo formation canon; phase→axes; ecosystem cues; precision / `detail_level` inference tables; pack-specific interview lists; `mirror_to_payload` cues | Phase 1–7 / affirmation **process** only (`story-interview-flow`, normalizer process) — **no** instance lists |
| **Language / jurisdiction** | `locale-jurisdiction.md` | Working languages/scripts for resident & operator; jurisdiction / demo-zone framing; STOP/422/standing copy language(s); Latin/Cyrillic (etc.) policy for geo labels | Wire i18n **slot-filling process** (`story-i18n-policy`, card `{ et, ru, en }`) — **which** languages/jurisdiction text = pack |

**MUST NOT** conflate:

- Admission BLOCK lists ↔ interview guidance tables.
- Jurisdiction copy ↔ inventing new OpenAPI field names.
- Pack JSON validate ↔ GPT (SEC-001 unchanged from REQ-45).

**Pack incomplete** if guidance or locale remain only in core prose or JSON comments.

---

## 5) Flat layout + prefix `schema-packs.` (moved from REQ-46 §4.1)

Файлы **инстанс-модели** schema pack (JSON + pack-owned GPT prose для **активной** ноды) — это **не** Pack Builder prompts и **не** Module-1 interview process.

| Rule | Requirement |
|------|-------------|
| **No subdirectory** | Must **not** live under `instructions/schema-packs/<schema_id>/<ver>/` for Custom GPT upload set (nested folders dropped). |
| **Prefix** | All instance files **MUST** use filename prefix `schema-packs.` and sit **flat** under `GPT UI/instructions/`. |
| **Naming pattern** | `schema-packs.<schema_id>.<schema_version>.<artifact>` where `<artifact>` ∈ {`pack.json`, `payload.schema.json`, `taxonomy.json`, `inbound-validation.md`, `interview-overlay.md`, `locale-jurisdiction.md`}. |
| **Root link retarget** | Все ссылки из **root** Module-1 Instructions (`story-interview-flow`, `api-orchestrator`, `story-normalizer`, `ingest-validation`, `instruction-modules-index`, operator manuals, active-pair index, etc.) на instance pack **MUST** указывать на flat `schema-packs.*` filenames — **без** path-сегментов субдиректории. Migration of on-disk links = follow-up implement wave; this REQ **requires** that outcome. |

Examples (target, **not** created by this REQ edit):

- `GPT UI/instructions/schema-packs.tallinn_civic.v1.pack.json`
- `GPT UI/instructions/schema-packs.tallinn_civic.v1.interview-overlay.md`
- `GPT UI/instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md`

Gateway on-disk layout `doge-complaints-gateway/schema-packs/<id>/<ver>/` **MAY** remain nested (executable SSOT). GPT Instructions copy for Custom GPT — **flat `schema-packs.*`**.

Operator swap (continuation of REQ-45 §4 item 5): replace the **flat** active set (+ align emitted `schema_binding` pair); core process unchanged.

---

## 6) Target instance pack set

| Flat filename pattern | Duty | As-built (2026-09-03) |
|-----------------------|------|------------------------|
| `schema-packs.<id>.<ver>.pack.json` | Model identity, field_policy, geo_intake, clustering/card | Nested copy present |
| `schema-packs.<id>.<ver>.payload.schema.json` | Payload / signals / geo JSON Schema | Nested copy present |
| `schema-packs.<id>.<ver>.taxonomy.json` | Taxonomy when required | Nested copy present |
| `schema-packs.<id>.<ver>.inbound-validation.md` | Content / admission | Nested present |
| `schema-packs.<id>.<ver>.interview-overlay.md` | Interview guidance | Nested present |
| `schema-packs.<id>.<ver>.locale-jurisdiction.md` | Language / jurisdiction | **Absent** — must be authored in implement wave |

---

## 7) Target behavior

1. Active node pair points at flat `schema-packs.<id>.<ver>.*` set (six artifacts when taxonomy + locale required).
2. Core modules read pack via flat names only; no hardcoded civic domain lists in core (AC-016).
3. Orchestrator emits `schema_binding` for the active pair; JSON in Instructions remains read-only (SEC-001).
4. Interview guidance and locale are first-class pack files — not Residual core weld.
5. Gateway nested pack remains executable SSOT for validate; GPT flat set is non-executable projection / copy-paste surface.
6. Pack Builder (REQ-46) **MUST** emit this same flat set; validation of candidates against builder meta-schemas stays in REQ-46.

---

## 8) Acceptance criteria

1. **AC-GPT-REQ45a-01:** Parent = REQ-45; this REQ continues pack-overlay logic (not a second interview product).
2. **AC-GPT-REQ45a-02:** Three pack prose identities documented: inbound-validation, interview-overlay, locale-jurisdiction — with duty split vs core process.
3. **AC-GPT-REQ45a-03:** Flat naming `schema-packs.<id>.<ver>.<artifact>` required for GPT Instructions upload set; nested instance tree **not** the target.
4. **AC-GPT-REQ45a-04:** Root Instruction links to instance packs **MUST** use flat names (required outcome; migrate wave later).
5. **AC-GPT-REQ45a-05:** `locale-jurisdiction` is a **required** GPT-facing pack artifact (alongside interview-overlay); content/admission-only pack is incomplete.
6. **AC-GPT-REQ45a-06:** No invented wire field names; SEC-001 / narrative-first / AC-016 from REQ-45 preserved.
7. **AC-GPT-REQ45a-07:** REQ-46 related as **consumer** of this layout (emit/validate authoring); SSOT of runtime identities/layout = this REQ.
8. **AC-GPT-REQ45a-08:** Verified state distinguishes nested as-built vs flat target; named paths **not** created in this requirements-only step.

---

## 9) Open questions

1. Whether git retains a nested mirror for operator readability alongside flat upload set — product choice at migrate wave (default target = flat only under `instructions/`).
2. Exact section outline of first `locale-jurisdiction.md` for `tallinn_civic/v1` — deferred to implement/story wave (seed from interview-overlay Latin/Cyrillic + i18n-policy language list + inbound STOP copy languages).
3. Whether gateway pack.json gains a pointer key to GPT prose filenames — optional; not required for GPT flat layout.

---

## 10) Dependencies / cross-links

- Parent: [REQ-45](./REQ-45-semantic-schema-intake-envelope-compatibility.md)
- Sibling gateway: [51-semantic-schema-runtime](../../../doge-complaints-gateway/docs/requirements/51-semantic-schema-runtime.md)
- Consumer (Pack Builder): [REQ-46](./REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md)
- Product parent Pack Builder: [REQ4](../../../docs/requirements%20backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md)
- Operator manual (as-built nested): [schema-pack-custom-model-operator-manual-ru.md](../runtime-docs/schema-pack-custom-model-operator-manual-ru.md)
- Index: [README](./README.md)
