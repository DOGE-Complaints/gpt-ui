# STORY-GPT-GUARD-01 — Zero junk throughput (дыры и MVP-швы)

## Meta
- **Key:** `STORY-GPT-GUARD-01-zero-junk-throughput`
- **Пакет:** [`intake-hardening/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** Critical — GAP-IH-03 (operator refine 2026-08-10: явный hole inventory)
- **Тип:** product requirement (admission / throughput guard)
- **Gap:** [`gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md`](../../../analysis/gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md) §3
- **Основание:** [REQ-30](../../../requirements/REQ-30-admission-gate-story-intake-strict-chain.md) §2.2 Done (GPT) + §2.4 SHOULD `INTAKE_NOT_SUBSTANTIVE` (**нет в gateway `src/`**); as-built validation
- **Pipeline / GIM:** GIM-233…237 (pkg-000034)

## Зачем простыми словами

GPT умеет отказывать в «мусоре», но есть конкретные **ходы**, которыми мусор всё ещё может оказаться в stash/БД. Эта стори перечисляет их и фиксирует, что закрываем на MVP (усиление GPT vs дешёвые non-AI эвристики на gateway). **Не** путать с GPT-PII-01: там backend AI запрещён; здесь LLM на сервере тоже не требуется.

## Текущее состояние (verified)

| Элемент | Реальность |
|---------|------------|
| GPT admission + junk rule | `api-orchestrator` §5.2.0a — Done (REQ-30) |
| Server substantive reject | **Нет** (`INTAKE_NOT_SUBSTANTIVE` — 0 в `src/`) |
| Server hard checks | non-empty / i18n / enums / geo |
| Post-create quality | `alpha_score` — **не** gate на create |
| As-built | validation as-built §§7–10 |

## Незакрытые ходы (Unclosed throughput holes)

| # | Ход | Почему дыра (факт) | MVP close |
|---|-----|--------------------|-----------|
| **H1** | Прямой schema-valid JSON → `POST /story-drafts` | Сервер не проверяет strict-chain artifacts / junk semantics | Residual **или** cheap **non-LLM** heuristic на gateway |
| **H2** | Submit уже застешанного junk | SPA `body: {}`; substance не пересматривается | Как H1 (содержимое stash) |
| **H3** | Placeholder title/desc («test», «asdf», «проверка») при non-empty | `_require_non_empty_string` проходит | GPT pre-flight + optional cheap keyword heuristic (no AI) |
| **H4** | `observation` / текст «нет реальной проблемы» | Schema OK; REQ-30 §2.4 не реализован | GPT admission substance + optional non-LLM server |
| **H5** | Модель пропускает/ослабляет admission artifacts | Instruction-only; сервер артефактов не видит | **Усилить GPT** (primary) |
| **H6** | Minimal assent («ок», «отправь тест»), если модель ошибается | §5.2.0a item 5 работает только при соблюдении | **Усилить GPT** |
| **H7** | Sandbox / authorized_tester exception mis-applied | §5.2.0a sandbox rules | **Усилить GPT** + явные STOP-копии |
| **H8** | Надеяться на `alpha_score` после persist | Score не reject на create | **Не** считать закрытием дыры |

## Требование / целевое состояние (MVP)

1. **Primary:** закрыть **H5–H7** усилением GPT admission / junk / confirmation (citizen Actions path).
2. **H1–H4:** либо documented residual на MVP, либо **дешёвые non-LLM** heuristics на gateway (placeholder/test keywords) — **без** серверного ИИ/токенов.
3. **H8** не использовать как substitute gate.
4. Классы отказа согласованы с GPT-MISSION-01 (standing/gossip) и не дублируют GPT-PII-01.
5. Sandbox / authorized tester — только при явных условиях REQ-30 §2.2; default = production reject junk.

## Acceptance Criteria (requirements)

- [x] Документ/инструкции закрывают H5–H7 с evidence (artifact missing → STOP; «отправь тест» → STOP; sandbox mis-use → STOP).
- [x] Таблица H1–H8 в AC evidence: для каждого — Closed (GPT) / Closed (cheap GW heuristic) / Residual documented.
- [x] Happy-path substantive personal/civic story → stash OK.
- [x] Явно: **нет** требования backend LLM для junk; отличие от GPT-PII-01 зафиксировано.
- [x] Не регрессирует Done GPT §5.2.0a без миграции.

## Границы

- **In scope:** hole inventory; GPT hardening; optional non-AI gateway heuristics as seam.
- **Вне scope:** server LLM; полный ML spam classifier; изменение Status Done REQ-30 файла (outcome в этой стори).
- **Не трогаем:** clustering `alpha_score` как create-gate; PII AI на backend.

## Швы

- GPT: `api-orchestrator.md` §5.2.0a / pre-flight; policy (mission classes → GPT-MISSION-01).
- Gateway: optional cheap heuristics only; **не** AI token path.
- Related: GPT-MISSION-01, GPT-PII-01, GPT-TAX-02.
