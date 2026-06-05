# REQ-36: Civic taxonomy expansion — multi-axis vocabulary для culture/youth/science/ecosystem/governance

> **Назначение:** расширить контролируемую таксономию меток (`story-label-taxonomy.md`, сейчас **v0.1**, ни разу не расширялась) с service-ticket-смещённой модели до многоосевой civic-модели, способной репрезентировать культурное развитие, молодёжь, науку, интеграцию, сообщество, дефицит среды (ecosystem) и модель управления (governance) как **first-class civic signals**, а не сжимать их в `education`.
> **Источник:** Gap Analysis 2026-06-04 (черновик REQ-36 §1–3); тест-история про культурно-научный центр → классификация свелась к `["education"]` (см. также [REQ-33](./REQ-33-label-extraction-multi-axis-improvement.md) multi-axis extraction).
> **Технический контекст:** [`story-label-taxonomy.md`](../../instructions/story-label-taxonomy.md) §3 определяет **11 осей**, но canonical-метки заданы только для 4 (`topic_domain` 12 ключей, `failure_mode` 9, `civic_signal` 7, `issue_archetype_support` 6); `service_object` — ось без canonical-меток; `deep_need`/`desired_outcome` — metadata-only by default (§3 L44–45); `affected_scope` хранит кандидатов metadata-only (§5 L118). REQ-33 уже требует multi-axis extraction в [`story-normalizer.md`](../../instructions/story-normalizer.md) §2.1/§4.1, но извлекать не из чего за пределами 4 заполненных осей.

**Версия:** 1.0 · 2026-06-04
**Статус:** Done (P3 pkg-000017 · GIM-156…160 · taxonomy v0.2 · matrix v1.0 · 2 advisory AC pending manual replay)
**Приоритет:** P2 (classification fidelity — влияет на кластеризацию, профили, dashboards; foundational для REQ-38)
**Тип:** GPT instruction update — `story-label-taxonomy.md` (SSOT расширение осей и canonical vocabulary); вторичные ссылки в `story-normalizer.md` §4.1a, `story-data-model.md` §5
**Серверная сторона:** не требует изменений (labels валидируются на instruction-слое; wire `canonical_labels` = `array<string>` без enum — [yaml L100–103](../custom-gpt-story-intake-actions.openapi.yaml))
**Парные REQ:** [REQ-20](./REQ-20-label-taxonomy-and-extraction-axes.md) (taxonomy axes baseline), [REQ-33](./REQ-33-label-extraction-multi-axis-improvement.md) (multi-axis extraction guidance), [REQ-34](./REQ-34-summary-generation-and-canonical-type-clarity.md) (type clarity) · downstream [REQ-38](./REQ-38-ecosystem-deficit-story-detection.md) (использует ось `ecosystem_signal`)

---

## 1. Текущее состояние (verified по коду 2026-06-04)

### 1.1 Таксономия service-ticket-смещена

[`story-label-taxonomy.md`](../../instructions/story-label-taxonomy.md) §4.1 — `topic_domain` canonical (12 ключей, дословно):

```
transport · roads · parking · public_space · waste · environment
housing · education · healthcare · digital_service · safety · accessibility
```

Хорошо ловит классические муниципальные жалобы (транспорт, дороги, мусор, парковка, жильё, медицина, digital, accessibility), но **нет** ключей для: культуры, молодёжи, науки, интеграции, языкового доступа, миграции, социальных служб, занятости, городского планирования, участия, наследия, сообщества, governance. История про «город как среду развития / культурный капитал / социальную ткань» сжимается в `education`.

### 1.2 Оси существуют, но canonical vocabulary не заполнен

[`story-label-taxonomy.md`](../../instructions/story-label-taxonomy.md) §3 — **11 осей**, но:

| Ось | Состояние в коде | Дельта |
|---|---|---|
| `topic_domain` | 12 canonical (§4.1) | расширить (§2.1) |
| `service_object` | ось объявлена (§3), **0 canonical-меток** в §4 | заполнить vocabulary |
| `location_context` | metadata-only (§5 L117) | частично promote (geo — см. REQ-38) |
| `failure_mode` | 9 canonical (§4.2) | без изменений |
| `issue_archetype_support` | 6 canonical (§4.4) | без изменений |
| `affected_scope` | metadata-only кандидаты (§5 L118): `parents, children_context, elderly_context, drivers, residents, …` | promote выбранных до canonical + расширить (§2.3) |
| `civic_signal` | 7 canonical (§4.3), включая `equity_access`, `city_for_people`, `systemic_pattern`, `not_only_me` | расширить (§2.4) |
| `deep_need` | metadata-only by default (§3 L44; §5 L119) | promote выбранных (§2.5) |
| `desired_outcome` | metadata-only by default (§3 L45; §5 L120) | promote выбранных (§2.6) |
| `risk_privacy_safety` | internal-only (§6) | без изменений |
| `confidence_state` | internal-only (§6) | без изменений |
| **`ecosystem_signal`** | **отсутствует** | **новая ось (§2.7)** |
| **`governance_signal`** | **отсутствует** | **новая ось (§2.8)** |

