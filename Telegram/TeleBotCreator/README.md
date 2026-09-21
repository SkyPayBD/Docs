# TeleBot Creator (TBC) — Complete AI Agent Training Documentation

> **Purpose:** This document is a complete and accurate reference guide for AI agents, developers, and bot creators who want to build powerful Telegram bots using the **TeleBot Creator (TBC)** platform. It covers every feature, syntax rule, built-in library, pattern, and best practice needed to write correct TPY code from scratch.
>
> **Language:** TPY (Telebot Python) — a custom, sandboxed Python-based scripting language. NOT standard Python.
>
> **Platform Website:** [https://telebotcreator.com/](https://telebotcreator.com/)
> **Full Docs:** [https://help.telebotcreator.com](https://help.telebotcreator.com)
> **Platform Version:** 7.1.2 · Telegram Bot API 10.1

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

  This document was created to help users build Telegram bots
  using the TeleBot Creator (TBC) platform with correct TPY syntax.
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

1. [What is TeleBot Creator (TBC)?](#1-what-is-telebot-creator-tbc)
2. [Platform Overview & Bot Setup](#2-platform-overview--bot-setup)
3. [TPY Language — What You Can and Cannot Do](#3-tpy-language--what-you-can-and-cannot-do)
4. [Pre-defined Global Variables](#4-pre-defined-global-variables)
5. [Commands — The Core Building Block](#5-commands--the-core-building-block)
6. [CRITICAL: How to Stop a Command Early (`raise ReturnCommand`)](#6-critical-how-to-stop-a-command-early)
7. [User Data Management — `User.*`](#7-user-data-management--user)
8. [Global Bot Data Management — `Bot.*`](#8-global-bot-data-management--bot)
9. [Sending Messages — `bot.sendMessage` & `bot.replyText`](#9-sending-messages)
10. [Keyboard Support — Inline & Reply Keyboards](#10-keyboard-support)
11. [Command Chaining — `Bot.handleNextCommand` & `Bot.runCommand`](#11-command-chaining)
12. [Scheduled Commands — `Bot.runCommandAfter`](#12-scheduled-commands)
13. [HTTP Requests — `HTTP` module & `libs.customHTTP`](#13-http-requests)
14. [Wait for Answer — Multi-Step Input Collection](#14-wait-for-answer--multi-step-input-collection)
15. [Sending Media — Photos, Documents, Videos, Audio](#15-sending-media)
16. [Editing & Deleting Messages](#16-editing--deleting-messages)
17. [Channel Membership Check — `MembershipCheck`](#17-channel-membership-check)
18. [Admin-Specific Functions](#18-admin-specific-functions)
19. [Broadcasting — `Bot.broadcast`](#19-broadcasting)
20. [Webhook Integration — `libs.Webhook`](#20-webhook-integration)
21. [Resources Library — `libs.Resources`](#21-resources-library--libsresources)
22. [All Available Libraries (30+)](#22-all-available-libraries-30)
23. [Referral System Pattern](#23-referral-system-pattern)
24. [Economy System Pattern](#24-economy-system-pattern)
25. [AI Integration — OpenAI & Gemini](#25-ai-integration--openai--gemini)
26. [HTML & Markdown Formatting](#26-html--markdown-formatting)
27. [Error Handling & Debugging](#27-error-handling--debugging)
28. [Complete Syntax Rules & Common Mistakes](#28-complete-syntax-rules--common-mistakes)
29. [Full Example — /start Command with All Features](#29-full-example--start-command-with-all-features)
30. [Quick Reference Card](#30-quick-reference-card)

---

## 1. What is TeleBot Creator (TBC)?

**TeleBot Creator (TBC)** is a free, cloud-based platform for building, hosting, and managing Telegram bots. Over **80,000 active bots** run on TBC, serving more than **20 million Telegram users** worldwide.

### Key Characteristics

- Uses **TPY (Telebot Python)** — a custom Python-like scripting language with 30+ built-in libraries.
- **100% Free** — 100,000 execution points per month per account (each command run costs 1 point).
- You write code in TBC's online editor; **no server setup required**.
- Built-in support for AI (GPT-4, Gemini), crypto payments, blockchain, broadcasting, webhooks, and more.
- Execution timeout: **up to 160 seconds** per command.

### Platform Statistics (2026)

| Metric | Value |
|--------|-------|
| Active Bots | 80,000+ |
| Total Bots Created | 150,000+ |
| Telegram Users Served | 20,000,000+ |
| TBC Platform Version | 7.1.2 |
| Telegram Bot API | 10.1 |
| Built-in Libraries | 30+ |

---

## 2. Platform Overview & Bot Setup

### Step-by-Step: Creating Your First Bot

1. **Register** — Create a free account at [telebotcreator.com/register](https://telebotcreator.com/register).
2. **Get a Bot Token** — Message [@BotFather](https://t.me/BotFather) on Telegram, use `/newbot`, and copy the API token.
3. **Add Your Bot** — On the TBC dashboard, click **"Add New Bot"**, paste your token, and click **"Create Bot"**.
4. **Write Your First Command** — Click on your bot → Commands → add `/start`:
   ```python
   bot.sendMessage("Hello! Welcome to my bot 🚀")
   ```
5. **Start Your Bot** — Click Start. Your bot is now live on Telegram!

### Dashboard Navigation

| Section | Purpose |
|---------|---------|
| **Dashboard** | Bot status, start/stop controls |
| **Commands** | Create, edit, and manage bot commands |
| **Users** | View all users who have interacted with the bot |
| **Errors** | Runtime error log for debugging |
| **Settings** | Bot configuration and token management |
| **Libraries** | Manage installed libraries |

---

## 3. TPY Language — What You Can and Cannot Do

TPY is a **sandboxed, synchronous Python-like language**. It looks like Python but runs in TBC's controlled environment.

### ✅ What You CAN Use

```python
# Variables
name = "Alice"
count = 0
balance = 100.5
is_active = True

# f-strings (template strings)
first_name = message.from_user.first_name
bot.sendMessage(f"Hello {first_name}!")

# Conditionals
if balance > 0:
    bot.sendMessage("You have funds!")
elif balance == 0:
    bot.sendMessage("Empty balance.")
else:
    bot.sendMessage("Negative balance!")

# Loops
for i in range(10):
    pass  # do something

while count < 5:
    count += 1

# Functions (you can define your own)
def greet(name):
    return "Hello, " + name + "!"

# String operations
text = "Hello " + user_name
upper = text.upper()
lower = text.lower()
stripped = text.strip()
split = text.split(",")

# Type conversion
num = int("42")
flt = float("3.14")
txt = str(100)

# Lists and dicts
items = ["apple", "banana", "cherry"]
data = {"key": "value", "count": 5}

# try/except
try:
    result = int(user_input)
except ValueError:
    bot.sendMessage("Invalid input!")
except Exception as e:
    bot.sendMessage(f"Error: {str(e)}")

# Built-in functions
length = len(items)
total = sum([1, 2, 3])
maximum = max([1, 5, 3])
minimum = min([1, 5, 3])
rounded = round(3.14159, 2)
sorted_list = sorted(items)
```

### ❌ What You CANNOT Use

```python
# ❌ No standard Python imports — EVER
import requests       # Not allowed
import os             # Not allowed
import json           # Not allowed
from datetime import datetime  # Not allowed

# ❌ No eval() or exec() — sandbox blocks these
eval("1+1")           # Blocked for security

# ❌ No file system access
open("file.txt")      # Not available
os.path.exists(...)   # Not available

# ❌ No async/await
async def handler():  # Not supported
    await something()

# ❌ No bare return at top level (see Section 6!)
# This is the MOST COMMON MISTAKE:
if condition:
    return            # ❌ SyntaxError — crashes the whole command!

# ✅ CORRECT way to stop early:
if condition:
    raise ReturnCommand   # ✅ This is the right way
```

### ✅ Allowed Built-ins

`str`, `int`, `float`, `bool`, `dict`, `list`, `set`, `len()`, `all()`, `any()`, `sum()`, `min()`, `max()`, `round()`, `sorted()`, `reversed()`, `enumerate()`, `isinstance()`, `range()`, `abs()`, `zip()`, `ord()`, `map()`, `slice()`, `type()`, `tuple()`, `hash()`, `hex()`, `bin()`, `chr()`, `format()`, `getattr()`, `hasattr()`, `callable()`, `filter()`, `divmod()`, `pow()`, `iter()`, `next()`

---

## 4. Pre-defined Global Variables

These are automatically available in every TPY command — **never redefine them**:

| Variable | Type | Description |
|----------|------|-------------|
| `msg` | `str` | Raw text content of the incoming message |
| `message` | Object | Full Telegram update object (user, chat, message details) |
| `message.from_user.first_name` | `str` | User's first name |
| `message.from_user.last_name` | `str` | User's last name |
| `message.from_user.username` | `str` | User's @username |
| `message.from_user.id` | `int` | User's Telegram ID |
| `message.chat.id` | `int` | Chat ID |
| `bot` | Object | Low-level bot object (Telegram API methods) |
| `Bot` | Object | High-level bot object (TBC platform methods) |
| `User` | Object | User data storage class |
| `u` | `str/int` | Current user's Telegram ID (shorthand) |
| `params` | `str` | Text after command name (e.g., in `/start ref123`, params = `"ref123"`) |
| `options` | Object | Data passed from another command or webhook |
| `update_type` | `str` | Type of update: `"message"`, `"callback_query"`, etc. |
| `left_points` | `int` | Remaining execution points for this month |
| `bot_token` | `str` | The bot's Telegram API token |
| `bot_id` | `str` | The bot's TBC internal ID |
| `HTTP` | Object | Built-in HTTP client |
| `libs` | Object | Access to all libraries (`libs.Resources`, `libs.Random`, etc.) |
| `CSV` | Object | CSV file management |
| `base64` | Object | Base64 encoding/decoding |
| `hashlib` | Object | SHA256, MD5, and other hash functions |
| `time` | Object | Time utilities (including `time.sleep()`) |
| `re` / `regex` | Object | Regular expressions |
| `isNumeric` | Function | Check if a value is numeric |
| `MembershipCheck` | Object | Channel membership verification |
| `encodejson` | Function | Encode dict to JSON string |
| `jsondumps` | Function | Serialize Python object to JSON string |
| `bunchify` | Function | Convert dict to object (access via attributes) |

---

## 5. Commands — The Core Building Block

A **command** is the fundamental unit of behavior in TBC. Each command is triggered by a specific text or event.

### Command Trigger Types

| Trigger | Description |
|---------|-------------|
| `/start` | Runs when user types `/start` |
| `/help` | Runs when user types `/help` |
| `any_name` | Custom command — triggered by exact text match |
| `Button Label` | Runs when a reply keyboard button is tapped (text matches) |
| `*` | Wildcard — matches any unrecognized message (fallback) |
| `@` | Runs before any other command — for preprocessing/logging |
| `handler_callback_query` | Runs when inline button is tapped |

### Basic Command Example

```python
# Command: /start
first_name = message.from_user.first_name
bot.sendMessage(f"Hey {first_name}! Welcome to my bot. Use /help to see what I can do.")
```

### Command with Parameters

```python
# Command: /order
# User sends: /order 5
qty = params   # params = "5"
if not isNumeric(qty):
    bot.sendMessage("Please enter a valid quantity.")
    raise ReturnCommand
bot.sendMessage(f"You ordered: {qty} items.")
```

---

## 6. CRITICAL: How to Stop a Command Early

> ⚠️ **This is the single most common mistake in TBC.**

A TPY command runs as a **flat script**, NOT inside a function. Therefore, using a bare `return` at the top level is a **SyntaxError** that crashes the entire command.

### ❌ WRONG (Will crash the command):

```python
# Command: /check
balance = User.getData("balance") or 0
if int(balance) <= 0:
    bot.sendMessage("You have no balance.")
    return  # ❌ SyntaxError: 'return' outside function — command crashes entirely
```

### ✅ CORRECT (Use `raise ReturnCommand`):

```python
# Command: /check
balance = User.getData("balance") or 0
if int(balance) <= 0:
    bot.sendMessage("You have no balance.")
    raise ReturnCommand  # ✅ Stops execution cleanly

# This line only runs if balance > 0
bot.sendMessage(f"Your balance is {balance} coins.")
```

### All Valid Ways to Stop Early

```python
raise ReturnCommand   # ✅ Recommended
raise returnCommand   # ✅ Also works
raise returncommand   # ✅ Also works
```

### `return` Inside a Function Is Fine

```python
# return IS allowed inside a def you defined yourself
def validate(amount):
    if amount <= 0:
        return False   # ✅ This is fine — it's inside a function
    return True

if not validate(int(params or 0)):
    bot.sendMessage("Invalid amount.")
    raise ReturnCommand  # ✅ Stop the command at top level
```

---

## 7. User Data Management — `User.*`

TBC provides built-in methods to store, retrieve, and delete **per-user data** that persists across sessions.

### `User.saveData(key, value)`

```python
# Save a string
User.saveData("name", "Alice")

# Save a number (stored as-is)
User.saveData("balance", 500)

# Save a flag
User.saveData("verified", "true")

# Save a referral ID
User.saveData("ref_by", str(referrer_id))
```

### `User.getData(key)`

Returns the stored value, or `None` if not set.

```python
# Read a value
name = User.getData("name")

# Safe read with fallback
balance = int(User.getData("balance") or 0)

# Check if key exists
registered = User.getData("registered")
if registered is None:
    bot.sendMessage("New user!")
    raise ReturnCommand
```

### `User.deleteData(key)`

```python
# Delete a specific key
User.deleteData("temp_code")

# Clear session data after a flow completes
User.deleteData("pending_amount")
User.deleteData("pending_action")
```

### Complete User Profile Pattern

```python
# Command: /start — Registration check
registered = User.getData("registered")
if not registered:
    User.saveData("registered", "true")
    User.saveData("balance", 0)
    User.saveData("join_date", str(libs.DateAndTime.date_now()))
    bot.sendMessage("✅ Account created! Welcome.")
else:
    bot.sendMessage("Welcome back! 👋")
```

---

## 8. Global Bot Data Management — `Bot.*`

Use `Bot.*` for **global data** shared across all users and commands.

### `Bot.saveData(key, value)`

```python
# Store global settings
Bot.saveData("total_users", 1500)
Bot.saveData("maintenance_mode", "false")
Bot.saveData("admin_ids", "123456789,987654321")
```

### `Bot.getData(key)`

```python
# Read global data
total = Bot.getData("total_users")
bot.sendMessage("Total users: " + str(total or 0))

# Check maintenance mode
if Bot.getData("maintenance_mode") == "true":
    bot.sendMessage("Bot is under maintenance.")
    raise ReturnCommand
```

### `Bot.deleteData(key)`

```python
Bot.deleteData("old_broadcast_message")
```

### `Bot.runCommand(command, options=None)`

Immediately run another command in the same user context.

```python
Bot.runCommand("help")
Bot.runCommand("process_order", {"item": "Gold", "qty": 5})
```

### User vs. Bot Data — When to Use Which

| Use Case | Correct Method |
|----------|---------------|
| User's balance | `User.saveData("balance", 100)` |
| User's name | `User.saveData("name", "Alice")` |
| Whether user is registered | `User.saveData("registered", "true")` |
| Total bot users count | `Bot.saveData("total_users", 500)` |
| Announcement text | `Bot.saveData("announcement", "Hello!")` |
| Admin Telegram IDs | `Bot.saveData("admin_ids", "123,456")` |
| Bot's API keys (config) | `Bot.saveData("payment_api_key", "xxxx")` |

---

## 9. Sending Messages

### `bot.sendMessage(text, **kwargs)` — Send to Current User

```python
# Plain text
bot.sendMessage("Hello, world!")

# With HTML formatting
bot.sendMessage("<b>Bold</b> and <i>italic</i>", parse_mode="HTML")

# With Markdown
bot.sendMessage("*Bold* and _italic_", parse_mode="Markdown")

# f-string (recommended for dynamic content)
first_name = message.from_user.first_name
bot.sendMessage(f"Hello {first_name}! You have {balance} coins.")
```

### `bot.replyText(chat_id, text, **kwargs)` — Send to Specific Chat

```python
# Send to a specific Telegram chat ID
bot.replyText(123456789, "Hello from the bot!")

# Send to current user (using 'u')
bot.replyText(u, "Your message was received!")

# Send with HTML
bot.replyText(u, "<b>Important notice!</b>", parse_mode="HTML")
```

> **Note:** `bot.sendMessage()` sends to the **current chat** automatically. `bot.replyText(chat_id, text)` requires you to specify the chat ID. Both methods support the same optional parameters.

### Optional Parameters for Both Methods

| Parameter | Type | Description |
|-----------|------|-------------|
| `parse_mode` | `str` | `"HTML"` or `"Markdown"` |
| `reply_markup` | `dict` | Inline or reply keyboard |
| `disable_notification` | `bool` | Silent message |
| `reply_to_message_id` | `int` | Reply to a specific message |
| `protect_content` | `bool` | Prevent forwarding/saving |

---

## 10. Keyboard Support

### Inline Keyboard (Buttons Inside Message)

```python
# Correct TBC/TPY syntax for inline keyboard
keyboard = {
    "inline_keyboard": [
        [
            {"text": "✅ Confirm", "callback_data": "confirm_yes"},
            {"text": "❌ Cancel", "callback_data": "confirm_no"}
        ],
        [
            {"text": "🌐 Visit Website", "url": "https://example.com"}
        ]
    ]
}
bot.sendMessage("Are you sure?", reply_markup=keyboard)
```

### Inline Keyboard — Multiple Rows

```python
keyboard = {
    "inline_keyboard": [
        [{"text": "💰 Balance", "callback_data": "check_balance"}],
        [{"text": "🔗 Referral Link", "callback_data": "get_referral"}],
        [{"text": "📋 Tasks", "callback_data": "view_tasks"}],
        [{"text": "❓ Help", "callback_data": "get_help"}]
    ]
}
bot.sendMessage("Main Menu:", reply_markup=keyboard)
```

### Handling Inline Button Press (Callback Query)

When a user taps an inline button with `callback_data="confirm_yes"`, TBC runs the command named `confirm_yes`:

```python
# Command: confirm_yes
balance = int(User.getData("balance") or 0)
cost = int(User.getData("pending_cost") or 0)

if balance < cost:
    bot.sendMessage("❌ Insufficient balance.")
    raise ReturnCommand

User.saveData("balance", balance - cost)
bot.sendMessage("✅ Purchase confirmed!")

# Answer the callback query (dismiss loading spinner)
bot.answerCallbackQuery(message.callback_query_id, text="Done! ✅")
```

### Reply Keyboard (Persistent Buttons)

```python
# Reply keyboard appears below text input
keyboard = {
    "keyboard": [
        ["💰 Balance", "🔗 Referral Link"],
        ["📋 Tasks", "💸 Withdraw"],
        ["👤 My Profile", "❓ Help"]
    ],
    "resize_keyboard": True,
    "one_time_keyboard": False
}
bot.sendMessage("Main Menu:", reply_markup=keyboard)
```

### Remove Reply Keyboard

```python
bot.sendMessage("Keyboard removed.", reply_markup={"remove_keyboard": True})
```

### Connecting Reply Buttons to Commands

Since tapping a button sends the button label as text, create a command with the **exact same name** as the button label:

- Button label: `💰 Balance`
- Command name: `💰 Balance`

When the user taps the button, TBC runs the `💰 Balance` command.

---

## 11. Command Chaining

### `Bot.handleNextCommand(command, options=None, cancel_at_command=None)`

Waits for the user's next message and routes it to another command.

```python
# Step 1: Ask for name
bot.sendMessage("What is your name?")
Bot.handleNextCommand("save_name")
```

```python
# Command: save_name
name = msg   # msg contains the user's reply
User.saveData("name", name)
bot.sendMessage(f"Thanks, {name}! Now enter your email:")
Bot.handleNextCommand("save_email")
```

```python
# Command: save_email
email = msg
User.saveData("email", email)
bot.sendMessage("✅ Your details have been saved!")
```

### `Bot.runCommand(command, options=None)`

Immediately trigger another command within the same execution context.

```python
bot.sendMessage("Loading your profile...")
Bot.runCommand("show_profile")
```

### Cancel Pattern

Always provide a `/cancel` command so users can exit multi-step flows:

```python
# Command: /cancel
User.deleteData("pending_amount")
User.deleteData("pending_action")
bot.sendMessage("❌ Action cancelled. Returning to main menu.")
Bot.runCommand("start")
```

---

## 12. Scheduled Commands

### `Bot.runCommandAfter(timeout, command, options=None)`

Schedule a command to run after a delay.

```python
# Signature
Bot.runCommandAfter(timeout, command, options=None)
```

- **`timeout`**: delay in seconds (min: 1, max: 366 days = `60*60*24*366`)
- **`command`**: the command name to run
- **`options`**: optional data passed to the scheduled command
- Returns: `{"id": "<job_id>", "command": "<command>", "timeout": <seconds>}`

```python
# Schedule a reminder in 1 hour
job = Bot.runCommandAfter(3600, "send_reminder", {"note": "Check your tasks!"})
User.saveData("reminder_job_id", job["id"])

bot.sendMessage("✅ Reminder set for 1 hour from now.")
```

```python
# Command: send_reminder
note = options.get("note", "Don't forget!")
bot.replyText(u, f"⏰ Reminder: {note}")
```

### Cancel a Scheduled Task

```python
# Cancel a scheduled command using the job ID
job_id = User.getData("reminder_job_id")
if job_id:
    bot.cancelScheduledTask(job_id)
    bot.sendMessage("✅ Reminder cancelled.")
```

### `time.sleep(seconds)` — Add a Delay Inside a Command

```python
bot.sendMessage("Processing step 1...")
time.sleep(2)  # wait 2 seconds (max: 10 seconds)
bot.sendMessage("Processing step 2...")
```

> ⚠️ `time.sleep()` max limit is **10 seconds**. For longer delays, use `Bot.runCommandAfter`.

---

## 13. HTTP Requests

### Built-in `HTTP` Module (Recommended)

```python
# GET request
response = HTTP.get("https://api.example.com/data")
bot.sendMessage("Status: " + str(response.status_code))

# GET with headers
response = HTTP.get(
    "https://api.example.com/user",
    headers={"Authorization": "Bearer YOUR_TOKEN"}
)
data = response.json()
bot.sendMessage("Name: " + data["name"])

# POST request with JSON
response = HTTP.post(
    "https://api.example.com/create",
    json={"user_id": str(u), "amount": 100},
    headers={"Content-Type": "application/json"}
)
result = response.json()
if result.get("status") == "success":
    bot.sendMessage("✅ Created!")
```

### Response Object

```python
response = HTTP.get("https://api.example.com/data")
response.status_code   # HTTP status code (e.g., 200)
response.text          # Raw response body as string
response.json()        # Parse response as JSON (returns dict)
response.content       # Raw bytes (for images/files)
response.headers       # Response headers dict
```

### `libs.customHTTP` — Custom HTTP Client

Use when you need custom timeout or proxy settings:

```python
http_client = libs.customHTTP()
response = http_client.get("https://api.weatherapi.com/v1/current.json?key=KEY&q=London")
weather = response.json()
bot.sendMessage(f"Temperature: {weather['current']['temp_c']}°C")
```

### Supported HTTP Methods

```python
HTTP.get(url, **kwargs)
HTTP.post(url, **kwargs)
HTTP.put(url, **kwargs)
HTTP.delete(url, **kwargs)
HTTP.patch(url, **kwargs)
HTTP.options(url, **kwargs)
HTTP.head(url, **kwargs)
```

### Error-Safe HTTP Request Pattern

```python
try:
    response = HTTP.get("https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd")
    if response.status_code != 200:
        bot.sendMessage("❌ API error: " + str(response.status_code))
        raise ReturnCommand
    data = response.json()
    price = data["bitcoin"]["usd"]
    bot.sendMessage(f"₿ Bitcoin: ${price:,}")
except Exception as e:
    bot.sendMessage("❌ Could not fetch price. Try again later.")
```

---

## 14. Wait for Answer — Multi-Step Input Collection

**Wait for Answer** is the pattern of using `Bot.handleNextCommand` to collect input from users step by step.

### How It Works

1. Bot sends a prompt message.
2. Bot calls `Bot.handleNextCommand("next_command")`.
3. User sends their reply.
4. TBC routes the user's reply to `next_command`.
5. Inside `next_command`, the user's reply is in the `msg` variable.

### Single-Step Input Example

```python
# Command: /setname
bot.sendMessage("What is your name?")
Bot.handleNextCommand("save_name")
```

```python
# Command: save_name
name = msg.strip()
if not name:
    bot.sendMessage("❌ Name cannot be empty. Try again:")
    Bot.handleNextCommand("save_name")
    raise ReturnCommand

User.saveData("name", name)
bot.sendMessage(f"✅ Name saved: {name}")
```

### Number Input with Validation

```python
# Command: /deposit
bot.sendMessage("Enter the amount you want to deposit (minimum 10):")
Bot.handleNextCommand("process_deposit")
```

```python
# Command: process_deposit
amount_str = msg.strip()
if not isNumeric(amount_str):
    bot.sendMessage("❌ Please enter a valid number:")
    Bot.handleNextCommand("process_deposit")
    raise ReturnCommand

amount = int(amount_str)
if amount < 10:
    bot.sendMessage("❌ Minimum is 10. Enter a higher amount:")
    Bot.handleNextCommand("process_deposit")
    raise ReturnCommand

User.saveData("pending_deposit", str(amount))
bot.sendMessage(f"✅ Amount set: {amount} coins. Confirm?", reply_markup={
    "inline_keyboard": [
        [{"text": "✅ Confirm", "callback_data": "deposit_confirm"},
         {"text": "❌ Cancel", "callback_data": "deposit_cancel"}]
    ]
})
```

---

## 15. Sending Media

### Send a Photo

```python
# By URL
bot.sendPhoto("https://example.com/image.jpg")

# With caption and HTML
bot.sendPhoto("https://example.com/banner.png",
    caption="<b>Welcome!</b>",
    parse_mode="HTML")

# With inline keyboard
keyboard = {"inline_keyboard": [[{"text": "🛒 Buy", "callback_data": "buy"}]]}
bot.sendPhoto("https://example.com/product.jpg",
    caption="🔥 Special Offer!",
    reply_markup=keyboard)

# To a specific user by file_id (Telegram file_id from a previously sent file)
bot.replyPhoto(u, "FILE_ID_HERE", caption="Here is your photo")
```

### Send a Document

```python
bot.sendDocument("https://example.com/guide.pdf", caption="📄 User Guide")
```

### Send a Video

```python
bot.sendVideo("https://example.com/tutorial.mp4", caption="🎬 Tutorial")
```

### Send an Animation (GIF)

```python
bot.sendAnimation("https://example.com/welcome.gif", caption="👋 Welcome!")
```

### Send Audio

```python
bot.sendAudio("https://example.com/music.mp3", caption="🎵 Enjoy!")
```

### Send a Voice Message

```python
bot.sendVoice("FILE_ID_OR_URL")
```

### Send a Sticker

```python
bot.sendSticker("FILE_ID_HERE")
```

### Send a Poll

```python
bot.sendPoll(
    question="What's your favorite feature?",
    options=["AI Integration", "Crypto Payments", "Broadcasting", "Webhooks"]
)
```

### All `bot.send*` Methods

| Method | Usage |
|--------|-------|
| `bot.sendMessage(text)` | Send text message to current chat |
| `bot.replyText(chat_id, text)` | Send text to specific chat_id |
| `bot.sendPhoto(photo, **kwargs)` | Send a photo |
| `bot.replyPhoto(chat_id, photo)` | Send photo to specific chat_id |
| `bot.sendDocument(document, **kwargs)` | Send a file/document |
| `bot.sendVideo(video, **kwargs)` | Send a video |
| `bot.sendAudio(audio, **kwargs)` | Send audio |
| `bot.sendVoice(voice, **kwargs)` | Send voice message |
| `bot.sendAnimation(animation, **kwargs)` | Send GIF/animation |
| `bot.sendVideoNote(video_note, **kwargs)` | Send circular video |
| `bot.sendSticker(sticker, **kwargs)` | Send sticker |
| `bot.sendPoll(question, options, **kwargs)` | Send a poll |
| `bot.sendDice(**kwargs)` | Send dice animation |
| `bot.sendLocation(latitude, longitude)` | Send location |
| `bot.sendMediaGroup(media, **kwargs)` | Send album of media |
| `bot.sendChatAction(action)` | Show "typing..." indicator |

---

## 16. Editing & Deleting Messages

### Save the Message ID After Sending

```python
# bot.sendMessage returns the sent message object
sent = bot.sendMessage("⏳ Processing your request...")
msg_id = sent.message_id
User.saveData("last_msg_id", str(msg_id))
```

### Edit a Sent Message

```python
msg_id = User.getData("last_msg_id")
chat_id = message.chat.id

bot.editMessageText(
    "✅ Done! Your request has been completed.",
    chat_id=chat_id,
    message_id=int(msg_id),
    parse_mode="HTML"
)
```

### Delete a Message

```python
msg_id = User.getData("last_msg_id")
chat_id = message.chat.id

try:
    bot.deleteMessage(chat_id=chat_id, message_id=int(msg_id))
except Exception:
    pass  # Message may already be deleted or too old (>48h)
```

### Edit Inline Keyboard of a Message

```python
msg_id = User.getData("last_msg_id")
new_keyboard = {
    "inline_keyboard": [[{"text": "✅ Completed", "callback_data": "done"}]]
}
bot.editMessageReplyMarkup(
    chat_id=message.chat.id,
    message_id=int(msg_id),
    reply_markup=new_keyboard
)
```

### Pattern: Updating a "Processing..." Message

```python
# Command: /pay
sent = bot.sendMessage("⏳ Processing payment...")
msg_id = sent.message_id
User.saveData("proc_msg_id", str(msg_id))

response = HTTP.get("https://api.payment.com/check?id=XYZ")
result = response.json()

msg_id = int(User.getData("proc_msg_id"))
if result.get("status") == "paid":
    bot.editMessageText("✅ Payment confirmed!", chat_id=message.chat.id, message_id=msg_id)
else:
    bot.editMessageText("❌ Payment not found.", chat_id=message.chat.id, message_id=msg_id)
```

---

## 17. Channel Membership Check

### Using `MembershipCheck` (Built-in Global)

```python
# Command: /start
# Check if user is member of a channel
result = MembershipCheck.check("@your_channel", u)

if result.status in ["left", "kicked"]:
    keyboard = {
        "inline_keyboard": [
            [{"text": "📢 Join Channel", "url": "https://t.me/your_channel"}],
            [{"text": "✅ I've Joined — Verify", "callback_data": "check_membership"}]
        ]
    }
    bot.sendMessage(
        "⚠️ <b>Join Required!</b>\n\nPlease join our channel to use this bot.",
        reply_markup=keyboard,
        parse_mode="HTML"
    )
    raise ReturnCommand

# User is a member — proceed
bot.sendMessage("✅ Verified! Welcome.")
```

### Multiple Channels Check

```python
channels = ["@channel1", "@channel2", "@channel3"]
all_joined = True

for ch in channels:
    result = MembershipCheck.check(ch, u)
    if result.status in ["left", "kicked"]:
        all_joined = False
        break

if not all_joined:
    keyboard = {
        "inline_keyboard": [
            [{"text": "📢 Join Channel 1", "url": "https://t.me/channel1"}],
            [{"text": "📢 Join Channel 2", "url": "https://t.me/channel2"}],
            [{"text": "📢 Join Channel 3", "url": "https://t.me/channel3"}],
            [{"text": "✅ Verify Membership", "callback_data": "verify_all"}]
        ]
    }
    bot.sendMessage(
        "⚠️ Please join ALL our channels to continue.",
        reply_markup=keyboard
    )
    raise ReturnCommand

bot.sendMessage("✅ All channels verified!")
```

### Membership Status Values

| Status | Meaning |
|--------|---------|
| `"member"` | Active member ✅ |
| `"administrator"` | Channel admin ✅ |
| `"creator"` | Channel owner ✅ |
| `"left"` | Not a member ❌ |
| `"kicked"` | Was banned ❌ |
| `"restricted"` | Restricted member ⚠️ |

---

## 18. Admin-Specific Functions

### Hardcoded Admin Check

```python
# At the top of any admin-only command
ADMIN_IDS = ["123456789", "987654321"]  # Telegram user IDs

if str(u) not in ADMIN_IDS:
    bot.sendMessage("❌ Access denied.")
    raise ReturnCommand

# Admin-only code below
bot.sendMessage("✅ Welcome, Admin!")
```

### Dynamic Admin Check (via Bot data)

```python
admin_ids_str = Bot.getData("admin_ids") or ""
admin_ids = admin_ids_str.split(",")

if str(u) not in admin_ids:
    bot.sendMessage("⛔ Access denied.")
    raise ReturnCommand
```

### Admin: View Bot Stats

```python
# Command: /stats (admin only)
ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    raise ReturnCommand

total_users = Bot.getData("total_users") or 0
total_withdrawals = Bot.getData("total_withdrawals") or 0

bot.sendMessage(
    f"📊 <b>Bot Statistics</b>\n\n"
    f"👥 Total Users: <code>{total_users}</code>\n"
    f"💸 Total Withdrawals: <code>{total_withdrawals}</code>\n"
    f"⚡ Points Left: <code>{left_points}</code>",
    parse_mode="HTML"
)
```

### Admin: Send Message to a Specific User

```python
# Command: /message_user (admin only)
# User sends: /message_user 123456789 Hello there!
parts = params.split(" ", 1)
if len(parts) < 2:
    bot.sendMessage("Format: /message_user <user_id> <message>")
    raise ReturnCommand

target_id = parts[0].strip()
msg_text = parts[1].strip()

try:
    bot.replyText(int(target_id), f"📨 Message from Admin:\n\n{msg_text}")
    bot.sendMessage(f"✅ Message sent to user {target_id}")
except Exception as e:
    bot.sendMessage(f"❌ Failed: {str(e)}")
```

---

## 19. Broadcasting

### `Bot.broadcast(...)` — Send to All Users

```python
# Broadcast a message to all users (function mode)
Bot.broadcast(
    function="send_message",
    text="📢 Hello, everyone! This is a broadcast."
)
```

```python
# Broadcast a custom command to all users
Bot.broadcast(command="promo_offer")
```

```python
# Multi-bot broadcast with speed control
Bot.broadcast(
    function="send_message",
    text="Hi {first_name}, here's our latest update!",
    mode="multi",
    bot_ids=["BOT_ID_1", "BOT_ID_2"],
    speed=12
)
```

> **`{first_name}`** is a built-in placeholder — the broadcast service replaces it with each recipient's name.

### Broadcast Control Methods

```python
Bot.stopBroadcast(broadcast_id)          # Stop a running broadcast
Bot.pauseBroadcast(broadcast_id)         # Pause
Bot.resumeBroadcast(broadcast_id)        # Resume
Bot.getBroadcastStatus(broadcast_id)     # Get status
Bot.listBroadcasts()                     # List all broadcasts
Bot.clearBroadcast(broadcast_id)         # Clear records
```

> **Limits:** Max 3 running broadcasts per bot, max 5000 globally on the platform.

---

## 20. Webhook Integration

### Generate a Webhook URL

```python
# Generate a URL that triggers a specific command
webhook_url = libs.Webhook.getUrlFor("payment_received", user_id=u)
bot.sendMessage(f"🔗 Your webhook URL:\n{webhook_url}")
```

### Receive Webhook Data

```python
# Command: payment_received (triggered when the webhook URL is called)

# In webhook commands, data comes via 'options', not 'msg'
raw_data = options.data         # Raw body string
json_data = options.json        # Parsed JSON dict
headers = options.headers       # HTTP request headers (new in 7.1.2)
caller_ip = options.ip          # Caller IP address (new in 7.1.2)

if json_data.get("status") == "paid":
    amount = json_data.get("amount", 0)
    balance = int(User.getData("balance") or 0)
    User.saveData("balance", balance + int(amount))
    bot.replyText(u, f"✅ Payment received: {amount} coins added!")
```

### Validate Webhook Signatures (Security)

```python
# Command: payment_received
sig = options.headers.get("X-Signature", "")
expected = libs.security.hmac_sign("MY_WEBHOOK_SECRET", options.data)

if sig != expected:
    # Invalid request — reject it silently
    raise ReturnCommand

# Valid request — process it
data = options.json
bot.replyText(u, "✅ Payment verified!")
```

---

## 21. Resources Library — `libs.Resources`

The Resources library manages numeric values (balance, points, XP, scores) per-user or globally.

### `libs.Resources.userRes(name, user=None)` — Per-User Resource

```python
# Get current user's balance
balance = libs.Resources.userRes("balance")
balance.add(100)                   # Add 100
balance.cut(50)                    # Subtract 50
balance.set(1000)                  # Set to absolute value
balance.reset()                    # Reset to 0
current = balance.value()          # Read current value
bot.sendMessage(f"Balance: {current}")
```

### `libs.Resources.anotherRes(name, user_id)` — Another User's Resource

```python
# Modify another user's resource by their Telegram ID
other_balance = libs.Resources.anotherRes("balance", "987654321")
other_balance.add(200)
bot.sendMessage("Added 200 to the other user.")
```

### `libs.Resources.globalRes(name)` — Global (Bot-wide) Resource

```python
jackpot = libs.Resources.globalRes("jackpot")
jackpot.add(10)
current = jackpot.value()
bot.sendMessage(f"Current jackpot: {current}")
```

### `libs.Resources.adminRes(name)` — Admin Operations

```python
# Get leaderboard (top 10)
admin = libs.Resources.adminRes("balance")
top = admin.getAllData(10)

msg = "🏆 <b>Top 10 Users</b>\n\n"
for i, entry in enumerate(top, 1):
    msg += f"{i}. User {entry['user']} — {entry['value']} coins\n"
bot.sendMessage(msg, parse_mode="HTML")
```

### Check Balance Before Spending

```python
balance = libs.Resources.userRes("balance")
cost = 50

if balance.value() < cost:
    bot.sendMessage(f"❌ Not enough coins. You need {cost} coins.")
    raise ReturnCommand

balance.cut(cost)
bot.sendMessage(f"✅ Purchased! Remaining: {balance.value()} coins.")
```

---

## 22. All Available Libraries (30+)

### Library Quick Reference

| Library | Code Access | Purpose |
|---------|-------------|---------|
| Resources | `libs.Resources` | Numeric values (balance, points, XP) |
| Random | `libs.Random` | Random numbers, strings, choices |
| Webhook | `libs.Webhook` | Receive external HTTP requests |
| CSV | `libs.CSV` | Manage CSV data files |
| DateAndTime | `libs.DateAndTime` | Date and time utilities |
| customHTTP | `libs.customHTTP` | Advanced HTTP client |
| Coinbase | `libs.Coinbase` | Coinbase crypto payments |
| Coinpayments | `libs.Coinpayments` | CoinPayments integration |
| Oxapay | `libs.Oxapay` | Oxapay payment gateway |
| MDxchange | `libs.MDxchange` | MDxchange crypto payments & Stars |
| TonLib | `libs.TonLib` | TON blockchain |
| web3lib | `libs.web3lib` | All EVM chains (ETH, BSC, etc.) |
| Crypto | `libs.Crypto` | Crypto price & conversion |
| openai_lib | `libs.openai_lib` | OpenAI GPT-4 integration |
| gemini_lib | `libs.gemini_lib` | Google Gemini AI integration |
| OpenCV | `libs.OpenCV` | Image processing |
| Pillow | `libs.Pillow` | Image editing |
| security | `libs.security` | HMAC, AES, Ed25519, hashing |
| PremiumGift | `libs.PremiumGift` | Send Telegram Premium gifts |
| translate | `libs.translate` | Auto-translation (92 languages) |

### `libs.Random` — Random Values

```python
# Random integer (inclusive)
num = libs.Random.randomInt(1, 100)

# Random string (alphanumeric)
code = libs.Random.randomStr(8)        # e.g. "Ax7Qz1Pp"

# Random choice from a list
winner = libs.Random.randomChoice(["Alice", "Bob", "Cara"])

# Random UUID
uid = libs.Random.randomUUID()

# Random hex string
token = libs.Random.randomHex(16)

# Weighted choice
prize = libs.Random.randomWeightedChoice([10, 50, 100], weights=[70, 25, 5])

# Shuffle a list (returns new list)
shuffled = libs.Random.randomShuffle(["a", "b", "c"])
```

### `libs.DateAndTime` — Date & Time

```python
now_utc = libs.DateAndTime.utcnow()       # e.g. "2026-09-12 10:30:00"
today = libs.DateAndTime.date_now()       # e.g. "2026-09-12"
timestamp = libs.DateAndTime.time()       # UNIX timestamp
now_tz = libs.DateAndTime.now("Asia/Dhaka")  # Timezone-aware

bot.sendMessage(f"Current UTC time: {now_utc}")
```

### `libs.CSV` — CSV File Management

```python
csv = libs.CSV.CSVHandler("leaderboard.csv")
csv.create_csv(["User", "Score", "Date"])
csv.add_row({"User": "Alice", "Score": 500, "Date": "2026-09-12"})
csv.edit_row({"User": "Alice"}, {"Score": 750})
row = csv.get({"User": "Alice"})
```

### `libs.security` — Cryptography (New in 7.1.2)

```python
# HMAC signature
sig = libs.security.hmac_sign("my_secret", "data_to_sign")
is_valid = libs.security.hmac_verify("my_secret", "data_to_sign", sig)

# AES encryption
key = libs.security.generate_key()
encrypted = libs.security.encrypt("sensitive data", key)
decrypted = libs.security.decrypt(encrypted, key)

# Hashing
sha = libs.security.sha256("my data")
md5_hash = libs.security.md5("my data")
```

### `libs.translate` — Auto-Translation (92 Languages)

```python
# Remember this user's language
libs.translate.setUser("bn")  # Bengali

# Set default language for all users
libs.translate.setGlobal("en")

# Translate a string
translated = libs.translate.text("Hello! Welcome to the bot.", "bn")
bot.sendMessage(translated)
```

---

## 23. Referral System Pattern

A complete referral system — generating links, tracking new users, rewarding referrers.

### Step 1 — Show User's Referral Link

```python
# Command: /refer  OR  "🔗 Referral Link" button

ref_link = f"https://t.me/YourBotUsername?start=ref{u}"
ref_count = libs.Resources.userRes("ref_count").value()
ref_earnings = libs.Resources.userRes("ref_earnings").value()

bot.sendMessage(
    f"🔗 <b>Your Referral Link</b>\n\n"
    f"<code>{ref_link}</code>\n\n"
    f"👥 Total Referrals: <b>{ref_count}</b>\n"
    f"💰 Total Earned: <b>{ref_earnings} coins</b>\n\n"
    f"📢 Share and earn <b>100 coins</b> per referral!",
    parse_mode="HTML"
)
```

### Step 2 — Handle New User via Referral Link

```python
# Command: /start
# When user clicks t.me/Bot?start=ref123456789
# params = "ref123456789"

first_name = message.from_user.first_name

# Check if new user
registered = User.getData("registered")
if not registered:
    User.saveData("registered", "true")

    # Initialize balance via Resources
    libs.Resources.userRes("balance").set(0)

    # Increment total users counter
    global_users = libs.Resources.globalRes("total_users")
    global_users.add(1)

    # Handle referral bonus
    start_param = params or ""
    if start_param.startswith("ref"):
        referrer_id = start_param[3:]  # Extract ID after "ref"

        # Prevent self-referral
        if referrer_id != str(u):
            already_credited = User.getData("ref_bonus_given")
            if not already_credited:
                User.saveData("ref_bonus_given", "true")

                # Give referrer 100 coins + increment their count
                libs.Resources.anotherRes("balance", referrer_id).add(100)
                libs.Resources.anotherRes("ref_count", referrer_id).add(1)
                libs.Resources.anotherRes("ref_earnings", referrer_id).add(100)

                # Notify referrer
                try:
                    bot.replyText(
                        int(referrer_id),
                        "🎉 Someone joined using your referral link!\n✅ +100 coins added!"
                    )
                except Exception:
                    pass  # Referrer may have blocked the bot

        bot.sendMessage(f"✅ Welcome, {first_name}! You joined via a referral.")
    else:
        bot.sendMessage(f"✅ Welcome, {first_name}! Great to have you here.")
else:
    balance = libs.Resources.userRes("balance").value()
    bot.sendMessage(f"👋 Welcome back, {first_name}!\n💰 Balance: {balance} coins")

# Show main menu
keyboard = {
    "keyboard": [
        ["💰 Balance", "🔗 Referral Link"],
        ["📋 Tasks", "💸 Withdraw"],
        ["👤 My Profile", "❓ Help"]
    ],
    "resize_keyboard": True
}
bot.sendMessage("📌 Main Menu:", reply_markup=keyboard)
```

### Step 3 — Check Referral Stats

```python
# Command: /refstats
ref_count = libs.Resources.userRes("ref_count").value()
ref_earnings = libs.Resources.userRes("ref_earnings").value()
balance = libs.Resources.userRes("balance").value()
ref_link = f"https://t.me/YourBotUsername?start=ref{u}"

bot.sendMessage(
    f"📊 <b>Your Referral Stats</b>\n\n"
    f"👥 Referrals: <b>{ref_count}</b>\n"
    f"💰 Earned: <b>{ref_earnings} coins</b>\n"
    f"🏦 Current Balance: <b>{balance} coins</b>\n\n"
    f"🔗 Your Link:\n<code>{ref_link}</code>",
    parse_mode="HTML"
)
```

---

## 24. Economy System Pattern

### Check Balance

```python
# Command: /balance  OR  "💰 Balance" button
balance = libs.Resources.userRes("balance").value()
bot.sendMessage(
    f"💰 <b>Your Balance</b>\n\n"
    f"Coins: <code>{balance}</code>",
    parse_mode="HTML"
)
```

### Daily Bonus with Cooldown

```python
# Command: /daily  OR  "🎁 Daily Bonus" button

today = str(libs.DateAndTime.date_now())
last_claim = User.getData("last_daily")

if last_claim == today:
    bot.sendMessage("⏳ Already claimed today! Come back tomorrow.")
    raise ReturnCommand

BONUS = 100
libs.Resources.userRes("balance").add(BONUS)
User.saveData("last_daily", today)
new_balance = libs.Resources.userRes("balance").value()

bot.sendMessage(
    f"✅ <b>Daily Bonus Claimed!</b>\n\n"
    f"+ {BONUS} coins added!\n"
    f"New Balance: <code>{new_balance}</code> coins",
    parse_mode="HTML"
)
```

### Transfer Coins Between Users

```python
# Command: /send
# User sends: /send 987654321 100

parts = (params or "").strip().split(" ", 1)
if len(parts) < 2:
    bot.sendMessage("❌ Format: /send <user_id> <amount>")
    raise ReturnCommand

receiver_id = parts[0].strip()
amount_str = parts[1].strip()

if not isNumeric(amount_str):
    bot.sendMessage("❌ Invalid amount.")
    raise ReturnCommand

amount = int(amount_str)

if receiver_id == str(u):
    bot.sendMessage("❌ You cannot send to yourself.")
    raise ReturnCommand

if amount <= 0:
    bot.sendMessage("❌ Amount must be greater than 0.")
    raise ReturnCommand

my_balance = libs.Resources.userRes("balance")
if my_balance.value() < amount:
    bot.sendMessage(f"❌ Insufficient balance.\nYour balance: {my_balance.value()}")
    raise ReturnCommand

# Execute transfer
my_balance.cut(amount)
libs.Resources.anotherRes("balance", receiver_id).add(amount)

bot.sendMessage(f"✅ Sent {amount} coins to user {receiver_id}\nNew balance: {my_balance.value()}")
try:
    bot.replyText(int(receiver_id), f"💰 You received {amount} coins from user {u}!")
except Exception:
    pass
```

### Admin Top-Up

```python
# Command: /topup  (admin only)
# Usage: /topup 123456789 500

ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    bot.sendMessage("❌ Access denied.")
    raise ReturnCommand

parts = (params or "").strip().split(" ", 1)
if len(parts) < 2:
    bot.sendMessage("Format: /topup <user_id> <amount>")
    raise ReturnCommand

target_id = parts[0].strip()
amount_str = parts[1].strip()

if not isNumeric(amount_str):
    bot.sendMessage("❌ Invalid amount.")
    raise ReturnCommand

amount = int(amount_str)
libs.Resources.anotherRes("balance", target_id).add(amount)
new_bal = libs.Resources.anotherRes("balance", target_id).value()

bot.sendMessage(f"✅ Added {amount} coins to user {target_id}\nNew balance: {new_bal}")
try:
    bot.replyText(int(target_id), f"💰 Admin added {amount} coins to your account!")
except Exception:
    pass
```

### Leaderboard

```python
# Command: /top
admin = libs.Resources.adminRes("balance")
top = admin.getAllData(10)

msg = "🏆 <b>Top 10 Richest Users</b>\n\n"
for i, entry in enumerate(top, 1):
    msg += f"{i}. <code>{entry['user']}</code> — {entry['value']} coins\n"

bot.sendMessage(msg, parse_mode="HTML")
```

---

## 25. AI Integration — OpenAI & Gemini

### OpenAI (GPT-4) Integration

```python
# Command: /ask
# Usage: /ask What is the capital of France?

question = params
if not question:
    bot.sendMessage("Usage: /ask <your question>")
    raise ReturnCommand

client = libs.openai_lib.OpenAIClient(api_key="YOUR_OPENAI_API_KEY", timeout=120)
response = client.create_chat_completion(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": question}
    ]
)
answer = response["choices"][0]["message"]["content"]
bot.sendMessage(f"🤖 <b>AI Answer:</b>\n\n{answer}", parse_mode="HTML")
```

### Google Gemini Integration

```python
client = libs.gemini_lib.GeminiClient(api_key="YOUR_GEMINI_API_KEY")
response = client.create_chat_completion(
    model="gemini-2.0-flash",
    messages=[
        {"role": "user", "content": "Explain quantum computing in simple terms."}
    ]
)
answer = response["choices"][0]["message"]["content"]
bot.sendMessage(answer)
```

### OpenRouter (100+ Models)

```python
client = libs.openai_lib.OpenAIClient(
    api_key="YOUR_OPENROUTER_API_KEY",
    timeout=120,
    base_url="https://openrouter.ai/api/v1"
)
assistant = libs.openai_lib.AIAssistant(
    openai_client=client,
    model="meta-llama/llama-3.3-8b-instruct:free",
    system_message="You are a helpful assistant."
)
response = assistant.send_message(params or "Hello!")
text = str(response.get("content")[0]['text']['value'])
bot.sendMessage(text)
```

---

## 26. HTML & Markdown Formatting

### HTML Formatting (Recommended)

Use `parse_mode="HTML"` in `bot.sendMessage()` or `bot.replyText()`.

| Tag | Result |
|-----|--------|
| `<b>text</b>` | **Bold** |
| `<i>text</i>` | *Italic* |
| `<u>text</u>` | Underline |
| `<s>text</s>` | ~~Strikethrough~~ |
| `<code>text</code>` | `Monospace code` |
| `<pre>text</pre>` | Code block |
| `<a href="URL">text</a>` | Hyperlink |
| `<tg-spoiler>text</tg-spoiler>` | Hidden spoiler |

```python
balance = libs.Resources.userRes("balance").value()
bot.sendMessage(
    f"<b>💰 Your Account</b>\n\n"
    f"Name: <code>{User.getData('name') or 'N/A'}</code>\n"
    f"Balance: <b>{balance} coins</b>\n"
    f"<i>Last updated: just now</i>",
    parse_mode="HTML"
)
```

### Markdown Formatting

Use `parse_mode="Markdown"`.

| Syntax | Result |
|--------|--------|
| `*text*` | **Bold** |
| `_text_` | *Italic* |
| `` `text` `` | `Monospace` |
| ` ```code``` ` | Code block |
| `[text](URL)` | Hyperlink |

```python
bot.sendMessage(
    f"*💰 Your Balance*\n\nCoins: `{balance}`\n_Updated just now_",
    parse_mode="Markdown"
)
```

---

## 27. Error Handling & Debugging

### Basic try/except

```python
try:
    amount = int(msg)
    if amount <= 0:
        raise ValueError("Amount must be positive")
    # process amount...
except ValueError as e:
    bot.sendMessage(f"❌ Invalid input: {str(e)}")
    raise ReturnCommand
except Exception as e:
    bot.sendMessage("⚠️ An unexpected error occurred.")
    Bot.saveData("last_error", str(e))
```

### The `@` Command — Pre-processor / Logger

```python
# Command: @  (runs BEFORE every other command)
user_id_str = str(u)
first_name = message.from_user.first_name

# Log activity (optional)
# Bot.saveData("last_active_" + user_id_str, str(libs.DateAndTime.utcnow()))

# Check maintenance mode
if Bot.getData("maintenance") == "true":
    if user_id_str not in ["123456789"]:  # admins bypass
        bot.sendMessage("🔧 Bot is under maintenance. Please try again later.")
        raise ReturnCommand
```

### The `*` Command — Fallback Handler

```python
# Command: *  (matches any unrecognized message)
bot.sendMessage(
    "❓ I didn't understand that.\n\nUse /help to see available commands.",
    reply_markup={
        "inline_keyboard": [[{"text": "❓ Help", "callback_data": "get_help"}]]
    }
)
```

### Viewing Errors

- Go to **Your Bot → Errors** in the TBC dashboard.
- Error log shows the command name, line number, and error message.
- Use `Bot.saveData("last_error", str(e))` to log errors to storage for inspection.

---

## 28. Complete Syntax Rules & Common Mistakes

### ✅ CORRECT Patterns

```python
# ✅ Correct: Use raise ReturnCommand to stop early (NEVER bare return at top level)
if not User.getData("registered"):
    bot.sendMessage("Please register first.")
    raise ReturnCommand

# ✅ Correct: f-string for dynamic messages
first_name = message.from_user.first_name
bot.sendMessage(f"Hello {first_name}!")

# ✅ Correct: Safe data reading with fallback
balance = int(User.getData("balance") or 0)

# ✅ Correct: Convert str to int before arithmetic
balance = int(User.getData("balance") or 0) + 100

# ✅ Correct: Check numeric input
if not isNumeric(msg):
    bot.sendMessage("Please enter a number.")
    raise ReturnCommand

# ✅ Correct: Use str(u) for comparisons (u may be int or str)
ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    raise ReturnCommand

# ✅ Correct: Wrap delete operations in try/except
try:
    bot.deleteMessage(chat_id=message.chat.id, message_id=int(msg_id))
except Exception:
    pass

# ✅ Correct: Access user info via message object
first_name = message.from_user.first_name
user_tg_id = message.from_user.id
```

### ❌ INCORRECT Patterns (Will Cause Errors)

```python
# ❌ NEVER use bare return at top level — SyntaxError
if not condition:
    return   # Crashes the whole command!

# ✅ Use this instead:
if not condition:
    raise ReturnCommand

# ❌ NEVER import anything
import requests   # Blocked — not available
import os         # Blocked — not available

# ❌ NEVER concatenate int + str directly
bot.sendMessage("Balance: " + balance)   # TypeError if balance is int
# ✅ Fix:
bot.sendMessage("Balance: " + str(balance))
# ✅ Or use f-string:
bot.sendMessage(f"Balance: {balance}")

# ❌ NEVER assume User.getData returns non-None
balance = int(User.getData("balance"))   # Crashes if never set (None)
# ✅ Fix:
balance = int(User.getData("balance") or 0)

# ❌ NEVER use eval() or exec()
eval("1+1")  # Blocked for security

# ❌ NEVER use old ReplyKeyboardMarkup/InlineKeyboardMarkup class syntax
keyboard = ReplyKeyboardMarkup()   # ❌ Wrong — use dict format
keyboard.addButton("Button")       # ❌ These methods don't exist in TBC

# ✅ Use dict format for keyboards:
keyboard = {"keyboard": [["Button 1", "Button 2"]], "resize_keyboard": True}
bot.sendMessage("Choose:", reply_markup=keyboard)

# ❌ NEVER use Bot.handleNextCommand on its own without bot.sendMessage first
Bot.handleNextCommand("command")  # Will work, but without a prompt the user won't know what to do
# ✅ Always prompt first:
bot.sendMessage("What is your name?")
Bot.handleNextCommand("save_name")
```

---

## 29. Full Example — /start Command with All Features

```python
# Command: /start
# This is a complete, production-ready /start command

first_name = message.from_user.first_name

# ─────────────────────────────────────────────
# STEP 1: Channel Membership Check
# ─────────────────────────────────────────────
channels_to_join = ["@your_channel1", "@your_channel2"]
not_joined = []

for ch in channels_to_join:
    result = MembershipCheck.check(ch, u)
    if result.status in ["left", "kicked"]:
        not_joined.append(ch)

if not_joined:
    buttons = []
    for ch in not_joined:
        buttons.append([{"text": f"📢 Join {ch}", "url": f"https://t.me/{ch.replace('@', '')}"}])
    buttons.append([{"text": "✅ I've Joined — Verify", "callback_data": "verify_membership"}])

    bot.sendMessage(
        "⚠️ <b>Join Required!</b>\n\nPlease join all our channels first.",
        reply_markup={"inline_keyboard": buttons},
        parse_mode="HTML"
    )
    raise ReturnCommand

# ─────────────────────────────────────────────
# STEP 2: New User Registration & Referral
# ─────────────────────────────────────────────
registered = User.getData("registered")

if not registered:
    User.saveData("registered", "true")

    # Initialize resources
    libs.Resources.userRes("balance").set(0)
    libs.Resources.userRes("ref_count").set(0)

    # Global user count
    libs.Resources.globalRes("total_users").add(1)

    # Handle referral
    start_param = params or ""
    if start_param.startswith("ref"):
        referrer_id = start_param[3:]
        if referrer_id and referrer_id != str(u):
            if not User.getData("ref_bonus_given"):
                User.saveData("ref_bonus_given", "true")
                libs.Resources.anotherRes("balance", referrer_id).add(100)
                libs.Resources.anotherRes("ref_count", referrer_id).add(1)
                libs.Resources.anotherRes("ref_earnings", referrer_id).add(100)
                try:
                    bot.replyText(
                        int(referrer_id),
                        "🎉 Someone joined via your referral!\n✅ +100 coins!"
                    )
                except Exception:
                    pass

    welcome_text = (
        f"🎉 <b>Welcome, {first_name}!</b>\n\n"
        f"Your account has been created.\n"
        f"Starting balance: <code>0 coins</code>"
    )
    bot.sendMessage(welcome_text, parse_mode="HTML")

else:
    # Returning user
    balance = libs.Resources.userRes("balance").value()
    bot.sendMessage(
        f"👋 <b>Welcome back, {first_name}!</b>\n\n"
        f"💰 Balance: <code>{balance} coins</code>",
        parse_mode="HTML"
    )

# ─────────────────────────────────────────────
# STEP 3: Show Main Menu
# ─────────────────────────────────────────────
keyboard = {
    "keyboard": [
        ["💰 Balance", "🔗 Referral Link"],
        ["📋 Tasks", "💸 Withdraw"],
        ["👤 My Profile", "❓ Help"]
    ],
    "resize_keyboard": True,
    "one_time_keyboard": False
}
bot.sendMessage("📌 <b>Main Menu</b>", reply_markup=keyboard, parse_mode="HTML")
```

---

## 30. Quick Reference Card

### Send Messages

```python
bot.sendMessage("text")                                  # To current chat
bot.sendMessage("<b>bold</b>", parse_mode="HTML")        # HTML formatted
bot.sendMessage(f"Hello {first_name}!")                  # f-string
bot.replyText(chat_id, "text")                           # To specific chat_id
```

### Stop Command Early (CRITICAL)

```python
raise ReturnCommand   # ALWAYS use this — NEVER bare return at top level
```

### Read / Write User Data

```python
User.saveData("key", value)
val = User.getData("key") or "default"
User.deleteData("key")
```

### Read / Write Global Data

```python
Bot.saveData("key", value)
val = Bot.getData("key") or "default"
Bot.deleteData("key")
```

### Resources (Balance, Points, etc.)

```python
res = libs.Resources.userRes("balance")
res.add(100)          # Add
res.cut(50)           # Subtract
res.set(1000)         # Set absolute
res.reset()           # Set to 0
current = res.value() # Read
```

### Reply Keyboard

```python
keyboard = {
    "keyboard": [["Button 1", "Button 2"], ["Button 3"]],
    "resize_keyboard": True
}
bot.sendMessage("Choose:", reply_markup=keyboard)
```

### Inline Keyboard

```python
keyboard = {
    "inline_keyboard": [
        [{"text": "Click", "callback_data": "my_command"}],
        [{"text": "Visit", "url": "https://example.com"}]
    ]
}
bot.sendMessage("Choose:", reply_markup=keyboard)
```

### Wait for Answer / Chain Commands

```python
bot.sendMessage("Enter your name:")
Bot.handleNextCommand("save_name")
# In save_name command: use msg to get the user's reply
```

### Run Another Command Immediately

```python
Bot.runCommand("another_command")
Bot.runCommand("another_command", {"key": "value"})
```

### Schedule a Command

```python
job = Bot.runCommandAfter(3600, "send_reminder")
bot.cancelScheduledTask(job["id"])   # Cancel it later
```

### HTTP Requests

```python
response = HTTP.get("https://api.example.com/data")
data = response.json()
status = response.status_code

response = HTTP.post("https://api.example.com/create",
    json={"key": "value"},
    headers={"Content-Type": "application/json"})
```

### Membership Check

```python
result = MembershipCheck.check("@channel", u)
if result.status in ["left", "kicked"]:
    bot.sendMessage("Please join first!")
    raise ReturnCommand
```

### Admin Check

```python
ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    bot.sendMessage("Access denied.")
    raise ReturnCommand
```

### Edit / Delete Messages

```python
sent = bot.sendMessage("Loading...")
msg_id = sent.message_id

# Edit later:
bot.editMessageText("Done!", chat_id=message.chat.id, message_id=msg_id)

# Delete:
try:
    bot.deleteMessage(chat_id=message.chat.id, message_id=msg_id)
except Exception:
    pass
```

### Webhook URL

```python
url = libs.Webhook.getUrlFor("my_command", user_id=u)
bot.sendMessage(f"Webhook URL: {url}")
```

### Random Values

```python
num = libs.Random.randomInt(1, 100)
code = libs.Random.randomStr(8)
winner = libs.Random.randomChoice(["Alice", "Bob", "Carol"])
```

### Date & Time

```python
today = str(libs.DateAndTime.date_now())
now = str(libs.DateAndTime.utcnow())
```

### Error Handling

```python
try:
    amount = int(msg)
except ValueError:
    bot.sendMessage("Enter a valid number.")
    raise ReturnCommand
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

*This document is the official TeleBot Creator (TBC) AI Agent training guide, created by MD Jaid Bin Siyam (Siyam). All syntax, methods, and examples are verified against the official TBC documentation at [help.telebotcreator.com](https://help.telebotcreator.com). © 2026–2027 SkyPayBD. All rights reserved.*
