# Required Knowledge Systems (DOGEstonia Issue)

## Purpose

Этот документ фиксирует рабочий минимум знаний для команды, которая поддерживает Issue-first контур Custom GPT и интеграцию с нодой/SPA.

## Required blocks

- Архитектура модулей: `root` → `base` → `safety` → `issue-policy-gate` → `issue-normalizer` → `api-orchestrator`.
- Контракт данных: `story-data-model.md` и требования в `docs/requirements`.
- Контракт API: `story-api-methods-reference.md` и импортированный Actions OpenAPI.
- Политика допуска: `story-policy-gate.md` + операторский rulebook.
- Практики безопасности и приватности: `safety-compliance.md`.

## Operational rule

Любые примеры, чеклисты и интеграционные заметки в `docs/` должны использовать только терминологию Issue-контура и актуальные `issues/*` контракты.
