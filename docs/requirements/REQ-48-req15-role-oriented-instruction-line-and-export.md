# REQ-48: REQ15 — Role-oriented instruction line, mapping, export/parity

Parent: [docs/requirements backlog/REQ15-DOGEstonia-Interview-DeepSeek-And-Telegram-Channel.md](<../../../docs/requirements backlog/REQ15-DOGEstonia-Interview-DeepSeek-And-Telegram-Channel.md>) §1, §3 L13, L17–L20, §4 O6, §6, AC-09, AC-11–14  
Parent-id: REQ15-interview-deepseek-telegram  
Siblings:
- [doge-ai-bridge REQ-13](../../../doge-ai-bridge/docs/requirements/REQ-13-REQ15-SEMANTIC-STATE-ROLES-INSTRUCTION-ASSEMBLY.md) (contract: per-role assembly from `DOGESTONIA_INSTRUCTIONS_DIR` + manifest / volume; **consume parity Pass**; **§3b export target** below)
- [doge-ai-bridge REQ-11](../../../doge-ai-bridge/docs/requirements/REQ-11-REQ15-DUAL-CONFIRM-GATEWAY-SUBMIT-SPA-HANDOFF.md) (stash + SPA continuation **in server code**, not in instruction prose)
- [doge-ai-bridge REQ-12](../../../doge-ai-bridge/docs/requirements/REQ-12-REQ15-INTERVIEW-DEEPSEEK-OPENAI-PROVIDERS.md) (no instruction authorship)
Status: Draft — awaiting PA.2 · **target tech-arch Present**  
Дата: 2026-10-31 · cascade refresh: 2026-11-02  
Locks: hybrid **C**; parity **both** sides; operator lock B — this REQ owns instruction **content** work in GPT UI  
**Disambiguation:** **GPT UI REQ-48** (this file, REQ15 role-oriented instructions) **≠** gateway [`48-early-signal-pre-cluster-data-readiness.md`](../../../doge-complaints-gateway/docs/requirements/48-early-signal-pre-cluster-data-readiness.md) (Early Signal program — different product)  
**Stitch audit:** [`docs/analysis/req15-aibridge-telegram-gpt48-stitch-audit-2026-11-02.md`](../../../docs/analysis/req15-aibridge-telegram-gpt48-stitch-audit-2026-11-02.md)  
**Architecture:** [`../architecture/req-48-role-oriented-instruction-line/00-overview.md`](../architecture/req-48-role-oriented-instruction-line/00-overview.md)  
**Interview report:** [`../analysis/zeya888.target-tech-arch-interview-req-48-role-oriented-instruction-line-2026-11-02.md`](../analysis/zeya888.target-tech-arch-interview-req-48-role-oriented-instruction-line-2026-11-02.md)  
**Backlog package:** [`../tasks/backlog-stories/req-48-role-oriented-instruction-line/INDEX.md`](../tasks/backlog-stories/req-48-role-oriented-instruction-line/INDEX.md) (GPT-ROL-01…04)

---

## Целевое состояние (простыми словами)

Это требование **проекта GPT UI**: здесь живут и правятся тексты инструкций (скиллы, диалоги, Module-1). Python aibridge их только читает с полки — не мастерская.

**Что делаем по шагам:**

1. **Архивируем текущие инструкции** — рабочее дерево `instructions/` (as-was, включая отработанный Custom GPT-процесс) копируем/переносим в явный архив **внутри GPT UI**, чтобы старая версия оставалась доступной для сверки и параллельной работы, пока готовится новая. Ничего ценного не затираем.
2. **Составляем карту (mapping):** каждый актуальный модуль → общий core / конкретная роль / только архив / уходит в серверный код bridge (stash и SPA-ссылка больше не в тексте инструкций).
3. **Пишем новую линию инструкций** под роли REQ15: настройка общения, интервьюер, экстрактор, нормализатор, checker — плюс общий shared core. Одна «книга» Custom GPT сменяется наборами текстов **под каждую роль**.
4. **Выпускаем export** (layout + **roles-schema** manifest) для `DOGESTONIA_INSTRUCTIONS_DIR` / volume aibridge и подтверждаем **export parity** (ничего не потеряли относительно задуманного набора).

