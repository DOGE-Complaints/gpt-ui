# REQ-46: Pack Builder — primary prompt, wrapper, repair debug & validation schemas

Parent: [docs/requirements backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md](../../../docs/requirements%20backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md) §5.1 Pack Builder, F1 (obtain builder prompt), F2 (validate + repair prompt loop)  
Parent-id: req4-node-onboarding-schema-pack-federation-gate  
Related (runtime SSOT): [REQ-45](./REQ-45-semantic-schema-intake-envelope-compatibility.md) — interview Custom GPT core vs pack + wire; **not** Pack Builder Instructions  
Related (runtime pack layout / identities): [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md) — three pack prose identities + flat `schema-packs.*` naming; Pack Builder **emits** that contract  
Related (app gate): [dogestonia-app REQ-APP-04](../../../dogestonia-app/docs/requirements/REQ-APP-04-node-onboarding-wave1.md) F2 validate / repair  
Related (backlog package): [node-onboarding/](../tasks/backlog-stories/node-onboarding/INDEX.md) — GPT-PB-01…04 (prompts → schemas → wrapper; stories only; `instructions/node-onboarding/` in P3)  
Status: Draft / Proposed (requirements only — artifact files not created in this step)  

Версия: 0.6 · 2026-09-03 (SSOT layout/identities → REQ-45a; REQ-46 keeps Pack Builder tooling only; supersedes 0.5)  
Проект: GPT UI  
Фокус: удалённый Pack Builder (wrapper + primary + repair) и meta-schemas валидации candidate schema pack — **без** связи с Module-1 interview Instructions

---

## 1) Goal

Дать оператору ноды (REQ4) способ **создать candidate schema pack** через диалог с Pack Builder:

1. Тонкий **wrapper** подтягивает и пинит удалённый **primary prompt** + **validation standards**.
2. Primary ведёт NL-диалог и эмитит файлы пака + `prompt_version_hash`.
3. Candidate валидируется против **тех же** protocol schemas/standards.
4. При fail оператор вставляет **repair / debug prompt** (ошибки + инструкции) обратно в builder dialogue и итерирует.

Pack Builder **не** заменяет DOGEstonia app validation / federation gate (REQ4 F2–F5). Он производит candidates.

**Runtime pack contract** (какие identities, какие flat имена, что остаётся в core) — **[REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md)**. Этот REQ задаёт **authoring tooling**, которое тот contract эмитит и проверяет.

---

## 2) Scope / Out of scope

### In scope

- Primary Pack Builder prompt (remote git / tagged raw URL).
- Synthetic wrapper (fetch/pin primary + bind standards).
- Repair / debug prompt template for validation failures.
- Validation schemas / checklists used to validate generated pack files (GPT UI-hosted copies for builder + transparency; align with gateway pack layout / REQ3 / **REQ-45a**).
- **Layout for Pack Builder only (§4):** prompts + meta-schemas under `instructions/node-onboarding/`.
- Pack output set: emit target = **REQ-45a** flat instance set (JSON + three GPT pack prose identities) — generation products, not Module-1 core edits.

### Out of scope

- Defining runtime pack identities / flat naming SSOT → [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md) (was briefly drafted here as §4.1 in 0.5; **moved**).
- Module-1 Story Interview **process** files (`story-interview-flow`, orchestrator, Actions OpenAPI) — process stays core; only **instance** pack data is authored by Pack Builder.
- Runtime migration of existing `tallinn_civic/v1/` nested tree (target paths named in REQ-45a; create/migrate = later explicit wave).
- Stripe, Arweave, EVM, app validate engine implementation, gateway Schema Runtime code.
- Full prose of primary/wrapper/repair, JSON Schema bodies, and instruction files — **out of this requirements-only step** (paths named below are future deliverables; analysis.mdc name ≠ create).

---

## 3) Verified current state (analysis.mdc)

