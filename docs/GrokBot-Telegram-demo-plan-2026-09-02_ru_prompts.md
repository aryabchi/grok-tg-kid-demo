Absolutely. I would make the demo deliberately **interactive first, autonomous second**. That gives the kid several visible “wow” moments before you introduce the recurring routine.

Also, one clarification from the current docs: Grok Bot routines can run on the cloud computer while your laptop is closed, and the routine has an explicit **Test run** facility. ([X.AI Documentation][1])

# 1. Prerequisites — everything prepared before the demo

The goal here is: **you arrive with accounts/access ready, but the actual assistant and integrations get assembled during the demo.**

### A. Your Cursor / Grok Bot access

You need:

* Cursor Pro account that has Grok Bot access.
* Grok Bot 0.30.0 on Windows.
* Ability to create a new Bot.
* Access to the Bot's cloud computer / terminal.

Cursor confirms Grok Bot is included with paid individual Cursor plans, including Pro. ([Cursor][2])

### B. Kid's Google account

Have available:

* the kid's Google account;
* the credentials needed for the Google OAuth flow;
* ability to complete Google authentication interactively.

**Do not create the demo calendar beforehand.**

The calendar and events will be created by Grok during the demo.

The Google Calendar plugin will be connected manually from **Grok Bot → Plugins**. Cursor's current instructions say to add a plugin there and complete provider authentication in the browser. ([Cursor][3])

### C. Telegram Bot

Do this **before the demo** because BotFather creation is not the interesting part.

You need:

* Telegram bot created with BotFather;
* bot username;
* bot token;
* kid has Telegram installed on Android;
* kid has opened the bot and pressed **Start**.

Keep the bot token somewhere you can securely supply to Grok when needed. **Don't put it into the normal Bot prompt.**

### D. Telegram destination

You need the kid's Telegram `chat_id`.

I would either have this already available or let the MCP setup discover it after the kid sends `/start`.

For the cleanest architecture, the final MCP configuration should use a **fixed destination**, rather than letting Grok choose arbitrary Telegram users.

Conceptually:

```text
TELEGRAM_BOT_TOKEN
TELEGRAM_DEFAULT_CHAT_ID = kid
```

### E. Telegram MCP candidates

Have these two options in mind:

**Primary**

```text
telegram-api-mcp
```

**Fallback**

```text
@node2flow/telegram-bot-mcp
```

You don't need to install either before the demo. Grok should do that on its cloud computer.

For `node2flow`, keep your already-proven workaround available as a fallback because you've already confirmed that Grok Bot can diagnose and work around the SDK/Zod issue.

### F. Demo content

Decide beforehand what events you want Grok to create. For example:

```text
* Сегодня 16:00 — Тренировка по футболу
* Завтра 09:00 — Контрольная по естествознанию
* Завтра 15:30 — День рождения Алекса
* Суббота 12:00 — День рождения бабушки
* Воскресенье 11:00 — Семейная поездка
```

That's all the manual preparation I would do.

---

# 2. Step-by-step demo

## Step 1 — Create the Bot

At the beginning of the demo:

**Grok Bot → New Bot**

Name:

**Calendar Buddy**

Then tell it:

> You are my personal calendar assistant. Your job is to help me understand important upcoming events and send useful reminders to my Telegram.

*RU:*
> Ты мой личный ассистент по календарю. Твоя задача — помогать мне отслеживать важные предстоящие события и отправлять полезные напоминания в мой Telegram.

Don't configure the routine yet.

---

## Step 2 — Connect Google Calendar

This is one of the few things **you do manually**.

In the Grok Bot app:

**Plugins → Google Calendar → Add**

Complete the Google OAuth flow using the kid's Google account.

This is account/plugin configuration, not something I'd delegate to the Bot. Cursor's documented flow is: open Plugins, add the plugin, then complete authorization in the browser. ([Cursor][3])

Then tell Grok:

> Check whether you can access my Google Calendar. Don't change anything yet. Tell me whether you can read calendar events.

