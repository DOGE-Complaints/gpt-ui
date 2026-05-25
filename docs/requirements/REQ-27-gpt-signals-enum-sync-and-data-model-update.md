# REQ-27: `gpt_signals` enum sync — `story-data-model.md §4.3` + orchestrator note fix

> **Назначение:** синхронизировать enum-значения `impact_estimation`, `problem_status`, `severity` в `story-data-model.md §4.3` с серверными frozenset-ами (`contracts.py:61-65`); исправить ошибочный hint `UNKNOWN` для `impact_estimation` в `api-orchestrator.md`; убрать устаревшую оговорку "non-wire metadata / must not be sent" (gpt_signals уже в контракте с REQ-42/23).  
> **Источник:** `doge-complaints-gateway/docs/analysis/audit-orchestrator-api-alignment-2026-05-24.md` (GAP-01, GAP-02, GAP-03) + дополнительный GAP-04 из прямого чтения файлов.  
> **Технический SoT (сервер):** `src/core/intake/contracts.py:61-65` (frozensets); `_parse_gpt_signal_enum()` (lines 128-148).

**Версия:** 1.0 · 2026-05-25  
**Статус:** requirements — ready for tasking  
**Приоритет:** P1 (GAP-01/03 — прямой HTTP 400); P2 (GAP-02/04 — runtime-risk / confusion-risk)  
**Тип:** документационные исправления — поведение GPT не меняется (normalizer уже содержит корректные значения); исправляется только data-model как SoT и один ошибочный hint в orchestrator  
**Серверная сторона:** не требует изменений  
**Парные REQ:** [REQ-23](./REQ-23-gpt-behavioral-extensions-pii-signals-consistency.md) (gpt_signals активация), [REQ-25](./REQ-25-canonical-type-labels-and-summary-wire-activation.md)

---

## 1. Текущее состояние (проблемы)

### 1.1 GAP-01 — `impact_estimation` enum несовместим с сервером (HIGH)

**Файл:** `GPT UI/instructions/story-data-model.md §4.3` (v0.11, строка 75)

```markdown
| `impact_estimation` | `personal` | `city/town` | `state` | `country` | `Earth` | ...
```

**Сервер (`contracts.py:62`):**
```python
GPT_SIGNAL_IMPACT_VALUES = frozenset({"LOCAL", "DISTRICT", "CITY", "NATIONAL"})
```

**`_parse_gpt_signal_enum()`** (строки 128–148): нормализует `.upper()`, затем проверяет `if normalized not in allowed → IntakeValidationError → HTTP 400 DOMAIN_ERROR`.

- `personal` → `PERSONAL` → NOT IN frozenset → **HTTP 400**
- `city/town` → `CITY/TOWN` → NOT IN frozenset → **HTTP 400**
- `state` → `STATE` → NOT IN frozenset → **HTTP 400**
- `country` → `COUNTRY` → NOT IN frozenset → **HTTP 400**
- `Earth` → `EARTH` → NOT IN frozenset → **HTTP 400**

**Статус normalizer:** `story-normalizer.md §4.3` (v0.2.1) уже содержит корректные `LOCAL, DISTRICT, CITY, NATIONAL`. Data-model — нет.

---

### 1.2 GAP-02 — `problem_status` частично несовместим с сервером (MEDIUM)

**Файл:** `GPT UI/instructions/story-data-model.md §4.3` (строка 76)

```markdown
| `problem_status` | `ongoing` | `resolved` | `worsened` | ...
```

**Сервер (`contracts.py:63-65`):**
```python
GPT_SIGNAL_PROBLEM_STATUS_VALUES = frozenset(
    {"ONGOING", "RESOLVED", "RECURRING", "UNKNOWN"}
)
```

- `worsened` → `WORSENED` → NOT IN frozenset → **HTTP 400**
- `RECURRING`, `UNKNOWN` — отсутствуют в data-model.md (неполная документация)

---

### 1.3 GAP-03 — orchestrator hint `UNKNOWN` для `impact_estimation` (HIGH)

**Файл:** `GPT UI/instructions/api-orchestrator.md §5.2.1` (строка 331)

```
| `gpt_signals.impact_estimation` | ... | No | Enum per REQ-42; use `UNKNOWN` or omit field if unsure |
```

