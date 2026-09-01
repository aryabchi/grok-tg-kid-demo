# Демо Grok Bot → Telegram Calendar Buddy

## 1. Предварительные условия — всё подготовлено до демо

**Цель:** заранее подготовить аккаунты и доступы, но самого ассистента и интеграции собрать непосредственно во время демо.

### A. Доступ к Cursor / Grok Bot

- Cursor Pro
- Grok Bot 0.30.0 on Windows
- Возможность создать нового Bot
- Доступ к cloud computer / terminal

### B. Google account ребёнка

Подготовьте только аккаунт и доступ, необходимые для Google OAuth:

- Google account существует и может использоваться для демо
- Пароль/учётные данные Google account доступны
- Если включены 2FA/passkey/другая проверка, во время демо должен быть доступен человек, который сможет её пройти
- Желательно, чтобы ребёнок присутствовал и мог пройти авторизацию аккаунта интерактивно
- Do **not** create the demo calendar beforehand
- Do **not** create the demo events beforehand
- Calendar и events будут созданы Grok во время демо
- Google Calendar plugin будет подключён вручную через **Grok Bot → Plugins**

### C. Telegram Bot

Подготовьте Telegram bot заранее:

- Создайте bot через **@BotFather**
- Выберите username, например `CalendarBuddyBot`
- Храните **bot token** в безопасном месте
- У ребёнка установлен Telegram на Android
- Ребёнок открывает bot и нажимает **Start** / отправляет сообщение

Token является секретом и **не должен** передаваться в обычном чате Grok Bot.

### D. Telegram destination

Для отправки private messages нужны следующие значения Telegram:

- **Bot token** — необходим для аутентификации bot
- **Child’s `chat_id`** — необходим для указания private destination

Отдельный числовой **Bot ID не требуется**. Bot ID можно получить из идентификатора bot через `getMe`, но отдельно настраивать его для отправки сообщений не нужно.

Рекомендуемая фиксированная конфигурация:

```text
TELEGRAM_BOT_TOKEN=<BotFather token>
TELEGRAM_DEFAULT_CHAT_ID=<kid chat id>
```

Token следует передать через secure secret/environment mechanism Grok Bot. `chat_id` не является секретом и может быть передан как configuration.

### E. Варианты Telegram MCP

Основной вариант:

- `telegram-api-mcp`

Запасной вариант:

- `@node2flow/telegram-bot-mcp`

Ни один из них не нужно устанавливать заранее: Grok может установить MCP server на своём cloud computer во время демо.

На случай проблемы совместимости MCP SDK/schema со штатным launcher держите под рукой уже известный workaround для `node2flow`.

### F. Материалы для демо

Подготовьте этот список как demo input, но не создавайте events заранее:

```text
Today 16:00       Football practice
Tomorrow 09:00   Science test
Tomorrow 15:30   Alex's birthday
Saturday 12:00   Grandma's birthday
Sunday 11:00     Family trip
```

---

# 2. Step-by-step demo

## Шаг 1 — Создать Bot

**[ВЫ]**

- Откройте Grok Bot.
- Создайте новый Bot.
- Назовите его **Calendar Buddy**.

**[В GROK BOT]**

> Ты мой персональный calendar assistant. Твоя задача — помогать мне понимать важные предстоящие events и отправлять полезные reminders в мой Telegram.

Пока не настраивайте routine.

---

## Шаг 2 — Подключить Google Calendar

**[ВЫ]**

В Grok Bot:

**Plugins → Google Calendar → Add**

Пройдите Google OAuth с помощью подготовленного Google account.

**[В GROK BOT]**

> Проверь, можешь ли ты получить доступ к моему Google Calendar. Пока ничего не изменяй. Скажи мне, можешь ли ты читать calendar events.

Это ручная настройка plugin/account; не поручайте настройку OAuth agent.

---

## Шаг 3 — Пусть Grok создаст demo calendar

**[В GROK BOT]**

> Create a new calendar called **Grok Demo**.
>
> Add these events:
>
> - Today 16:00 — Football practice
> - Tomorrow 09:00 — Science test
> - Tomorrow 15:30 — Alex's birthday
> - Saturday 12:00 — Grandma's birthday
> - Sunday 11:00 — Family trip
>
> Keep event names simple and clear.

Затем:

**[В GROK BOT]**

> Покажи мне events, которые ты только что создал.

Это первый наглядный момент **«AI сделал что-то в реальном мире»**.

---

## Шаг 4 — Пусть Grok установит Telegram MCP

**[В GROK BOT]**

> I want you to be able to send messages to my Telegram bot from your cloud computer.
>
> Set up a **local stdio MCP server** for the Telegram Bot API.
>
> First try `telegram-api-mcp`. Install it on your cloud computer, configure it securely with my Telegram bot token and the fixed Telegram chat ID, and verify that it starts correctly.
>
> Keep the bot token out of source files, prompts, logs, and normal chat history. Use the MCP server environment/secret mechanism.
>
> Do not send any message yet.
>
> First tell me:
> 1. whether the MCP server started successfully;
> 2. what Telegram messaging tools are available;
> 3. whether you can successfully authenticate with the bot.

### Передача значений Telegram в Grok

Use:

```text
TELEGRAM_BOT_TOKEN=<BotFather token>
TELEGRAM_DEFAULT_CHAT_ID=<kid chat id>
```

Предполагаемая схема:

```text
Grok Bot Agent
      ↓ stdio MCP
telegram-api-mcp
      ↓ HTTPS
Telegram Bot API
      ↓
Kid ↔ Calendar Buddy private chat
```