*RU:*
> Проверь, есть ли у тебя доступ к моему Google Calendar. Пока ничего не меняй. Скажи мне, можешь ли ты читать события календаря.

---

## Step 3 — Let Grok create the demo calendar

Now the kid gets to participate.

Tell Grok:

> Create a new calendar called **Grok Demo**.
>
> Add these events:
>
> * Today 16:00 — Football practice
> * Tomorrow 09:00 — Science test
> * Tomorrow 15:30 — Alex's birthday
> * Saturday 12:00 — Grandma's birthday
> * Sunday 11:00 — Family trip
>
> Keep the event names simple and clear.

*RU:*
> Создай новый календарь под названием **Grok Demo**.
>
> Добавь следующие события:
>
> * Сегодня 16:00 — Тренировка по футболу
> * Завтра 09:00 — Контрольная по естествознанию
> * Завтра 15:30 — День рождения Алекса
> * Суббота 12:00 — День рождения бабушки
> * Воскресенье 11:00 — Семейная поездка
>
> Названия событий делай простыми и понятными.

Grok should use the Calendar tools to create the calendar/events.

Then ask:

> Show me the events you just created.

*RU:*
> Покажи мне события, которые ты только что создал.

This is the first useful “AI did something in the real world” moment.

---

# Step 4 — Have Grok install Telegram MCP

Now tell Grok:

> I want you to be able to send messages to my Telegram bot from your cloud computer.
>
> Set up a **local stdio MCP server** for the Telegram Bot API.
>
> First try `telegram-api-mcp`. Install it on your cloud computer, configure it securely with my Telegram bot token, and verify that it starts correctly.
>
> Do not send any message yet.
>
> First tell me:
>
> 1. whether the MCP server started successfully;
> 2. what Telegram messaging tools are available;
> 3. whether you can successfully authenticate with the bot.

*RU:*
> Я хочу, чтобы ты мог отправлять сообщения в мой Telegram-бот со своего облачного компьютера.
>
> Настрой **локальный stdio MCP server** для Telegram Bot API.
>
> Сначала попробуй `telegram-api-mcp`. Установи его на своём облачном компьютере, безопасно настрой с моим Telegram bot token и проверь, что он корректно запускается.
>
> Пока не отправляй никаких сообщений.
>
> Сначала скажи мне:
>
> 1. успешно ли запустился MCP server;
> 2. какие инструменты для отправки сообщений в Telegram доступны;
> 3. удалось ли успешно авторизоваться в боте.

### Fallback if `telegram-api-mcp` fails

Then tell it:

> `telegram-api-mcp` did not work. Use `@node2flow/telegram-bot-mcp` instead.
>
> You previously solved a compatibility issue with this package by inspecting its schemas and adapting the launcher. Reproduce the working approach rather than insisting on the stock `npx` launch.
>
> Keep the Telegram bot token only in the connector environment, never in source files or chat.
>
> First perform a `getMe`/`tg_get_me` smoke test and do not send a message yet.

*RU:*
> `telegram-api-mcp` не сработал. Вместо него используй `@node2flow/telegram-bot-mcp`.
>
> Ранее ты уже решал проблему совместимости с этим пакетом, изучив его схемы и адаптировав launcher. Воспроизведи этот рабочий подход, а не настаивай на стандартном запуске через `npx`.
>
> Храни Telegram bot token только в окружении коннектора, никогда — в исходных файлах или в чате.
>
> Сначала выполни smoke test через `getMe`/`tg_get_me` и пока не отправляй сообщение.

This is where your previous successful experience is valuable: **you're deliberately allowing the agent to debug the integration instead of pre-solving it yourself.**

The upstream `node2flow` package documents a local stdio installation and `TELEGRAM_BOT_TOKEN`; its current published version is 1.0.4. ([npmjs.com][4])

---

# Step 5 — Verify the Telegram connection

Once Grok says the MCP works:

> Verify the Telegram bot identity without sending a message to the child.

*RU:*
> Проверь идентичность Telegram-бота, не отправляя сообщение ребёнку.

Expected conceptually:

