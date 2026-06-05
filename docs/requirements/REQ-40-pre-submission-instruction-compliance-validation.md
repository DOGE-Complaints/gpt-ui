# REQ-40: Pre-submission instruction-compliance validation — evidence↔payload missing-trigger detection

> **Назначение:** добавить в pre-flight перед `POST /intake/stories` слой проверки **соответствия payload фактическим инструкциям**: сравнить evidence истории (что резидент сообщил / что произвели upstream-модули) с построенным `StoryIntakeRequest` и выявлять **expected-but-missing** поля (location, summary, multi-axis labels, severity) с исходом FAIL/warning. Сейчас инструкции существуют (REQ-26/32/33/34/35), но execution path не гарантирует их срабатывание — а pre-flight проверяет только **форму** уже собранного payload, не его **полноту относительно evidence**.
> **Источник:** Gap Analysis 2026-06-04 (черновик «Trigger Reliability, Instruction Compliance & Execution Determinism» §2.2, §3.2, §6.2, §8); тест-прогон 2026-06-01 (поля пустые потому что GPT не сработал, не из-за бага backend).
> **Технический контекст (verified по коду):** [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.2 содержит checks **1–12** (GIM-28, L487–526) — это **shape/consistency** проверки построенного payload (non-empty слоты, enum-валидность, gate approved); item 12 (L524) — единственная evidence-aware строка, но только **informational trace** в узком post-demo institution-кейсе. Evidence↔payload сравнения как **FAIL** в §5.2.2 нет. Evidence-сторона доступна: `location.freeform` из ingest validation artifact ([`story-normalizer.md`](../../instructions/story-normalizer.md) §4.6 L303), `ingest_validation_report` из [`ingest-validation.md`](../../instructions/ingest-validation.md).

**Версия:** 1.0 · 2026-06-05
**Статус:** requirements — ready for tasking
**Приоритет:** P1 (Critical — execution determinism; напрямую закрывает корневую причину пустых полей demo)
**Тип:** GPT instruction update — `api-orchestrator.md` §5.2.2 (расширение pre-flight), вторичное `ingest-validation.md` (evidence-сторона), `story-normalizer.md` (severity_confidence)
**Серверная сторона:** не требует изменений (валидация на instruction-слое до HTTP)
**Парные REQ:** [REQ-26](./REQ-26-location-query-normalizer-to-wire.md), [REQ-32](./REQ-32-origin-traceability-sending-enforcement.md), [REQ-33](./REQ-33-label-extraction-multi-axis-improvement.md), [REQ-34](./REQ-34-summary-generation-and-canonical-type-clarity.md), [REQ-35](./REQ-35-demo-field-population-gpt-fixes.md) (правила, которые этот REQ enforce'ит) · [REQ-30](./REQ-30-admission-gate-story-intake-strict-chain.md) (admission gate §5.2.0a — родственный enforcement) · downstream [REQ-41](./REQ-41-trigger-observability-audit-trail.md) (observability над теми же триггерами)

---

## 1. Текущее состояние (verified по коду 2026-06-05)

### 1.1 Pre-flight проверяет форму, не полноту относительно evidence

[`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.2 checks 1–12 (L487–526):

| Check | Что проверяет | Тип |
|---|---|---|
| 1–5 | session_language / title / description / detected_input_language / gate approved non-empty/valid | shape + chain |
| 6 | normalizer_module ref match | informational |
| 7–8 | institution demo-gate / i18n completeness | drop-field |
| 9 | consistency_notes non-empty if present | shape |
| 10 | gpt_signals enum-валидность если present | shape |
| 11 | summary все 3 слота non-empty если present | shape |
| 12 | institution present + location_query omitted + address есть → **trace_notes (informational, не STOP)** | evidence-aware, **narrow** |

**Ни один check не делает evidence→payload сравнение как блокирующее**: «история содержала локацию/несколько доменов/severity-сигнал, но payload их не emit'нул». Item 12 — единственный намёк, и он informational + только в post-demo institution-ветке.

### 1.2 Поля-правила существуют, но enforcement отсутствует

| REQ-40 ask | Правило уже есть | Доказательство | Чего нет |
|---|---|---|---|
| §3.1 location MUST при упоминании | REQ-35 | [`story-normalizer.md:317`](../../instructions/story-normalizer.md#L317) MUST-when-confirmed | блокирующего pre-flight FAIL (§3.2) |
| §4.1 origin.source mandatory | REQ-32 | [`api-orchestrator.md:326`](../../instructions/api-orchestrator.md#L326) | ✅ покрыто (вне дельты) |
| §5.1/5.2 summary distinct + problem/desired | REQ-34 | `story-normalizer.md §4.1` L140–147 | проверки distinctness как check |
| §6.1 multi-axis labels | REQ-33 | `story-normalizer.md` §4.1a | collapse-detection check (§6.2) |
| §7.1 severity только при evidence | REQ-35 | [`story-normalizer.md:243-249`](../../instructions/story-normalizer.md#L243-L249) | явный `severity_confidence` концепт (§7.2) |

### 1.3 Расхождение черновика (для tasking)

Черновик §6.1 перечисляет ось `affected_population` — фактическая ось в SSOT `affected_scope` (см. [REQ-36](./REQ-36-civic-taxonomy-expansion-multi-axis.md) §1.3). Использовать `affected_scope`.

---

## 2. Целевое состояние — расширить §5.2.2 (НЕ параллельный валидатор)

> Все проверки ниже — **новые items 13+ в существующем §5.2.2** (additive к checks 1–12, как §5.2.0a additive к §5.2.2). Single source of truth: один pre-flight, не второй валидатор.

### 2.1 Location coverage check (§3.2 → FAIL)

Если `ingest_validation_report` содержит подтверждённую `location.freeform` (резидент назвал/подтвердил место) **И** `narrative.location_query` отсутствует в payload → **FAIL** (stop-the-line), сообщить какой модуль (`story-normalizer.md` §4.6) должен re-run. Усиление item 12 с informational до блокирующего для confirmed-location кейса.

### 2.2 Single-label collapse detection (§6.2 → warning)

Если evidence/нарратив указывает на несколько доменов/осей, а `canonical_labels` содержит ровно один ключ → **validation warning** + trace_notes (не STOP — labels не обязательны на wire). Cross-ref REQ-33 multi-axis.

### 2.3 Missing-trigger detection (§2.2, §8 — consolidated compliance pass)

Перед HTTP сравнить evidence ↔ payload по набору required-when-applicable триггеров и собрать список expected-but-missing:

| Триггер | Expected когда | Missing если |
|---|---|---|
| location | confirmed location в ingest report | `narrative.location_query` отсутствует |
| origin.source | всегда (Action submit) | поле пустое/отсутствует |
| summary | контент достаточен (REQ-34) | `narrative.summary` отсутствует при достаточном нарративе |
| multi_axis_labels | нарратив ≥2 осей | один label (см. §2.2) |
| gpt_signals | есть субъективный сигнал | sidecar пуст при явном сигнале |

Required-триггеры (location при confirmed, origin.source) → **FAIL**; soft-триггеры (summary, labels, signals) → **warning** + trace_notes.

### 2.4 Runtime capability distinction (§4.2)

Для `origin.conversation_id` / `tool_call_id`: явно различать **unavailable** (runtime не отдаёт поле → omit-key, REQ-35 D3) vs **silent omit при доступности** (последнее — нарушение). Зафиксировать как trace_notes-запись «runtime did not expose conversation_id», чтобы omit был объяснимым, а не молчаливым.

### 2.5 Severity confidence (§7.2)

В [`story-normalizer.md`](../../instructions/story-normalizer.md) §4.3: если `severity` выведена без явного/сильного evidence — пометить низкую уверенность и **omit** поле (семантически уже есть в REQ-35 L247 «omit when no defensible signal»; REQ-40 делает критерий явным через `severity_confidence = LOW → omit`).

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/api-orchestrator.md` | §5.2.2 | Items 13+: location-coverage FAIL (§2.1), single-label-collapse warning (§2.2), missing-trigger compliance pass (§2.3), runtime-capability trace (§2.4) |
| `GPT UI/instructions/ingest-validation.md` | evidence-сторона | Гарантировать, что `ingest_validation_report` несёт confirmed-location и multi-domain signal для сравнения |
| `GPT UI/instructions/story-normalizer.md` | §4.3 | Явный `severity_confidence = LOW → omit` (§2.5) |
| `GPT UI/instructions/api-orchestrator.md` | Version + history | Bump + changelog REQ-40 |

---

## 4. Acceptance Criteria

**Static (verifiable по `api-orchestrator.md` §5.2.2):**
- [ ] Есть check: confirmed `location.freeform` в evidence + `location_query` отсутствует → **STOP/FAIL** (не informational)
- [ ] Есть check: multi-domain evidence + один label → **warning** + trace_notes
- [ ] Есть consolidated missing-trigger pass (location/origin/summary/labels/signals) с разделением FAIL vs warning
- [ ] `origin.conversation_id` omit при unavailable сопровождается trace_notes-объяснением (не silent)
- [ ] `story-normalizer.md` §4.3 содержит явный `severity_confidence = LOW → omit`
- [ ] Все новые проверки оформлены как items §5.2.2 (additive к 1–12), не параллельный валидатор
- [ ] Регрессия: checks 1–12 (GIM-28), §5.2.0a admission gate, §5.2.0 PII не изменены семантически

**Advisory (live replay):**
- [ ] История «в Таллине» без `location_query` → pre-flight FAIL с указанием re-run §4.6
- [ ] Multi-domain история с одним label → warning зафиксирован

---

## 5. Не в scope

- **Trigger observability / audit trail** (экспозиция «какой триггер сработал» как диагностика) → [REQ-41](./REQ-41-trigger-observability-audit-trail.md).
- Сами поля-правила (location MUST, origin.source, summary gen, multi-axis, severity) — уже REQ-26/32/33/34/35; здесь только их **enforcement**.
- Изменения backend / wire-схемы.
- `affected_scope` canonical promotion — [REQ-36](./REQ-36-civic-taxonomy-expansion-multi-axis.md).
