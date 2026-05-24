# REQ-25: Активация `canonical_type`, `canonical_labels` и исправление правила `summary`

> **Назначение:** (1) снять статус "Deferred" с полей `narrative.canonical_type` и `narrative.canonical_labels` — GPT их уже генерирует, но не передаёт серверу, что блокирует pipeline публикации issue; (2) исправить правило omit для `narrative.summary` — текущая инструкция "omit key if empty" ведёт к HTTP 400.  
> **Источник gap-анализа:** `GPT UI/docs/analysis/gap-analysis-api-reference-vs-gpt-integration-2026-05-22.md` (GAP-3, GAP-4).  
> **Технический SoT (сервер):** `doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md §6.3`, `src/core/domain/narrative_i18n.py:parse_required_i18n_dict()`.

**Версия:** 1.0 · 2026-05-22  
**Статус:** requirements — ready for tasking  
**Приоритет:** P1  
**Тип:** активация функционала (canonical fields) + исправление бага (summary rule)  
**Серверная сторона:** не требует изменений — `contracts.py` и `parse_story_intake_request()` уже принимают оба поля  
**Парные REQ:** [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md), [REQ-24](./REQ-24-gpt-orchestrator-response-contract-corrections.md)

---

## 1. Расширение A: активация `canonical_type` и `canonical_labels`

### 1.1 Текущее состояние (broken)

`api-orchestrator.md §5.2.1` field mapping table (~строки 317–318):

```
| narrative.canonical_type   | canonical_payload.type     | Deferred | OP-01 |
| narrative.canonical_labels | canonical_payload.labels[] | Deferred | OP-02 |
```

`story-api-methods-reference.md §1.1`:
```
| narrative.canonical_type   | no | deferred by runtime policy |
| narrative.canonical_labels | no | deferred by runtime policy |
```

GPT-нормализатор (`story-normalizer.md §4.1`) **уже генерирует** эти поля:
- `canonical_payload.type` — одно из: `complaint`, `observation`, `system_bug`, `absurdity`
- `canonical_payload.labels[]` — taxonomy-approved строки из `story-label-taxonomy.md`

Поля не передаются на сервер.

### 1.2 Что сервер делает с этими полями (API_REFERENCE.md §6.3)

**`canonical_type`:**
- Отсутствие поля → **снижает `alpha_score` на 12 очков** (влияет на кластеризацию)
- Только `canonical_type ∈ {complaint, system_bug}` **проходят promotion gate** для создания публичного issue
- Значения `observation` и `absurdity` принимаются intake-ом, но не дают доступа к pipeline создания issues
- Enum не валидируется на стороне сервера при intake — любая строка принимается

**`canonical_labels`:**
- Нормализуются сервером: `.strip().lower()`, дедупликация с сохранением порядка
- Драйвят cluster key derivation для civic lenses
- Без labels — кластеризация строится только на text similarity

**Текущий эффект дефолта:** каждая история от GPT:
1. Никогда не становится публичным issue (нет `canonical_type ∈ {complaint, system_bug}`)
2. Теряет 12 очков alpha_score
3. Имеет слабые кластерные ключи

### 1.3 Целевое состояние

После REQ-25 GPT передаёт:
```json
{
  "narrative": {
    ...
    "canonical_type": "complaint",
    "canonical_labels": ["road_maintenance", "accessibility"]
  }
}
```

**Правила маппинга:**
- `canonical_type` → `canonical_payload.type` (когда normalizer вернул значение)
- `canonical_labels` → `canonical_payload.labels[]` (только canonical-метки из story-label-taxonomy.md; low-confidence и metadata-only — не включать)
- Оба поля опциональны — если normalizer не сгенерировал → не включать в запрос

**Важно:** сервер принимает любую строку в `canonical_type` без enum-валидации. GPT должен передавать только значения из taxonomy (`complaint`, `observation`, `system_bug`, `absurdity`).

---

## 2. Расширение B: исправление правила `summary` (GAP-4)

### 2.1 Текущее состояние (broken)

**`api-orchestrator.md §5.2.1` JSON body пример** (~строки 272–275):
```json
"summary": {
  "et": "<canonical_payload.summary.et — omit key if empty>",
  "ru": "<canonical_payload.summary.ru — omit key if empty>",
  "en": "<canonical_payload.summary.en — omit key if empty>"
}
```

**`story-api-methods-reference.md §1.1`** (~строка 35):
```
| narrative.summary | no | canonical_payload.summary when present; omit empty keys |
```

Инструкция "omit key if empty" означает: GPT может отправить частичный summary, например `{"et": "...", "ru": "..."}` (без `en`).

### 2.2 Что сервер делает (API_REFERENCE.md §6.3, narrative_i18n.py)

```
summary — If present: must have all three et, ru, en non-empty strings.
```

Сервер вызывает `parse_required_i18n_dict()` для summary-объекта если он не `null`. Эта функция (`narrative_i18n.py`) итерирует все три языка и **поднимает `ValueError`** (`→ HTTP 400`) если любой ключ отсутствует или пустой.

