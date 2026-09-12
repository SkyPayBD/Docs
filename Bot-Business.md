# Bots.Business (BB) — Complete AI Agent Training Documentation

> **Purpose:** This document is a complete reference guide for AI agents, developers, and bot creators who want to build powerful Telegram bots using the **Bots.Business (BB)** platform. It covers every feature, syntax rule, library, pattern, and best practice needed to write correct BJS code from scratch.
>
> **Intended Audience:** AI language models used as coding assistants, Telegram bot developers, and learners who want to understand BB's custom JavaScript dialect (BJS) end-to-end.

---

<!--
  ════════════════════════════════════════════════════════════
  Copyright © 2026–2027  MD Jaid Bin Siyam (Siyam)
  All rights reserved.

  Author        : MD Jaid Bin Siyam (Siyam)
  Telegram      : @BD_Prime_Minister
  Channel       : https://t.me/Hyper_10_Squad
  Email         : support@skypaybd.top
  WhatsApp      : +8801761844968
  Phone (Call)  : (contact via Telegram or WhatsApp)
  Address       : Panchagarh, Rangpur, Bangladesh
  Payment GW    : https://skypaybd.top/
  Digital Store : https://chapri.shop/
  GitHub        : https://github.com/SkyPayBD
  YouTube       : https://youtube.com/@SkyPayBD
  Facebook      : https://facebook.com/siyamahmedjsx

  This document was created to help users of the SkyPayBD
  payment gateway build Telegram bots using Bots.Business.
  Unauthorized redistribution without credit is prohibited.
  ════════════════════════════════════════════════════════════
-->

---

## 🤖 AI Agent Initialization

When this document is provided as a system prompt or context to any AI agent, the agent **must reply with the following message first** before doing anything else:

> **"I am completely ready to help with this, tell me what I should make for you? 🤝"**

After that, the agent will wait for the user's task and respond based on the full knowledge in this document.

---

## Table of Contents