**Important:** do not put the bot token into normal chat. Give it to the MCP connector through Grok Bot's secure secret/environment mechanism.

### Запасной вариант, если `telegram-api-mcp` не сработает

**[В GROK BOT]**

> `telegram-api-mcp` did not work. Use `@node2flow/telegram-bot-mcp` instead.
>
> You previously solved a compatibility issue with this package by inspecting its schemas and adapting the launcher. Reproduce the working approach rather than insisting on the stock `npx` launch.
>
> Pin the MCP SDK to the compatible version if necessary and use the custom launcher approach.
>
> Keep the Telegram bot token only in the connector environment, never in source files or chat.
>
> First perform a `getMe` / `tg_get_me` smoke test and do not send a message yet.

Известная рабочая структура fallback:

```text
/workspace/mcp-telegram/
├── package.json
└── run.mjs
```

Workaround использует `@node2flow/telegram-bot-mcp` 1.0.4 с MCP SDK 1.26.0 и custom launcher вокруг `dist/client.js`.

---

## Шаг 5 — Проверить подключение Telegram

**[В GROK BOT]**

> Проверь identity Telegram bot, не отправляя message ребёнку.

Ожидаемый результат:

```text
is_bot: true
username: CalendarBuddyBot
```

Затем убедитесь, что fixed destination настроен:

```text
TELEGRAM_DEFAULT_CHAT_ID=<kid chat id>
```

**[В GROK BOT]**

> Подтверди, что fixed Telegram destination настроен корректно. Пока не отправляй message.

Использовать фиксированный `chat_id` предпочтительнее, чем позволять agent выбирать произвольных получателей Telegram.

---

## Шаг 6 — Первое простое Telegram message

**[В GROK BOT]**

> Send this exact message to the configured Telegram recipient:
>
> **Hello! 👋 Your Calendar Buddy is working.**

Ребёнок должен получить обычное Telegram message в private bot chat.

Это демонстрирует цепочку:

```text
Grok Bot
  → MCP
  → Telegram Bot API
  → private Telegram chat
  → Android notification
```

---

# 3. Дать ребёнку поэкспериментировать

**Пока не создавайте** recurring routine.

## 3.1 События на завтра

**[В GROK BOT]**

> Прочитай calendar **Grok Demo** и отправь мне Telegram message, содержащий только события на завтра.

Example:

```text
📅 Tomorrow

🧪 09:00 — Science test
🎂 15:30 — Alex's birthday
```

## 3.2 Добавить категории / emojis

**[В GROK BOT]**

> Отправь тот же summary, но пометь birthdays символом 🎂, а tests — 🧪.

## 3.3 Креатив для дней рождения

**[В GROK BOT]**

> Если есть birthday, предложи мне три варианта congratulation messages: friendly, funny, very short. Отправь их в Telegram.

## 3.4 Дать ребёнку изменить поведение

Спросите ребёнка, что он хочет изменить.

Examples:

**[В GROK BOT]**

> Не упоминай football practice.

**[В GROK BOT]**

> Всё-таки упоминай football practice, но ставь sports в начало.

**[В GROK BOT]**

> Сделай messages короче.

Это показывает, что ребёнок может менять поведение ассистента в диалоге, а не настраивать жёстко заданное приложение.

---

# 4. Превратить это в recurring routine

Только после того, как ребёнка устраивает поведение:

**[В GROK BOT]**

> Turn what we just built into a recurring routine.
>
> Every day, check the **Grok Demo** calendar for today and tomorrow.
>
> Ignore ordinary recurring lessons unless unusual or important.
>
> Mention tests, sports, trips, and birthdays.
>
> Mark birthdays with 🎂 and tests with 🧪.
>
> For birthdays include three short congratulation options: friendly, funny, very short.
>
> Send the resulting summary to the configured Telegram recipient.
>
> Do not send any other Telegram messages.

Для живого демо временно задайте легко наблюдаемый schedule (например, каждые 5 минут) или используйте возможность **Test run** у routine.

После демонстрации верните нужный schedule, например по будням в 07:30.

---

# 5. Показать автономность routine

Измените calendar после настройки routine.

For example:

- Add another birthday.
- Change the time of an event.
- Add an important trip.

Then wait for the temporary scheduled run.

Ожидаемая последовательность:

```text
Calendar изменён
      ↓
Routine запускается
      ↓
Calendar читается заново
      ↓
Grok определяет, что важно
      ↓
Формируется Telegram message
      ↓
Ребёнок получает его
```

Ручной prompt не должен требоваться.

---

# 6. Выключить laptop

Настройте routine так, чтобы она запустилась в ближайшее время и её можно было наблюдать.

Затем:

1. Измените calendar.
2. Убедитесь, что routine запланирована.
3. Полностью выключите Windows laptop.
4. Подождите.
5. Следите за Telegram ребёнка.

Цель — показать, что cloud computer Grok Bot продолжает выполнять routine, даже когда local laptop выключен.

После демо верните обычный schedule.

---

# 7. Сценарий демо

The complete story:

```text
Я создаю AI assistant
          ↓
Я даю ему доступ к моему calendar
          ↓
Он создаёт свой demo calendar
          ↓
Я показываю ему, как должны выглядеть reminders
          ↓
Он учится отправлять Telegram messages
          ↓
Я меняю правила
          ↓
Он сразу меняет свои messages
          ↓
Я говорю ему «делай это каждое утро»
          ↓
Он создаёт routine
          ↓
Я выключаю компьютер
          ↓
📱 Telegram всё равно приходит
```

### Что демонстрирует это демо

**Tools → agentic reasoning → external action → user-directed refinement → automation → cloud autonomy**
