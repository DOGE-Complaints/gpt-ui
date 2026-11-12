# STORY-GPT-PB-11 — Primary prompt: enforce load-gate knobs before emit

## Meta
- **Key:** `STORY-GPT-PB-11-primary-prompt-enforce-load-gate-emit`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) · pkg-000060 GIM-414…419 🟢 P3 story-only 2026-09-06T11:37:37Z
- **Приоритет:** P1 — primary всё ещё порождает pack.json, который meta отвергает на том же классе дыр
- **Тип:** product / Pack Builder primary prompt — **только постановка** (Layer 1)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md)
- **Prior story:** [`STORY-GPT-PB-10`](./STORY-GPT-PB-10-primary-prompt-elicit-load-gates.md) (Done story-only — тело primary **не** доведено)
- **Evidence (instance):** live `pack:validate` на `uus_veerenni_civic`/`v2` → pack.json INVALID (readiness / civic lenses / min_size_by_lens); payload + taxonomy + prose PASS
- **Skill declared (future P3):** `prompt-engineer` · `openai-custom-gpt-builder`
- **Depends:** GPT-PB-01 (primary on disk); GPT-PB-06…09 (meta + checklist уже кодируют gates); soft GPT-PB-10 (intent)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Delivery class:** **requirements / story only** — в этой волне **не** править `pack-builder-primary-prompt.md`

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

Meta и checklist уже умеют **отклонять** неполный `pack.json` (exact lenses без readiness, civic clustering без нужных полей / с «своими» lens id вместо официального набора).

Но **primary prompt** на диске по-прежнему ведёт discovery и emit так, что оператор может получить «красивый» flat-пак (prose + payload + taxonomy ок), а `pack.json` снова провалит первую валидацию. Это повторилось на `uus_veerenni_civic`/`v2`: валидатор честно сказал FAIL только по pack.json; остальное PASS.

GPT-PB-10 описал нужное поведение (elicit load gates до emit), но волна была **story-only** — тело [`pack-builder-primary-prompt.md`](../../../../instructions/node-onboarding/pack-builder-primary-prompt.md) **не** содержит явных обязанностей про readiness / civic required keys / официальные civic lens ids / запрет emit без них (verified: в primary нет этих строк; readiness emit = общий §6 без load-gate knobs).

Нужна **новая** итерация на **правку primary prompt**: чтобы Pack Builder **не выпускал** финальный pack.json без этих knobs — иначе оператор снова упрётся в repair после первой валидации.

## Проблема (факт)

| Наблюдение | Evidence |
|------------|----------|
| Primary §2.2 clustering — общий one-liner; нет elicit readiness / civic load-gate knobs | `pack-builder-primary-prompt.md` §2 / §6 |
| Meta уже отвергает неполный exact/civic | GPT-PB-06…09 Done · `pack-builder-pack.schema.json` |
| Новый emit `uus`/`v2`: prose+payload+taxonomy PASS; pack.json INVALID тем же классом | live `npm run pack:validate -- --id uus_veerenni_civic --ver v2` · flat `schema-packs.uus_veerenni_civic.v2.*` |
| PB-10 Done без правки primary body | INDEX + PB-10 Delivery class story-only |

## Scope

В **будущем P3** (не в этой постановке) доработать **только** Pack Builder primary:

1. **Discovery:** явно спрашивать / собирать knobs, без которых meta режет pack.json для нод с clustering (readiness на exact lenses; полный civic block включая `min_size_by_lens` и `readiness_threshold`; civic `active_lenses` / `primary_lens` только из официального ClusterLens-набора standards).
2. **Emit gate:** MUST NOT финальный emit `pack.json`, пока эти knobs не собраны (для civic / clustering нод).
3. **Ориентир оператору:** ссылка на checklist §5 / pack meta как «зачем спрашиваем» — без переписывания meta.
4. Сохранить ортогональность Module-1 и flat six-file emit (REQ-45a).

Формулировки — языком Pack Builder / operator interview, не копировать чужие runtime-спеки в текст primary.

## Out of scope



## Verified current state

| Fact | Evidence |
|------|----------|
| Primary on disk, version header v1.0-style readiness без load-gate knobs | `GPT UI/instructions/node-onboarding/pack-builder-primary-prompt.md` |
| PB-10 intent Done; body primary не обновлён под AC PB-10 | `STORY-GPT-PB-10-…` Delivery class · grep primary |
| Standards отвергают неполный pack.json | `pack-builder-pack.schema.json` · checklist §5 |
| v2 instance: тот же fail-класс на pack.json | `schema-packs.uus_veerenni_civic.v2.pack.json` + live validate |

## Target / Acceptance Criteria (stubs)

- [ ] **AC-GPT-PB11-01:** В primary явно описан discovery load-gate knobs (exact readiness + civic required fields + ClusterLens ids) до emit.
- [ ] **AC-GPT-PB11-02:** Primary MUST NOT разрешать финальный emit `pack.json` для clustering/civic ноды без этих knobs.
- [ ] **AC-GPT-PB11-03:** Readiness §6 / topic checklist обновлены так, что оператор видит связь «спросили → иначе meta FAIL» (без правки meta bodies).
- [ ] **AC-GPT-PB11-04:** Нет Part B instance repair; нет Module-1 edits; нет ослабления standards.

## Dependencies

- Hard: primary on disk (PB-01); gates in meta + checklist (PB-06…09).
- Soft: PB-10 (intent / AC map — реализовать в primary body этой story).
- Soft: SCR-NODEONB-01 — механическая проверка после emit.

## Related links

| Artifact | Role |
|----------|------|
| `pack-builder-primary-prompt.md` | целевой файл будущего P3 |
| GPT-PB-10 | prior intent (story-only) |
| `pack-builder-overlays.checklist.md` §5 | advisory load gates для формулировок discovery |
| `pack-builder-pack.schema.json` | что meta уже отвергает |
| uus v2 flat set + `pack:validate` | свежий пример того же класса gaps |