| Fact | Evidence |
|------|----------|
| Product Pack Builder intent | REQ4 §5.1, F1–F2 |
| Pack Builder prompts / meta-schemas on disk | **Not present** — named in §5 as required future paths only |
| Existing instance pack tree | Nested `GPT UI/instructions/schema-packs/<schema_id>/<ver>/` (REQ-45 / Wave 2 as-built); flat `schema-packs.*` = **REQ-45a target**, not present yet |
| Gateway executable packs | `doge-complaints-gateway/schema-packs/<id>/<ver>/` (`pack.json`, `payload.schema.json`, `taxonomy.json`) |
| GPT pack prose as-built | inbound + interview-overlay present nested; **locale-jurisdiction** absent — see REQ-45a §3 |
| Interview Instructions ≠ Pack Builder | REQ-45 / `instruction-modules-index.md` |
| Runtime layout / identity SSOT | [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md) |

---

## 4) Path rules — Pack Builder tree only

Различать **два** класса файлов. Не смешивать.

### 4.1 Instance schema-pack model — SSOT in REQ-45a

Instance pack flat naming, three prose identities, and root Instruction retarget are defined in **[REQ-45a §4–§6](./REQ-45a-pack-prose-identities-and-flat-layout.md)**.

This REQ **MUST NOT** redefine that layout. Pack Builder **MUST** emit filenames and duties matching REQ-45a.

Gateway nested `doge-complaints-gateway/schema-packs/<id>/<ver>/` **MAY** remain nested. GPT Instructions copy — flat `schema-packs.*` per REQ-45a.

### 4.2 Pack Builder prompts + validation meta-schemas — subdirectory `node-onboarding/`

| Rule | Requirement |
|------|-------------|
| **Directory** | Pack Builder **prompts** and **validation schemas/checklists** (standards для валидации candidate pack) **MUST** live under `GPT UI/instructions/node-onboarding/`. |
| **Why** | Отделить авторский tooling (onboarding / Pack Builder) от flat instance pack upload set; в git это читаемее. |
| **Not instance model** | Ничего из REQ-45a instance set не класть в `node-onboarding/`. |

Examples (target): `…/node-onboarding/pack-builder-primary-prompt.md`, `…/node-onboarding/pack-builder-pack.schema.json`.

---

## 5) Required artifacts

### 5.0 Pack Builder set (`instructions/node-onboarding/`)

| Path under `GPT UI/instructions/node-onboarding/` | Duty |
|--------------------------------------------------|------|
| `pack-builder-README.md` | Builder vs interview Instructions; raw/tag URL; `prompt_version_hash`; index; standards set hash |
| `pack-builder-wrapper.md` | Tiny synthetic wrapper: fetch/pin primary; bind validation schemas/checklist |
| `pack-builder-primary-prompt.md` | Full Pack Builder — **задача подробно в §5.1** |
| `pack-builder-repair-debug-prompt.md` | Validation-fail repair template |
| `pack-builder-pack.schema.json` | Meta-schema for candidate `pack.json` |
| `pack-builder-payload-schema.schema.json` | Meta-schema for candidate `payload.schema.json` |
| `pack-builder-taxonomy.schema.json` | Meta-schema for candidate `taxonomy.json` (when required) |
| `pack-builder-overlays.checklist.md` | Presence/shape rules for **three** pack prose identities ([REQ-45a §4](./REQ-45a-pack-prose-identities-and-flat-layout.md)) |

### 5.0b Instance pack set (flat `schema-packs.*`) — emit target = REQ-45a

Named by primary emit. **Normative file list and duties:** [REQ-45a §6](./REQ-45a-pack-prose-identities-and-flat-layout.md). **Not present** until create/migrate wave.

Summary (consumer view only):

| Flat filename pattern | Duty (REQ-45a) |
|-----------------------|----------------|
| `schema-packs.<id>.<ver>.pack.json` | Model identity, field_policy, geo_intake, clustering/card |
| `schema-packs.<id>.<ver>.payload.schema.json` | Payload / signals / geo JSON Schema |
| `schema-packs.<id>.<ver>.taxonomy.json` | Taxonomy when required |
| `schema-packs.<id>.<ver>.inbound-validation.md` | Content / admission |
| `schema-packs.<id>.<ver>.interview-overlay.md` | Interview guidance |
| `schema-packs.<id>.<ver>.locale-jurisdiction.md` | Language / jurisdiction |

