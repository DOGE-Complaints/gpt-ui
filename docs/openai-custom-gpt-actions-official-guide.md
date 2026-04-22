# Custom GPT — Actions и авторизация (конспект официальной документации OpenAI)

## TODO: Точки трансформации под DOGEstonia

- [ ] Оставить как инфраструктурный референс; добавить приложение с DOGEstonia-конвенциями именования Action-операций.
- [ ] После утверждения OpenAPI ноды DOGEstonia добавить ссылку на канонический schema-файл для Issue API.
- [ ] Добавить короткую матрицу «какие Actions нужны MVP, какие отложены».

**Конвенция маркеров:** см. `DOGEstonia-doc-root-mirror-and-TODO-convention.md`.

**Назначение:** единая локальная справка для команды DOGEstonia при настройке **Actions** в Custom GPT.  
**Методология:** только опора на официальные материалы OpenAI; дата сверки **2026-03-28**.  
**Связь с проектом:** интеграция с Bot API — [task-implement-gpt-backend-activity-creation-integration](./analysis/tasks/task-implement-gpt-backend-activity-creation-integration/operator-deploy-checklist.md), SSOT `bot/docs/tech/api/api.md`.

**TODO (DOGEstonia-transform):** поддерживать ссылки на OpenAPI/чеклист **Issue** после появления ноды DOGEstonia (не удалять официальную часть документа).

---

## 1. Официальные источники (первичные)

| Тема | URL |
|------|-----|
| Введение в GPT Actions | https://platform.openai.com/docs/actions/introduction |
| Getting started (пример weather.gov, OpenAPI, тесты) | https://developers.openai.com/api/docs/actions/getting-started/ |
| Аутентификация Actions | https://developers.openai.com/api/docs/actions/authentication |
| Production (лимиты, TLS, IP, consequential) | https://developers.openai.com/api/docs/actions/production |
| Настройка Actions в GPT (Help Center) | https://help.openai.com/en/articles/9442513 |
| Создание и редактирование GPT | https://help.openai.com/en/articles/8554397 |
| Actions GPT (помощь со схемой) | https://chatgpt.com/g/g-TYEliDU6A-actionsgpt |
| Политики / приватность GPT | https://openai.com/policies/usage-policies , https://help.openai.com/en/articles/8554402-gpts-data-privacy-faqs |

---

## 2. Как устроены GPT Actions (официально)

