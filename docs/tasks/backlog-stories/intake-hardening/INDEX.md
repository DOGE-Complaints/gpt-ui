# Intake hardening — package INDEX

**Зона:** `GPT UI/docs/tasks/backlog-stories/intake-hardening/`  
**Gap SSOT:** [`gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md`](../../analysis/gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md)  
**As-built:** [`story-pii-processing-before-send-as-built.md`](../../runtime-docs/story-pii-processing-before-send-as-built.md) · [`story-validation-and-garbage-filtering-as-built.md`](../../runtime-docs/story-validation-and-garbage-filtering-as-built.md)  
**Package candidate:** `pkg-000028` remains for PII (empty paths; **не** mutate). MISSION-01 via `pkg-000029`. TAX-02 via `pkg-000033`. GUARD-01 via `pkg-000034`.  
**Operator refine:** 2026-08-10 — PII GPT-only; TAX English labels; MISSION standing; GUARD hole inventory

## Context

Post-MVP hardening (requirements layer). MVP content/PII/standing — **GPT instructions**; gateway AI не для PII; junk — GPT primary + optional cheap non-LLM heuristics.

**Cross-project:** gateway consumer AI **не** для PII MVP; TAX vocabulary reject **не** требуется. Не переоткрывать Done `taxonomy-contract` / `story-submit-handoff`. REQ-21 Deferred ≠ GPT-PII-01.

## Stories

| Order | Key | Story | Status |
|------|-----|-------|--------|
| 1 | GPT-MISSION-01 | [Standing, personal stories, anti-gossip](./STORY-GPT-MISSION-01-mission-aligned-content-cleanliness.md) | Done |
| 2 | GPT-TAX-02 | [Axes lockstep + English label values](./STORY-GPT-TAX-02-taxonomy-label-key-sync.md) | Done |
| 3 | GPT-GUARD-01 | [Zero junk — holes H1–H8](./STORY-GPT-GUARD-01-zero-junk-throughput.md) | Done |
| 4 | GPT-PII-01 | [Zero PII on wire — GPT-only MVP](./STORY-GPT-PII-01-zero-pii-on-wire.md) | Done |

## Progress

| Metric | Value |
|--------|-------|
| Stories | 4 |
| Done | 4 |
| Todo | 0 |
| **Progress** | **4/4 (100%)** |