### 1.3 Расхождения черновика с фактическими осями (для корректного tasking)

| Черновик предлагал | Факт в SSOT | Решение REQ-36 |
|---|---|---|
| Новая ось `affected_population` | Уже есть ось `affected_scope` (§3, §5 L118) | **НЕ создавать дубль** — расширять/promote `affected_scope`; рассмотреть переименование как open decision (§6) |
| Новые оси `service_object`, `location_context`, `desired_outcome`, `deep_need` | Все 4 уже существуют как оси (§3) | заполнить/promote vocabulary, не «добавлять ось» |
| Новая ось `equity_dimension` | Перекрывается с `civic_signal.equity_access` (§4.3) | реконсилировать — не плодить параллельную ось (§6 open decision) |

---

## 2. Целевое состояние

> Все списки ниже — **research draft vocabulary** из источника; при tasking каждый ключ проходит сверку с §2 «Non-negotiable rules» таксономии (не плодить дубли осей, не превращать internal/privacy в card-labels) и получает `Meaning`-строку в стиле существующих таблиц §4.

### 2.1 Расширить `topic_domain` (новые домены для Tallinn/Estonia)

Сохранить 12 существующих (§1.1). Добавить:

```
culture · youth_development · science_and_research · arts_and_creativity
sports_and_recreation · community_life · integration · language_access
migration_adaptation · social_services · employment_and_skills · local_economy
urban_planning · public_participation · heritage · mental_wellbeing_civic
family_support · elderly_support · childcare · school_environment
after_school_ecosystem · civil_society · trust_and_governance
```

### 2.2 Заполнить `service_object` canonical vocabulary

Ось объявлена, но пуста. Заполнить конкретными объектами/сервисами (kindergarten, youth_center, cultural_center, library, sports_facility, …) — точный список определить при tasking из resident-нарративов.

### 2.3 Расширить/promote `affected_scope` (НЕ новая ось)

Promote из metadata-only до canonical выбранных существующих (`parents`, `children_context`→`children`, `elderly_context`→`elderly`, `residents`) + добавить:

```
children · youth · students · families · people_with_disabilities
pedestrians · cyclists · drivers · public_transport_users · newcomers
refugees · ukrainian_refugees · russian_speaking · estonian_speaking
multilingual_communities · low_income_residents · small_businesses
cultural_workers · teachers · mentors · volunteers
```

### 2.4 Расширить `civic_signal`

К 7 существующим добавить (реконсилировать с `equity_access`):

```
information_gap_civic · participation_signal · social_cohesion · brain_drain_signal
```

### 2.5 Promote `deep_need` (из metadata-only)

```
belonging · dignity · agency · being_heard · predictability · safety_need
fairness · cultural_identity · intellectual_growth · creative_self_realization
meaningful_community · future_opportunity · trust · continuity
```

### 2.6 Promote `desired_outcome` (из metadata-only)

```
new_public_service · new_community_hub · better_access · transparent_process
stronger_standards · more_mentors · more_programs · better_coordination
replicable_solution · open_methodology · cooperative_institution
multilingual_access · safe_environment · human_centered_city
```

### 2.7 Новая ось `ecosystem_signal`

Для историй, где проблема — **отсутствие среды**, а не один сломанный сервис (критично для REQ-38):

```
ecosystem_gap · institutional_decline · missing_infrastructure · weak_coordination
capacity_shortage · mentor_shortage · community_fragmentation · loss_of_continuity
brain_drain · underused_resources · replicable_model_needed
```

### 2.8 Новая ось `governance_signal`

Для историй о модели управления как части desired solution (кооператив, прозрачность, риск узурпации):

```
transparent_governance · participatory_governance · cooperative_model
power_concentration_risk · accountability_gap · open_source_model
replicability · community_ownership
```

### 2.9 Sub-domain vocabulary (culture / science / youth)

Расширенные кластеры под `topic_domain`/`service_object` (полные списки — §3.6 источника): `cultural_infrastructure, cultural_access, cultural_continuity, minority_culture, creative_spaces, performing_arts, music_education, theatre_education, cultural_events, community_culture, heritage_preservation`; `science_education, research_projects, stem_development, intellectual_environment, academic_mentorship, talent_development, critical_thinking, innovation_capacity`; `youth_development, after_school_programs, youth_mentorship, youth_spaces, youth_participation, youth_creativity, youth_science, youth_culture`. Разместить по правильным осям при tasking (избегая дублей с §2.1).

### 2.10 Tallinn/Estonia coverage map (research baseline)

Кластерная карта тем (Mobility / Urban space / Housing / Education / Youth / Culture / Science / Integration / Migration / Social support / Digital state / Governance / Environment / Economy / Safety) — сохранена из источника §3.10 как baseline для распределения ключей по осям.

