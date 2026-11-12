# STORY-GPT-PB-03 — Validation meta-schemas + overlays checklist

## Meta
- **Key:** `STORY-GPT-PB-03-validation-meta-schemas-and-overlays-checklist`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (pkg-000053 · GIM-366…371 🟢 · P3 2026-09-03T18:00:24Z · Awaiting Commits)
- **Приоритет:** P1
- **Тип:** product / validation standards (doc + JSON Schema)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.0 schemas · checklist · Protocol · AC-03/09
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** —
- **Blocks:** GPT-PB-04 (bind standards)

## Зачем простыми словами

Primary должен генерировать то, что проходит checks. Meta-schemas для `pack.json` / `payload.schema.json` / `taxonomy.json` + checklist трёх prose identities (REQ-45a).

**Глубина (operator lock):** **минимальный tallinn exemplar**, но **полная структура верхних слоёв** (`field_policy`, `geo_intake`, `geo_model` / territory hooks, `exact_lenses`, `node_clustering`, `card_fields` optional, taxonomy axes/keys/map, three prose presence) — чтобы на gateway не достраивать крупные ветки с нуля.

## Целевые файлы

```text
GPT UI/instructions/node-onboarding/
  pack-builder-pack.schema.json
  pack-builder-payload-schema.schema.json
  pack-builder-taxonomy.schema.json
  pack-builder-overlays.checklist.md
```

## Scope

- T01–T03: Three meta-schemas — full top-level branch skeleton; tallinn-minimal examples.
- T04: Overlays checklist — MUST three identities (flat names REQ-45a); presence/shape, not empty prose.
- T05: Protocol — semantic align gateway; **gateway wins** on federation drift.

## Out of scope

- Deep gateway enrichment; invent wire names; primary/repair/wrapper bodies; runtime flat instances (SSR).

## Verified current state

| Fact | Evidence |
|------|----------|
| Meta-schemas / checklist | **Present** — four files under `instructions/node-onboarding/` (P3 2026-09-03T18:00:24Z) |
| Tallinn exemplar | Gateway `…/schema-packs/tallinn_civic/v1/`; GPT flat targets REQ-45a / SSR-11…13 |
| Three identities | REQ-45a §4; SSR-12 locale |

## Target / Acceptance Criteria

- [x] **AC-GPT-PB03-01:** Four standards files under `node-onboarding/` (AC-GPT-REQ46-03).
- [x] **AC-GPT-PB03-02:** Checklist covers **three** pack prose identities (AC-09).
- [x] **AC-GPT-PB03-03:** Full top-level structural branches; tallinn = minimal exemplar only.
- [x] **AC-GPT-PB03-04:** Gateway wins on drift; no Module-1 process edits.

## Точки в docs (P3)

| Action | Path |
|--------|------|
| CREATE | four files listed above |

## Dependencies

- Soft: SSR-11…13 Done for flat+locale example paths in checklist.
- Soft: PB-01/02 already reference these filenames.
