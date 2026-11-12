# STORY-GPT-MISSION-01 — Standing, personal stories, anti-gossip (mission cleanliness)

## Meta
- **Key:** `STORY-GPT-MISSION-01-mission-aligned-content-cleanliness`
- **Пакет:** [`intake-hardening/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Pipeline / GIM:** GIM-216…220 🟢 (pkg-000029)
- **Приоритет:** High — GAP-IH-01 (operator refine 2026-08-10)
- **Тип:** product requirement (mission / policy standing)
- **Gap:** [`gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md`](../../../analysis/gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md) §1
- **Основание:** [REQ-01](../../../requirements/REQ-01-mission.md); [`story-policy-gate.md`](../../../../instructions/story-policy-gate.md) §7.1; [`story-interview-flow.md`](../../../../instructions/story-interview-flow.md) §5/§7.5 personal story; as-built validation
- **MVP enforcement seam:** GPT policy-gate + interview + admission — **без** server mission LLM

## Вердикт: не избыточна ли стори?

**Не избыточна.** Текущая система валидации закрывает структуру (§4.1), safety checkpoints и тонкий demo policy pack — но **не** продуктовый standing и **не** явную норму «личные истории = ценный сигнал».

Факт по [`story-policy-gate.md`](../../../../instructions/story-policy-gate.md) §7.1 demo baseline:

- BLOCK: `IRRELEVANT_NON_CIVIC`, `SCAM_OR_SPAM`, `OBSCENE_OR_TROLL`
- approved: «content that is **recognizably civic**»

**Пробелы §7.1 vs миссия оператора:**

| Тема | В §7.1 / текущих gates | Нужно продукту |
|------|------------------------|----------------|
| Личные истории (семья, ребёнок, своя боль) | Не названы как ACCEPT; «civic» можно прочитать узко | **ACCEPT** — статистика / кластеры / коллективное внимание |
| Standing: только про себя / personally affected | **Нет** | Явный критерий |
| Чужие дела (соседи, сплетни) | **Нет** отдельного reject | **REJECT** — склочничество |
| Spam / troll / off-topic | Есть | Сохранить |

Interview уже допускает personal story без Q7 ([`story-interview-flow.md`](../../../../instructions/story-interview-flow.md) §5/§7.5) — стори **согласует policy + admission** с этим и с миссией REQ-01, а не дублирует server junk-guard (→ GPT-GUARD-01).

## Зачем простыми словами

Мы собираем человеческий опыт, который на уровне кластеров становится общественным сигналом. Можно рассказать про неадекватного мужа или измученного ребёнка — это боль, которая **лично затрагивает**. Нельзя тащить чужие соседские драмы из любопытства.

## Текущее состояние (verified)

| Элемент | Реальность |
|---------|------------|
| Миссия REQ-01 | Мягкое извлечение опыта → структурированный общественный сигнал |
| Interview | Personal story valid; Q7 non-blocking |
| Policy §7.1 | Тонкий civic/spam/troll pack — без standing / anti-gossip |
| Gateway | Schema only — без mission/standing |
| As-built | [`story-validation-and-garbage-filtering-as-built.md`](../../../runtime-docs/story-validation-and-garbage-filtering-as-built.md) |

## Требование / целевое состояние

### Классы контента (product outcomes)

| Класс | Outcome | Пояснение |
|-------|---------|-----------|
| Личная история (супруг(а), ребёнок, своя боль, свои родители-пенсионеры) | **ACCEPT** | Ценный сигнал; при кластеризации — коллективная потребность во внимании/поддержке |
| Про себя **или** personally affected (боль детей = боль родителей; супруг(а); собственные родители) | **ACCEPT** | **Standing** |
| Чужие дела без собственной затронутости (соседи, «у них в семье», склоки) | **REJECT** | Nose into others’ affairs / склочничество |
| Off-topic / non-civic noise, spam/scam, obscene/troll, test/junk | **REJECT** | Уже demo pack + GPT-GUARD-01 |
| Узкое «только формальная civic-жалоба» | **Неверно** | Личное ≠ вне scope миссии |

### Цепочка

1. Зафиксировать критерии в policy SSOT (demo baseline и/или OP-DOC) + согласовать interview/admission формулировки.
2. Verify: policy gate и admission **реально** применяют standing / anti-gossip (не только «recognizably civic»).
3. MVP: **только GPT**; server mission LLM **не** в scope.

## Acceptance Criteria (requirements)

- [x] Policy/demo (или OP-DOC) явно: personal stories ACCEPT; standing = self / personally affected; neighbor-gossip REJECT.
- [x] Evidence scenarios: (A) «муж / ребёнок / я» → path to approved+admission OK; (B) «у соседей скандал, мне просто интересно» → rejected / STOP до stash.
- [x] Checklist verify: `story-policy-gate` ↔ interview §7.5 ↔ orchestrator admission согласованы (нет противоречия «личное = IRRELEVANT»).
- [x] Spam/troll/off-topic не ослаблены.
- [x] Явно **нет** AC «gateway mission AI reject обязателен на MVP».

## Границы

- **In scope:** requirements для mission standing + anti-gossip; deepen policy/interview/admission instructions.
- **Вне scope:** backend AI; алгоритм кластеризации; PII scrub (→ GPT-PII-01); pure junk placeholders (→ GPT-GUARD-01).
- **Смежно:** safety third-party PII — не заменяет standing (можно говорить про ребёнка без паспортных данных).

## Швы

- GPT: `story-policy-gate.md`, `story-interview-flow.md`, `safety-compliance.md`, `api-orchestrator.md` admission.
- Gateway: не MVP enforcement для standing.
- Related: GPT-GUARD-01, GPT-PII-01.