### 5.1 Primary prompt — постановка задачи (функциональная)

Файл-артефакт `node-onboarding/pack-builder-primary-prompt.md` **пока не создаётся** (analysis.mdc: name ≠ create). Ниже — обязательная постановка того, что primary **должен уметь**, когда его напишут.

#### 5.1.1 Роль и аудитория

- **Роль:** единственный полный Pack Builder instruction set (удалённый git / tagged raw URL), который ведёт диалог и **эмитит candidate schema pack**.
- **Аудитория:** оператор ноды (город / NGO / community / private deployer), часто **не разработчик** — NL о нуждах ноды, не «напиши JSON Schema с нуля».
- **Граница:** primary **не** Module-1 interview GPT. Он **авторит** flat `schema-packs.*` per **REQ-45a** для runtime Custom GPT / gateway, но interview **process** не переписывает.

#### 5.1.2 Входы сессии

Primary (через wrapper) обязан исходить из:

1. Закреплённых **validation standards** в `node-onboarding/` — генерировать только то, что проходит эти проверки.
2. Описания оператором: домен; signals/поля; geo; taxonomy / card / clustering; **и все три GPT pack-идентичности** ([REQ-45a §4](./REQ-45a-pack-prose-identities-and-flat-layout.md)).
3. Явного `schema_id` / `schema_version` — **не** путать с envelope `m2.story_intake_envelope.v2`.

#### 5.1.3 Диалог (что primary обязан выяснить до эмита)

Пока не собрано достаточно — **не** выдавать финальный pack.

Колонка **Gateway (as-built)** — executable pack на `doge-complaints-gateway` (`schema-packs/<id>/<ver>/`; SCHEMA-005).

Колонка **GPT Custom GPT** — interview Custom GPT (Module-1 process + active pack). Ownership of identities = **REQ-45a** (not redefined here).

**Paragraph — pack ownership of GPT identities (emit contract):**  
As-built и target duties for content/admission, interview guidance, and language/jurisdiction are normative in **[REQ-45a §4](./REQ-45a-pack-prose-identities-and-flat-layout.md)**. Primary Pack Builder **MUST** elicit and emit all three prose files plus the JSON set with **flat** `schema-packs.*` names. Нельзя считать candidate «готовым», если guidance или locale оставлены «в голове core» или только в комментариях к JSON.

Идентичности процесса (краткая consumer-таблица; полный SSOT = REQ-45a):

| Идентичность | Где живёт (target) | Что делает |
|--------------|-------------------|------------|
| **content / admission** | pack `…inbound-validation.md` + core process gates | Domain BLOCK/clarify/ACCEPT; completeness; label checklists; GEO_SCOPE / territory STOP copy |
| **interview guidance** | pack `…interview-overlay.md` + core phase **process** without instance lists | Geo formation; phase→axes; ecosystem; precision tables; pack interview cues |
| **язык / юрисдикция** | pack `…locale-jurisdiction.md` + core i18n **slot** process | Languages/scripts; jurisdiction frames; STOP/422 copy language(s); geo label script policy — **MUST NOT** invent wire names |

JSON на GPT-стороне — **read-only reference** (SEC-001 / REQ-45). Orchestrator **MUST** emit `schema_binding` с активной пары; пути — flat `schema-packs.*` ([REQ-45a §5](./REQ-45a-pack-prose-identities-and-flat-layout.md)).

