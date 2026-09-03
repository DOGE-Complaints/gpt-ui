# Мануал: как настроить Custom GPT под кастомную модель и правила валидации

> **Для кого:** оператор / продукт, которому нужно «переключить» GPT на другую ноду или другую модель полей — без переписывания всего интервью.  
> **Метод:** [`.cursor/rules/analysis.mdc`](../../../.cursor/rules/analysis.mdc) — факты с диска.  
> **Сверка:** 2026-09-03 (GPT-SSR-13 flat six-file + locale).  
> **Не** юридический документ. **Не** marketplace / §30.

As-built по «мусору» и допускам: [`story-validation-and-garbage-filtering-as-built.md`](./story-validation-and-garbage-filtering-as-built.md).  
PII pre-send: [`story-pii-processing-before-send-as-built.md`](./story-pii-processing-before-send-as-built.md).

---

## 1. Зачем это нужно (простыми словами)

Один Custom GPT ведёт разговор с жителем (фазы интервью, privacy, отправка черновика).  
**Модель данных ноды** (какие поля в `signals.*`, как устроен geo, какие domain-листы для «civic / spam») **не должна быть вшита** в ядро инструкций.

Иначе при смене города/ноды пришлось бы править orchestrator и interview-flow под каждое имя поля.

Решение (REQ-45 / GPT-SSR-01…03):

- **Ядро** — процесс (как спрашивать, когда STOP, как слать HTTP).
- **Шесть сменных файлов пака (flat)** — JSON trio (*модель*: pack / payload / taxonomy) + prose trio (*inbound* / *interview-overlay* / *locale-jurisdiction*) конкретной ноды.
- **OpenAPI Actions** — транспортный контракт (`schema_binding` обязателен, `geo_detail` опционален).
- **Gateway** — настоящий validate (пак в GPT **не** исполняет схему).

---

## 2. Что где лежит

| Слой | Что это | Где на диске |
|------|---------|--------------|
| **Ядро инструкций** | Интервью, safety, policy *process*, normalizer process, orchestrator HTTP | [`GPT UI/instructions/`](../../instructions/) (`story-interview-flow.md`, `story-policy-gate.md`, `story-normalizer.md`, `api-orchestrator.md`, …) |
| **Active pair** | Какая нода сейчас активна (`schema_id` + `schema_version`) | [`GPT UI/instructions/schema-packs/README.md`](../../instructions/schema-packs/README.md) **v2.1** |
| **Pack JSON (модель)** | `field_policy`, `geo_intake`, clustering, payload shape, taxonomy | Flat: [`schema-packs.tallinn_civic.v1.pack.json`](../../instructions/schema-packs.tallinn_civic.v1.pack.json) + [`payload.schema.json`](../../instructions/schema-packs.tallinn_civic.v1.payload.schema.json) + [`taxonomy.json`](../../instructions/schema-packs.tallinn_civic.v1.taxonomy.json) — COPY gateway |
| **Interview overlay** | Phase→axes, ecosystem cues (prose); script/canon → locale | [`schema-packs.tallinn_civic.v1.interview-overlay.md`](../../instructions/schema-packs.tallinn_civic.v1.interview-overlay.md) |
| **Правила валидации (inbound)** | Domain lists / completeness vs pack fields | [`schema-packs.tallinn_civic.v1.inbound-validation.md`](../../instructions/schema-packs.tallinn_civic.v1.inbound-validation.md) |
| **Locale / jurisdiction** | Working languages, scripts, jurisdiction framing, STOP copy language | [`schema-packs.tallinn_civic.v1.locale-jurisdiction.md`](../../instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md) |
| **Archive (не SSOT)** | Nested wave-2 snapshot (historical) | [`archive/schema-packs-nested-tallinn_civic-v1/`](../../instructions/archive/schema-packs-nested-tallinn_civic-v1/) |
| **OpenAPI Actions** | Wire: обязательный `schema_binding`, опциональный `geo_detail` | [`GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml`](../custom-gpt-story-intake-actions.openapi.yaml) **info.version 0.8.0** |
| **Справка по Actions** | Lockstep с OpenAPI | [`story-api-methods-reference.md`](../../instructions/story-api-methods-reference.md) |
| **Gateway SSOT** | Исполняемый pack + payload schema | `doge-complaints-gateway/schema-packs/<id>/<ver>/` |

**Текущий active pair (as-built):** `tallinn_civic` / `v1`.

