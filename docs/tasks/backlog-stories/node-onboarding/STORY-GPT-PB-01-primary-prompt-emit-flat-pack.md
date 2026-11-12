# STORY-GPT-PB-01 — Primary Pack Builder prompt (NL → flat pack emit)

## Meta
- **Key:** `STORY-GPT-PB-01-primary-prompt-emit-flat-pack`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Приоритет:** P1 — first Pack Builder artifact
- **Тип:** product / primary Pack Builder Instructions body
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.1 · AC-04/06/09/10/11
- **Skill declared:** prompt-engineer · openai-custom-gpt-builder
- **Depends:** **assumes GPT-SSR-11…13 Done** (flat + locale reference); soft: GPT-PB-03 (standards bind by path name)
- **Blocks:** GPT-PB-02

## Зачем простыми словами

Primary — полный Pack Builder: NL-диалог с оператором ноды → candidate **flat** `schema-packs.*` (JSON + inbound + interview-overlay + locale-jurisdiction) + `prompt_version_hash`. Не Module-1 interview GPT. Директория `instructions/node-onboarding/` появляется как побочный эффект первого файла — **отдельной стори на mkdir нет**.

## Assumed executed runtime (operator-first)

```text
GPT UI/instructions/
  schema-packs.tallinn_civic.v1.pack.json
  schema-packs.tallinn_civic.v1.payload.schema.json
  schema-packs.tallinn_civic.v1.taxonomy.json
  schema-packs.tallinn_civic.v1.inbound-validation.md
  schema-packs.tallinn_civic.v1.interview-overlay.md
  schema-packs.tallinn_civic.v1.locale-jurisdiction.md
```

Primary **SHALL** use this as reference shape and support other `schema_id` / `schema_version` (not civic-only).

## Целевой файл

```text
GPT UI/instructions/node-onboarding/pack-builder-primary-prompt.md
```

## Scope

- T01: Implement REQ-46 §5.1.1–5.1.7 (role, inputs, discovery incl. geo a–f + three identities, emit table, MUST/MUST NOT, repair accept hook, readiness).
- T02: Emit mandate — flat names only; all three prose files required.
- T03: Reference validation standards paths under `node-onboarding/` (bodies in PB-03); generation must be checkable against them when present.
- T04: Invariants — SEC-001; no Module-1 rewrite; semantic `schema_version` ≠ envelope id; no invent wire names.

## Out of scope

- Meta-schema / checklist bodies (→ PB-03); wrapper/README (→ PB-04); repair template (→ PB-02).
- SSR-11…13 P3 (assumed Done before this P3).

## Verified current state

| Fact | Evidence |
|------|----------|
| primary file | **Present** — `GPT UI/instructions/node-onboarding/pack-builder-primary-prompt.md` (P3 2026-09-03) |
| `instructions/node-onboarding/` | **Present** (created with primary) |
| Emit SSOT | REQ-45a §4–§6 |
| SSR-11…13 | **Done** — flat ×6 + locale on disk |

## Target / Acceptance Criteria

- [x] **AC-GPT-PB01-01:** `pack-builder-primary-prompt.md` on disk implementing §5.1.
- [x] **AC-GPT-PB01-02:** Mandates flat emit + three prose identities (AC-GPT-REQ46-04/09/10).
- [x] **AC-GPT-PB01-03:** Aligns REQ-45a; no nested instance emit (AC-06).
- [x] **AC-GPT-PB01-04:** MUST NOT invent wire names; SEC-001 pack=data.
- [x] **AC-GPT-PB01-05:** Orthogonality to interview Instructions (AC-07).
- [x] **AC-GPT-PB01-06:** Reference = post-SSR-11…13 tallinn flat+locale; other nodes supported.

## Точки в docs (P3)

| Action | Path |
|--------|------|
| CREATE | `…/node-onboarding/pack-builder-primary-prompt.md` |

## Dependencies

- Prerequisite package: SSR-11 → SSR-12 → SSR-13 Done before P3.
- Soft: PB-03 standards files (named contract; created later in sequence).
