# STORY-GPT-SSR-17 — Retire `activity-legacy-paths-inventory` from Custom GPT upload set

## Meta
- **Key:** `STORY-GPT-SSR-17-retire-activity-legacy-paths-inventory`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md) (Wave 5 — Instruction upload hygiene)
- **Status:** Done (Awaiting Commits) · P3 Layer 1+2 2026-09-08T11:27:34Z · product retire deferred
- **Приоритет:** P1 — Custom GPT file-count limit; inventory занимает слот без runtime-ценности
- **Тип:** product / instruction upload surface + soft link cleanup
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) (Module-1 Issue-first) · layout sibling [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md) (flat upload pressure)
- **Trigger:** operator Custom GPT create — file count limit; inventory is editor checklist, not ingest module
- **Skill declared (future P3):** `openai-custom-gpt-builder`
- **Depends:** Wave 4 Done (SSR-11…13); soft: Issue-first cleanup already in core (2026-04-20)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Delivery class (эта волна):** **story Layer 1+2 only** — не move/delete inventory, не править 4 link sites здесь

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

При сборке Custom GPT Instructions упираются в **лимит числа файлов**.  
[`activity-legacy-paths-inventory.md`](../../../../instructions/activity-legacy-paths-inventory.md) лежит рядом с runtime modules, но это **не** шаг Issue-цепочки: это короткий **editor checklist** «не вернуть donor Activity routes / `normalized_activity_payload`».

Нужно **убрать файл из upload-набора** так, чтобы:

1. Runtime ingest (lifecycle → policy → normalizer → orchestrator) **не** изменился.
2. Четыре soft-ссылки на inventory **не** стали битыми в загруженном GPT.
3. Editor discipline checklist **не** исчезла из git (переезд в docs / archive).

## Что делает файл сейчас (as-built)

| Секция | Содержание |
|--------|------------|
| Issue-first edit checklist | 9 пунктов merge-gate для авторов instructions |
| Canonical chain refs | lifecycle / normalizer / policy / story-api-methods-reference |
| High-risk files table | `base`, `ingest-validation`, `api-orchestrator`, `instruction-modules-index` |

Размер ~41 строка. **Нет** правил диалога, HTTP, schema pack, geo, taxonomy.

## Почему безопасно для runtime

| Факт | Evidence |
|------|----------|
| Не в strict-chain | [`story-lifecycle-instructions.md`](../../../../instructions/story-lifecycle-instructions.md) §2.1: validation → safety → policy → **`normalized_issue_payload`** → API — inventory не назван |
| Не в `root.md` routing | grep `activity-legacy` in `root.md` = empty |
| Не в required-knowledge upload narrative | [`docs/required-knowledge-systems.md`](../../../required-knowledge-systems.md) не перечисляет inventory |
| Terminal artifact уже Issue | lifecycle / base / normalizer: `normalized_issue_payload` (не activity) |
| Donor HTTP уже historical в orchestrator | `api-orchestrator.md` L19 + L83 / L756 — donor removed; inventory только «inventory:» soft cite |

## Проблема (upload)

| Наблюдение | Evidence |
|------------|----------|
| Inventory в `instructions/` рядом с live modules | path under `GPT UI/instructions/` |
| Ровно **4** inbound markdown links | `base.md` L7; `ingest-validation.md` L7; `api-orchestrator.md` L19; `instruction-modules-index.md` L11 |
| Index changelog упоминает inventory | `instruction-modules-index.md` L77 (0.35) — historical row only |
| Flat pack wave уже давит file budget | REQ-45a / SSR-11: six-file pack + core |

Если просто **не загрузить** inventory, но оставить ссылки — GPT откроет битый path / «file not found» при follow-link. Нужна **self-contained** donor one-liner в 4 файлах **или** перенос checklist в docs + обновление ссылок на docs (docs обычно **не** upload).

## Scope

