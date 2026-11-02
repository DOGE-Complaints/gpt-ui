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
| 4 | [STORY-GPT-ROL-04 — export roles-schema + parity + ops](./STORY-GPT-ROL-04-export-roles-schema-parity-ops.md) | Todo | GPT-ROL-03 |

**Progress:** 2/4 Done (50%) · **As-of:** 2026-11-02T13:25:46Z  
**Technical subtaks:** ROL-01 archive Present; ROL-02 mapping Present (`docs/analysis/req-48-module1-role-mapping.md`); ROL-03 sibling Present (`instructions-roles-req15/`); ROL-04 name ≠ create until P1.3/P3

## Граница

- GPT UI owns instruction **texts**, mapping, export layout/parity evidence, operator publish note.
- **OOS:** aibridge Python (REQ-11/12/13 handlers), gateway/SPA contracts, Telegram, Actions OpenAPI archive.
