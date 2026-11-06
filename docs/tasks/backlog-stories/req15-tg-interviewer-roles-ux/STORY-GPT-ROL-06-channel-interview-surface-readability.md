# STORY-GPT-ROL-06 — Channel interview surface readability

## Meta
- **Key:** `STORY-GPT-ROL-06-channel-interview-surface-readability`
- **Status:** Done (Awaiting Commits)
- **P3:** `2026-11-06T12:10:14Z` · GIM-475…479 🟢
- **Pipeline (pkg-000069):** [`STORY-GPT-ROL-06 pipeline`](../../../analysis/tasks/epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-06-channel-interview-surface-readability/STORY-GPT-ROL-06-channel-interview-surface-readability.md)
- **Package:** [`INDEX.md`](INDEX.md) · `req15-tg-interviewer-roles-ux`
- **Parent:** peer REQ-48 roles line residual UX
- **Architecture:** [`../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [`02-surface-readability.md`](../../../architecture/req15-tg-interviewer-roles-ux/02-surface-readability.md)
- **Evidence:** gap Goals 5–6 · [triage](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-triage-2026-11-06.md) · [target UX](../../../../../docs/analysis/req15-tg-interviewer-target-ux-2026-11-06.md)
- **Soft-depends:** after/parallel [STORY-GPT-ROL-05](STORY-GPT-ROL-05-channel-interview-stage-proactivity.md) (avoid co-editing same handoff paragraphs)
- **Volume:** `GPT-UI/instructions-roles/` citizen-facing guidance (+ short style rules in `core.md`)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Emitted:** 2026-11-06 · **Tech deepen:** 2026-11-06T11:51:22Z
- **Supersedes:** root accumulative batch **surface** todos (air / markers)

## Scope (requirements)

1. Roles **MUST** instruct sparse, scannable Telegram replies: short paragraphs, blank lines between blocks, avoid dense bullet walls.
2. Prefer **light emoji** and orange-ish Unicode markers for lists over heavy `**` star-bold spam.
3. Document honesty: Telegram **cannot** apply CSS brand orange — markers are best-effort Unicode/emoji only.
4. Emoji budget: avoid overload (e.g. at most one small marker cluster per section).
5. Do **not** change stage/handoff policy (that is ROL-05) except where formatting touches the same lines — prefer sequential edit after 05.

## Out of scope

- Stage proactivity / stash bans / vague-UI policy — **STORY-GPT-ROL-05**.  
- Runtime code; language picker; ops logging.  
- Applying instruction body patches in this deepen wave (name≠create until Apply/P3).

## Target technical architecture

SSOT: [`architecture/req15-tg-interviewer-roles-ux/`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [`02-surface-readability.md`](../../../architecture/req15-tg-interviewer-roles-ux/02-surface-readability.md).

- Style rules live in instruction prose (`core` + citizen-facing roles); TG has no CSS orange.
- Edit **after** ROL-05 on shared handoff lines; soft-parallel OK if no co-edit of those paragraphs.
- Must not regress ROL-05 / core stash bans.

## Acceptance (product)

| ID | Criterion |
|----|-----------|
| AC-ROL-06-01 | Style rules Present for paragraph air / blank lines vs packed walls |
| AC-ROL-06-02 | Preference for light markers/emoji over dense `**`; TG orange honesty noted |
| AC-ROL-06-03 | Emoji-load guidance Present (anti-spam) |
| AC-ROL-06-04 | No regression of ROL-05 / core stash bans introduced by style-only edits |

## Подзадачи (технические)

> **Delivery class этой волны:** постановка T01–T05 + tech-arch. **Не** патчить `instructions-roles/**` здесь — Apply/P3 в другом диалоге.

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | Add TG surface style block in **`core.md`**: air / blank lines; light markers (e.g. 🔸) over dense `**`; TG orange honesty; emoji budget ≤1 cluster/section | 01–03 | Style block Present in core |
| **T02** | Apply scannable reply guidance in **`interviewer.md`** (citizen-facing); other roles only if they emit long packed walls | 01,02 | interviewer.md updated; others N/A or touched |
| **T03** | **Sequential fence:** after/parallel ROL-05 — no rewrite of stage/handoff **behavior** except formatting on shared lines | 04 | Diff review: behavior unchanged |
| **T04** | **Grep/regression:** stash/SPA invent bans still Present; style prefers markers over dense `**` spam | 04 | Grep + spot-check Pass |
| **T05** | AC checklist AC-ROL-06-01…04 Pass | all | Gate recorded |

### Locked touch paths

| Path | Role |
|------|------|
| `GPT-UI/instructions-roles/core.md` | T01 |
| `GPT-UI/instructions-roles/interviewer.md` | T02 |
| Other role md | T02 optional |

## Verified current state (honesty)

| Fact | Status |
|------|--------|
| Pille bot turns | Dense `**` + long bullet blocks Observed |
| Roles volume | Present; TG style rules **Present** (P3) |

## Related

- Sibling: [STORY-GPT-ROL-05](STORY-GPT-ROL-05-channel-interview-stage-proactivity.md)  
- Arch: [00-overview](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [02](../../../architecture/req15-tg-interviewer-roles-ux/02-surface-readability.md)  
