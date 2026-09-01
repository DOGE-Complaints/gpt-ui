# GPT UI — Runtime Docs (as-built)

> **Назначение:** персистентная зона as-built документации Custom GPT / story intake для операторов, compliance и юридики.  
> **Не путать с** [`docs/analysis/`](../analysis/) (gitignore, working notes) и [`docs/requirements/`](../requirements/) (REQ-контракты / Planned).

## Правило достоверности

1. Утверждение о **текущем поведении** должно ссылаться на один из источников:
   - [`instructions/`](../../instructions/) (GPT instruction SSOT),
   - [`docs/custom-gpt-story-intake-actions.openapi.yaml`](../custom-gpt-story-intake-actions.openapi.yaml),
   - `doge-complaints-gateway/src/**`,
   - `spa-app/src/**` (где путь касается submit UI).
2. Текст REQ без реализации в `src` / без нормы в instructions → секция **Planned**, не Current.
3. Метод: [`.cursor/rules/analysis.mdc`](../../../.cursor/rules/analysis.mdc). Где поведения нет — писать «в коде нет».
4. Документы **не** являются юридическим заключением.

## Карта документов

| Документ | Вопрос |
|----------|--------|
| [schema-pack-custom-model-operator-manual-ru.md](./schema-pack-custom-model-operator-manual-ru.md) | Как настроить GPT под кастомную модель и правила валидации (swap двух pack-файлов; active pair; `schema_binding`) |
| [story-pii-processing-before-send-as-built.md](./story-pii-processing-before-send-as-built.md) | Что считается PII в story-path; есть ли очистка перед отправкой на сервер; что персистится; где `redact_pii`; scan set включая `geo_detail` |
| [story-validation-and-garbage-filtering-as-built.md](./story-validation-and-garbage-filtering-as-built.md) | По каким критериям история допускается; schema packs / `schema_binding`; как фильтруется «мусор»; что гарантирует gateway vs GPT |

Смежные as-built вне GPT UI:

- Identity phone / GDPR: [`doge-identity-service/docs/runtime-docs/10-phone-personal-data-processing.md`](../../../doge-identity-service/docs/runtime-docs/10-phone-personal-data-processing.md)
- Gateway story persistence: [`doge-complaints-gateway/docs/runtime-docs/story-persistence-model.md`](../../../doge-complaints-gateway/docs/runtime-docs/story-persistence-model.md)