`UNKNOWN` **не входит** в `GPT_SIGNAL_IMPACT_VALUES`. Если GPT следует этому совету:
- `impact_estimation: "UNKNOWN"` → `.upper()` → `"UNKNOWN"` → NOT IN `{LOCAL, DISTRICT, CITY, NATIONAL}` → **HTTP 400**

Корректная инструкция для `UNKNOWN` уже есть для `problem_status` (строка 386): `use UNKNOWN for problem_status only` — это правильно.

---

### 1.4 GAP-04 — "non-wire metadata / must not be sent" — устаревшая оговорка (MEDIUM)

**Файл:** `GPT UI/instructions/story-data-model.md §4.3` (строки 70–78)

Текущий текст:
```
These fields are **non-wire metadata** for the current runtime contract and must not be 
sent as `StoryIntakeRequest` fields until HTTP/OpenAPI explicitly includes them.
```

**Фактическое состояние (REQ-42/REQ-23, 2026-05-22):**
- `contracts.py:61-65` определяет `GPT_SIGNAL_*_VALUES` frozensets
- `contracts.py:69-81` — `GptSignalsBlock` dataclass с тремя полями
- `contracts.py:92` — `StoryIntakeRequest.gpt_signals: GptSignalsBlock | None`
- OpenAPI v0.4.0 включает `GptSignals` схему

Поля `gpt_signals` **уже в wire-контракте** с момента REQ-42/23. Текст "must not be sent" является ложным и может заставить GPT-агента, читающего data-model как SoT, не отправлять `gpt_signals`.

---

### 1.5 GAP-05 — case inconsistency для `severity` (LOW/COSMETIC)

**Файл:** `story-data-model.md §4.3` (строка 74): `low | medium | high | critical` (lowercase)  
**Normalizer:** `story-normalizer.md §4.3`: `LOW, MEDIUM, HIGH, CRITICAL` (uppercase)  
**Server:** нормализует `.upper()`, runtime-риска нет.

Cosmetic — несоответствие между data-model и normalizer снижает читаемость и создаёт неоднозначность в документации.

---

## 2. Целевое состояние

### 2.1 `story-data-model.md §4.3` — исправленная таблица subjective fields

**Было:**
```markdown
| Field | Type (logical) | Rule |
|-------|------------------|------|
| `severity` | `low` \| `medium` \| `high` \| `critical` | Resident-perceived seriousness; optional... |
| `impact_estimation` | `personal` \| `city/town` \| `state` \| `country` \| `Earth` | Self-reported perceived scope of impact. |
| `problem_status` | `ongoing` \| `resolved` \| `worsened` | How the resident frames change over time, if stated. |

These fields are **non-wire metadata** for the current runtime contract and must not be 
sent as `StoryIntakeRequest` fields until HTTP/OpenAPI explicitly includes them.
```

**Стало:**
```markdown
| Field | Type (logical) | Rule |
|-------|------------------|------|
| `severity` | `LOW` \| `MEDIUM` \| `HIGH` \| `CRITICAL` | Resident-perceived seriousness. Optional; collect only if stated without leading the user. |
| `impact_estimation` | `LOCAL` \| `DISTRICT` \| `CITY` \| `NATIONAL` | Administrative scope of perceived impact (Estonia scale). Omit if unsure — no UNKNOWN fallback for this field. |
| `problem_status` | `ONGOING` \| `RESOLVED` \| `RECURRING` \| `UNKNOWN` | How the resident frames the current state of the problem. `UNKNOWN` is valid only for this field; use when status was not clarified. |

These fields are sent via **`gpt_signals`** block in `StoryIntakeRequest` (REQ-42/23, wire since OpenAPI v0.4.0). 
Values are normalized `.upper()` by the server; enum values above must match `contracts.py` frozensets exactly.
Normalizer placement: `non_wire_metadata` sidecar → orchestrator maps to `gpt_signals` per [`story-normalizer.md §4.3`](story-normalizer.md) and [`api-orchestrator.md §5.2.1`](api-orchestrator.md).
```

### 2.2 `api-orchestrator.md §5.2.1` — исправить hint для `impact_estimation`

**Было (строка 331):**
```
| `gpt_signals.impact_estimation` | `non_wire_metadata.impact_estimation` | No | Enum per REQ-42; use `UNKNOWN` or omit field if unsure |
```