| Тема | Зачем в паке | Gateway (as-built) — связанный функционал | GPT Custom GPT — связанный функционал (target) |
|------|----------------|-------------------------------------------|--------------------------------------------------|
| Идентичность модели | `schema_id`, `schema_version` | Пара каталога pack + `NODE_SCHEMA_*`; intake `schema_binding` **MUST** совпасть, иначе 4xx. Не путать с envelope `m2.story_intake_envelope.v2`. | **Active pair** → flat `schema-packs.<id>.<ver>.*` (REQ-45a). Orchestrator **читает** → **MUST** эмитить `schema_binding`. Swap = смена active flat set + upload; **не** правка domain-имён в core (AC-016). |
| Поля / signals | `payload.schema.json` + `field_policy` в `pack.json` | **Validate:** `SchemaRuntime.validate`. **Policy map:** `field_policy`. Exact lenses. | inbound completeness; normalizer handoff; orchestrator STOP; SEC-001. |
| Geo mode | `geo_intake` + pack-блоки | Node zone / instance territory / precision as-built. | interview-overlay formation/precision; inbound GEO_SCOPE/territory STOP; **locale-jurisdiction** script/jurisdiction framing. |
| Taxonomy | `taxonomy.json` when required | Contour1 wire vs Contour2 pack taxonomy. | inbound enum checklists; interview-overlay phase→axes. |
| Clustering / card | `exact_lenses`, `node_clustering`, `card_fields` | Pack exact + civic + `schema_card`. | GPT не запускает ClusteringEngine. |
| Content / admission | `…inbound-validation.md` | Gateway **не** исполняет prose. | Pack Contour B; Primary **MUST** elicit domain lists. |
| Interview guidance | `…interview-overlay.md` | Gateway **не** исполняет. | Pack-owned per REQ-45a; Primary **MUST** elicit place-naming, phase/axis, ecosystem, mirror cues. |
| Язык / юрисдикция | `…locale-jurisdiction.md` | Gateway **не** исполняет. | Pack-owned per REQ-45a; Primary **MUST** elicit languages + jurisdiction frames. |

Reference shape: gateway `tallinn_civic/v1` JSON + GPT prose → flat `schema-packs.tallinn_civic.v1.*` including `locale-jurisdiction.md` (REQ-45a). Primary **SHALL** уметь порождать **другие** node models.

**Geo discovery cues for primary:** (a) зона ноды (`geo_scope`); (b) GPT-instance territory; (c) `geo_intake.mode`; (d) `mirror_to_payload`; (e) depth levels / `detail_level` / `geo_model`; (f) **locale/script** rules (§ locale pack file / REQ-45a).

#### 5.1.4 Обязательный выход (candidate pack)

Primary **MUST** эмитить полный набор с flat именами per **[REQ-45a §5–§6](./REQ-45a-pack-prose-identities-and-flat-layout.md)**:

| Файл (flat name) | Продуктовая обязанность |
|------------------|-------------------------|
| `schema-packs.<id>.<ver>.pack.json` | Пара; field_policy; geo_intake; clustering/card |
| `schema-packs.<id>.<ver>.payload.schema.json` | Payload / signals / geo schema |
| `schema-packs.<id>.<ver>.taxonomy.json` | Когда требуется taxonomy |
| `schema-packs.<id>.<ver>.inbound-validation.md` | Content / admission |
| `schema-packs.<id>.<ver>.interview-overlay.md` | Interview guidance (**обязателен** для GPT-facing pack) |
| `schema-packs.<id>.<ver>.locale-jurisdiction.md` | Language / jurisdiction (**обязателен** для GPT-facing pack) |

Метаданные: **`prompt_version_hash`** (REQ4 §5.1); рекомендуется `standards_set_hash` из `node-onboarding` standards.

#### 5.1.5 Инварианты и запреты primary

| MUST | MUST NOT |
|------|----------|
| Соответствовать `node-onboarding` meta-schemas / checklist | Переписывать Module-1 process files / OpenAPI |
| Emit flat `schema-packs.*` names (REQ-45a) | Класть instance pack в субдиректории или в `node-onboarding/` |
| Три pack prose identities (REQ-45a §4) | Оставлять guidance/locale только в core |
| SEC-001: pack = data | Validate вместо gateway / app gate |
| Semantic `schema_version` ≠ envelope id | Invent wire names вне gateway contracts |
| Candidate готов к REQ4 F2 validate | Stripe / Arweave / EVM / federation |
| Уточнять при нехватке данных | Сливать Pack Builder с interview Instructions |