**Важно:** `schema_version` в паке (`v1`) — это **семантическая** версия модели ноды.  
На проводе envelope остаётся `schema_version: "m2.story_intake_envelope.v2"`. Это **разные** поля. Не путать.

**Wave 2:** active `data-model.md` **нет** — форма полей = JSON; prose = overlay + inbound.

---

## 3. Pack bundle: что писать куда (flat six-file — GPT-SSR-11…13)

Layout для любой ноды (Custom GPT **upload** = flat prefix names):

```text
GPT UI/instructions/
  schema-packs.<schema_id>.<schema_version>.pack.json              ← COPY gateway
  schema-packs.<schema_id>.<schema_version>.payload.schema.json    ← COPY gateway
  schema-packs.<schema_id>.<schema_version>.taxonomy.json          ← COPY gateway
  schema-packs.<schema_id>.<schema_version>.inbound-validation.md  ← content / admission
  schema-packs.<schema_id>.<schema_version>.interview-overlay.md   ← geo canon, phase→axes
  schema-packs.<schema_id>.<schema_version>.locale-jurisdiction.md ← languages / jurisdiction
```

Плюс правка active pair в [`schema-packs/README.md`](../../instructions/schema-packs/README.md).

> **Historical nested GPT tree** `schema-packs/<schema_id>/<schema_version>/` — **не** live upload layout; snapshot only under [`archive/schema-packs-nested-tallinn_civic-v1/`](../../instructions/archive/schema-packs-nested-tallinn_civic-v1/). Gateway executable SSOT stays nested under `doge-complaints-gateway/schema-packs/<id>/<ver>/`.

### 3.1 JSON pack files — модель (read-only)

Сюда **byte-identical copy** gateway `pack.json` + `payload.schema.json` + `taxonomy.json`:

- Binding pair (`schema_id`, `schema_version`) — в `pack.json`.
- Required `signals.*` / optional `geo.*` — `field_policy` + `payload.schema.json`.
- `geo_intake.mode` и матрица emit.
- `axis_to_signal_map` — **`taxonomy.json`** (не MD §2.3).

Файлы **не** валидатор и **не** OpenAPI. Gateway validate = SEC-001. Core orchestrator/normalizer **читают JSON**, не archived `data-model.md`.

### 3.2 `inbound-validation.md` — правила валидации контента ноды

Сюда — **domain lists** и completeness относительно pack fields:

- Какие классы BLOCK / clarify / ACCEPT для *этой* ноды (списки).
- Label / multi-axis чеклисты с отсылкой к taxonomy.
- Pack-required fields before stash (дополнительно к core §4.1).
- GEO_SCOPE copy текста для жителя при 422.

### 3.3 `interview-overlay.md` — interview guidance

Geo formation canon, phase→axes tables, ecosystem-deficit cues, demo stash examples. **Не** дублирует JSON shape — см. §3.1.

**Process** (когда BLOCK vs approve) остаётся в ядре:

- [`story-policy-gate.md`](../../instructions/story-policy-gate.md)
- [`ingest-validation.md`](../../instructions/ingest-validation.md)

Пак даёт **списки и поля ноды**, ядро — **алгоритм решения**.

---

## 4. Active pair и `schema_binding`

Перед stash orchestrator **читает** README (active pair), открывает flat six-file set — **`pack.json`** + **`payload.schema.json`** + **`taxonomy.json`** + inbound / overlay / **`locale-jurisdiction.md`** (+ normalizer handoff), и **MUST** положить в HTTP:

```json
"schema_binding": {
  "schema_id": "<из README>",
  "schema_version": "<из README>",
  "structured_payload": { "signals": { … }, "geo": { … } }
}
```

