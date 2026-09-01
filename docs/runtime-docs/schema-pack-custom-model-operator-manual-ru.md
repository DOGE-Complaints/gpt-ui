# Мануал: как настроить Custom GPT под кастомную модель и правила валидации

> **Для кого:** оператор / продукт, которому нужно «переключить» GPT на другую ноду или другую модель полей — без переписывания всего интервью.  
> **Метод:** [`.cursor/rules/analysis.mdc`](../../../.cursor/rules/analysis.mdc) — факты с диска.  
> **Сверка:** 2026-09-01.  
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
- **Два сменных файла пака** — *модель* и *правила валидации* конкретной ноды.
- **OpenAPI Actions** — транспортный контракт (`schema_binding` обязателен, `geo_detail` опционален).
- **Gateway** — настоящий validate (пак в GPT **не** исполняет схему).

---

## 2. Что где лежит

| Слой | Что это | Где на диске |
|------|---------|--------------|
| **Ядро инструкций** | Интервью, safety, policy *process*, normalizer process, orchestrator HTTP | [`GPT UI/instructions/`](../../instructions/) (`story-interview-flow.md`, `story-policy-gate.md`, `story-normalizer.md`, `api-orchestrator.md`, …) |
| **Active pair** | Какая нода сейчас активна (`schema_id` + `schema_version`) | [`GPT UI/instructions/schema-packs/README.md`](../../instructions/schema-packs/README.md) **v1.0** |
| **Модель (data-model)** | Форма `structured_payload`, `geo_intake`, geo-канон, примеры | [`…/tallinn_civic/v1/data-model.md`](../../instructions/schema-packs/tallinn_civic/v1/data-model.md) **v1.3** |
| **Правила валидации (inbound)** | Domain lists / completeness vs pack fields | [`…/tallinn_civic/v1/inbound-validation.md`](../../instructions/schema-packs/tallinn_civic/v1/inbound-validation.md) **v1.2** |
| **OpenAPI Actions** | Wire: обязательный `schema_binding`, опциональный `geo_detail` | [`GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml`](../custom-gpt-story-intake-actions.openapi.yaml) **info.version 0.8.0** |
| **Справка по Actions** | Lockstep с OpenAPI | [`story-api-methods-reference.md`](../../instructions/story-api-methods-reference.md) |
| **Gateway SSOT** | Исполняемый pack + payload schema | `doge-complaints-gateway/schema-packs/<id>/<ver>/` |

**Текущий active pair (as-built):** `tallinn_civic` / `v1`.

**Важно:** `schema_version` в паке (`v1`) — это **семантическая** версия модели ноды.  
На проводе envelope остаётся `schema_version: "m2.story_intake_envelope.v2"`. Это **разные** поля. Не путать.

---

## 3. Pack bundle: что писать куда (wave 2 — GPT-SSR-09)

Layout для любой ноды:

```text
GPT UI/instructions/schema-packs/<schema_id>/<schema_version>/
  pack.json                 ← COPY gateway (field_policy, geo_intake, clustering)
  payload.schema.json       ← COPY gateway (signals/geo shape)
  taxonomy.json             ← COPY gateway (axes + axis_to_signal_map)
  inbound-validation.md     ← content / admission rules (dual contour)
  interview-overlay.md      ← geo canon, phase→axes, ecosystem cues
  archive/data-model.v*.md  ← archived MD projection (не SSOT)
```

Плюс правка active pair в [`schema-packs/README.md`](../../instructions/schema-packs/README.md).

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

Перед stash orchestrator **читает** README, открывает **`pack.json`** + **`payload.schema.json`** (+ normalizer handoff / overlay), и **MUST** положить в HTTP:

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

### Шаг 1 — copy-paste gateway JSON (v3 — GPT-SSR-09)

**Default policy:** byte-identical copy from gateway SSOT.

