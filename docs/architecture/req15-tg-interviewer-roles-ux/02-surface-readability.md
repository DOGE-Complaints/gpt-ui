# Channel interview surface readability (ROL-06)

**Parent:** [00-overview.md](./00-overview.md)  
**Story:** [STORY-GPT-ROL-06](../../tasks/backlog-stories/req15-tg-interviewer-roles-ux/STORY-GPT-ROL-06-channel-interview-surface-readability.md)  
**As-of:** 2026-11-06T11:51:22Z

## Target style rules (instruction layer)

| Rule | Meaning |
|------|---------|
| Air | Short paragraphs; blank lines between blocks; avoid dense bullet walls. |
| Markers | Prefer **light emoji** and orange-ish Unicode list markers (e.g. 🔸) over heavy `**` star-bold spam. |
| TG orange honesty | Telegram **cannot** apply CSS brand orange — markers are best-effort Unicode/emoji only. |
| Emoji budget | Avoid overload — at most one small marker cluster per section. |
| Behavior fence | Do **not** change stage/handoff policy (ROL-05) except where formatting touches the same lines. |

## Edit order

- Prefer **after** ROL-05 Apply on shared handoff paragraphs.  
- Soft-parallel OK if editors do not co-edit the same handoff sentences.

## Touch set (implement)

1. `instructions-roles/core.md` — short TG surface style block (shared).  
2. `instructions-roles/interviewer.md` — citizen-facing scannable reply guidance.  
3. Other role files — only if they emit long packed walls in channel turns.

## Verify (implement gate)

- Style rules Present in core (air / markers / orange honesty / emoji budget).  
- Regression: ROL-05 / stash bans in core still Present after style edits.  
- AC-ROL-06-01…04 checklist Pass.

## Out of scope

- Stage proactivity / vague-UI / stash bans policy authorship (ROL-05).  
- Runtime parse-mode / HTML coloring in telegram service.  
- Language picker (STORY-69).