```text
is_bot: true
username: CalendarBuddyBot
```

Now establish the destination:

> Find the private Telegram chat with the user who started this bot and configure that chat as the only destination for calendar reminders. Do not send anything yet.

*RU:*
> Найди приватный Telegram-чат с пользователем, который запустил этого бота, и настрой этот чат как единственный пункт назначения для напоминаний из календаря. Пока ничего не отправляй.

---

# Step 6 — Send the first simple Telegram message

Now the kid gets the first tangible result.

Tell Grok:

> Send this exact message to the configured Telegram recipient:
>
> **Hello! 👋 Your Calendar Buddy is working.**

*RU:*
> Отправь настроенному получателю в Telegram именно это сообщение:
>
> **Привет! 👋 Твой Calendar Buddy работает.**

Kid receives:

```text
Calendar Buddy 🤖
Hello! 👋 Your Calendar Buddy is working.
```

At this point you've proven:

**Grok → local MCP → Telegram Bot API → kid's Android**

---

# Step 7 — Let the kid experiment with calendar → Telegram

This is where I would spend most of the fun demo time.

Don't create the routine yet.

### 7a. First variation

Tell Grok:

> Read the **Grok Demo** calendar and send me a Telegram message containing only tomorrow's events.

*RU:*
> Прочитай календарь **Grok Demo** и отправь мне сообщение в Telegram, содержащее только завтрашние события.

Kid gets:

> 📅 **Tomorrow**
>
> 🧪 09:00 — Science test
> 🎂 15:30 — Alex's birthday

### 7b. Second variation

Then let the kid change the requirement:

> Send the same summary, but mark birthdays with 🎂 and tests with 🧪.

*RU:*
> Отправь такую же сводку, но отмечай дни рождения значком 🎂, а контрольные — значком 🧪.

Grok sends a new Telegram message with the requested format.

### 7c. Third variation

Now:

> When there is a birthday, give me three possible congratulation messages: friendly, funny, and very short. Put them in the Telegram message.

*RU:*
> Когда есть день рождения, предложи мне три варианта поздравления: дружелюбный, смешной и очень короткий. Включи их в сообщение в Telegram.

The kid receives something like:

> 🎂 **Alex's birthday tomorrow!**
>
> **Friendly:** Happy birthday, Alex! 🎉 Hope you have a fantastic day!
>
> **Funny:** Happy birthday! 🎂 You're officially one year more awesome.
>
> **Short:** Happy birthday, Alex! 🎉

### 7d. Let the kid change the assistant

This is the really good part.

Ask the kid:

> What should your Calendar Buddy change?

*RU:*
> Что бы ты хотел изменить в своём Calendar Buddy?

For example:

> Don't mention football practice.

*RU:*
> Не упоминай тренировку по футболу.

Then:

> Actually, mention it but put sports at the top.

*RU:*
> На самом деле, упомяни её, но поставь спорт на первое место.

Then:

> Make the messages shorter.

*RU:*
> Делай сообщения короче.

The Bot repeatedly changes the Telegram output based on the kid's instructions.

**Only after the kid is happy with the content and format do we automate it.**

---

# Step 8 — Create the recurring routine — but don't make the kid wait

Now that the behavior is proven manually, turn it into a routine.

Tell Grok:

> Turn what we just built into a recurring routine.
>
> Every day, check the Grok Demo calendar for today's and tomorrow's events.
>
> Ignore ordinary recurring lessons unless they're unusual or important.
>
> Mention important events such as tests, sports, trips and birthdays.
>
> Mark birthdays with 🎂 and tests with 🧪.
>
> For birthdays, include three short congratulation options: friendly, funny and very short.
>
> Send the resulting summary to the configured Telegram recipient.
>
> Do not send any other Telegram messages.

