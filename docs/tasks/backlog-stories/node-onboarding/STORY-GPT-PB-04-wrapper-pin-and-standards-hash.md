# STORY-GPT-PB-04 — Pack Builder wrapper + pin / standards hash (+ README)

## Meta
- **Key:** `STORY-GPT-PB-04-wrapper-pin-and-standards-hash`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (pkg-000054 · GIM-372…376 🟢 · P3 2026-09-03T18:08:15Z · Awaiting Commits)
- **Приоритет:** P1
- **Тип:** product / synthetic wrapper + tooling README
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.0 wrapper · README · §6 · AC-01/02/05/07/08/12
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** soft GPT-PB-01 (primary to pin); soft GPT-PB-03 (standards to bind)
- **Blocks:** —

## Зачем простыми словами

Тонкий **wrapper**: fetch/pin remote primary + bind validation standards, зафиксировать `prompt_version_hash` / `standards_set_hash`. Полный Pack Builder текст — в primary (PB-01), не здесь.

**README** (бывшая отдельная «папка»-стори) — короткий индекс siblings + bans: не instance pack в `node-onboarding/`; emit SSOT = REQ-45a; ортогональность Module-1. **Не** empty stubs «чтобы path существовал».

## Целевые файлы

```text
GPT UI/instructions/node-onboarding/
  pack-builder-wrapper.md
  pack-builder-README.md
```

## Scope

- T01: Wrapper — pin primary URL/tag; bind PB-03 standards; hash recording.
- T02: Operator flow REQ-46 §6.1–6.4 (obtain → pin → dialogue → emit → validate → repair via PB-02).
- T03: README — Builder vs interview; sibling index; AC-05 ban; consumer of REQ-45a.
- T04: Wrapper **does not** embed full primary body.

## Out of scope

- Primary/repair/schema bodies (PB-01…03); app hosting of raw URLs (REQ-APP-04 F1); empty placeholders for missing siblings.

## Verified current state

| Fact | Evidence |
|------|----------|
| wrapper / README | **Present** — P3 2026-09-03T18:08:15Z |
| Product flow | REQ4 §5.1; REQ-46 §1, §6 |

## Target / Acceptance Criteria

- [x] **AC-GPT-PB04-01:** `pack-builder-wrapper.md` exists; duties distinct from primary/repair (AC-02).
- [x] **AC-GPT-PB04-02:** `prompt_version_hash` required; `standards_set_hash` recommended (AC-08).
- [x] **AC-GPT-PB04-03:** `pack-builder-README.md` — orthogonality + REQ-45a emit SSOT + ban instance under `node-onboarding/` (AC-01/05/07/12).
- [x] **AC-GPT-PB04-04:** No empty sibling stubs created by this story.

## Точки в docs (P3)

| Action | Path |
|--------|------|
| CREATE | `…/pack-builder-wrapper.md` |
| CREATE | `…/pack-builder-README.md` |

## Dependencies

- Soft: PB-01, PB-03 present for pin/bind; PB-02 for loop docs in §6.