Правила (as-built, [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.1 / item 20):

- `schema_binding` **никогда не опускать**.
- `schema_id` / `schema_version` = пара из README (не envelope id).
- Pack-required `signals.*` (для civic: `civic_domain`, `failure_pattern`) — непустые строки из normalizer handoff; иначе **STOP** до HTTP.
- Значения **не выдумывать**.

Опционально на корне: `geo_detail` (gateway GeoDetail) — по режиму `geo_intake.mode`.

Citizen Mode может **не показывать** binding в разговоре; в Actions body он всё равно **должен** быть.

---

## 5. Пошаговый swap на другую модель (чеклист оператора)

Цель: указать GPT на другой gateway pack (`NODE_SCHEMA_ID` / `NODE_SCHEMA_VERSION`).

### Шаг 1 — copy-paste gateway JSON → flat GPT names (GPT-SSR-11…13)

**Default policy:** byte-identical copy from gateway SSOT; GPT upload uses **flat** filenames.

1. Открыть gateway nested SSOT `doge-complaints-gateway/schema-packs/<new_id>/<new_ver>/` (`pack.json`, `payload.schema.json`, **`taxonomy.json`**).
2. Скопировать **три JSON** в flat GPT paths:
   - `GPT UI/instructions/schema-packs.<new_id>.<new_ver>.pack.json`
   - `GPT UI/instructions/schema-packs.<new_id>.<new_ver>.payload.schema.json`
   - `GPT UI/instructions/schema-packs.<new_id>.<new_ver>.taxonomy.json`
3. **Optional diff-check** перед upload: `cmp` все три файла с gateway (должны совпадать байт-в-байт).
4. Сохранить или обновить flat prose trio: `inbound-validation.md` + **`interview-overlay.md`** + **`locale-jurisdiction.md`** (`schema-packs.<new_id>.<new_ver>.*`).
5. **Не** перепроецировать JSON→MD вручную. **Не** re-import Actions, если менялись только JSON/MD.
6. **Не** использовать archived `data-model.md` / nested GPT tree как SSOT — только flat six-file + README.

> **Gateway prerequisite:** on-disk `taxonomy.json` SSOT — [GW-SSR-26/28/29](../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/INDEX.md) Done on gateway side.

> **Historical (wave 1 — до GPT-SSR-04):** ручная проекция gateway JSON → `data-model.md` + `inbound-validation.md` («спроецировать»). Deprecated для новых swap; оставлено только как fallback до SSR-09 archive.

### Шаг 2 — объявить active pair

В [`schema-packs/README.md`](../../instructions/schema-packs/README.md) обновить таблицу Active pair:

| Key | Value |
|-----|--------|
| `schema_id` | `<new_id>` |
| `schema_version` | `<new_ver>` |

И ссылки на **flat six-file** pack artifacts (JSON trio + inbound + overlay + **locale-jurisdiction**); README swap table v5 / SSR-13.

### Шаг 3 — загрузить Instructions в Custom GPT

Загрузить **ядро** + **README** + **flat six-file** pack artifacts (`pack.json`, `payload.schema.json`, `taxonomy.json`, `inbound-validation.md`, `interview-overlay.md`, **`locale-jurisdiction.md`**).

Если менялись **только** pack files и README → **Actions re-import не нужен**.

### Шаг 4 — когда нужен Actions re-import

| Изменение | Действие |
|-----------|----------|
| Только pack / README | Upload Instructions |
| Менялся OpenAPI (`schema_binding` / `geo_detail` / paths / required) | Publish YAML + **re-import Actions** в ChatGPT Configure |

Текущий контракт: OpenAPI **0.8.0** уже содержит required `schema_binding`. Менять его «под ноду» обычно **не** нужно: меняется **содержимое** `structured_payload`, не имена sidecar на wire.

### Шаг 5 — smoke после swap

1. Пройти короткий диалог до stash (God Mode удобнее: видно полный JSON).
2. В теле запроса есть `schema_binding.schema_id` / `schema_version` = новая пара.
3. Pack-required keys в `structured_payload.signals` заполнены (или GPT остановился с STOP — это тоже ожидаемо, если taxonomy не дала canonical).
4. Geo: поведение соответствует `geo_intake.mode` нового пака.
5. HTTP 2xx от node (или понятный 4xx от gateway schema — тогда править проекцию пака, не invent поля).

### Шаг 6 — AC-016 (swap без правок domain в core)

После swap **не** должны понадобиться правки orchestrator / interview-flow **ради имён полей ноды**. Если понадобились — pack неполный или поля утекли в core. Чеклист — §9 ниже.

---

## 6. Правила валидации: кто что режет

| Вопрос | Кто отвечает | Где |
|--------|--------------|-----|
| Достаточно ли эпизода (Q1–Q3)? | GPT interview | `story-interview-flow.md` |
| Spam / off-topic / gossip? | GPT policy **process** + domain lists пака | `story-policy-gate.md` + `inbound-validation.md` |
| Labels canonical? | GPT ingest + pack taxonomy.json | `ingest-validation.md` + pack `taxonomy.json` (stub: `story-label-taxonomy.md`) |
| Pack-required signals перед stash? | GPT normalizer + orchestrator STOP | `pack.json` field_policy + orchestrator item 20 |
| Schema / required `schema_binding` / payload shape? | **Gateway** | `contracts.py` Schema Runtime |
| Content «мусор» на сервере? | **Нет** (as-built) | см. validation as-built |

Итог: **содержательная** фильтрация — в GPT; **схемная** — на gateway. Прямой JSON-клиент с валидным envelope обходит instruction filters.

---

## 7. Режимы geo (просто)

Читаются из **data-model** пака (`geo_intake.mode`):

| Mode | Что должен сделать GPT | Сейчас у `tallinn_civic/v1` |
|------|------------------------|----------------------------|
| `optional` | Можно без `geo_detail` и без `location_query` | **Да, текущий** |
| `require_location_or_detail` | Нужен непустой `location_query` **или** `geo_detail` | — |
| `require_detail` | Нужен непустой `geo_detail` | — |

`merge` / `mirror_to_payload` в pack.json — **только gateway**; в OpenAPI их **нет**.

Текст для жителя при `GEO_SCOPE_MISMATCH` — из inbound-validation / data-model pack, не из hardcoded city list в core.

---

## 8. Частые ошибки

1. **Забыли `schema_binding`** → GPT должен STOP (item 20); gateway всё равно отвергнет (`Missing or invalid root.schema_binding`).
2. **Вшили civic поля в core** → ломается AC-016; следующий swap = боль.
3. **Invent enum / `"example"` из §7 pack как значение** → gateway/логика расходятся; источник = taxonomy canonical (data-model §2.3).
4. **Сменили OpenAPI, не сделали Actions re-import** → ChatGPT шлёт старый контракт.
5. **Перепутали envelope `m2.story_intake_envelope.v2` и pack `v1`** → кладут envelope string в `schema_binding.schema_version`.
6. **Думают, что pack «валидирует» как сервер** → pack = инструкция; validate = gateway.
7. **Ждут, что сервер отрежет spam** → as-built нет; только GPT policy gate.

---

## 9. Чеклист AC-016 (после swap)

Из [`schema-packs/README.md`](../../instructions/schema-packs/README.md):

| # | Проверка | Pass когда |
|---|----------|------------|
| 1 | Фазы интервью / Phase 7 | Не менялись в `story-interview-flow.md`; списки полей — из `pack.json` / `interview-overlay.md` |
| 2 | Envelope id | Всё ещё `m2.story_intake_envelope.v2` |
| 3 | Active pair | Берётся из README, не hardcoded civic в core |
| 4 | Geo gate | Параметризован `geo_intake.mode` пака (`pack.json`) |
| 5 | PII / admission process | Без правок process в normalizer / orchestrator |
| 6 | Domain field names | Живут в pack JSON + inbound / overlay (не core) |

Fail: ради имён полей ноды пришлось править orchestrator / interview-flow.

---

## 10. Быстрый указатель файлов (копипаста)

```text
# Active pair + swap
GPT UI/instructions/schema-packs/README.md
GPT UI/instructions/schema-packs.README.md

# Flat six-file pack bundle (tallinn_civic/v1) — Custom GPT upload
GPT UI/instructions/schema-packs.tallinn_civic.v1.pack.json
GPT UI/instructions/schema-packs.tallinn_civic.v1.payload.schema.json
GPT UI/instructions/schema-packs.tallinn_civic.v1.taxonomy.json
GPT UI/instructions/schema-packs.tallinn_civic.v1.inbound-validation.md
GPT UI/instructions/schema-packs.tallinn_civic.v1.interview-overlay.md
GPT UI/instructions/schema-packs.tallinn_civic.v1.locale-jurisdiction.md

# Emit / STOP
GPT UI/instructions/api-orchestrator.md          # §5.2.1, items 13 / 19 / 20
GPT UI/instructions/story-normalizer.md          # §4.6 geo, §4.7 structured_payload_handoff

# Wire contract
GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml   # 0.8.0
GPT UI/instructions/story-api-methods-reference.md

# Gateway projection source (executable — not GPT upload layout)
doge-complaints-gateway/schema-packs/tallinn_civic/v1/
```

---

## 11. Сверка

| Поле | Значение |
|------|----------|
| Дата | 2026-09-03 |
| Pack README | v2.1 (flat six-file + locale) |
| pack JSON / payload / taxonomy | flat `schema-packs.tallinn_civic.v1.*` (gateway copy) |
| interview-overlay / inbound / locale | flat MD trio (SSR-11/12/13) |
| nested GPT upload paths | retired — archive only |
| OpenAPI | 0.8.0 |
| Orchestrator (emit) | 0.5.9+ + SSR-13 locale in active artifact list |