1. **Перенести** `activity-legacy-paths-inventory.md` из `instructions/` → `GPT UI/docs/analysis/` (или `docs/archive/instructions/`) как human SSOT checklist.
2. **В 4 файлах** заменить link на inventory на **самодостаточный** donor/Issue one-liner (без зависимости от upload file).
3. **Operator note:** Custom GPT upload set **MUST NOT** include retired inventory; git retains checklist under docs.
4. Soft: bump changelog row on `instruction-modules-index.md` when pointer changes.
5. Verify grep: zero live links to `activity-legacy-paths-inventory.md` under `instructions/` after P3.

## Out of scope

- Rewrite donor wording elsewhere in core (bulk purge).
- Pack Builder / PB-13; uus pack; OpenAPI; gateway.
- Creating parallel inventory stubs «чтобы path существовал» в Knowledge.
- Deleting checklist content without docs home (lose editor discipline).

## Target / Acceptance Criteria (постановка)

- [ ] **AC-GPT-SSR17-01:** Inventory **не** требуется в Custom GPT Instructions/Knowledge upload set.
- [ ] **AC-GPT-SSR17-02:** Все 4 former link sites self-contained (no broken relative link to missing upload file).
- [ ] **AC-GPT-SSR17-03:** Checklist content preserved under `GPT UI/docs/**` (analysis or archive).
- [ ] **AC-GPT-SSR17-04:** Strict-chain / `normalized_issue_payload` / orchestrator-only HTTP **unchanged** in behavior text.
- [ ] **AC-GPT-SSR17-05:** `rg activity-legacy-paths-inventory instructions/` = 0 (or only git-history notes outside live upload files).

## Dependencies

- Soft: Issue-first cleanup 2026-04-20 already removed donor bodies (base Purpose).
- Soft: SSR-11…13 flat pack pressure (motivation for file budget).
- Parallel: other Wave 5 stories (SSR-16 Pack Builder meta) — orthogonal.

## Related

| Artifact | Role |
|----------|------|
| `activity-legacy-paths-inventory.md` | Retire target (upload) / move to docs |
| `base.md` / `ingest-validation.md` / `api-orchestrator.md` / `instruction-modules-index.md` | 4 link sites |
| `story-lifecycle-instructions.md` | Runtime SSOT chain (must stay authoritative) |
| Prior plan notes | Custom GPT file-count trigger; editor-only role |

---

# Layer 2 — Техническая архитектура и субтаски (P3 later)

## 0. Contours

```text
                    ┌─────────────────────────────┐
  Custom GPT upload │ core story-* + orchestrator │  ← MUST
                    │ + schema-packs.* ×6 + README│
                    │ + base / ingest / index     │
                    └─────────────┬───────────────┘
                                  │ today soft cite
                                  ▼
                    ┌─────────────────────────────┐
                    │ activity-legacy-paths-      │  ← DROP from upload
                    │ inventory.md (editor only)  │
                    └─────────────────────────────┘
                                  │ after P3
                                  ▼
                    ┌─────────────────────────────┐
  Git human SSOT    │ GPT UI/docs/analysis/…      │  ← KEEP checklist
                    │ (or docs/archive/…)         │
                    └─────────────────────────────┘
```

## 1. Checklist vs already-in-core (do not re-litigate at P3)

| Inventory checklist item | Already enforced in runtime docs? | Evidence |
|--------------------------|-----------------------------------|----------|
| 1. Lifecycle = story-lifecycle | Yes | lifecycle file + index Canonical table |
| 2. `normalized_issue_payload` | Yes | lifecycle §2.1 / base / normalizer |
| 3. HTTP from story-api-methods-reference | Yes | orchestrator lockstep cites |
| 4. `/activities` historical only | Yes (partial) | orchestrator L19 donor guardrail |
| 5. policy before normalizer | Yes | lifecycle strict-chain |
| 6. orchestrator-only HTTP | Yes | orchestrator role + lifecycle |
| 7. index Issue-first pointer | Yes | index L9–11 (minus inventory link) |
| 8. Issue domain terms in examples | Discipline | editor; not a separate file duty |
| 9. "not for Issue ingest" label | Pattern in orchestrator | keep in one-liners |

**Thesis:** Retire does **not** require inventing new process rules — only **relocate** checklist + **inline** the one sentence still useful for GPT when it reads base/ingest/orch/index.