### 2.11 Целевая классификация (hypothesis, тест-история культурно-научный центр)

После расширения история ожидаемо получает мультиосевую классификацию (≥5 осей) вместо `["education"]`:

```json
{
  "topic_domain": ["culture", "education", "youth_development", "science_and_research", "community_life"],
  "affected_scope": ["children", "youth", "russian_speaking", "ukrainian_refugees", "families"],
  "ecosystem_signal": ["ecosystem_gap", "institutional_decline", "mentor_shortage", "loss_of_continuity", "replicable_model_needed"],
  "governance_signal": ["cooperative_model", "transparent_governance", "open_source_model", "community_ownership"],
  "deep_need": ["intellectual_growth", "creative_self_realization", "cultural_continuity", "meaningful_community", "future_opportunity"],
  "civic_signal": ["not_only_me", "systemic_pattern", "equity_access", "city_for_people"]
}
```

> Примечание: `affected_population` из черновика заменён на фактическую ось `affected_scope`.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-label-taxonomy.md` | §3 Axis model | Добавить оси `ecosystem_signal`, `governance_signal`; реконсилировать `affected_scope` vs draft `affected_population`, `equity_dimension` vs `civic_signal` |
| `GPT UI/instructions/story-label-taxonomy.md` | §4 Canonical allowed labels | Расширить `topic_domain` (§2.1); заполнить `service_object` (§2.2); добавить блоки `ecosystem_signal` (§2.7), `governance_signal` (§2.8) |
| `GPT UI/instructions/story-label-taxonomy.md` | §5 Metadata-only | Promote выбранных `affected_scope` / `deep_need` / `desired_outcome` до canonical; оставить остальное metadata |
| `GPT UI/instructions/story-label-taxonomy.md` | Version header + §8 history | Bump 0.1 → след.; changelog REQ-36 |
| `GPT UI/instructions/story-normalizer.md` | §4.1a extraction hints | Добавить hints для новых высокоценных осей (`ecosystem_signal`, `culture`, `youth_development`) — согласовать с REQ-33 |
| `GPT UI/instructions/story-data-model.md` | §5 enums source | Cross-ref на расширенную таксономию (labels SoT) |

---

## 4. Acceptance Criteria

**Static (verifiable по `story-label-taxonomy.md`):**
- [ ] `topic_domain` §4.1 содержит `culture`, `youth_development`, `science_and_research` (минимум) с `Meaning`-строками
- [ ] Ось `ecosystem_signal` присутствует в §3 и имеет canonical-блок в §4 (включая `ecosystem_gap`, `institutional_decline`)
- [ ] Ось `governance_signal` присутствует в §3 и §4 (включая `cooperative_model`, `transparent_governance`)
- [ ] `service_object` имеет ≥1 canonical-ключ (ось более не пустая)
- [ ] `affected_scope` имеет canonical-ключи (`children`, `youth` минимум) — **без** создания параллельной оси `affected_population`
- [ ] Выбранные `deep_need` / `desired_outcome` повышены до canonical disposition с явным правилом
- [ ] §6 open decisions (`affected_scope` rename, `equity_dimension` reconcile) разрешены или явно отложены
- [ ] Version bump + changelog REQ-36; no internal/privacy label promoted to card labels (§2 rule 4 не нарушен)

**Advisory (live replay, из кода не верифицируются):**
- [ ] Тест-история культурно-научный центр → классификация ≥5 осей (§2.11), не `["education"]`
- [ ] История про научные школы → `science_and_research` + `academic_mentorship` когда evidence

---

## 5. Не в scope

- **Ecosystem-deficit detection поведение** (когда нет одного сломанного сервиса) → [REQ-38](./REQ-38-ecosystem-deficit-story-detection.md) (зависит от оси `ecosystem_signal`, определяемой здесь).
- **Geographic intelligence** (location confidence, canonicalization) → [REQ-39](./REQ-39-geographic-intelligence-confidence-canonicalization.md).
- Изменения backend / wire-схемы (`canonical_labels` уже `array<string>`).
- Публичный UI-словарь (instruction-таксономия ≠ UI dictionary, §1 таксономии).
- Live web/social research тем — отдельный research-этап (§6 ниже).

---

## 6. Open decisions / research tasks

- **OD-1:** переименовать ось `affected_scope` → `affected_population` (выразительнее) или оставить `affected_scope` (меньше propagation). Влияет на `story-normalizer.md` §4.2.1 candidate axis-строки.
- **OD-2:** `equity_dimension` как отдельная ось vs расширение `civic_signal.equity_access`. Рекомендация: не плодить ось без доказанной нужды (analysis.mdc single-source).
- **OD-3 (research, live):** сверить vocabulary с реальными темами — Tallinn city channels, ERR/rus.err.ee, русско-/эстонско-/украиноязычные community channels, Tallinn development/integration/youth strategy, existing civic platforms; найти темы, повторяющиеся в resident narratives, но отсутствующие в официальных taxonomy (источник §3.12).