1. Открыть `doge-complaints-gateway/schema-packs/<new_id>/<new_ver>/` (`pack.json`, `payload.schema.json`, **`taxonomy.json`**).
2. Скопировать **три JSON** в:
   - `GPT UI/instructions/schema-packs/<new_id>/<new_ver>/pack.json`
   - `GPT UI/instructions/schema-packs/<new_id>/<new_ver>/payload.schema.json`
   - `GPT UI/instructions/schema-packs/<new_id>/<new_ver>/taxonomy.json`
3. **Optional diff-check** перед upload: `cmp` все три файла с gateway (должны совпадать байт-в-байт).
4. Сохранить или обновить `inbound-validation.md` + **`interview-overlay.md`** (content + interview guidance).
5. **Не** перепроецировать JSON→MD вручную. **Не** re-import Actions, если менялись только JSON/MD.
6. **Не** использовать archived `data-model.md` как SSOT — только JSON + overlay.

> **Gateway prerequisite:** on-disk `taxonomy.json` SSOT — [GW-SSR-26/28/29](../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/INDEX.md) Done on gateway side.

> **Historical (wave 1 — до GPT-SSR-04):** ручная проекция gateway JSON → `data-model.md` + `inbound-validation.md` («спроецировать»). Deprecated для новых swap; оставлено только как fallback до SSR-09 archive.

### Шаг 2 — объявить active pair

В [`schema-packs/README.md`](../../instructions/schema-packs/README.md) обновить таблицу Active pair:

| Key | Value |
|-----|--------|
| `schema_id` | `<new_id>` |
| `schema_version` | `<new_ver>` |

И ссылки на pack artifacts (wave 2 target: JSON pair + inbound; README rename → SSR-09).

### Шаг 3 — загрузить Instructions в Custom GPT

Загрузить **ядро** + **README** + **pack artifacts** (`pack.json`, `payload.schema.json`, `taxonomy.json`, `inbound-validation.md`, `interview-overlay.md`).

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
| Labels canonical? | GPT ingest + taxonomy | `ingest-validation.md`, `story-label-taxonomy.md` |
| Pack-required signals перед stash? | GPT normalizer + orchestrator STOP | data-model §2 + orchestrator item 20 |
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
| 1 | Фазы интервью / Phase 7 | Не менялись в `story-interview-flow.md`; списки полей — из data-model |
| 2 | Envelope id | Всё ещё `m2.story_intake_envelope.v2` |
| 3 | Active pair | Берётся из README, не hardcoded civic в core |
| 4 | Geo gate | Параметризован `geo_intake.mode` пака |
| 5 | PII / admission process | Без правок process в normalizer / orchestrator |
| 6 | Domain field names | Живут в двух pack-файлах |

Fail: ради имён полей ноды пришлось править orchestrator / interview-flow.

---

## 10. Быстрый указатель файлов (копипаста)

```text
# Active pair + swap
GPT UI/instructions/schema-packs/README.md

# Текущая модель + валидация (экстрагированы)
GPT UI/instructions/schema-packs/tallinn_civic/v1/data-model.md
GPT UI/instructions/schema-packs/tallinn_civic/v1/inbound-validation.md

# Emit / STOP
GPT UI/instructions/api-orchestrator.md          # §5.2.1, items 13 / 19 / 20
GPT UI/instructions/story-normalizer.md          # §4.6 geo, §4.7 structured_payload_handoff

# Wire contract
GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml   # 0.8.0
GPT UI/instructions/story-api-methods-reference.md

# Gateway projection source
doge-complaints-gateway/schema-packs/tallinn_civic/v1/
```

---

## 11. Сверка

| Поле | Значение |
|------|----------|
| Дата | 2026-09-01 |
| Pack README | v1.0 |
| data-model | v1.3 |
| inbound-validation | v1.2 |
| OpenAPI | 0.8.0 |
| Orchestrator (emit) | 0.5.9 (GIM-273…275) |
