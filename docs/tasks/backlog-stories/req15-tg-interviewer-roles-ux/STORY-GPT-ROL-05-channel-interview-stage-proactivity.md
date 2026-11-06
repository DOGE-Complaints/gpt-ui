# STORY-GPT-ROL-05 — Channel interview stage proactivity + handoff wording

## Meta
- **Key:** `STORY-GPT-ROL-05-channel-interview-stage-proactivity`
- **Status:** Done (Awaiting Commits)
- **P3:** `2026-11-06T11:58:46Z` · GIM-468…474 🟢
- **Pipeline (pkg-000068):** [`STORY-GPT-ROL-05 pipeline`](../../../analysis/tasks/epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-05-channel-interview-stage-proactivity/STORY-GPT-ROL-05-channel-interview-stage-proactivity.md)
- **Package:** [`INDEX.md`](INDEX.md) · `req15-tg-interviewer-roles-ux`
- **Parent:** peer REQ-48 roles line (ROL-01…04 Done) · residual UX
- **Architecture:** [`../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [`01-stage-proactivity-handoff.md`](../../../architecture/req15-tg-interviewer-roles-ux/01-stage-proactivity-handoff.md)
- **Evidence:** gap Goals 1/2/4 prose · [triage](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-triage-2026-11-06.md) · [target UX](../../../../../docs/analysis/req15-tg-interviewer-target-ux-2026-11-06.md)
- **Soft-depends:** aibridge [`STORY-AIBRIDGE-68`](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-68-channel-seam-send-actions-always.md) (server CTAs)
- **Volume:** `GPT-UI/instructions-roles/` (`core.md`, `interviewer.md`, handoff phrases in `checker.md` / `normalizer.md` as needed)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Emitted:** 2026-11-06 · **Tech deepen:** 2026-11-06T11:51:22Z
- **Supersedes:** root [`req15-roles-interview-ux-prose-accum-batch`](../../../../../docs/analysis/req15-roles-interview-ux-prose-accum-batch.md) **behavior** todos (proactive / no-vague-ui / core-align)

## Scope (requirements)

1. Roles **MUST** instruct the model to **auto-advance** interview stages (intake → check → normalize → i18n prep) when enough citizen data is Present — without waiting for user prompts «дальше», «продолжай», «проверяй».
2. At stage boundaries, prose **MAY** acknowledge progress; it **MUST NOT** be the only path — wording **MUST** align with server-offered continue/edit/Send **actions** when those are Present (peer STORY-68).
3. Roles **MUST NOT** tell the user to use a vague «интерфейс» for submit without either naming in-chat actions or stating that after server Send confirm a SPA `continuation_url` may appear.
4. Roles **MUST** preserve ROL-03 / `core.md` bans: **no** model-owned gateway stash, **no** inventing SPA redirect URLs or Bearer/HTTP submit.
5. Touch set for implement: primarily `core.md` + `interviewer.md`; only handoff-related sentences in `checker.md` / `normalizer.md` if needed for consistency.

## Out of scope

- Surface formatting (air, emoji, `**`) — **STORY-GPT-ROL-06**.  
- Minting channel actions / aibridge/telegram code — STORY-68/69.  
- Schema packs, archive Module-1 rewrite, export parity.  
- Applying instruction body patches in this deepen wave (name≠create until Apply/P3).

## Target technical architecture

SSOT: [`architecture/req15-tg-interviewer-roles-ux/`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md).

- **Prose** owns auto-advance + handoff wording; **server** owns action mint + stash + `continuation_url` ([01](../../../architecture/req15-tg-interviewer-roles-ux/01-stage-proactivity-handoff.md)).
- Forbidden: model POST gateway; bare «интерфейс» without control; invent SPA/Bearer.
- Soft-dep STORY-68 for action presence; do not invent action schemas in roles files.
- Edit before ROL-06 on shared handoff paragraphs.

## Acceptance (product)

| ID | Criterion |
|----|-----------|
| AC-ROL-05-01 | Instructions require proactive stage advance without user coaching «дальше/проверяй» when data enough |
| AC-ROL-05-02 | Submit/handoff wording never relies on bare «интерфейс»; points to actions and/or post-stash URL behavior |
| AC-ROL-05-03 | `core.md` (or equivalent) still forbids model-owned stash and inventing redirect/submit HTTP |
| AC-ROL-05-04 | No contradiction with peer STORY-68 ownership (server mints actions; model does not POST gateway) |

## Подзадачи (технические)

> **Delivery class этой волны:** постановка T01–T07 + tech-arch. **Не** патчить `instructions-roles/**` здесь — Apply/P3 в другом диалоге.

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | **Audit** `instructions-roles/`: gaps vs target Goals 1/2/4 — missing auto-advance; any vague-UI / coaching-only handoff | — | Short audit note (gate / analysis) listing gaps |
| **T02** | Patch **`core.md`**: proactive stage constitution + handoff wording contract (actions and/or post-stash URL); **keep** stash/SPA invent bans | 01,03,04 | core.md updated; bans Present |
| **T03** | Patch **`interviewer.md`**: auto-advance dialogue phases without waiting for «дальше/проверяй/продолжай» when data enough | 01 | interviewer.md updated |
| **T04** | **If needed only:** align handoff sentences in `checker.md` / `normalizer.md` with AC-02/04 (no behavior invent beyond consistency) | 02,04 | Touched only if inconsistent; else N/A recorded |
| **T05** | Document wording alignment with **STORY-68** in role prose or README pointer: server mints actions; model does not POST gateway | 04 | Explicit ownership sentence Present |
| **T06** | **Grep gate** on volume: no model duty for `POST /story-drafts` / `postStoryDraftStash` / invent SPA redirect procedure; no bare «интерфейс» as sole handoff | 02,03 | Grep clean or findings fixed |
| **T07** | AC checklist AC-ROL-05-01…04 Pass | all | Gate recorded |

### Locked touch paths

| Path | Role |
|------|------|
| `GPT-UI/instructions-roles/core.md` | T02 |
| `GPT-UI/instructions-roles/interviewer.md` | T03 |
| `…/checker.md`, `…/normalizer.md` | T04 optional |

## Verified current state (honesty)

| Fact | Status |
|------|--------|
| Roles volume Present | `GPT-UI/instructions-roles/` (~7 md) |
| Strip stash in core | **Present** (ROL-03) |
| Proactivity / anti-vague-UI rules | **Present** (P3) |
| Pille session | User coached stages; bot refused send with vague UI |

## Related

- Sibling: [STORY-GPT-ROL-06](STORY-GPT-ROL-06-channel-interview-surface-readability.md)  
- Peer runtime: [STORY-AIBRIDGE-68](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-68-channel-seam-send-actions-always.md)  
- Arch: [00-overview](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [01](../../../architecture/req15-tg-interviewer-roles-ux/01-stage-proactivity-handoff.md)  
