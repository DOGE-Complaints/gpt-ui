# REQ-48 — Role-oriented instruction line (backlog package)

**Зона:** `GPT UI/docs/tasks/backlog-stories/req-48-role-oriented-instruction-line/`  
**Parent REQ:** [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md)  
**Tech-arch:** [`../../../architecture/req-48-role-oriented-instruction-line/00-overview.md`](../../../architecture/req-48-role-oriented-instruction-line/00-overview.md)  
**Метод:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)  
**Skill:** `openai-custom-gpt-builder`

| Order | Story | Status | Depends |
|-------|-------|--------|---------|
| 1 | [STORY-GPT-ROL-01 — archive Module-1 snapshot](./STORY-GPT-ROL-01-archive-module1-snapshot.md) | Done (Awaiting Commits) | — |
| 2 | [STORY-GPT-ROL-02 — Module-1 role mapping](./STORY-GPT-ROL-02-module1-role-mapping.md) | Done (Awaiting Commits) | GPT-ROL-01 |
| 3 | [STORY-GPT-ROL-03 — author roles line + strip stash](./STORY-GPT-ROL-03-author-roles-line-strip-stash.md) | Done (Awaiting Commits) | GPT-ROL-02 |
| 4 | [STORY-GPT-ROL-04 — export roles-schema + parity + ops](./STORY-GPT-ROL-04-export-roles-schema-parity-ops.md) | Done (Awaiting Commits) | GPT-ROL-03 |

**Progress:** 4/4 Done (Awaiting Commits) · **As-of:** 2026-11-02T14:39:55Z  
**Technical subtaks:** ROL-01…04 product Present (archive · mapping · sibling · manifest/evidence/operator-note)

## Peer residual (do not reopen 01–04)

REQ15 TG interviewer roles UX (proactivity + surface): package [`../req15-tg-interviewer-roles-ux/INDEX.md`](../req15-tg-interviewer-roles-ux/INDEX.md) · **STORY-GPT-ROL-05** / **06** · Todo · requirements-only · [root triage](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-triage-2026-11-06.md).

## Граница

- GPT UI owns instruction **texts**, mapping, export layout/parity evidence, operator publish note.
- **OOS:** aibridge Python (REQ-11/12/13 handlers), gateway/SPA contracts, Telegram, Actions OpenAPI archive.
