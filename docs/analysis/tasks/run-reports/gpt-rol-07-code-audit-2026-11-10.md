# P4 Code Audit — STORY-GPT-ROL-07 (citizen surface: no role theater)

**Дата:** 2026-11-10  
**UTC verify:** 2026-11-10T12:37:02Z  
**Метод:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — claims с путями; findings only; RO (no product edits)  
**builder_project:** gpt  
**Объект (backlog):** [`STORY-GPT-ROL-07-citizen-surface-no-role-theater.md`](../../../tasks/backlog-stories/req15-tg-interviewer-roles-ux/STORY-GPT-ROL-07-citizen-surface-no-role-theater.md)  
**Pipeline:** [`…/STORY-GPT-ROL-07-….md`](../epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-07-citizen-surface-no-role-theater/STORY-GPT-ROL-07-citizen-surface-no-role-theater.md)  
**Volume:** [`GPT UI/instructions-roles/`](../../../instructions-roles/) (alias `GPT-UI/instructions-roles/`)  
**P3 claim:** GIM-480…486 🟢 · UTC `2026-11-10T12:35:03Z` · [`run-summary-20261110-1235-gpt-rol-07-p3-execute.md`](./run-summary-20261110-1235-gpt-rol-07-p3-execute.md)  
**Pkg:** [`pkg-000070-…yaml`](../gpt-active-packages/pkg-000070-20261110-gpt-rol-07-citizen-surface-no-role-theater.yaml) · pointer [`gpt-active-package.current.yaml`](../gpt-active-package.current.yaml)  
**Scope:** Findings only. Write = this audit path. Phases not advanced. Do not invent AC.

**Вердикт:** vs-AC **4/4 PASS** (AC-ROL-07-01…04).  
**OPEN blocking:** **0**.  
**OPEN residual (story-accepted):** soft-dep **STORY-AIBRIDGE-71** / cite **72** (ownership Present; do not invent CTAs).  
**WAVE COMPLETE:** **yes**.  
**Grep stash:** **CLEAN**.

---

## 1. vs-AC (story ACs only)

| AC | Итог | Verified path / fact |
|----|------|----------------------|
| **AC-ROL-07-01** | **PASS** | [`core.md`](../../../instructions-roles/core.md) §Citizen surface covenant — forbid operable extractor/checker/normalizer/HANDOFF/«следующий процесс» · [`interviewer.md`](../../../instructions-roles/interviewer.md) L22–23 · **Citizen fence** on extractor/checker/normalizer/communication_setup |
| **AC-ROL-07-02** | **PASS** | core covenant item 2 + interviewer L24 — forbid fake handoff/validation/send/«передал» without server CTA/FSM advance · Ownership rows STORY-71/72 · empty actions → no theater substitute |
| **AC-ROL-07-03** | **PASS** | core covenant item 3 + interviewer L25 — forbid free-text command school («валидируй/запускай/передай/отправляй») as **primary** when CTAs are the path · MAY point to Present CTAs only |
| **AC-ROL-07-04** | **PASS** | core L20 + Ownership Forbidden stash/SPA/Bearer · interviewer stash ban · live `rg` **GREP_STASH_CLEAN** (no `POST /story-drafts` / `postStoryDraftStash` / `#/story/submit`) |

Gate: [`story-acceptance-gate.md`](../epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-07-citizen-surface-no-role-theater/story-acceptance-gate.md) PASS · T01 audit note Present · task READMEs GIM-480…486 🟢.

---

## 2. Fact verify (product)

| Check | Result |
|-------|--------|
| Citizen covenant in core | **Y** — dual-surface + MUST NOT ×3 + MAY ×2 + Keep ROL-03/05/06 |
| interviewer no-theater utterances | **Y** — calm draft + bans; internal handoff row model-facing only |
| Fences on other roles | **Y** — 4/4 (extractor, checker, normalizer, communication_setup) |
| README ROL-07 + 71/72 ownership | **Y** |
| Peer STORY-71 / 72 backlog Present | **Y** |
| Theater strings as **prohibitions** only | **Y** (grep hits are covenant/fence bans, not coaching duties) |
| Stash invent bans | **Present** + GREP_STASH_CLEAN |
| ROL-05/06 retained | **Y** (auto-advance, style block Present) |

### Grep (re-run)

```text
rg -n 'POST /story-drafts|postStoryDraftStash|#/story/submit' "GPT UI/instructions-roles/"
→ GREP_STASH_CLEAN

rg -n 'Citizen surface covenant|STORY-AIBRIDGE-71|do not invent CTAs' "GPT UI/instructions-roles/"
→ hits (covenant + ownership)
```

---

## 3. OPEN — blocking vs residual

### Blocking GAP

| ID | Severity | Finding |
|----|----------|---------|
| — | — | **none** |

### Story-accepted residual Open (non-blocking)

| Item | Severity | Fate |
|------|----------|------|
| Soft-dep **STORY-AIBRIDGE-71** / cite **72** runtime CTAs/Send FSM | residual Open | Ownership sentence Present in core + README; roles must not invent CTAs — does **not** fail AC-01…04 |

### Not OPEN (Info)

| Item | Fate |
|------|------|
| Separate `acceptance-verification-GIM-480…486.md` files Absent | Gate + run-summary + README Done checkboxes Present; product evidence on volume |
| Some task README «Code Facts» still say covenant Absent (scaffold) | Stale vs P3 product; does not fail product AC |

---

## 4. Регрессии

| Инвариант | Состояние |
|-----------|-----------|
| ROL-03 stash/SPA invent bans | **intact** |
| ROL-05 auto-advance / STORY-68 | **intact** |
| ROL-06 TG surface style | **intact** |
| Dual-surface (model may name roles) | **ok** — internal tables allowed; citizen theater forbidden |

---

## 5. Counts

| Metric | Value |
|--------|-------|
| vs-AC PASS | **4/4** |
| vs-AC FAIL | **0** |
| OPEN blocking | **0** |
| OPEN residual (story-accepted) | **1** (STORY-71/72 soft-dep) |
| WAVE COMPLETE | **yes** |
| Grep stash CLEAN | **Y** |

---

## 6. Handoff (OP fence)

VAL wrote audit. Phases **not** advanced. No product edits. Residual 71/72 stays peer Open. Next: P8 / further residual UX as operator directs.