## 2. Exact edit map (P3)

| ID | Action | Path |
|----|--------|------|
| T01 | Re-verify grep links = 4 sites only | workspace `GPT UI/` |
| T02 | Copy inventory body → docs home | e.g. `GPT UI/docs/analysis/activity-legacy-paths-inventory-editor-checklist.md` (+ header «not Custom GPT upload») |
| T03 | `base.md` L7 | Replace `(see inventory)` with self-contained Issue-first / donor-historical sentence |
| T04 | `ingest-validation.md` L7 | Same — drop `(guardrails: inventory)` |
| T05 | `api-orchestrator.md` L19 | Keep donor `/activities` note; drop `(inventory: …)` cite |
| T06 | `instruction-modules-index.md` L11 | Point editors to docs checklist path **or** inline; changelog bump |
| T07 | Remove/move file out of `instructions/` | no upload sibling left |
| T08 | Operator smoke | Upload set without inventory; God Mode short path still reaches stash mapping without looking for inventory |
| T09 | Out-of-scope guard | No OpenAPI / pack / Module-1 process rewrite beyond 4 lines + move |

**Locked default for T02 destination:** [`GPT UI/docs/analysis/`](../../../analysis/) — create file there (folder may be empty today; SSR INDEX already cites `GPT UI/analysis/` sibling paths). Prefer **move** (git mv) over delete so history stays clear.

**Locked default for link replacement text (copy at P3):**

> Donor `/activities` routes and `normalized_activity_payload` are historical only and not executable for DOGEstonia Issue ingest. Active terminal artifact is `normalized_issue_payload`; HTTP SSOT is `story-api-methods-reference.md`. Editor checklist (human): `docs/analysis/activity-legacy-paths-inventory-editor-checklist.md` — **do not upload** to Custom GPT.

(Index may shorten the last sentence to avoid forcing docs into GPT context.)

## 3. Risk register

| Risk | Mitigation |
|------|------------|
| Broken relative links in uploaded GPT | T03–T06 self-contained before T07 remove |
| Editors lose checklist | T02 move to docs with clear title |
| Accidental re-upload of old path | Operator manual / story AC; optional note in `schema-pack-custom-model-operator-manual-ru.md` only if upload inventory lists inventory (verified: manual §0 does not list it) |
| Over-edit core process | Touch only cited lines + move |

## 4. Acceptance Criteria (refined)

- [ ] **AC-GPT-SSR17-01:** Custom GPT upload set documented without inventory; file absent from `instructions/` live tree (or redirect stub ≤5 lines saying «moved to docs — do not upload» — **prefer full move, no stub**, to save the file slot).
- [ ] **AC-GPT-SSR17-02:** Four former sites compile donor/Issue rule without inventory path.
- [ ] **AC-GPT-SSR17-03:** Checklist 9 items + high-risk table live under `GPT UI/docs/analysis/…`.
- [ ] **AC-GPT-SSR17-04:** No change to lifecycle order or artifact names beyond link hygiene.
- [ ] **AC-GPT-SSR17-05:** Grep clean under `instructions/` for `activity-legacy-paths-inventory`.

## 5. Anti-patterns

| Anti-pattern | Why fail |
|--------------|----------|
| Delete inventory without docs home | Lose editor SSOT |
| Leave stubs in `instructions/` «for path» | Still consumes upload slot / confuses operators |
| Paste full 9-item checklist into all 4 files | Token bloat; duplicate SSOT |
| Expand gateway enums / fix packs in this story | Wrong package |

## Claims hygiene

| Claim | Evidence |
|-------|----------|
| Inventory ~41 lines editor checklist | file body L1–41 |
| Exactly 4 instruction links | grep `activity-legacy-paths-inventory` under `GPT UI/` |
| Not in lifecycle strict-chain | `story-lifecycle-instructions.md` §2.1 |
| File-count trigger | operator report (product constraint); REQ-45a flat already multiplies files |
| Prefer move to docs/analysis | this story Layer2 T02 locked default |

No instruction body / move in this постановка wave.
