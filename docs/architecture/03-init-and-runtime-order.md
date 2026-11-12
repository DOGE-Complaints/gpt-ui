# 03 — Инициализация и runtime-порядок до stash

**Зона:** [`architecture/`](./README.md)  
**As-of:** 2026-09-16  
**SSOT:** [`bootstrap.md`](../../instructions/bootstrap.md) · [`story-lifecycle-instructions.md`](../../instructions/story-lifecycle-instructions.md) · [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2

---

## 1. Точка инициализации сессии

1. **`root.md`** — активация hierarchy; bootstrap **до** functional routing.  
2. **`bootstrap.md`** — заполняет `comm_context` (`ui_lang`, tone, verbosity, presets) **до** functional modules.  
3. **`base.md`** — mode / constitution.  
4. Дальше — interview + strict chain ([01](./01-module1-instruction-layers.md)).

Без bootstrap session-language / tone контракты downstream (i18n, redirect copy) ломаются по инструкции.

---

## 2. Диалог → артефакты (без HTTP)

Упрощённый поток Module 1:

```text
story-interview-flow
  → ingest-validation (+ optional deep-parsing)
  → safety-compliance (may BLOCK)
  → story-policy-gate (admission)
  → story-normalizer → normalized_issue_payload
  → (user confirm in orchestrator §5.2.0b)
  → api-orchestrator maps → StoryDraftStashRequest
```

Граница Module 1 vs wire: lifecycle — Module 1 заканчивается нормализованным артефактом; user handoff Module 2 = stash request + SPA ([`story-lifecycle-instructions.md`](../../instructions/story-lifecycle-instructions.md)).

---

## 3. Stash и SPA (citizen path)

Единственный user path (`api-orchestrator.md` L242–250):

1. Guards §5.2.0a → §5.2.0 → §5.2.0b → §5.2.2 на **том же** теле, что уйдёт в stash.  
2. `postStoryDraftStash` / `POST /story-drafts` / Bearer `GPT_ACTIONS_BEARER_SECRET`.  
3. HTTP **201** → `data.draft_id`.  
4. Redirect:

```text
{SPA_BASE}/#/story/submit?draft_id=<draft_id>
```

5. **Browser** делает `POST /story-drafts/{draft_id}/submit` (GW-DRAFT-02) → **202**.  
6. GPT **не** ждёт submit и **не** утверждает «история создана» (§5.2.4 / L242).

`SPA_BASE` задаётся в ChatGPT Configure → Instructions (не в repo env файла orchestrator) — L270–276.

---

## 4. Канал B (bridge)

Тот же stash/SPA контракт; init/confirm живут в Python + n8n ([00](./00-two-channels-overview.md), aibridge confirm docs).  
Instruction delivery → [05](./05-python-bridge-instruction-delivery.md).