Итог для стори-пакета: архив + mapping + новая role-oriented линия + export. Архивация — обязательный первый результат, не «мимоходом».

---

## 1) Goal

1. **Archive** current GPT UI `instructions/` (as-was) for parallel access and baseline — first deliverable in this project.  
2. **Mapping** from Module-1 files → shared / role(s) / archive-only / server-only (REQ-11 owns stash/redirect).  
3. **Author** role-oriented instruction line (shared core + five v1 roles) for aibridge assembly.  
4. **Strip** model-owned gateway stash + SPA redirect from the new line.  
5. **Export** contract for hybrid **C** + **export parity Pass** (roles-schema — §3b).

## 2) Scope / Out of scope

**In scope (story weight)**

- Inventory of active `GPT UI/instructions/` Module-1 + active pack files used by interview.  
- **Mapping table** (as-was file/module → shared | role(s) | archive-only | server-only) — deliverable for PA.2/implement; must cover methodology preservation intent (parent AC-11).  
- **New instruction line:** shared core + modules aligned to five roles (display → ids in §3b).  
- Role templates state purpose, bounds, result shape expectations in prose (runtime registry remains aibridge REQ-13).  
- Remove/relocate **POST /story-drafts + SPA redirect** duties from instruction prose → aibridge REQ-11.  
- Export layout for `DOGESTONIA_INSTRUCTIONS_DIR` + `DOGESTONIA_INSTRUCTIONS_MANIFEST` matching **§3b** (not Phase-A `files[]` as SSOT of the new line).  
- **Export parity Pass** vs intended new-line inventory.  
- Operator docs: archive vs new line; publish-to-volume (v1 may be manual).

**Prerequisite (required, not the epic headline)**

- Archive move/copy of **instruction content only** so Custom GPT / as-was process remains readable in parallel.

**Out of scope**

- Python role coordinator, provider adapters, dual-confirm tokens, gateway clients (aibridge REQ-11/12/13).  
- Implementing per-role **handlers** in code (REQ-13) — this REQ owns **texts/templates and mapping**, not Python.  
- Changing gateway/spa stash contracts.  
- Archiving Actions OpenAPI / ChatGPT Configure blobs (unless later expanded).  
- Sense-brief / poster worker content.  
- Telegram transport.

## 3) Verified current state (cascade 2026-11-02)

