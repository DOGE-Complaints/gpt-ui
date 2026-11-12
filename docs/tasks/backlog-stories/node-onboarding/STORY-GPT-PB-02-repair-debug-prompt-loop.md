# STORY-GPT-PB-02 — Repair / debug prompt + validation fail loop

## Meta
- **Key:** `STORY-GPT-PB-02-repair-debug-prompt-loop`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1
- **Тип:** product / repair prompt template
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.0 repair · §5.1.6 · §6.3–4 · AC-02
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-PB-01
- **Blocks:** —

## Зачем простыми словами

При fail валидации оператор вставляет repair/debug prompt (ошибки + инструкции) обратно в Pack Builder dialogue; primary переиздаёт **тот же flat file set**. Candidate loop — **не** замена app/gateway federation gate (REQ4 F2–F5 / REQ-APP-04).

## Целевой файл

```text
GPT UI/instructions/node-onboarding/pack-builder-repair-debug-prompt.md
```

## Scope

- T01: Author repair template — slots for validator errors, standards path refs (PB-03), regenerate flat REQ-45a set.
- T02: Document loop with primary (PB-01): fail → repair → regenerate → re-validate (wrapper flow finalized in PB-04).
- T03: Explicit non-replacement of dogestonia-app / gateway admission.

## Out of scope

- App validator UI; gateway SchemaRuntime; Module-1 interview edits; meta-schema bodies (PB-03).

## Verified current state

| Fact | Evidence |
|------|----------|
| repair file | **Present** — `GPT UI/instructions/node-onboarding/pack-builder-repair-debug-prompt.md` (P3 2026-09-03T17:46:23Z) |
| Loop intent | REQ-46 §1 item 4; §6.3–4 |

## Target / Acceptance Criteria

- [x] **AC-GPT-PB02-01:** Repair prompt file exists; distinct from primary/wrapper (AC-GPT-REQ46-02).
- [x] **AC-GPT-PB02-02:** Loop regenerates flat `schema-packs.*` only; no Module-1 process edits.
- [x] **AC-GPT-PB02-03:** Explicit: does not replace app/gateway gates.

## Точки в docs (P3)

| Action | Path |
|--------|------|
| CREATE | `…/pack-builder-repair-debug-prompt.md` |

## Dependencies

- GPT-PB-01.
- Soft: PB-03 standards path names in template slots.
