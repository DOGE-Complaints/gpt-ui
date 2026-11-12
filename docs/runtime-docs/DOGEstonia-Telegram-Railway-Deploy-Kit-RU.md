# DOGEstonia Telegram Bot — Railway deploy kit

Готовая схема без форка и без собственного репозитория:

`Telegram → готовый bot-wrapper в Railway → OpenAI-compatible LLM endpoint`

## 1. Что будет развернуто

- Open-source wrapper: [n3d1117/chatgpt-telegram-bot](https://github.com/n3d1117/chatgpt-telegram-bot)
- Готовый Docker image: `n3d1117/chatgpt-telegram-bot:latest`
- Registry: [Docker Hub](https://hub.docker.com/r/n3d1117/chatgpt-telegram-bot)
- Лицензия: GPL-2.0
- Соединение с Telegram: long polling. Публичный домен, webhook и переменная `PORT` не нужны.
- LLM endpoint должен поддерживать OpenAI-compatible Chat Completions API (`/v1/chat/completions`).

## 2. Сначала создать Telegram-бота через BotFather

1. Открыть только официальный аккаунт: [@BotFather](https://t.me/BotFather).
2. Нажать **Start** или отправить `/start`.
3. Отправить `/newbot`.
4. Ввести отображаемое имя, например: `DOGEstonia — Uus Veerenni`.
5. Ввести уникальный username, обязательно заканчивающийся на `bot`, например: `uus_veerenni_doge_bot`.
6. BotFather пришлет токен вида `123456789:AA...`. Скопировать его. Это значение переменной `TELEGRAM_BOT_TOKEN` в Railway.

Важно: имя, username, описание и картинка не записываются в `.env`. Из BotFather в Railway нужен только токен.

### Брендинг через BotFather — без изменения кода

Отправить `/mybots` → выбрать нового бота → **Edit Bot**:

- **Edit Name** — название;
- **Edit Description** — описание на стартовом экране;
- **Edit About** — короткий текст профиля;
- **Edit Botpic** — квадратный логотип DOGEstonia;
- **Edit Commands** — вставить:

```text
start - Start conversation
help - Help
reset - Start a new conversation
stats - Usage statistics
```

Для бота только в личных сообщениях: `/setjoingroups` → выбрать бота → **Disable**.

Если токен случайно опубликован: в BotFather выполнить `/revoke`, получить новый токен и заменить `TELEGRAM_BOT_TOKEN` в Railway.

## 3. Получить свой Telegram user ID

Он нужен для `ADMIN_USER_IDS` и закрытого пилота. Открыть [@getidsbot](https://t.me/getidsbot), нажать **Start** и скопировать числовой `User ID`.

## 4. Создать сервис в Railway

1. Открыть [Railway — New Project](https://railway.com/new).
2. Выбрать **Deploy a Docker Image**. Если такого пункта нет: создать **Empty Service**, открыть **Settings → Service Source → Docker Image**.
3. Вставить image:

```text
n3d1117/chatgpt-telegram-bot:latest
```

4. Открыть сервис → **Variables** → **Raw Editor**.
5. Вставить блок из следующего раздела, заменить все `CHANGE_ME_...` и применить изменения.
6. Нажать **Deploy** или **Redeploy**.

Railway официально поддерживает развертывание публичных образов Docker Hub напрямую: [Railway Services documentation](https://docs.railway.com/services#deploying-a-public-docker-image).

## 5. Переменные Railway — готовый блок

В Railway раздел **Variables** заменяет `.env`: отдельный файл загружать не нужно.

```dotenv
# Telegram
TELEGRAM_BOT_TOKEN=CHANGE_ME_TOKEN_FROM_BOTFATHER
ADMIN_USER_IDS=CHANGE_ME_YOUR_NUMERIC_TELEGRAM_ID
ALLOWED_TELEGRAM_USER_IDS=CHANGE_ME_YOUR_NUMERIC_TELEGRAM_ID

# Remote LLM: OpenAI-compatible Chat Completions
OPENAI_API_KEY=CHANGE_ME_REMOTE_LLM_API_KEY
OPENAI_BASE_URL=https://CHANGE_ME_LLM_HOST/v1
OPENAI_MODEL=CHANGE_ME_EXACT_MODEL_ID

# Instructions / behavior
ASSISTANT_PROMPT=CHANGE_ME_PASTE_THE_FULL_SYSTEM_INSTRUCTIONS_HERE
BOT_LANGUAGE=en
TEMPERATURE=0.2
MAX_TOKENS=1200
MAX_HISTORY_SIZE=30
MAX_CONVERSATION_AGE_MINUTES=1440
STREAM=true
SHOW_USAGE=false
N_CHOICES=1
PRESENCE_PENALTY=0.0
FREQUENCY_PENALTY=0.0

# Keep the wrapper text-only and predictable
ENABLE_QUOTING=true
ENABLE_IMAGE_GENERATION=false
ENABLE_TRANSCRIPTION=false
ENABLE_TTS_GENERATION=false
ENABLE_VISION=false
ENABLE_FUNCTIONS=false
```

### Что именно заменить

| Переменная | Значение |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Токен, который выдал BotFather после `/newbot` |
| `ADMIN_USER_IDS` | Твой числовой Telegram user ID |
| `ALLOWED_TELEGRAM_USER_IDS` | На пилоте — твой ID; для публичного доступа позднее поставить `*` |
| `OPENAI_API_KEY` | Ключ удаленного LLM-сервиса |
| `OPENAI_BASE_URL` | Базовый URL OpenAI-compatible API, обычно заканчивается на `/v1` |
| `OPENAI_MODEL` | Точный model ID, который принимает удаленный сервис |
| `ASSISTANT_PROMPT` | Вся система инструкций; Railway принимает длинное и многострочное значение |

Не добавлять `PORT`, `WEBHOOK_URL` или Railway Domain: этот контейнер использует Telegram long polling.

## 6. Если подключать OpenAI напрямую

Использовать:

```dotenv
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_API_KEY=CHANGE_ME_OPENAI_PLATFORM_PROJECT_KEY
OPENAI_MODEL=CHANGE_ME_OPENAI_MODEL_ID
```

- Ключ: [OpenAI Platform API keys](https://platform.openai.com/api-keys)
- API billing: [OpenAI Platform billing](https://platform.openai.com/settings/organization/billing/overview)

Подписка ChatGPT Team/Business сама по себе не является API-ключом и не подставляется в бота. Для прямого OpenAI API нужен ключ проекта на Platform и настроенный там API billing.

## 7. Где держать систему инструкций

Выбрать один источник истины:

- Если удаленный LLM endpoint уже всегда применяет нужные инструкции, поставить короткое нейтральное значение `ASSISTANT_PROMPT`, чтобы не дублировать правила.
- Если endpoint — обычная модель без закрепленных правил, вставить полную систему инструкций в `ASSISTANT_PROMPT`.

При последующих изменениях: Railway → сервис → **Variables** → изменить `ASSISTANT_PROMPT` → **Deploy staged changes**. Форк и пересборка image не нужны.

## 8. Проверка после deploy

1. В Railway открыть **Deployments → View Logs**.
2. Убедиться, что процесс запустился без traceback и продолжает работать.
3. Открыть `https://t.me/USERNAME_БОТА`.
4. Нажать **Start** и отправить тестовое сообщение.
5. Отправить `/reset`, затем пройти короткий тестовый диалог заново.

### Быстрая диагностика

| Симптом в логах/боте | Что проверить |
|---|---|
| `InvalidToken` / бот не стартует | `TELEGRAM_BOT_TOKEN`, без пробелов и кавычек |
| `Unauthorized` / `403` от LLM | `OPENAI_API_KEY` |
| `404`, model not found | `OPENAI_BASE_URL` и точный `OPENAI_MODEL` |
| Бот отвечает «нет доступа» | ID в `ALLOWED_TELEGRAM_USER_IDS` |
| `Conflict: terminated by other getUpdates request` | Бот запущен еще где-то; оставить один Railway instance/replica |
| Бот отвечает как обычный ассистент | Проверить `ASSISTANT_PROMPT` и отсутствие дублирующих инструкций на endpoint |

## 9. Перевод из пилота в публичный режим

После проверки заменить:

```dotenv
ALLOWED_TELEGRAM_USER_IDS=*
```

Затем применить staged changes. До публичного открытия желательно поставить расходный лимит у самого LLM-провайдера: встроенный денежный счетчик этого wrapper может быть неточным для кастомной модели.

## 10. Все нужные URL в одном месте

- Railway new project: https://railway.com/new
- Railway dashboard: https://railway.com/dashboard
- Railway Docker image docs: https://docs.railway.com/services#deploying-a-public-docker-image
- GitHub source: https://github.com/n3d1117/chatgpt-telegram-bot
- Docker Hub image: https://hub.docker.com/r/n3d1117/chatgpt-telegram-bot
- Official BotFather: https://t.me/BotFather
- Telegram bot-token tutorial: https://core.telegram.org/bots/tutorial#obtain-your-bot-token
- Telegram ID helper: https://t.me/getidsbot
- OpenAI API keys: https://platform.openai.com/api-keys
- OpenAI API billing: https://platform.openai.com/settings/organization/billing/overview

---

Verified against the repository README and `.env.example`, plus Railway public Docker image documentation, on 2026-09-08.