**Сценарий поломки:** GPT сгенерировал `summary.et` и `summary.ru`, но `summary.en` пустой. Следуя инструкции "omit key if empty", GPT отправляет:
```json
"summary": {"et": "Lühikokkuvõte.", "ru": "Краткое изложение."}
```
Сервер возвращает `HTTP 400: Missing or invalid narrative.summary.en`.

### 2.3 Целевое состояние (исправление)

Правило: **если хотя бы один языковой слот summary пустой → опустить весь блок `summary`**.

```json
// Отправлять только если все три слота непустые:
"summary": {
  "et": "Lühikokkuvõte.",
  "ru": "Краткое изложение.",
  "en": "Brief summary."
}
// Если хотя бы один пустой — summary блок не включать в запрос вообще
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | JSON body пример | Добавить `canonical_type` и `canonical_labels` в тело запроса с условием |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | Field mapping table (~строки 317–318) | Заменить "Deferred" на активные строки |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | JSON body пример (`summary`) | Изменить комментарий с "omit key if empty" на "omit entire block if any slot empty" |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | "Do NOT include" список | Убрать canonical_type и canonical_labels из запрещённых (если есть) |
| `GPT UI/instructions/story-api-methods-reference.md §1.1` | Field lock table | Заменить "deferred by runtime policy" для canonical_type и canonical_labels |
| `GPT UI/instructions/story-api-methods-reference.md §1.1` | summary строка | Изменить "omit empty keys" на "omit entire block if any slot empty" |

---

## 4. Конкретные изменения

### 4.1 api-orchestrator.md — field mapping table

**Было:**
```
| narrative.canonical_type   | canonical_payload.type     | Deferred | OP-01 |
| narrative.canonical_labels | canonical_payload.labels[] | Deferred | OP-02 |
```

**Стало:**
```
| narrative.canonical_type   | canonical_payload.type     | No | Значения: complaint, observation, system_bug, absurdity; только complaint/system_bug дают доступ к issue promotion |
| narrative.canonical_labels | canonical_payload.labels[] | No | Только canonical-метки из story-label-taxonomy.md; low-confidence и metadata-only — исключить |
```

### 4.2 api-orchestrator.md — JSON body (summary)

**Было:**
```json
"summary": {
  "et": "<canonical_payload.summary.et — omit key if empty>",
  ...
}
```

**Стало:**
```json
// Include summary only if ALL three slots are non-empty:
"summary": {
  "et": "<canonical_payload.summary.et>",
  "ru": "<canonical_payload.summary.ru>",
  "en": "<canonical_payload.summary.en>"
}
// If any slot is empty → omit the entire summary block
```

### 4.3 story-api-methods-reference.md §1.1

**Было:**
```
| narrative.canonical_type   | no | deferred by runtime policy |
| narrative.canonical_labels | no | deferred by runtime policy |
| narrative.summary          | no | canonical_payload.summary when present; omit empty keys |
```

**Стало:**
```
| narrative.canonical_type   | no | canonical_payload.type; complaint/system_bug → issue promotion gate |
| narrative.canonical_labels | no | canonical_payload.labels[]; только canonical disposition из story-label-taxonomy |
| narrative.summary          | no | canonical_payload.summary; omit entire block if any et/ru/en slot empty |
```

---

## 5. Acceptance Criteria

### Extension A (canonical fields)
- [ ] GPT отправляет `narrative.canonical_type` = значение из `canonical_payload.type` (когда normalizer его сгенерировал)
- [ ] GPT отправляет `narrative.canonical_labels` = массив canonical-меток (без metadata-only и low-confidence)
- [ ] `POST /intake/stories` с `canonical_type: "complaint"` → HTTP 202
- [ ] `POST /intake/stories` с `canonical_labels: ["road_maintenance"]` → HTTP 202
- [ ] Если normalizer не сгенерировал canonical_type → поле отсутствует в запросе (не null, не пустая строка)
- [ ] api-orchestrator.md содержит информацию что `complaint`/`system_bug` дают доступ к issue promotion gate

### Extension B (summary rule)
- [ ] GPT не отправляет summary с частично заполненными языковыми слотами
- [ ] Если `canonical_payload.summary.en` пустой → весь блок `summary` отсутствует в запросе
- [ ] `POST /intake/stories` без `summary` → HTTP 202 (summary опционален)
- [ ] `POST /intake/stories` с полным `summary` `{et, ru, en}` → HTTP 202

---

## 6. Не в scope этого REQ

- Изменения на стороне сервера (контракт и так поддерживает оба поля)
- Изменения в `story-normalizer.md` (normalizer уже генерирует эти поля; только orchestrator-маппинг нужен)
- `location_query` wire activation (REQ-26)
- Логика самой кластеризации и promotion gate (серверная задача)
- Добавление enum-валидации `canonical_type` на стороне сервера (не требуется по REQ-25)
