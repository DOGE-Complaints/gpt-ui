# REQ-26: `location_query` — добавить в pipeline normalizer → wire

> **Назначение:** добавить `location_query` в выходной артефакт story-normalizer и в wire-маппинг api-orchestrator, чтобы данные о локации, собранные в ходе интервью (Phase 3–4), передавались на сервер и резолвились в geo-координаты.  
> **Источник gap-анализа:** `GPT UI/docs/analysis/gap-analysis-api-reference-vs-gpt-integration-2026-05-22.md` (GAP-6).  
> **Технический SoT (сервер):** `doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md §6.3`, `src/core/application/services.py:117-137`, `src/core/geo/`.

**Версия:** 1.0 · 2026-05-22  
**Статус:** requirements — ready for tasking  
**Приоритет:** P2  
**Тип:** функциональное расширение — добавление нового поля в normalizer output и wire  
**Серверная сторона:** не требует изменений — `contracts.py` уже принимает `narrative.location_query`; geo-резолвер уже реализован  
**Парные REQ:** [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md), [REQ-25](./REQ-25-canonical-type-labels-and-summary-wire-activation.md)

---

## 1. Текущее состояние (gap)

### 1.1 Где теряется location_query

Фазы 2–3 интервью собирают данные о локации (адрес, район, название места). `ingest-validation.md` фиксирует их в validation report (`"location": { "freeform": "Pärnu mnt, Tallinn" }`).

**story-normalizer.md §4** — выходной артефакт `canonical_payload`:
```json
{
  "canonical_payload": {
    "type": "complaint",
    "labels": [...],
    "title": {...},
    "description": {...},
    "summary": {...},
    "institution": {...}
  }
}
```
`location_query` отсутствует в `canonical_payload` и во всей структуре `normalized_issue_payload`. Поле не выходит из нормализатора — grep по `story-normalizer.md` для `"location_query"` возвращает **ноль результатов**.

**api-orchestrator.md §5.2.1** field mapping table:
`story-api-methods-reference.md §1.1` строка `narrative.location_query | no | optional, when available` — источник маппинга не определён.

### 1.2 Что происходит без location_query

Сервер (`services.py:117-137`):
```python
geo = (
    self.geo_service.resolve_for_story(
        request.narrative.location_query,
        debug_logger=debug_logger,
    )
    if self.geo_service is not None
    else None
)
```

Если `location_query` отсутствует или пустой — geo-резолвер не вызывается. `StoryRecord.geo = None`.

**Последствия:**
- Истории не имеют координат (`lat`, `lon`, `admin_district`, etc.)
- `GET /tallinn/issues` bounding-box и district-фильтры (15 параметров) не работают для этих историй — issues с пустым geo не попадают в гео-запросы SPA
- Кластеризация по географическому lens строится без реальных координат
- `alpha_score` не получает geo-бонус

---

## 2. Целевое состояние

### 2.1 story-normalizer.md — добавить location_query в output

`normalized_issue_payload` должен содержать опциональное поле `location_query`:

```json
{
  "normalized_issue_payload": {
    "canonical_payload": {
      "type": "complaint",
      "title": {...},
      "description": {...}
    },
    "normalization_metadata": {
      "session_language": "et",
      "detected_input_language": "et",
      ...
    },
    "location_query": "Tartu mnt 80, Tallinn"
  }
}
```

**Правила формирования `location_query`:**
1. Включать только если пользователь явно назвал или подтвердил локацию в ходе интервью (Phase 2–3)
2. Формат: freeform-строка, предпочтительно `<улица/место>, <город>` или `<район>, Tallinn`
3. Если пользователь упомянул несколько локаций без уточнения — использовать ту, что ближе всего к сути жалобы
4. Если локация не была подтверждена или неоднозначна — `location_query` не включать
5. Пустую строку не передавать — сервер хранит пустой как `None`