| Fact | Path / note |
|------|-------------|
| Module-1 SSOT (as-was) | `GPT UI/instructions/` · [`instruction-modules-index.md`](../../instructions/instruction-modules-index.md) |
| One-personality chain | ingest → safety → policy → normalizer → api-orchestrator (+ interview-flow, packs) |
| Model-owned stash + SPA redirect | [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2 / §5.2.B — must become **server-only** in new line |
| Channel path (aibridge) | Still single assembled string / Keep `InterviewEngine` on `/v1/channel/turns` — **not** per-role orchestration yet |
| Role registry + assembly (harness) | **Present** in aibridge REQ-13 (STORY-62…64): `roles/registry.py`, `roles/assembly.py`, fixture [`roles.manifest.json`](../../../doge-ai-bridge/tests/fixtures/content/roles.manifest.json) |
| ASGI / channel wire for roles + semantic state | **Absent / WAIVED** (REQ-13 G-02) — blocking live AC-06/09, **not** blocking GPT-48 authoring under §3b |
| Phase-A content load | `content.py` + `files[]` manifest — **legacy/transitional** readiness; **not** SSOT of the new export line |
| Sync GPT UI → aibridge | Script **not found**; v1 may be manual volume upload |
| This REQ export bodies / archive tree / mapping table | **Absent** (name ≠ create until PA.2 / implement ordered) |

### 3b) Export consumer contract (locked · copy from aibridge REQ-13 §3b)

**Do not invent** alternate layout. Authoring and export must match this table.

| Contract | Value |
|----------|--------|
| Env root | `DOGESTONIA_INSTRUCTIONS_DIR` |
| Manifest env | `DOGESTONIA_INSTRUCTIONS_MANIFEST` — after REQ15 cutover points at **roles-schema** manifest (not Phase-A `files[]` alone) |
| Roles schema | JSON object: `bundle_version` (string) · `core` (filename) · `roles` (object) |
| Role ids (exact) | `communication_setup` · `interviewer` · `extractor` · `normalizer` · `checker` |
| Display → id | Communication setup → `communication_setup`; Interviewer → `interviewer`; Extractor → `extractor`; Normalizer → `normalizer`; Checker → `checker` |
| Per role | `{ "file": "<root-child.md>", "version": "<pin>" }` — both required strings |
| Path rule | `core` and every `roles.*.file` = **non-empty root children** of the instructions dir — no `/`, `\`, `.`, `..`, absolute paths (enforced in aibridge `roles/assembly.py`) |
| Phase-A `files[]` | Remains for **legacy/transitional** readiness until ASGI role wire; **not** SSOT of this REQ’s new line |
| Dual schema | Concurrent `files[]` vs roles-schema on one env = **wire residual** (blocking live channel orchestration), **not** authoring ambiguity if export follows this table |
| Forbidden in new texts | Gateway stash URL choice, `POST /story-drafts`, SPA redirect procedure (peer REQ-11) — Checker **prose** only; submit stays code |
| Parity | **Export Pass** (this REQ) ≠ **Consume Pass** (aibridge REQ-13 AC-05); inventory/hash wire format = joint Open — do not invent here |
| Consume reference fixture | [`doge-ai-bridge/tests/fixtures/content/roles.manifest.json`](../../../doge-ai-bridge/tests/fixtures/content/roles.manifest.json) + `roles-instructions/*.md` |

## 4) Target behavior

1. **Archive first** (prerequisite): as-was tree reachable; not deleted.  
2. **Mapping Present:** every active interview-relevant module classified; no silent drop of methodology without explicit archive-only or server-only decision.  
3. **New line:** role-callable instruction sets + shared core; coordinator/server owns submit (REQ-11).  
4. **Export:** versioned **roles-schema** bundle for local dir and Railway volume (§3b); exact folder names under GPT UI = PA.2 — do not invent paths on disk here.  
5. **Parity (export):** inventory/hash Pass before content Done.  
6. Authoring skills stay in GPT UI — not in aibridge Python package.

## 5) Acceptance criteria

| ID | Criterion |
|----|-----------|
| **AC-GPT-48-01** | **Mapping table** Present: as-was → shared / role(s) / archive-only / server-only; covers active interview Module-1 + active pack used in runtime |
| **AC-GPT-48-02** | **New role-oriented line** Present for all five v1 roles + shared core (ids §3b); documented as REQ15 authoring SSOT |
| **AC-GPT-48-03** | New line does **not** require the model to perform gateway stash or SPA redirect |
| **AC-GPT-48-04** | Archive of pre-cutover instruction tree Present and reachable (prerequisite) |
| **AC-GPT-48-05** | Export layout documented and produceable for `DOGESTONIA_INSTRUCTIONS_DIR` + `DOGESTONIA_INSTRUCTIONS_MANIFEST` as **roles-schema** (§3b) — Phase-A `files[]` is **not** the new-line SSOT |
| **AC-GPT-48-06** | **Export parity Pass** evidence: released set matches intended new-line inventory (no silent subset) |
| **AC-GPT-48-07** | Operator note: archive vs new line; publish-to-volume steps (v1 may be manual SSH/upload) |

## 6) Non-goals / Open

- Exact archive directory name and new-line folder layout — PA.2 / implement (name ≠ create until ordered).  
- Whether Custom GPT Actions channel stays long-term — Open; archive enables continuity.  
- Consume-side fail-closed — aibridge REQ-13.  
- Joint inventory/hash format with consume Pass — Open (do not invent in GPT UI alone).  
- Merge/split of roles after latency measurement — parent allows later; v1 keeps five.  
- ASGI role wire / live channel orchestration — aibridge residual; does not block authoring under §3b.  
- Stitch mismatches (Bearer names, SPA delivery, dual schema): [`req15-aibridge-telegram-gpt48-stitch-audit-2026-11-02.md`](../../../docs/analysis/req15-aibridge-telegram-gpt48-stitch-audit-2026-11-02.md).