**Стало:**
```
| `gpt_signals.impact_estimation` | `non_wire_metadata.impact_estimation` | No | Enum per REQ-42: LOCAL, DISTRICT, CITY, NATIONAL; omit field if unsure (no UNKNOWN fallback for this field) |
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-data-model.md §4.3` | Таблица subjective fields | Все три поля: исправить enum-значения, привести к UPPERCASE, обновить семантику |
| `GPT UI/instructions/story-data-model.md §4.3` | Текст после таблицы | Удалить "must not be sent / non-wire until included"; добавить корректный текст о `gpt_signals` wire |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | Field mapping table, строка `impact_estimation` | Убрать `UNKNOWN`, указать 4 допустимых значения явно |
| `GPT UI/instructions/story-data-model.md` | Version header + version history | Bump → 0.12 · 2026-05-25 |

---

## 4. Конкретные изменения

### 4.1 story-data-model.md — таблица §4.3

Заменить строки `severity`, `impact_estimation`, `problem_status` (см. §2.1 выше).

Ключевые различия по полям:
- `severity`: `low/medium/high/critical` → `LOW/MEDIUM/HIGH/CRITICAL`
- `impact_estimation`: `personal|city/town|state|country|Earth` → `LOCAL|DISTRICT|CITY|NATIONAL`; добавить: "Omit if unsure — no UNKNOWN fallback"
- `problem_status`: `ongoing|resolved|worsened` → `ONGOING|RESOLVED|RECURRING|UNKNOWN`; `RECURRING` — "периодически повторяется"; `UNKNOWN` — "статус не уточнён"

### 4.2 story-data-model.md — параграф после таблицы §4.3

Удалить:
```
These fields are **non-wire metadata** for the current runtime contract and must not be 
sent as `StoryIntakeRequest` fields until HTTP/OpenAPI explicitly includes them. They are 
not required for §4.1 completeness and must not block StoryIntake handoff.
```

Заменить на текст, описывающий фактическую wire-архитектуру (см. §2.1 выше).

### 4.3 api-orchestrator.md — одна строка в field mapping table

Строка `gpt_signals.impact_estimation` (текущая строка 331): убрать `UNKNOWN`, перечислить допустимые значения (см. §2.2 выше).

---

## 5. Acceptance Criteria

- [ ] `story-data-model.md §4.3` содержит `impact_estimation: LOCAL | DISTRICT | CITY | NATIONAL`
- [ ] `story-data-model.md §4.3` содержит `problem_status: ONGOING | RESOLVED | RECURRING | UNKNOWN` (без `worsened`)
- [ ] `story-data-model.md §4.3` содержит `severity: LOW | MEDIUM | HIGH | CRITICAL` (UPPERCASE)
- [ ] `story-data-model.md §4.3` не содержит текст "must not be sent as StoryIntakeRequest fields"
- [ ] `story-data-model.md §4.3` содержит ссылку на `gpt_signals` как wire-путь (REQ-42/23, OpenAPI v0.4.0)
- [ ] `api-orchestrator.md §5.2.1` строка `impact_estimation` не содержит `UNKNOWN`
- [ ] `api-orchestrator.md §5.2.1` строка `impact_estimation` явно перечисляет допустимые значения: `LOCAL, DISTRICT, CITY, NATIONAL`
- [ ] `story-data-model.md` версия ≥ `0.12` с changelog-записью для REQ-27
- [ ] `POST /intake/stories` с `impact_estimation: "LOCAL"` → HTTP 202 (сервер принимает)
- [ ] `POST /intake/stories` с `impact_estimation: "UNKNOWN"` → HTTP 400 (не допустимо)
- [ ] `POST /intake/stories` с `problem_status: "RECURRING"` → HTTP 202 (сервер принимает)
- [ ] `POST /intake/stories` с `problem_status: "WORSENED"` → HTTP 400 (не допустимо)

---

## 6. Не в scope этого REQ

- Изменения в `story-normalizer.md §4.3` (уже содержит корректные uppercase-значения — v0.2.1)
- Изменения в `contracts.py` или frozensets на стороне сервера
- Изменения в OpenAPI schema (поля уже присутствуют в v0.4.0)
- Добавление `impact_estimation: UNKNOWN` на сервере (не запрошено)
- Изменения в interview flow для сбора этих полей