#### 5.1.6 Связь с repair loop

Primary принимает заполненный repair prompt из `node-onboarding/` после fail и переиздаёт тот же flat file set (§5.1.4 / REQ-45a).

#### 5.1.7 Критерий готовности текста primary

NL нужды → §5.1.3 (включая три GPT identities / REQ-45a) → полный flat set §5.1.4 с `prompt_version_hash` → validate против `node-onboarding` meta-schemas — **без** правок Module-1 core.

### Protocol alignment

- Semantic shape **SHOULD** align with gateway pack layout / REQ3; GPT Instructions path layout follows **REQ-45a**.
- Drift GPT meta-schemas vs gateway enforcement: **gateway wins** for federation admission.

---

## 6) Target behavior

1. Operator obtains wrapper + pinned primary + standards from `node-onboarding/`.
2. Primary emits flat `schema-packs.<id>.<ver>.*` including three prose identities (**REQ-45a**).
3. Validator checks against `node-onboarding` meta-schemas + overlays checklist (covers REQ-45a identities).
4. Fail → repair prompt → regenerate → re-validate.
5. Success → admission-ready for REQ4 F3–F5 (outside this REQ).
6. Root Instructions link only to flat `schema-packs.*` (**REQ-45a** retarget).
7. Pack Builder never merged into Module-1 interview Instruction core.

---

## 7) Acceptance criteria

1. **AC-GPT-REQ46-01:** Parent link to REQ4 §5.1 / F1–F2; child = Pack Builder authoring only.
2. **AC-GPT-REQ46-02:** Wrapper, primary, repair-debug under `node-onboarding/`; duties distinct.
3. **AC-GPT-REQ46-03:** Validation meta-schemas + checklist in `node-onboarding/`; cover three pack prose identities (REQ-45a).
4. **AC-GPT-REQ46-04:** Instance pack emit uses flat prefix `schema-packs.*` per **REQ-45a**; **no** instance subdirectory under Instructions.
5. **AC-GPT-REQ46-05:** Pack Builder prompts/schemas **in** `instructions/node-onboarding/` (not mixed with instance `schema-packs.*`).
6. **AC-GPT-REQ46-06:** Root Instruction retarget to flat names is **REQ-45a** AC; Pack Builder emit must not contradict it.
7. **AC-GPT-REQ46-07:** Orthogonality to Module-1 process; civic reference instance only.
8. **AC-GPT-REQ46-08:** `prompt_version_hash` + wrapper standards hash.
9. **AC-GPT-REQ46-09:** Primary / checklist treat interview guidance + language/jurisdiction as **pack-owned** per **REQ-45a** (not content/admission-only); this REQ does not redefine identity SSOT.
10. **AC-GPT-REQ46-10:** Primary emit mandates all three prose files + JSON set with flat names (REQ-45a §6).
11. **AC-GPT-REQ46-11:** Primary task statement complete without requiring prompt files on disk yet (name ≠ create).
12. **AC-GPT-REQ46-12:** Related link to REQ-45a as runtime layout/identity SSOT; Pack Builder = consumer only.

---

## 8) Cross-links

- Parent: [REQ4](../../../docs/requirements%20backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md)
- Runtime core vs pack: [REQ-45](./REQ-45-semantic-schema-intake-envelope-compatibility.md)
- Runtime pack identities + flat layout (SSOT): [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md)
- Pack layout reference (as-built nested; target flat per REQ-45a): [schema-pack-custom-model-operator-manual-ru.md](../runtime-docs/schema-pack-custom-model-operator-manual-ru.md)
- App onboarding tasks: [REQ-APP-04](../../../dogestonia-app/docs/requirements/REQ-APP-04-node-onboarding-wave1.md)
- Index: [README](./README.md)