- Custom GPT может подключать **внешние REST API** через Actions.
- В основе — **function calling**: модель (1) выбирает вызов, (2) формирует JSON-аргументы, (3) платформа выполняет HTTP-запрос к вашему API.
- Пользователь формулирует запрос на естественном языке; ответ снова превращается в текст в чате.
- Источник: [Introduction](https://platform.openai.com/docs/actions/introduction).

---

## 3. Важное ограничение: Apps **или** Actions

- В одном GPT можно использовать **либо** встроенные **Apps**, **либо** **Actions** — **не оба одновременно**.
- Источники: [Creating a GPT](https://help.openai.com/en/articles/8554397), [Configuring actions](https://help.openai.com/en/articles/9442513).

---

## 4. Настройка Action в редакторе GPT (пошагово по Help Center)

1. Редактор GPT → раздел **Actions** → **Create new action**.
2. Задать **аутентификацию** (см. §5).
3. Добавить **схему OpenAPI** (см. §6).
4. Протестировать в **Preview**; у каждого action есть кнопка **Test** с детальным вводом/выводом.
5. Для публичных GPT со Actions нужен **URL политики конфиденциальности** (Privacy Policy URL).

Источник: [Configuring actions in GPTs](https://help.openai.com/en/articles/9442513).

---

## 5. Аутентификация (официально)

По умолчанию для действий выбран режим **None**; тип можно сменить. Разные действия в одной схеме в общем случае описываются в рамках выбранного механизма (см. production про смешение схем).

### 5.1 None

- Без API-ключа и без OAuth.
- Рекомендуется рассматривать для первых шагов, чтобы снизить отток пользователей из‑за обязательного входа.
- Можно позже добавить отдельное действие с входом.

Источник: [Authentication](https://developers.openai.com/api/docs/actions/authentication).

### 5.2 API Key

- Для **сервер–сервер** сценариев.
- Секрет **шифруется** при хранении у OpenAI (по документации).
- В UI доступны варианты: **Basic**, **Bearer**, **Custom header** (конкретное имя заголовка задаётся в продукте при настройке).

Источники: [GPT Action authentication](https://developers.openai.com/api/docs/actions/authentication), [Configuring actions](https://help.openai.com/en/articles/9442513).

### 5.3 OAuth

- Вход **пользователя**; подходит для персонализированного доступа.
- В редакторе указываются: **Client ID**, **Client Secret**, **Authorization URL**, **Token URL**, **Scope** (и связанные настройки обмена токена).
- **Client Secret** хранится в зашифрованном виде; **Client ID** виден конечным пользователям (официальное предупреждение).
- В запросе к token endpoint участвуют в т.ч. `grant_type`, `code`, `redirect_uri`. Пример `redirect_uri` из документации:
  - `https://chat.openai.com/aip/{g-YOUR-GPT-ID-HERE}/oauth/callback`
  - также валиден: `https://chatgpt.com/aip/{g-YOUR-GPT-ID-HERE}/oauth/callback`
- Эти **redirect URL** должны быть зарегистрированы у провайдера OAuth (оба варианта домена, если используете оба).
- Требуется использование параметра **state** (безопасность).
- После успешного входа токен пользователя передаётся в заголовке **`Authorization`** в формате `Bearer` / `Basic` (как настроено).
- В документации приведён ожидаемый вид ответа с **access_token** (и опционально **refresh_token**, **expires_in**); детали flow — у провайдера и в вашей реализации token/authorization endpoints.

Источник: [Authentication](https://developers.openai.com/api/docs/actions/authentication).

---

## 6. OpenAPI-схема

- Формат: **OpenAPI 3.x** (JSON или YAML), стандарт как у Swagger/OpenAPI.
- Должны быть заданы: **servers** (базовый URL), **paths**, параметры, **operationId** (идентификаторы операций для модели).
- **Импорт:** вставить текст, импортировать **по URL** или начать с шаблонов (Weather JSON, Pet Store YAML, Blank) — см. Help Center.
- Имена и **descriptions** в схеме критичны: по ним модель решает, **какой** метод вызвать и **какие** аргументы заполнить.
- Ограничения на длину описаний — см. §7.2.

Рекомендация OpenAI: отлаживать вызовы в **Postman** (или аналоге) до отладки в ChatGPT.  
Источник: [Getting started](https://developers.openai.com/api/docs/actions/getting-started/).

---

## 7. Production и ограничения (официально)

### 7.1 Сеть и надёжность

- Только **HTTPS**, **TLS 1.2+**, порт **443**, валидный **публичный** сертификат.
- **Таймаут** запроса: до **45 секунд** round-trip.
- Рекомендуется **rate limiting**; ChatGPT учитывает **429** и частые **500** (backoff).

Источник: [Production notes](https://developers.openai.com/api/docs/actions/production).

### 7.2 Лимиты на текст и OpenAPI

- Тело запроса и ответа: каждое **до 100 000 символов**.
- Только **текст** (без картинок/видео в payload).
- В спецификации: до **300** символов на description/summary **endpoint**; до **700** символов на description **параметра** (лимиты могут меняться — смотреть актуальную страницу).

Источник: [Production notes](https://developers.openai.com/api/docs/actions/production).

### 7.3 IP-адреса исходящих вызовов

- Запросы к вашему API идут с IP из диапазонов из файла **chatgpt-connectors.json** (CIDR-блоки); список периодически обновляется. При необходимости — allowlist на стороне API.

URL файла в документации: https://openai.com/chatgpt-connectors.json  
Источник: [Production notes](https://developers.openai.com/api/docs/actions/production).

### 7.4 Флаг `x-openai-isConsequential`

- В OpenAPI можно пометить операцию: `x-openai-isConsequential: true`.
- При **true**: ChatGPT **всегда** запрашивает подтверждение пользователя перед вызовом; нет кнопки «always allow».
- При **false**: возможна «always allow».
- Если поле **не задано**: по умолчанию **GET** считаются несущественными (`false`), остальные методы — **true**.

Источник: [Production notes](https://developers.openai.com/api/docs/actions/production).

### 7.5 Критично: произвольные заголовки (custom headers)

Официально в [Production notes](https://developers.openai.com/api/docs/actions/production) указано:

> **Custom headers are not supported**

Это означает: **нельзя** полагаться на произвольный набор заголовков вроде отдельного `X-User-Id` **наряду** с тем, что описано в механизмах аутентификации Actions, так же как в типичных обсуждениях на форуме разработчиков описывают проблемы с header-параметрами в OpenAPI для Actions.

**Связь с Issue API:** текущий бекенд для `POST /issues/draft` может опираться на заголовок **`X-User-Id`** (см. `bot/docs/tech/api/api.md`). В среде **Custom GPT Actions** это **может быть недоступно** как отдельный заголовок. Варианты на уровне продукта (не из документации OpenAI): прокси, изменение API (например передача идентификатора в теле), использование **OAuth** с идентификатором в токене, или сценарий только с **`mock_user`** без стабильного user id. Детали — в операторском чеклисте таска BCK-1.

### 7.6 OAuth и домены

- Для OAuth (кроме перечисленных в доке исключений вроде Google, Microsoft, Adobe) **домены** в flow OAuth должны **совпадать** с доменом **основных** endpoint’ов схемы (официальное правило на странице Production).

Источник: [Production notes](https://developers.openai.com/api/docs/actions/production).

### 7.7 Enterprise / Education

- Workspace может **запретить** домены или **разрешить** только allowlist; при нуле разрешённых доменов Actions не выполняются.

Источники: [Configuring actions](https://help.openai.com/en/articles/9442513), [Managing GPT access](https://help.openai.com/en/articles/8555535).

---

## 8. Инструкции GPT (Instructions) и Actions

- В инструкциях обычно выделяют: **контекст**, **пошаговый сценарий** (с явными **operationId** / именами действий из схемы), **доп. замечания**.
- Плохие практики (официально в Production): провоцировать вызов action без запроса пользователя; заставлять пользователя говорить «yes» для триггера; возвращать из API длинный «разговорный» текст вместо сырых данных, если достаточно JSON.

Источники: [Getting started](https://developers.openai.com/api/docs/actions/getting-started/), [Production notes](https://developers.openai.com/api/docs/actions/production).

---

## 9. Типичная отладка (из Getting started)

| Симптом | Рекомендация OpenAI |
|--------|---------------------|
| Вызывается не тот метод или не вызывается | Уточнить descriptions в схеме; в инструкциях сослаться на имена действий |
| Неверные параметры | Уточнить descriptions параметров |
| Нет ясной ошибки | Кнопка **Test** у action; затем Postman с теми же auth/URL |
| Ошибка аутентификации | Проверить callback URL у OAuth; повторить настройки в Postman |

Источник: [Getting started](https://developers.openai.com/api/docs/actions/getting-started/).

---

## 10. Чеклист перед интеграцией с Issue API (кратко)

1. Публичный **HTTPS** API (не localhost с ноутбука без туннеля).
2. OpenAPI: **servers.url**, **operationId**, короткие **description** в лимитах.
3. Выбрать **None** / **API Key** / **OAuth** по модели угроз и UX.
4. Учесть **отсутствие custom headers** — спроектировать обход для `X-User-Id`, если нужен стабильный пользователь.
5. При OAuth — зарегистрировать **оба** redirect URL (`chat.openai.com` и `chatgpt.com`) при необходимости.
6. Для мутаций рассмотреть **`x-openai-isConsequential`**.
7. Опционально: allowlist IP из `chatgpt-connectors.json`.

---

## 11. Версионирование этого файла

| Версия | Дата | Изменение |
|--------|------|-----------|
| 1.0 | 2026-03-28 | Первый конспект по официальным страницам OpenAI (Introduction, Getting started, Authentication, Production, Help Center Creating GPT / Configuring actions). |

---

*Если официальные страницы изменятся, обновляйте этот файл и ссылки; расхождения с продуктом ChatGPT проверяйте по актуальной документации OpenAI.*

**Связь с архитектурой инструкций DOGEstonia:** концептуальный мост «Actions + OpenAPI + лимиты + как это стыкуется с Orchestrator» — [custom-gpt-architecture-principles-for-dialog-ingest-api.md](./custom-gpt-architecture-principles-for-dialog-ingest-api.md) §11–§16.