**Где брать данные:** validation artifact `"location": { "freeform": "..." }` из `ingest-validation.md` — это уже извлечённый и нормализованный текст локации. Normalizer должен перенести его в `location_query` если локация была подтверждена пользователем в Phase 7.2.

### 2.2 api-orchestrator.md §5.2.1 — добавить маппинг

**В JSON body пример добавить:**
```json
"narrative": {
  ...
  "location_query": "<normalized_issue_payload.location_query — omit if absent>"
}
```

**В field mapping table добавить строку:**
```
| narrative.location_query | normalized_issue_payload.location_query | No | Freeform location; сервер резолвит в geo; omit if null |
```

**Правило omit:** если `normalized_issue_payload.location_query` отсутствует или пустой — поле в запрос не включать. Сервер принимает пустой как `None`, но чистить стоит на GPT-стороне.

### 2.3 api-orchestrator.md §5.2.2 — добавить pre-flight проверку (опционально)

Если `narrative.institution` включён и location_query отсутствует при наличии явного адреса в `canonical_payload` — warning в `trace_notes`. Это информационная проверка, не stop-the-line.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-normalizer.md §4` | Output envelope | Добавить опциональное поле `location_query` в `normalized_issue_payload` (не внутри `canonical_payload`) |
| `GPT UI/instructions/story-normalizer.md` | Правила формирования | Добавить §4.6 или подраздел: критерии включения location_query, формат строки, приоритет при нескольких локациях |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | JSON body пример | Добавить `narrative.location_query` с условием omit |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | Field mapping table | Добавить строку `narrative.location_query` с источником |
| `GPT UI/instructions/story-api-methods-reference.md §1.1` | Field lock table | Обновить строку `narrative.location_query` — добавить источник маппинга |

---

## 4. Дизайн-решения

### D-01: Где хранить location_query в normalized_issue_payload?

**Принятое решение:** как поле верхнего уровня в `normalized_issue_payload` (рядом с `canonical_payload`, не внутри него). Обоснование:
- `canonical_payload` — контент Issue (title, description, labels). Geo — это метаданные intake, не Issue-content.
- `normalization_metadata` — рефсы и трейсинг, не данные. Туда тоже не подходит.
- Отдельное поле на верхнем уровне — чистая семантика.

### D-02: Один или несколько location_query?

**Принятое решение:** одна строка. Если пользователь упомянул несколько мест — normalizer выбирает наиболее специфичное/подтверждённое. Геo-резолвер сервера принимает один location_query. Множественные локации — edge case, который может быть отражён в `consistency_notes` (REQ-23 §C).

---

## 5. Acceptance Criteria

- [ ] `story-normalizer.md` содержит правило формирования `location_query` с критериями включения
- [ ] `normalized_issue_payload` структура документирует опциональный `location_query`
- [ ] `api-orchestrator.md §5.2.1` field mapping table содержит строку `narrative.location_query` с источником
- [ ] GPT отправляет `narrative.location_query` когда пользователь назвал и подтвердил локацию
- [ ] GPT не отправляет `narrative.location_query` когда локация не была названа или неоднозначна
- [ ] `POST /intake/stories` с `location_query: "Tartu mnt 80, Tallinn"` → HTTP 202
- [ ] После intake: если geo_service настроен — `story.geo` заполнен координатами и district
- [ ] `POST /intake/stories` без `location_query` → HTTP 202 (geo = null, поле опционально)

---

## 6. Не в scope этого REQ

- Реализация или изменение geo-резолвера на сервере (уже реализован)
- Map pin / геокодинг в SPA (SA-07)
- Валидация того, что location_query находится в пределах CLUSTER_GEO_SCOPE (сервер делает это сам — HTTP 422 GEO_SCOPE_MISMATCH при несоответствии)
- Множественные location_query в одном intake (не поддерживается контрактом)
- Обратная связь GPT-у об успехе geo-резолвинга (сервер не возвращает geo в response, только `story_id` и `status`)