*RU:*
> Преврати то, что мы только что сделали, в повторяющуюся routine.
>
> Каждый день проверяй календарь Grok Demo на предмет сегодняшних и завтрашних событий.
>
> Игнорируй обычные повторяющиеся уроки, если они не необычные или важные.
>
> Упоминай важные события, такие как контрольные, спорт, поездки и дни рождения.
>
> Отмечай дни рождения значком 🎂, а контрольные — значком 🧪.
>
> Для дней рождения включай три коротких варианта поздравления: дружелюбный, смешной и очень короткий.
>
> Отправляй итоговую сводку настроенному получателю в Telegram.
>
> Не отправляй никаких других сообщений в Telegram.

Grok Bot's current routine model explicitly supports taking a successful one-time task and turning it into a repeatable process. It also provides **Test run** before enabling the routine. ([X.AI Documentation][1])

### Timing

Don't set it for tomorrow morning.

For the demo, set the routine to something like:

**Every 5 minutes**, temporarily.

For example:

```text
09:30
09:35
09:40
...
```

Then run **Test run** immediately.

That lets the kid see:

```text
routine created
      ↓
Test run
      ↓
Calendar read
      ↓
reasoning
      ↓
Telegram message
```

The test run performs real work, so use the demo calendar and the kid's Telegram as the safe target. Cursor's documentation specifically recommends using safe inputs because a test run can perform real external actions. ([X.AI Documentation][1])

Once you've seen it work, you can change the production schedule to something sensible like **07:30 every weekday**.

---

# Step 9 — Prove the routine is autonomous

Now comes the second “wow” moment.

After the routine works:

> Change tomorrow's **Grok Demo** calendar entry from "Science test" to "Science test — 09:00". Add another birthday tomorrow.

*RU:*
> Измени завтрашнюю запись в календаре **Grok Demo** с «Science test» на «Science test — 09:00». Добавь ещё один день рождения на завтра.

Then wait for the next temporary scheduled run.

The next Telegram message should reflect the changed Calendar data **without you manually telling Grok to run the task again**.

This demonstrates:

**Calendar changed → routine wakes up → Grok re-reads → reasons → Telegram changes.**

---

# Step 10 — Turn off the laptop

Finally, test the thing you actually care about.

First change the routine to an easily observable temporary schedule, e.g.:

**Run 5–10 minutes from now.**

Then:

1. Make one small change to the demo calendar.
2. Confirm the laptop is not needed for the actual run.
3. Shut down your Windows laptop completely.
4. Wait for the scheduled run.
5. Watch the kid's Telegram.

Grok Bot's current documentation is explicit:

> “Bot work runs on the cloud computer. Closing the app, laptop, or iPhone does not stop a background turn or routine.” ([X.AI Documentation][5])

If the Telegram message arrives with the updated calendar information, you've demonstrated the **full autonomous architecture**.

Afterwards, restore the routine to:

**Every weekday at 07:30**

---

# The resulting story for the kid

The demo now has a very natural narrative:

```text
I create an AI assistant
          ↓
I give it access to my calendar
          ↓
It creates its own demo calendar
          ↓
I teach it how I want reminders to look
          ↓
It learns to send Telegram messages
          ↓
I change the rules
          ↓
It immediately changes its messages
          ↓
I tell it "do this every morning"
          ↓
It creates a routine
          ↓
I turn off my computer
          ↓
📱 Telegram still arrives
```

That's much stronger than simply demonstrating “Grok can read Google Calendar.” It shows the whole progression:

**tools → agentic reasoning → external action → user-directed refinement → automation → cloud autonomy.**

[1]: https://docs.x.ai/grok-bot/skills-routines-and-automations?utm_source=chatgpt.com "Skills and routines | SpaceXAI Docs"
[2]: https://cursor.com/help/grok-bot/plans?utm_source=chatgpt.com "Plans and billing | Cursor Docs"
[3]: https://cursor.com/help/grok-bot/connect-plugins?utm_source=chatgpt.com "Connect plugins | Cursor Docs"
[4]: https://www.npmjs.com/package/%40node2flow/telegram-bot-mcp?utm_source=chatgpt.com "@node2flow/telegram-bot-mcp - npm"
[5]: https://docs.x.ai/grok-bot/faq?utm_source=chatgpt.com "Frequently asked questions | SpaceXAI Docs"