1. [What is Bots.Business?](#1-what-is-botsbusiness)
2. [Platform Overview & Account Setup](#2-platform-overview--account-setup)
3. [BJS — The Custom JavaScript Dialect](#3-bjs--the-custom-javascript-dialect)
4. [Commands — The Core Building Block](#4-commands--the-core-building-block)
5. [Command Editor Fields Explained](#5-command-editor-fields-explained)
6. [Wait for Answer — The Input Collection System](#6-wait-for-answer--the-input-collection-system)
7. [Bot Methods — `Bot.*`](#7-bot-methods--bot)
8. [Telegram API Methods — `Api.*`](#8-telegram-api-methods--api)
9. [HTTP Requests — `HTTP.*`](#9-http-requests--http)
10. [Properties — Storing & Reading Data](#10-properties--storing--reading-data)
11. [Context Variables — `user`, `chat`, `request`, `params`, `options`, `message`](#11-context-variables--user-chat-request-params-options-message)
12. [Chaining Commands — `Bot.runCommand` & `Bot.run`](#12-chaining-commands--botruncommand--botrun)
13. [Inline Keyboards & Callback Queries](#13-inline-keyboards--callback-queries)
14. [Reply Keyboards](#14-reply-keyboards)
15. [Saving & Using Message IDs (Edit / Delete Messages)](#15-saving--using-message-ids-edit--delete-messages)
16. [Webhooks Library — Receiving External Requests](#16-webhooks-library--receiving-external-requests)
17. [All Available Libraries](#17-all-available-libraries)
    - [Library Quick Reference Table](#library-quick-reference-table)
    - [Lists, Admin Panel, WebApp, Deep Links, Inactivity Reminder](#17b-additional-platform-features)
18. [Economy System Pattern](#18-economy-system-pattern)
19. [Referral System Pattern](#19-referral-system-pattern)
20. [Leaderboard Pattern](#20-leaderboard-pattern)
21. [Broadcast System](#21-broadcast-system)
22. [Scheduling & Background Work](#22-scheduling--background-work)
23. [HTML Formatting in Messages](#23-html-formatting-in-messages)
24. [Security Best Practices](#24-security-best-practices)
25. [Errors & Debugging](#25-errors--debugging)
26. [Complete Syntax Rules & Common Mistakes](#26-complete-syntax-rules--common-mistakes)

---

## 1. What is Bots.Business?

**Bots.Business (BB)** is a no-code/low-code cloud platform for creating Telegram bots. You connect a Telegram bot token to the platform, create **commands**, write logic in **BJS** (a synchronous JavaScript dialect), and the platform executes that logic when users interact with your bot.

Key characteristics:
- All code runs **synchronously** on BB's cloud servers.
- You do **not** host anything yourself.
- Code is written in the **mobile app** (Android/iOS) or via **VS Code** integration.
- Built-in objects like `Bot`, `Api`, `HTTP`, `User`, `Libs` are provided by the runtime — you never import them.
- Results from async operations (Telegram API calls, HTTP requests) are delivered to **callback commands**, not returned inline.

---

## 2. Platform Overview & Account Setup

### Creating a Bot

1. Create a Telegram bot via [@BotFather](https://t.me/BotFather) and copy the token.
2. Sign up at [bots.business](https://bots.business) (email or Telegram login).
3. Add a new bot and paste the token.
4. Launch the bot from the Dashboard.
5. Open **Commands** → tap **+** → **New command** to start building.

### Mobile App Navigation

| Tab | Purpose |
|-----|---------|
| **Dashboard** | Bot status, launch/stop |
| **Commands** | Create, edit, search, organize commands |
| **Libraries** | Install/manage libraries |
| **Properties** | Inspect stored bot/user data |
| **Chats** | View known users/chats |
| **Errors** | Runtime error log |
| **Admin Panel** | Bot settings form (configurable) |

---

## 3. BJS — The Custom JavaScript Dialect

BJS is **synchronous JavaScript** running inside a sandboxed BB runtime. It looks like ordinary JavaScript but with important restrictions.

### ✅ What You CAN Use

```js
var, let, const
if / else / else if
for loops, while loops
functions (regular, not async)
arrays, objects, JSON.parse / JSON.stringify
String(), Number(), Boolean(), parseInt(), parseFloat()
Math.*, Array.*, String.*
try / catch / finally
return (exits the current command early)
```

### ❌ What You CANNOT Use

```js
async / await          // Not supported — BJS is synchronous
Promises / .then()     // Not supported
fetch()                // Not available — use HTTP.get / HTTP.post
setTimeout / setInterval  // Not available — use Bot.run({ run_after: N })
require()              // Not available — no module system
document / window      // No browser DOM
Node.js built-ins      // Not available (fs, path, etc.)
new Function()         // Sandbox-blocked
WebAssembly            // Not supported
```

### Key Syntax Rules

- Use `===` for equality checks, never `==` where type matters.
- A single `=` assigns a value; it does **not** compare.
- String input from users (`message`) is always text — convert with `Number(message)` before arithmetic.
- Built-in BJS names (`user`, `chat`, `Bot`, `User`, `Api`, `HTTP`, `Libs`, `params`, `options`, `message`, `content`, `request`) are **reserved** — do not use them as your own variable names.
- Comments use `//` for single line and `/* */` for block comments.
- Curly braces `{ }` from formatted text editors can break syntax — always use straight quotes `"` or `'`.

---

## 4. Commands — The Core Building Block

A **command** is the fundamental unit of behavior in BB. Each command has:
- A **name** (the trigger text)
- Optional **Answer** (a static text reply sent automatically)
- Optional **BJS code** (dynamic logic)
- Optional **Keyboard** (reply buttons)
- Optional **Aliases** (alternative trigger names)
- Optional **Wait for answer** flag
- Optional **Folder** assignment
- Optional **Group restriction**

### Command Naming Rules

Command names are **very flexible** in BB:

```
/start          ← standard slash command
/cmd2           ← any slash name
Help            ← no slash required
🎮 Play         ← emojis and spaces are valid
My Settings     ← spaces allowed
*               ← wildcard: matches any unrecognized message
@               ← runs when a user joins the bot for the first time
@@              ← runs in group chats when a new member joins
!               ← runs when a BJS error occurs (error handler)
```

> **Important:** Command matching is exact for the main name. Aliases are normalized to lowercase. `/start` and `/START` can be different commands.

### Aliases

Aliases let one command respond to multiple trigger words:

```
Command: /help
Aliases: Help, /support, Support, 🆘 Help
```

Any of those texts typed by the user will trigger `/help`. This is also how reply keyboard buttons connect to commands — the button label becomes the text sent, and an alias matches it.

### Folders

Folders are **organizational only** — they do not restrict who can run a command. Create folders under **Commands → + → Folder manager** to keep your commands tidy.

---

## 5. Command Editor Fields Explained

| Field | Description |
|-------|-------------|
| **Command** | The trigger text (e.g. `/start`, `Help`, `🎮 Play`) |
| **Answer** | Static message sent when this command is triggered. Supports basic Markdown. Can include `<property_name>` placeholders. |
| **Aliases** | Comma-separated alternative trigger names: `Help, /support, Support` |
| **Help** | Description shown in the bot's help behavior |
| **Keyboard** | Comma-separated reply button labels. Use `\n` to start a new row. Requires an Answer to be set. |
| **Allowed only for group** | Restricts command to a BB user group (not a Telegram group). Assign groups via `User.addToGroup()`. |
| **Wait for answer** | When ON: sends the Answer/prompt first, then waits for the user's next message before running the BJS code. |
| **Auto retry time in seconds** | Runs BJS periodically without user interaction (scheduled background task). Requires explicit destination config. |
| **Select Folder** | Assigns the command to an editor folder |

### Answer Property Placeholders

In the **Answer** field only (not BJS), you can embed stored property values:

```
Answer: Hello <first_name>, your balance is <balance> coins!
```

This inserts the user's `first_name` property and their `balance` property automatically.

---

## 6. Wait for Answer — The Input Collection System

**Wait for Answer** is one of the most important features in BB. It creates a two-step interaction:

### How It Works

1. User sends the command trigger (e.g. `/setname`).
2. BB sends the **Answer** text (the prompt) immediately.
3. BB **pauses** and waits for the user's next message.
4. When the user replies, BB runs the **BJS code** of that command.
5. Inside the BJS, the user's reply is available as `message`.

### Step-by-Step Example

**Command: `/askname`**
- Answer: `What is your name? Send /cancel to stop.`
- Wait for answer: **ON**
- BJS:

```js
const name = (message || "").trim();
if (!name) {
  Bot.sendMessage("Please send your name as text.");
  return;
}
User.setProp("display_name", name);
Bot.sendMessage("Your name has been saved as: " + name);
```

**Command: `/cancel`**
- Answer: `Cancelled. Send /askname to try again.`
- Wait for answer: **OFF**
- BJS: (empty)

> When the user types `/cancel` while a wait is pending, BB cancels the pending wait and runs `/cancel` instead. This is the standard cancel pattern.

### Redirecting to Another Command That Waits

You can chain commands so that one command immediately redirects to another that has Wait for Answer ON:

**Command: `/start`** (Wait for answer: OFF)
```js
Bot.sendMessage("Welcome! Let's set up your account.");
Bot.runCommand("/askname");
```

**Command: `/askname`** (Wait for answer: ON, Answer: "What is your name?")
```js
const name = (message || "").trim();
User.setProp("display_name", name);
Bot.sendMessage("Name saved: " + name);
```

When `/start` runs, it sends the welcome message, then runs `/askname`, which sends its prompt and waits. When the user replies, `/askname`'s BJS runs with the user's input in `message`.

---

## 7. Bot Methods — `Bot.*`

`Bot.*` methods are BB's own higher-level API. They are simpler but do not return Telegram response data.

### Sending Messages

```js
Bot.sendMessage("Hello!");
Bot.sendMessage("Hello <b>World</b>", { parse_mode: "HTML" });
Bot.sendMessage({ text: "Hello", parse_mode: "HTML" });

// Send to a specific chat by Telegram chat ID
Bot.sendMessageToChatWithId(telegramChatId, "Hello there!");

// Send by chat title (less reliable — titles can be duplicated)
Bot.sendMessageToChat("My Group Name", "Announcement!");

// Debug: format any value as JSON and send it
Bot.inspect(someObject);
```

### Editing Messages

```js
// Edit a message in the current chat
Bot.editMessage("Updated text", message_id);

// Edit a message in another chat
Bot.editMessageInChat(telegramChatId, "Updated text", message_id);

// Replace inline keyboard on an existing message
Bot.editInlineKeyboard(buttonsArray, message_id, telegramChatId);
```

> Always keep the `message_id` together with its `chat_id` — a message ID is only unique within its chat.

### Running Commands

```js
// Run another command immediately in the same context
Bot.runCommand("/mycommand");

// Run with options passed to the target command
Bot.runCommand("/mycommand", { key: "value" });

// Advanced run with scheduling and context
Bot.run({
  command: "/mycommand",
  options: { amount: 100 },
  run_after: 30,      // delay in seconds (optional)
  label: "reminder"   // label for cancellation (optional)
});

// Cancel scheduled commands by label
Bot.clearRunAfter({ label: "reminder" });
```

### Inline Keyboards (Bot style)

```js
Bot.sendInlineKeyboard([
  { title: "Option A", command: "/option_a" },
  { title: "Visit Site", url: "https://example.com" }
], "Choose an option:");

// Send to a specific chat
Bot.sendInlineKeyboardToChatWithId(telegramChatId, buttons, "Choose:");
```

### Reply Keyboards (Bot style)

```js
Bot.sendKeyboard("Button1,Button2\nButton3,Button4", "Choose:");
// \n starts a new row of buttons
```

### Properties (shorthand — same as User/Bot.setProp)

```js
Bot.setProp("key", value);
Bot.getProp("key", defaultValue);
Bot.deleteProp("key");
```

---

## 8. Telegram API Methods — `Api.*`

`Api.*` gives you direct access to every Telegram Bot API method. Unlike `Bot.*`, API calls can return response data via **callback commands**.

### Core Pattern

```js
Api.methodName({
  param1: value1,
  param2: value2,
  on_result: "/callback_command",   // runs on success
  on_error: "/error_command"        // runs on failure
});
```

The result is NOT returned inline. It arrives in the callback command via `options.result`.

### Sending Messages

```js
Api.sendMessage({
  chat_id: user.telegramid,
  text: "Hello <b>World</b>!",
  parse_mode: "HTML"
});
```

### Sending Photos

```js
Api.sendPhoto({
  chat_id: user.telegramid,
  photo: "https://example.com/image.jpg",
  caption: "Here is the photo!"
});

// Or use a Telegram file_id
Api.sendPhoto({
  chat_id: user.telegramid,
  photo: fileId,
  caption: "Your photo"
});
```

### Sending Documents

```js
Api.sendDocument({
  chat_id: user.telegramid,
  document: "https://example.com/file.pdf",
  caption: "Your document"
});
```

### Other Media Methods

```js
Api.sendAudio({ chat_id: ..., audio: fileId, caption: "..." });
Api.sendVideo({ chat_id: ..., video: fileId, caption: "..." });
Api.sendVoice({ chat_id: ..., voice: fileId });
Api.sendAnimation({ chat_id: ..., animation: fileId });
Api.sendVideoNote({ chat_id: ..., video_note: fileId });
Api.sendMediaGroup({ chat_id: ..., media: [...] });
```

### Editing Messages

```js
// Edit text of a sent message
Api.editMessageText({
  chat_id: telegramChatId,
  message_id: savedMessageId,
  text: "Updated content",
  parse_mode: "HTML"
});

// Edit message with new inline keyboard
Api.editMessageText({
  chat_id: telegramChatId,
  message_id: savedMessageId,
  text: "Pick an option:",
  parse_mode: "HTML",
  reply_markup: {
    inline_keyboard: [[
      { text: "Button", callback_data: "/mycommand" }
    ]]
  }
});

// Edit only the reply markup (keyboard), not the text
Api.editMessageReplyMarkup({
  chat_id: telegramChatId,
  message_id: savedMessageId,
  reply_markup: { inline_keyboard: [[...]] }
});
```

### Deleting Messages

```js
Api.deleteMessage({
  chat_id: telegramChatId,
  message_id: messageIdToDelete
});
```

> You can only delete messages sent by the bot, or user messages in groups where the bot is admin.

### Getting a Sent Message's ID (on_result)

To capture the `message_id` of a message you just sent:

**In sending command:**
```js
Api.sendMessage({
  text: "Processing...",
  parse_mode: "HTML",
  on_result: "/save_msg_id"
});
```

**Command `/save_msg_id`:**
```js
if (!options || !options.result) { return; }
const id = options.result.message_id;
User.setProp("last_msg_id", id, "string");
```

### Inline Keyboards (Api style)

```js
Api.sendMessage({
  chat_id: user.telegramid,
  text: "Choose:",
  reply_markup: {
    inline_keyboard: [
      [
        { text: "Button A", callback_data: "/cmd_a" },
        { text: "Button B", callback_data: "/cmd_b" }
      ],
      [
        { text: "Visit", url: "https://example.com" }
      ]
    ]
  }
});
```

> **Important:** `Bot.sendInlineKeyboard` uses `{ title, command }`. `Api.sendMessage` uses `{ text, callback_data }`. Do **not** mix these two formats.

### Answer Callback Query (for inline button presses)

When a user taps an inline button, Telegram sends a callback query. BB runs the command in `callback_data`. To dismiss the loading spinner:

```js
Api.answerCallbackQuery({
  callback_query_id: request.callback_query_id,
  text: "Done!",
  show_alert: false
});
```

### Get Chat Member (check membership)

```js
Api.getChatMember({
  chat_id: "@yourchannel",
  user_id: user.telegramid,
  on_result: "/check_result"
});
```

```js
// /check_result
if (!options || !options.result) { return; }
const status = options.result.status;
// status: "member", "administrator", "creator", "left", "kicked"
if (status === "member" || status === "administrator" || status === "creator") {
  Bot.sendMessage("You are subscribed ✅");
} else {
  Bot.sendMessage("Please join our channel first.");
}
```

### Forward a Message

```js
Api.forwardMessage({
  chat_id: targetChatId,
  from_chat_id: sourceChatId,
  message_id: messageId
});
```

### Pin a Message

```js
Api.pinChatMessage({
  chat_id: telegramChatId,
  message_id: messageId
});
```

---

## 9. HTTP Requests — `HTTP.*`

Use `HTTP.*` to call external web services. **Responses are never returned inline** — they go to callback commands.

### GET Request

```js
HTTP.get({
  url: "https://api.example.com/data",
  success: "/on_http_success",
  error: "/on_http_error",
  headers: { "Authorization": "Bearer TOKEN" }
});
```

**`/on_http_success`:**
```js
if (http_status == null) { return; }
const status = Number(http_status);
if (status < 200 || status >= 300) {
  Bot.sendMessage("Error: HTTP " + status);
  return;
}
let data;
try { data = JSON.parse(content); } catch(e) {
  Bot.sendMessage("Invalid JSON response");
  return;
}
Bot.sendMessage("Result: " + data.someField);
```

**`/on_http_error`:**
```js
Bot.sendMessage("Could not reach the service. Try again later.");
```

### POST Request (JSON Body)

```js
HTTP.post({
  url: "https://api.example.com/create",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice", amount: 100 }),
  success: "/on_post_success",
  error: "/on_http_error"
});
```

### Available Callback Variables in success command

| Variable | Content |
|----------|---------|
| `content` | Response body as decoded text |
| `http_status` | HTTP status code (e.g. `200`, `404`) |
| `http_headers` | Decoded response headers object |
| `cookies` | Decoded Set-Cookie info |

### Supported Methods

```js
HTTP.get({ url, success, error, headers })
HTTP.post({ url, body, success, error, headers })
HTTP.put({ url, body, success, error, headers })
HTTP.delete({ url, success, error, headers })
HTTP.options({ url, success, error, headers })
```

> ⚠️ `HTTP.patch` and `HTTP.trace` are **not implemented** — do not use them.
> ⚠️ Each request has a **5-second timeout**. For long operations, return a job ID immediately and poll later with `Bot.run({ run_after: N })`.

---

## 10. Properties — Storing & Reading Data

Properties are the primary persistent storage in BB. Data survives between executions.

### User Properties (per-user data)

```js
// Save
User.setProp("key", value);
User.setProp("key", value, "string");  // explicit type

// Read
const val = User.getProp("key");
const val = User.getProp("key", "default");  // with fallback

// Delete
User.deleteProp("key");
```

### Bot Properties (shared by all users)

```js
// Save
Bot.setProp("key", value);
Bot.setProp("key", value, "json");

// Read
const val = Bot.getProp("key");
const val = Bot.getProp("key", 0);   // with fallback

// Delete
Bot.deleteProp("key");
```

### Supported Storage Types

| Type | Use for |
|------|---------|
| `"string"` | Short text (≤255 chars) |
| `"text"` | Long text (>255 chars) |
| `"integer"` | Whole numbers |
| `"float"` | Decimal numbers |
| `"boolean"` | `true` / `false` |
| `"json"` | Arrays, objects |
| `"datetime"` | Date strings |

> **Best practice:** Always use `"string"` for IDs and numbers you don't calculate with. Use `"json"` explicitly for arrays and objects.

### ⚠️ The Zero/False/Empty Problem

The property reader uses truthy fallback logic. A saved `0`, `false`, or `""` may return the default value instead. **Wrap scalar values in a JSON object** when you need to distinguish zero from "not set":

```js
// Save
Bot.setProp("settings", { enabled: false, count: 0 }, "json");

// Read
const s = Bot.getProp("settings", { enabled: false, count: 0 });
Bot.sendMessage(s.enabled ? "ON" : "OFF");  // correctly reads false
```

### Storing Arrays (Broadcast Groups Pattern)

```js
// Add user to a group array
let group = Bot.getProp("group_0") || [];
if (!group.includes(user.telegramid)) {
  if (group.length >= 200) {
    // Start a new group when full
    let i = 1;
    while (Bot.getProp("group_" + i)) { i++; }
    group = [];
    Bot.setProp("group_" + i, group, "json");
  } else {
    group.push(user.telegramid);
    Bot.setProp("group_0", group, "json");
  }
}
```

---

## 11. Context Variables — `user`, `chat`, `request`, `params`, `options`, `message`

These are automatically provided by the BB runtime in each execution.

### `user` — Current User

```js
user.id             // Internal BB user ID (integer)
user.telegramid     // Telegram user ID (integer)
user.first_name     // First name
user.last_name      // Last name (may be empty)
user.username       // Telegram @username (may be empty)
user.language_code  // e.g. "en", "ru"
```

> Always check `if (!user) { return; }` at the top of commands that require a user context.

### `chat` — Current Chat

```js
chat.id         // Internal BB chat ID
chat.chatid     // Telegram chat ID
chat.chat_type  // "private", "group", "supergroup", "channel"
chat.title      // Chat title (groups/channels)
```

### `request` — The Incoming Telegram Update

```js
request.message_id       // ID of the triggering message
request.callback_query_id  // ID of a callback query (button tap)
request.text             // Raw text of the message
```

### `params` — Text After the Command Name

If a user sends `/order 5`, then `params` is `"5"`:

```js
const qty = Number(params || "0");
```

### `message` — The User's Text (in Wait for Answer)

When **Wait for answer** is ON and the user replies, the reply text is in `message`:

```js
const input = (message || "").trim();
```

### `options` — Data Passed from Another Command or Callback

When you use `Bot.runCommand("/cmd", { key: "val" })` or `on_result: "/cmd"`, the receiving command reads:

```js
options.key          // "val"
options.result       // Telegram API result (in on_result callbacks)
options.error        // Error info (in on_error callbacks)
options.bb_options   // Extra context passed via bb_options field
```

### `content` — Webhook / HTTP Response Body

In webhook receiver commands and HTTP success callbacks:

```js
const data = JSON.parse(content);
```

---

## 12. Chaining Commands — `Bot.runCommand` & `Bot.run`

### Simple Chain

```js
Bot.sendMessage("Going to step 2...");
Bot.runCommand("/step2");
```

`/step2` runs immediately after with the same user/chat context.

### Pass Data to Next Command

```js
Bot.runCommand("/process_order", { amount: 500, item: "Gold" });
```

In `/process_order`:
```js
const amount = options.amount;  // 500
const item = options.item;      // "Gold"
```

### Delayed Execution

```js
Bot.run({
  command: "/send_reminder",
  run_after: 3600,    // 1 hour in seconds
  label: "reminder_" + user.telegramid,
  options: { message: "Don't forget your task!" }
});
```

### Cancel a Scheduled Command

```js
Bot.clearRunAfter({ label: "reminder_" + user.telegramid });
```

> ⚠️ `Bot.clearRunAfter()` without a label cancels ALL pending scheduled tasks for the bot.

---

## 13. Inline Keyboards & Callback Queries

### Creating an Inline Keyboard (Api style — recommended for complex layouts)

```js
Api.sendMessage({
  chat_id: user.telegramid,
  text: "Choose an action:",
  parse_mode: "HTML",
  reply_markup: {
    inline_keyboard: [
      [
        { text: "✅ Option A", callback_data: "/option_a" },
        { text: "❌ Option B", callback_data: "/option_b" }
      ],
      [
        { text: "🌐 Visit Site", url: "https://example.com" }
      ],
      [
        { text: "💳 Pay", web_app: { url: "https://pay.example.com" } }
      ]
    ]
  }
});
```

### Handling the Callback (Button Press)

When a user taps a button with `callback_data: "/option_a"`, BB runs the `/option_a` command. Inside that command:

```js
// Dismiss the loading indicator on the button
Api.answerCallbackQuery({
  callback_query_id: request.callback_query_id,
  text: "Selected!"
});

// Do your logic
Bot.sendMessage("You chose Option A!");
```

### Creating Inline Keyboard (Bot style — simpler)

```js
Bot.sendInlineKeyboard([
  [
    { title: "Option A", command: "/option_a" },
    { title: "Option B", command: "/option_b" }
  ],
  [
    { title: "Visit Site", url: "https://example.com" }
  ]
], "Choose an action:");
```

> **Format difference summary:**
> - `Bot.sendInlineKeyboard` → `{ title: "...", command: "/..." }`
> - `Api.sendMessage` → `{ text: "...", callback_data: "/..." }`
> Never mix these formats.

---

## 14. Reply Keyboards

A reply keyboard shows persistent buttons below the text input. Tapping a button sends that button's text as a message.

### Via Command Editor (Keyboard field)

In the command editor's **Keyboard** field:
```
Button1,Button2
Button3,Button4
```
`\n` or a new line starts a new row.

### Via BJS

```js
Bot.sendKeyboard("🛍 Shop,📦 Orders\n⚙️ Settings,❓ Help", "Main Menu:");
```

### Remove Reply Keyboard

```js
Api.sendMessage({
  chat_id: chat.chatid,
  text: "Keyboard removed.",
  reply_markup: { remove_keyboard: true }
});
```

### Connecting Buttons to Commands via Aliases

Since tapping a button sends the button text, create a command with that exact text as an alias:

- Button label: `🛍 Shop`
- Command `/shop` with alias `🛍 Shop`
- When user taps the button, the text "🛍 Shop" is sent and BB runs `/shop`

---

## 15. Saving & Using Message IDs (Edit / Delete Messages)

This is a critical pattern for creating dynamic messages (e.g., payment links, processing indicators).

### Step 1 — Send a message and capture its ID

```js
Api.sendMessage({
  text: "<b>Processing...</b>",
  parse_mode: "HTML",
  on_result: "/save_message_id"
});
```

### Step 2 — Save the message ID in the callback

**Command `/save_message_id`:**
```js
if (!options || !options.result) { return; }
User.setProp("last_msg_id", options.result.message_id, "string");
```

### Step 3a — Edit that message later

```js
const msgId = User.getProp("last_msg_id");
Api.editMessageText({
  chat_id: user.telegramid,
  message_id: msgId,
  text: "<b>Done! Here is your result.</b>",
  parse_mode: "HTML"
});
```

### Step 3b — Delete that message later

```js
const msgId = User.getProp("last_msg_id");
try {
  Api.deleteMessage({
    chat_id: user.telegramid,
    message_id: msgId
  });
} catch(e) {}
```

### Deleting Multiple Messages at Once

```js
try {
  Api.deleteMessage({ message_id: User.getProp("msg_id_1") });
  Api.deleteMessage({ message_id: User.getProp("msg_id_2") });
} catch(e) {}
```

> Wrap delete operations in `try/catch` — Telegram will error if a message was already deleted or is too old (>48h).

---

## 16. Webhooks Library — Receiving External Requests

The **Webhooks** library lets external services (payment gateways, GitHub, etc.) send data into your bot.

### Installation

1. Open your bot → **Libraries** → **Go to install**
2. Find and install **Webhooks**

### Generate a Webhook URL

Run this in a setup command to get a URL for a specific command:

```js
const url = Libs.Webhooks.getUrlFor({
  command: "/payment_received",
  user_id: user.id    // Internal BB user ID
});
Bot.sendMessage("Webhook URL: " + url);
```

### Receiving the Webhook

**Command `/payment_received`** (this is triggered when the URL is called):
```js
if (!options || options.method !== "POST") { return; }
let data;
try {
  data = JSON.parse(content);
} catch(e) { return; }

// Validate and process
if (data.status === "paid") {
  const amount = data.amount;
  User.setProp("balance", (User.getProp("balance") || 0) + Number(amount), "integer");
  Bot.sendMessage("Payment received: " + amount);
}

// Acknowledge to the external service
WebApp.render({ content: { ok: true }, mime_type: "application/json" });
```

### Available in the webhook command

```js
content          // Raw request body (string)
options.method   // "POST", "GET", etc.
options.headers  // Request headers
options.params   // Query string params
options.ip       // Sender IP address
```

> ⚠️ **Always validate webhook payloads** with a signature or secret key. Anyone who has the URL can call it.

---

## 17. All Available Libraries

Libraries are reusable modules installed per-bot. Some are built into the runtime (no install needed); others must be installed from the Store.

### Core Libraries (No Install Needed)

#### `ResLib` — Numeric Resources

Manage numeric values like coins, points, XP, energy. Supports per-user and per-chat scope, and optional time-based growth (e.g. energy regeneration).

```js
// Get current user's resource
const bal = ResLib.userRes("balance");
bal.value()          // Read current value (returns number)
bal.add(100)         // Increase by 100
bal.remove(50)       // Decrease by 50 (will not go below 0 by default)
bal.set(1000)        // Set to absolute value
bal.exist()          // Returns true if the resource has been set before

// Access another user's resource by their Telegram ID
const otherBal = ResLib.anotherUserRes("balance", targetTelegramId);
otherBal.add(50);
otherBal.value();

// Chat-scoped resource (shared within a chat/group)
const chatPot = ResLib.chatRes("pot");
chatPot.add(10);

// Resource with time-based growth (e.g. energy regenerates 1 per minute)
// Configure in the resource's contract via Admin Panel
// Then simply read — it auto-calculates growth since last update
const energy = ResLib.userRes("energy");
Bot.sendMessage("Energy: " + energy.value());
```

**Checking before spending (safe pattern):**

```js
const bal = ResLib.userRes("balance");
const cost = 50;
if (bal.value() < cost) {
  Bot.sendMessage("❌ Not enough coins. You need " + cost + " coins.");
  return;
}
bal.remove(cost);
Bot.sendMessage("✅ Purchased! Remaining: " + bal.value());
```

> **Old name:** `Libs.ResourcesLib` — still works but `ResLib` is preferred in new code.

#### `RefLib` — Referral Links

Track user referrals. When a user starts the bot via a referral link (e.g. `https://t.me/YourBot?start=ref_12345`), BB automatically records who referred them. `RefLib` wraps this tracking system.

**How referral links work:**
- Each user gets a unique link generated by `RefLib.getLink()`.
- When a new user clicks that link and starts the bot, BB records the referrer.
- In the `/start` command BJS, `RefLib.getAttractedBy()` returns the referrer's user object (or `null` if not referred).
- This is a **one-time** record — only the first time the new user starts the bot.

```js
// Get the current user's unique referral link
const link = RefLib.getLink();
Bot.sendMessage("🔗 Your referral link:\n" + link);

// Get who referred the current user (returns user object or null)
const referrer = RefLib.getAttractedBy();
if (referrer) {
  // Notify the referrer
  Bot.sendMessageToChatWithId(referrer.telegramid, "🎉 Someone joined using your link!");
  // Give them a bonus
  const refBal = ResLib.anotherUserRes("balance", referrer.telegramid);
  refBal.add(100);
}

// Get how many users this person has referred
const count = RefLib.getCount();
Bot.sendMessage("You have referred " + count + " users.");
```

**Deep Links — Passing a start parameter:**

When a user clicks a referral link, the start parameter is available in `params` inside `/start`:

```js
// Command: /start
// If the user arrived via a referral link, params contains the referral data
const startParam = params || "";
if (startParam.startsWith("ref_")) {
  // RefLib handles this automatically — just call getAttractedBy()
  const referrer = RefLib.getAttractedBy();
  if (referrer && !User.getProp("ref_bonus_given")) {
    ResLib.anotherUserRes("balance", referrer.telegramid).add(100);
    User.setProp("ref_bonus_given", "true", "string");
    Bot.sendMessageToChatWithId(referrer.telegramid, "✅ +100 coins for your referral!");
  }
}
Bot.sendMessage("Welcome to the bot! 🎉");
```

> **Note:** `Libs.ReferralLib` is the old name — use `RefLib` in new code. Both access the same data.

#### `TopBoardLib` — Leaderboards

```js
// Add/update score for current user
TopBoardLib.addScore("my_board", user.telegramid, scoreValue);

// Get top N users
const top = TopBoardLib.getTop("my_board", 10);
for (let i = 0; i < top.length; i++) {
  Bot.sendMessage((i+1) + ". " + top[i].telegramid + " — " + top[i].score);
}

// Reset the board
TopBoardLib.reset("my_board");
```

#### `CommonLib` — Display Name & Utilities

```js
// Get a formatted display name for a user
const name = CommonLib.getDisplayName(user);
Bot.sendMessage("Hello, " + name + "!");
```

#### `CryptoJS` — Hashing

```js
const hash = CryptoJS.MD5("somestring").toString();
const sha256 = CryptoJS.SHA256("data").toString();
const hmac = CryptoJS.HmacSHA256("data", "secret").toString();
```

#### `CurrencyQuote` — Exchange Rates

```js
// Get rate (cached)
const rate = CurrencyQuote.getRate("USD", "EUR");
Bot.sendMessage("1 USD = " + rate + " EUR");
```

---

### Installable Libraries (Install from Store)

#### `Libs.CooldownLib` — Rate Limiting / Cooldowns

Prevent spam and limit repeated actions.

```js
// Check if cooldown has passed (returns true if OK to proceed)
if (!Libs.CooldownLib.check("daily_bonus", 86400)) {
  Bot.sendMessage("⏳ You already claimed today. Come back in 24h.");
  return;
}
// If we get here, the action is allowed and cooldown is reset
bal.add(50);
Bot.sendMessage("✅ Daily bonus claimed!");
```

The second argument is the cooldown duration in **seconds**.

#### `Libs.Guard` — Restrict Commands to Admins

```js
// At the top of an admin-only command
if (!Libs.Guard.isVerifiedUser(user.telegramid)) {
  Bot.sendMessage("❌ Access denied.");
  return;
}
// Admin-only logic here
```

Setup: Configure admin Telegram IDs in the Guard library's Admin Panel.

#### `Libs.MembershipChecker` — Channel Subscription Check

```js
Libs.MembershipChecker.check({
  chat_id: "@yourchannel",
  user_id: user.telegramid,
  on_success: "/member_ok",
  on_fail: "/not_member"
});
```

**`/member_ok`:**
```js
Bot.sendMessage("✅ Thanks for subscribing!");
```

**`/not_member`:**
```js
Bot.sendMessage("❌ Please join @yourchannel first!");
```

> ⚠️ Do not use cached membership results for security decisions — always perform a fresh check.

#### `Libs.Webhooks` — External Webhooks

*(See Section 16 for full details)*

```js
const url = Libs.Webhooks.getUrlFor({ command: "/receive", user_id: user.id });
```

#### `Libs.Lang` — Multi-Language Support

```js
// Set user language
Libs.Lang.setUserLang("en");

// Get a translation
const text = Libs.Lang.get("welcome_message");
Bot.sendMessage(text);
```

Configure translations in the Admin Panel or via JSON files.

#### `SmartBot`, `SmartAmountDialog`, `SmartTasker` — Template Dialogs

`SmartBot` provides template-driven reply menus. `SmartAmountDialog` handles numeric input with validation and re-prompting. `SmartTasker` manages task/checklist progress per user. These have their own template definition format — see BB documentation for full SmartBot template syntax.

#### `Libs.OxaPayLibV1` — OxaPay Payment Integration

```js
// Create a payment invoice
Libs.OxaPayLibV1.createInvoice({
  amount: 10,
  currency: "USDT",
  description: "Premium access",
  on_success: "/payment_success",
  on_error: "/payment_error"
});
```

Configure your OxaPay API key in the library's Admin Panel. Verify signed callbacks before crediting users.

---

### Library Quick Reference Table

| Library | Object in Code | Install Required | Purpose |
|---------|---------------|-----------------|---------|
| Resources | `ResLib` | ❌ Built-in | Numeric values (balance, XP, energy) |
| Referrals | `RefLib` | ❌ Built-in | Referral links and tracking |
| Leaderboard | `TopBoardLib` | ❌ Built-in | Top N scores |
| Display Name | `CommonLib` | ❌ Built-in | Formatted user names |
| Hashing | `CryptoJS` | ❌ Built-in | MD5, SHA256, HMAC |
| Exchange Rates | `CurrencyQuote` | ❌ Built-in | Cached forex rates |
| Cooldowns | `Libs.CooldownLib` | ✅ Store | Rate limiting |
| Admin Guard | `Libs.Guard` | ✅ Store | Restrict to admin IDs |
| Channel Check | `Libs.MembershipChecker` | ✅ Store | Subscription verification |
| Webhooks | `Libs.Webhooks` | ✅ Store | Receive external HTTP |
| Language | `Libs.Lang` | ✅ Store | Multi-language messages |
| SmartBot | `SmartBot` | ✅ Store | Template menus & dialogs |
| OxaPay | `Libs.OxaPayLibV1` | ✅ Store | Crypto payment gateway |

---

## 17b. Additional Platform Features

### Lists — Paginated Collections of Properties or Users

BB's **Lists** system lets you store collections that grow beyond a single property. Use when you need paginated data or membership sets.

```js
// Add current user to a named list
User.addToGroup("vip_members");

// Check if current user is in a group
const group = User.getGroup();
if (group === "vip_members") {
  Bot.sendMessage("You are a VIP!");
}

// Remove from group
User.removeGroup();
```

> `User.addToGroup(name)` stores a **single group string** per user — adding a new group replaces the previous one. For multiple memberships, use a `"json"` property containing an array.

### Admin Panel — Configurable Settings Form

The **Admin Panel** is a settings form visible to the bot owner in the mobile app. Use it to store API keys, configuration, and settings that should be editable without touching code.

```js
// Read a value from the Admin Panel field named "api_key"
const apiKey = Bot.getProp("api_key");

// Read a field with a fallback
const welcomeText = Bot.getProp("welcome_text", "Welcome to the bot!");
Bot.sendMessage(welcomeText);
```

To set up Admin Panel fields: go to your bot → **Admin Panel** → fill in fields and save. Each field name becomes a `Bot.getProp("field_name")` readable value.

### WebApp — Return JSON from BJS to External Callers

`WebApp` is used inside webhook receiver commands to send a JSON response back to the caller:

```js
// Acknowledge a webhook call with JSON
WebApp.render({
  content: { ok: true, received: true },
  mime_type: "application/json"
});

// Return data
WebApp.render({
  content: { status: "success", balance: bal.value() },
  mime_type: "application/json"
});
```

> `WebApp.render` must be used inside a webhook-triggered command. Current BB backend only supports JSON responses — `text/plain` is not supported.

### Deep Links — Passing Parameters via Start Link

You can pass a parameter to your bot via the start link: `https://t.me/YourBot?start=YOURPARAM`

In the `/start` command, that parameter is available in `params`:

```js
// Command: /start
const startParam = (params || "").trim();

if (startParam === "premium") {
  Bot.sendMessage("You came from our premium invite link!");
} else if (startParam.startsWith("ref_")) {
  // Handle referral (RefLib does this automatically too)
  Bot.sendMessage("You joined via a referral!");
} else {
  Bot.sendMessage("Welcome to the bot!");
}
```

Deep links are useful for: referral tracking, campaign tracking, direct feature access, and onboarding flows.

### Inactivity Reminder — Message Users After Silence

Send a follow-up message when a user hasn't interacted for a set period:

```js
// Command: /start or any active command — schedule a reminder
Bot.clearRunAfter({ label: "inactive_" + user.telegramid });  // cancel old one first
Bot.run({
  command: "/inactivity_reminder",
  run_after: 86400,   // 24 hours
  label: "inactive_" + user.telegramid
});
```

**Command `/inactivity_reminder`:**
```js
// This runs if the user hasn't triggered the cancellation
Bot.sendMessageToChatWithId(user.telegramid,
  "👋 Hey! We miss you. Come back and check what's new.");
```

Reset the timer on every user interaction by calling `Bot.clearRunAfter` at the start of active commands, then scheduling again.

---

## 18. Economy System Pattern

A full economy system uses `ResLib` for balances, `CooldownLib` for rate limiting, `TopBoardLib` for leaderboards, and `RefLib` for referral bonuses. Below is a complete blueprint.

### Check Balance

```js
// Command: /balance
const bal = ResLib.userRes("balance");
Bot.sendMessage(
  "💰 <b>Your Balance</b>\n\n" +
  "Coins: <code>" + bal.value() + "</code>",
  { parse_mode: "HTML" }
);
```

### Daily Bonus

```js
// Command: /daily
if (!Libs.CooldownLib.check("daily_" + user.telegramid, 86400)) {
  Bot.sendMessage("⏳ Already claimed today. Come back in 24 hours.");
  return;
}
const bal = ResLib.userRes("balance");
bal.add(100);
Bot.sendMessage("✅ You received 100 coins! Balance: " + bal.value());
```

### Transfer Coins

```js
// Command: /send (Wait for answer ON — prompt asks for: amount receiverId)
const parts = (message || "").trim().split(" ");
const amount = Number(parts[0]);
const receiverId = parts[1];

if (!amount || amount <= 0) {
  Bot.sendMessage("❌ Invalid amount.");
  return;
}
if (String(receiverId) === String(user.telegramid)) {
  Bot.sendMessage("❌ Cannot send to yourself.");
  return;
}

const myBal = ResLib.userRes("balance");
if (myBal.value() < amount) {
  Bot.sendMessage("❌ Insufficient balance.");
  return;
}

const theirBal = ResLib.anotherUserRes("balance", receiverId);
if (!theirBal.exist()) {
  Bot.sendMessage("❌ That user is not registered.");
  return;
}

myBal.remove(amount);
theirBal.add(amount);
Bot.sendMessage("✅ Sent " + amount + " coins to " + receiverId);
Bot.sendMessageToChatWithId(receiverId, "💰 You received " + amount + " coins from " + user.telegramid);
```

### Shop / Purchase Item

```js
// Command: /buy_premium
const COST = 500;
const bal = ResLib.userRes("balance");

if (bal.value() < COST) {
  Bot.sendMessage("❌ You need " + COST + " coins. Your balance: " + bal.value());
  return;
}
if (User.getProp("is_premium") === "true") {
  Bot.sendMessage("✅ You already have Premium!");
  return;
}

bal.remove(COST);
User.setProp("is_premium", "true", "string");
Bot.sendMessage("🎉 Premium activated! Your new balance: " + bal.value() + " coins.");
```

### Admin — Top Up a User's Balance

```js
// Command: /topup (admin only, Wait for answer ON)
// Answer: "Enter: telegramId amount"
const ADMIN_IDS = ["YOUR_TELEGRAM_ID"];
if (!ADMIN_IDS.includes(String(user.telegramid))) {
  Bot.sendMessage("❌ Access denied.");
  return;
}
const parts = (message || "").trim().split(" ");
const targetId = parts[0];
const amount = Number(parts[1]);
if (!targetId || isNaN(amount) || amount <= 0) {
  Bot.sendMessage("❌ Invalid format. Use: telegramId amount");
  return;
}
const targetBal = ResLib.anotherUserRes("balance", targetId);
targetBal.add(amount);
Bot.sendMessage("✅ Added " + amount + " coins to user " + targetId);
Bot.sendMessageToChatWithId(targetId, "💰 Admin added " + amount + " coins to your balance!");
```

---

## 19. Referral System Pattern

A complete referral system has three parts: generating the link, handling the new user, and rewarding the referrer.

### Step 1 — User gets their referral link

```js
// Command: /refer
const link = RefLib.getLink();
const count = RefLib.getCount();
Bot.sendMessage(
  "🔗 <b>Your Referral Link</b>\n\n" +
  "<code>" + link + "</code>\n\n" +
  "👥 Total referrals: <b>" + count + "</b>\n" +
  "💰 Earn 100 coins per referral!",
  { parse_mode: "HTML" }
);
```

### Step 2 — New user starts the bot via referral link

```js
// Command: /start
// Only give bonus once per new user
if (!User.getProp("joined")) {
  User.setProp("joined", "true", "string");

  const referrer = RefLib.getAttractedBy();
  if (referrer) {
    // Give bonus to referrer
    if (!User.getProp("ref_bonus_given")) {
      const refBal = ResLib.anotherUserRes("balance", referrer.telegramid);
      refBal.add(100);
      User.setProp("ref_bonus_given", "true", "string");
      Bot.sendMessageToChatWithId(
        referrer.telegramid,
        "🎉 You earned 100 coins! Someone joined using your referral link."
      );
    }
    Bot.sendMessage("✅ Welcome! You joined via a referral link.");
  } else {
    Bot.sendMessage("✅ Welcome to the bot!");
  }
} else {
  Bot.sendMessage("Welcome back!");
}
```

### Step 3 — Check your referral count (leaderboard style)

```js
// Command: /refstats
const count = RefLib.getCount();
const bal = ResLib.userRes("balance");
Bot.sendMessage(
  "📊 <b>Your Stats</b>\n\n" +
  "👥 Referrals: <b>" + count + "</b>\n" +
  "💰 Balance: <b>" + bal.value() + " coins</b>",
  { parse_mode: "HTML" }
);
```

---

## 20. Leaderboard Pattern

```js
// Update leaderboard when balance changes
const bal = ResLib.userRes("balance");
TopBoardLib.addScore("richest", user.telegramid, bal.value());
```

```js
// Command: /top
const top = TopBoardLib.getTop("richest", 10);
let msg = "🏆 <b>Top 10 Richest Users</b>\n\n";
for (let i = 0; i < top.length; i++) {
  msg += (i + 1) + ". ID " + top[i].telegramid + " — " + top[i].score + " coins\n";
}
Bot.sendMessage(msg, { parse_mode: "HTML" });
```

---

## 21. Broadcast System

Send a message to all users of your bot.

### Using `Bot.runAll`

```js
// Command: /broadcast_send (admin only)
Bot.runAll({
  command: "/send_to_user",
  options: { text: "📢 Important announcement!" },
  include: "private"  // only private chats
});
```

**Command `/send_to_user`:**
```js
Bot.sendMessage(options.text);
```

### Manual Broadcast Groups (Alternative)

Store user IDs in arrays (200 per group) and iterate:

```js
// Registration: store user in broadcast list
let i = 0;
while (Bot.getProp("Broadcast_" + i)) { i++; }
let group = Bot.getProp("Broadcast_" + (i - 1 >= 0 ? i - 1 : 0)) || [];
if (!group.includes(user.telegramid)) {
  if (group.length >= 200) {
    group = [];
    Bot.setProp("Broadcast_" + i, group, "json");
  } else {
    group.push(user.telegramid);
    Bot.setProp("Broadcast_" + (i - 1 >= 0 ? i - 1 : 0), group, "json");
  }
}
```

---

## 22. Scheduling & Background Work

### Run a Command After a Delay

```js
Bot.run({
  command: "/remind_user",
  run_after: 1800,   // 30 minutes
  label: "remind_" + user.telegramid,
  options: { note: "Check your tasks!" }
});
```

### Periodic Commands (Auto Retry)

In the command editor, set **Auto retry time in seconds** (e.g. `3600` for hourly). This runs the BJS code periodically without user input. You must set an explicit destination context.

> ⚠️ Auto Retry commands have **no user context** — do not use `user.*` inside them.

### Cancel a Pending Scheduled Command

```js
Bot.clearRunAfter({ label: "remind_" + user.telegramid });
```

---

## 23. HTML Formatting in Messages

Use `parse_mode: "HTML"` to format messages with HTML tags.

| Format | Syntax | Example |
|--------|--------|---------|
| Bold | `<b>text</b>` | **text** |
| Italic | `<i>text</i>` | *text* |
| Underline | `<u>text</u>` | <u>text</u> |
| Strikethrough | `<s>text</s>` | ~~text~~ |
| Inline code | `<code>text</code>` | `text` |
| Spoiler | `<tg-spoiler>text</tg-spoiler>` | Hidden text |
| Blockquote | `<blockquote>text</blockquote>` | Quoted |
| Expandable quote | `<blockquote expandable>text</blockquote>` | Collapsed |
| Code block | `<pre><code class="language-js">code</code></pre>` | Code |
| Link | `<a href="https://example.com">text</a>` | Hyperlink |

### Example

```js
Bot.sendMessage(
  "<b>Your Balance</b>\n\n" +
  "💰 Coins: <code>" + balance + "</code>\n" +
  "<i>Updated just now</i>",
  { parse_mode: "HTML" }
);
```

> Always use `parse_mode: "HTML"` explicitly when using HTML tags. When sending user-supplied text, use `parse_mode: null` to prevent Markdown/HTML injection.

---

## 24. Security Best Practices

### Always Validate User Input

```js
const amount = Number(message);
if (isNaN(amount) || amount <= 0 || amount > 10000) {
  Bot.sendMessage("❌ Invalid amount. Enter a number between 1 and 10000.");
  Bot.runCommand("/ask_amount");  // Re-ask
  return;
}
```

### Restrict Admin Commands

```js
const ADMIN_IDS = ["123456789", "987654321"];
if (!ADMIN_IDS.includes(String(user.telegramid))) {
  Bot.sendMessage("❌ Access denied.");
  return;
}
```

Or use `Libs.Guard` for more robust admin restriction.

### Use Cooldowns to Prevent Spam

```js
if (!Libs.CooldownLib.check("action_" + user.telegramid, 60)) {
  Bot.sendMessage("⏳ Please wait before doing that again.");
  return;
}
```

### Validate Webhook Signatures

```js
// In /payment_received webhook handler
const receivedSig = options.headers["X-Signature"];
const expectedSig = CryptoJS.HmacSHA256(content, "YOUR_SECRET_KEY").toString();
if (receivedSig !== expectedSig) {
  return;  // Silently reject invalid requests
}
```

### Never Trust `params` or `options` Blindly

Always sanitize and validate before using in logic, especially in financial operations.

### Keep API Keys in Admin Panel

Store sensitive keys in the bot's Admin Panel fields, not hardcoded in BJS:

```js
const apiKey = Bot.getProp("api_key");
```

---

## 25. Errors & Debugging

### The `!` Command (Error Handler)

Create a command named `!` — it runs whenever a BJS error occurs in any command:

```js
// Command: !
Bot.sendMessage("⚠️ An error occurred. Please try again.");
// Optionally log it:
Bot.setProp("last_error", String(error_message || "unknown"), "string");
```

### Check Syntax in the Editor

In the BJS editor, tap **⋮ → Check** to validate syntax before saving.

### Use `Bot.inspect` for Debugging

```js
Bot.inspect(user);        // Sends full user object as JSON
Bot.inspect(options);     // Inspect callback options
Bot.inspect(request);     // Inspect incoming Telegram update
```

### Error Log

In the mobile app: **Your Bot → Errors** — shows a log of recent runtime failures with command name and line number.

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| `user is not defined` | The trigger has no user context; guard with `if (!user) { return; }` |
| `options is not defined` | Callback called manually without options; guard with `if (!options) { return; }` |
| `Libs.SomeLib is not defined` | Library not installed for this bot, or wrong capitalization |
| `Cannot read property of null` | Data was null/undefined; use fallback: `|| {}` or `|| []` |
| Silent failure on `Api.sendMessage` | User may have blocked the bot; use `on_error` to handle it |
| `0` or `false` read back as default | Wrap scalars in JSON object (see Section 10) |

---

## 26. Complete Syntax Rules & Common Mistakes

### ✅ Correct Patterns

```js
// String comparison
if (message === "yes") { ... }

// Number conversion and validation
const n = Number(params);
if (isNaN(n) || n < 1) { return; }

// Safe property access
const name = user && user.first_name ? user.first_name : "User";

// Safe JSON parse
let data;
try { data = JSON.parse(content); } catch(e) { return; }

// Array with explicit type
Bot.setProp("list", ["a", "b", "c"], "json");

// Read array safely
const arr = Bot.getProp("list") || [];

// Use string for stored IDs
User.setProp("msg_id", options.result.message_id, "string");
const id = User.getProp("msg_id");

// Trim user input
const input = (message || "").trim();

// Guard against no user
if (!user) { return; }

// Guard against no options in callback
if (!options || !options.result) { return; }
```

### ❌ Incorrect Patterns (Will Cause Errors)

```js
// ❌ Never use async/await
const result = await Bot.sendMessage("hi");

// ❌ Never use fetch
fetch("https://api.example.com").then(r => r.json());

// ❌ Never use setTimeout
setTimeout(() => { Bot.sendMessage("hi"); }, 1000);

// ❌ Never use require
const axios = require("axios");

// ❌ Never mix Bot inline keyboard format with Api format
Bot.sendInlineKeyboard([{ text: "btn", callback_data: "/cmd" }], "Pick:");
// ↑ Wrong! Bot.sendInlineKeyboard uses { title, command }, not { text, callback_data }

// ❌ Never use == for equality check
if (message == 1) { ... }  // Use === instead

// ❌ Never trust raw user input as a number
const n = message;   // It's a string! Always Number(message)

// ❌ Never ignore HTTP status
// In success callback, always check: if (Number(http_status) !== 200) { ... }

// ❌ Never use Http (lowercase H) — it's HTTP (uppercase)
Http.post(...)  // Wrong!
HTTP.post(...)  // Correct
```

---

## Quick Reference Card

### Send a Message
```js
Bot.sendMessage("text");
Bot.sendMessage("<b>bold</b>", { parse_mode: "HTML" });
```

### Read / Write Property
```js
User.setProp("key", value);
const v = User.getProp("key", "default");
Bot.setProp("key", value, "json");
const v = Bot.getProp("key");
```

### Wait for Answer Flow
1. Create command with **Answer** as prompt and **Wait for answer: ON**
2. In BJS: read `message` for the user's reply

### Capture Message ID
```js
Api.sendMessage({ text: "...", on_result: "/save_id" });
// In /save_id:
User.setProp("msg_id", options.result.message_id, "string");
```

### Edit a Sent Message
```js
Api.editMessageText({ message_id: User.getProp("msg_id"), text: "New text", parse_mode: "HTML" });
```

### Delete a Message
```js
try { Api.deleteMessage({ message_id: User.getProp("msg_id") }); } catch(e) {}
```

### HTTP POST
```js
HTTP.post({ url: "...", body: JSON.stringify({...}), headers: {"Content-Type":"application/json"}, success: "/ok", error: "/err" });
```

### Webhook URL
```js
const url = Libs.Webhooks.getUrlFor({ command: "/receive", user_id: user.id });
```

### Chain Commands
```js
Bot.runCommand("/next_step");
Bot.runCommand("/next_step", { key: "value" });
```

### Delay a Command
```js
Bot.run({ command: "/later", run_after: 3600, label: "my_label" });
```

---

## Contact & Support

| | |
|-|-|
| **Author** | MD Jaid Bin Siyam (Siyam) |
| **Telegram** | [@BD_Prime_Minister](https://t.me/BD_Prime_Minister) |
| **Channel** | [@Hyper_10_Squad](https://t.me/Hyper_10_Squad) |
| **Email** | support@skypaybd.top |
| **WhatsApp** | +8801761844968 |
| **Phone (Call)** | *(contact via Telegram or WhatsApp)* |
| **Address** | Panchagarh, Rangpur, Bangladesh |
| **Payment Gateway** | [skypaybd.top](https://skypaybd.top/) |
| **Digital Products** | [chapri.shop](https://chapri.shop/) |
| **GitHub** | [github.com/SkyPayBD](https://github.com/SkyPayBD) |
| **YouTube** | [youtube.com/@SkyPayBD](https://youtube.com/@SkyPayBD) |
| **Facebook** | [facebook.com/siyamahmedjsx](https://facebook.com/siyamahmedjsx) |

---

*This document is the official Bots.Business (BB) AI Agent training guide, created by MD Jaid Bin Siyam (Siyam) for helping users of SkyPayBD build Telegram bots. All content, code examples, and explanations are original. © 2026–2027 SkyPayBD. All rights reserved.*
