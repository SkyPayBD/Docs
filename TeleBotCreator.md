# TeleBot Creator (TBC) — Complete AI Agent Training Documentation

> **Purpose:** This document is a complete reference guide for AI agents, developers, and bot creators who want to build powerful Telegram bots using the **TeleBot Creator (TBC)** platform. It covers every feature, syntax rule, built-in function, pattern, and best practice needed to write correct TBC-compatible code from scratch.
>
> **Intended Audience:** AI language models used as coding assistants, Telegram bot developers, and learners who want to understand TBC's custom Python-like scripting system end-to-end.
>
> **Platform Website:** [https://telebotcreator.com/](https://telebotcreator.com/)

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
  YouTube       : https://youtube.com/@Sky-Pay-BD
  Facebook      : https://facebook.com/siyamahmedjsx

  This document was created to help users build Telegram bots
  using the TeleBot Creator (TBC) platform.
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
3. [TBC Scripting Language — Python-Like Syntax](#3-tbc-scripting-language--python-like-syntax)
4. [Commands — The Core Building Block](#4-commands--the-core-building-block)
5. [User Data Management — `User.*`](#5-user-data-management--user)
6. [Global Bot Data Management — `Bot.*`](#6-global-bot-data-management--bot)
7. [Message Handling — Sending & Receiving](#7-message-handling--sending--receiving)
8. [Keyboard Support](#8-keyboard-support)
9. [Command Chaining — `Bot.handleNextCommand`](#9-command-chaining--bothandlenextcommand)
10. [HTTP Requests — `HTTP.get` & `HTTP.post`](#10-http-requests--httpget--httppost)
11. [Wait for Answer — Input Collection System](#11-wait-for-answer--input-collection-system)
12. [Inline Keyboards & Callback Queries](#12-inline-keyboards--callback-queries)
13. [Sending Media — Photos, Documents, Videos](#13-sending-media--photos-documents-videos)
14. [Editing & Deleting Messages](#14-editing--deleting-messages)
15. [Channel Membership Check](#15-channel-membership-check)
16. [Admin-Specific Functions](#16-admin-specific-functions)
17. [Error Handling](#17-error-handling)
18. [Properties & Data Storage Patterns](#18-properties--data-storage-patterns)
19. [Referral System Pattern](#19-referral-system-pattern)
20. [Economy System Pattern](#20-economy-system-pattern)
21. [Broadcast System](#21-broadcast-system)
22. [HTML & Markdown Formatting](#22-html--markdown-formatting)
23. [Security Best Practices](#23-security-best-practices)
24. [Complete Syntax Rules & Common Mistakes](#24-complete-syntax-rules--common-mistakes)
25. [Full Example — /start Command with All Features](#25-full-example--start-command-with-all-features)
26. [Quick Reference Card](#26-quick-reference-card)

---

## 1. What is TeleBot Creator (TBC)?

**TeleBot Creator (TBC)** is a cloud-based platform specifically designed to simplify the creation and management of Telegram bots. It uses a **Python-like scripting syntax** — but it is **NOT standard Python**. TBC does not support `import`, `from`, or any standard Python library. Instead, it provides its own **built-in objects and functions** that the runtime makes available automatically.

### Key Characteristics

- All code runs on **TBC's cloud servers** — you do not host anything yourself.
- The scripting language **looks like Python** but uses TBC's own built-in runtime objects (`bot`, `Bot`, `User`, `HTTP`, `ReplyKeyboardMarkup`, `InlineKeyboardMarkup`).
- You **never use `import` or `from`** — all libraries are built-in.
- You write your bot logic as **commands**, and TBC executes them when users interact with your bot.
- Results from Telegram API calls are often handled through TBC's higher-level wrappers, making development fast and simple.

### TBC vs. Standard Python

| Feature | Standard Python | TBC Scripting |
|---------|----------------|---------------|
| Import libraries | `import requests` | ❌ Not allowed |
| HTTP requests | `requests.get(url)` | `HTTP.get(url)` |
| Send message | `bot.send_message(...)` | `bot.replyText(chat_id, text)` |
| Store data | Database/file | `User.saveData(key, value)` |
| Keyboards | Custom classes | `ReplyKeyboardMarkup()` |
| Run next command | Manual routing | `Bot.handleNextCommand(cmd)` |

---

## 2. Platform Overview & Bot Setup

### Step-by-Step: Creating Your First Bot

1. **Create a Telegram bot** via [@BotFather](https://t.me/BotFather) and copy your bot token.
2. **Sign up** at [telebotcreator.com](https://telebotcreator.com/) using your Telegram account or email.
3. **Add a new bot** in the TBC dashboard and paste your bot token.
4. **Launch** the bot from the dashboard.
5. Open the **Commands** section → tap **+** → **New Command** to start writing code.
6. Write your TBC-compatible script in the code editor and **save**.

### Platform Navigation

| Section | Purpose |
|---------|---------|
| **Dashboard** | Bot status, start/stop controls |
| **Commands** | Create, edit, and manage bot commands |
| **Users** | View all users who have interacted with the bot |
| **Properties** | Inspect stored user and bot data |
| **Errors** | Runtime error log for debugging |
| **Settings** | Bot configuration and token management |

---

## 3. TBC Scripting Language — Python-Like Syntax

TBC uses a **custom scripting language** that resembles Python in appearance but runs in TBC's own sandboxed runtime environment.

### ✅ What You CAN Use

```python
# Variables and assignment
name = "Alice"
count = 0
balance = 100.5
is_active = True

# Conditionals
if balance > 0:
    bot.replyText(user_id, "You have funds!")
elif balance == 0:
    bot.replyText(user_id, "Balance is zero.")
else:
    bot.replyText(user_id, "Insufficient funds.")

# Loops
for i in range(10):
    # do something

while count < 5:
    count += 1

# Functions
def greet_user(name):
    return "Hello, " + name + "!"

# String operations
text = "Hello " + user_name
upper = text.upper()
lower = text.lower()

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
except:
    bot.replyText(user_id, "Invalid input!")

# return (exits a function or command early)
return
```

### ❌ What You CANNOT Use

```python
# ❌ No standard Python imports
import requests          # Not allowed
import json              # Not allowed
from telegram import Bot # Not allowed

# ❌ No async/await
async def handler():     # Not supported
    await something()

# ❌ No standard Python libraries
os.path.exists(...)      # Not available
sys.argv                 # Not available
open("file.txt")         # Not available

# ❌ No external package manager
pip install anything     # Not applicable
```

### Reserved Built-in Names in TBC

These are provided by the TBC runtime automatically — do **not** reuse them as variable names:

| Name | Purpose |
|------|---------|
| `bot` | Send messages, photos, etc. |
| `Bot` | Data storage and command chaining |
| `User` | Per-user data management |
| `HTTP` | External API calls |
| `u` | Current user's Telegram ID (shorthand) |
| `user_id` | Alias for current user's Telegram ID |
| `ReplyKeyboardMarkup` | Create reply keyboard |
| `InlineKeyboardMarkup` | Create inline keyboard |
| `message` | The user's incoming message text |
| `params` | Parameters passed from another command |

---

## 4. Commands — The Core Building Block

A **command** is the fundamental unit of behavior in TBC. Each command is triggered by a specific text or event.

### Command Types

| Trigger | Description |
|---------|------------|
| `/start` | Runs when user types `/start` |
| `/help` | Runs when user types `/help` |
| `Button Label` | Runs when a reply keyboard button is tapped |
| `*` | Wildcard — matches any unrecognized message |
| `@` | Runs when a user starts the bot for the first time |

### Command Structure

Each TBC command has:
- A **trigger name** (e.g., `/start`, `My Profile`)
- Optional **Answer text** (static auto-reply)
- **TBC Script code** (the logic you write)
- Optional **Keyboard** (buttons to show)
- Optional **Wait for answer** flag (to collect user input)

### Example: Basic `/start` Command

```python
# Command: /start

# Send welcome message
bot.replyText(user_id, "👋 Welcome to the bot! Choose an option below.")

# Create reply keyboard
keyboard = ReplyKeyboardMarkup()
keyboard.addButton("💰 Balance")
keyboard.addButton("👤 My Profile")
keyboard.addButton("📋 Tasks")
keyboard.addButton("❓ Help")

# Send keyboard with message
bot.replyText(user_id, "Please select an option:", reply_markup=keyboard)
```

---

## 5. User Data Management — `User.*`

TBC provides built-in methods to store, retrieve, and delete **per-user data** that persists across sessions.

### `User.saveData(key, value)`

Saves a value associated with the current user.

```python
# Save user's name
User.saveData("name", "Alice")

# Save user's balance
User.saveData("balance", 500)

# Save user's referral code
User.saveData("ref_code", "REF123")

# Save a list as JSON string
import_not_needed = str(["item1", "item2"])
User.saveData("cart", import_not_needed)
```

### `User.getData(key)`

Retrieves a value stored for the current user. Returns `None` if not set.

```python
# Read user's name
name = User.getData("name")

# Read with fallback (manual)
balance = User.getData("balance")
if balance is None:
    balance = 0

# Display to user
bot.replyText(user_id, "Your balance: " + str(balance))
```

### `User.deleteData(key)`

Removes a stored value for the current user.

```python
# Delete a specific key
User.deleteData("temp_cart")

# Common pattern: clear session data after completing a flow
User.deleteData("pending_amount")
User.deleteData("pending_action")
```

### Practical Pattern: Saving User Profile on Registration

```python
# Command: /start
# Check if user is already registered
existing = User.getData("registered")
if existing == "true":
    bot.replyText(user_id, "Welcome back! 👋")
else:
    # Register new user
    User.saveData("registered", "true")
    User.saveData("balance", 0)
    User.saveData("join_date", "2026-09-12")
    bot.replyText(user_id, "✅ Account created! Welcome to the bot.")
```

---

## 6. Global Bot Data Management — `Bot.*`

Use `Bot.*` methods for **global data** shared across all users.

### `Bot.saveData(key, value)`

Stores a global value (accessible by all users/commands).

```python
# Store total user count
Bot.saveData("total_users", 1500)

# Store a global announcement
Bot.saveData("announcement", "Server maintenance on Sunday!")

# Store admin list
Bot.saveData("admin_ids", "123456789,987654321")
```

### `Bot.getData(key)`

Retrieves a global value.

```python
# Read total users
total = Bot.getData("total_users")
bot.replyText(user_id, "Total users: " + str(total))

# Read announcement
ann = Bot.getData("announcement")
if ann:
    bot.replyText(user_id, "📢 " + ann)
```

### `Bot.deleteData(key)`

Deletes a global value.

```python
# Clear an old announcement
Bot.deleteData("announcement")
```

### User vs. Bot Data — When to Use Which

| Use Case | Method |
|----------|--------|
| User's balance | `User.saveData("balance", 100)` |
| User's name | `User.saveData("name", "Alice")` |
| Total bot users | `Bot.saveData("total_users", 500)` |
| Global config / API keys | `Bot.saveData("api_key", "xxxx")` |
| Broadcast message text | `Bot.saveData("broadcast_text", "Hello!")` |
| Admin Telegram IDs | `Bot.saveData("admins", "123,456")` |

---

## 7. Message Handling — Sending & Receiving

### Send a Plain Text Message

```python
# Send to current user
bot.replyText(user_id, "Hello! How are you?")

# Send to a specific chat ID
bot.replyText(1234567890, "This is a direct message.")
```

### Send a Message with Markdown Formatting

```python
bot.replyText(user_id, "*Bold text* and _italic text_", parse_mode="Markdown")
```

### Send a Message with HTML Formatting

```python
bot.replyText(user_id, "<b>Bold</b> and <i>italic</i> text", parse_mode="HTML")
```

### Send a Message with Inline Keyboard

```python
keyboard = InlineKeyboardMarkup()
keyboard.addButton("✅ Confirm", callback_data="/confirm")
keyboard.addButton("❌ Cancel", callback_data="/cancel")

bot.replyText(user_id, "Do you want to proceed?", reply_markup=keyboard)
```

### Reading the User's Incoming Message

When the user sends a text message, it is available as the variable `message`:

```python
# Command: * (wildcard — catches any unrecognized text)
user_input = message
bot.replyText(user_id, "You said: " + user_input)
```

### Reading Command Parameters

If the user sends `/order 5`, the `params` variable contains `"5"`:

```python
# Command: /order
qty = int(params) if params else 0
bot.replyText(user_id, "You ordered: " + str(qty) + " items.")
```

---

## 8. Keyboard Support

### Reply Keyboard (Persistent Buttons)

A reply keyboard appears below the text input. Tapping a button sends that button's label as a message.

```python
# Create a reply keyboard
keyboard = ReplyKeyboardMarkup()
keyboard.addButton("💰 Balance")
keyboard.addButton("📋 My Tasks")
keyboard.newRow()           # Start a new row of buttons
keyboard.addButton("👤 Profile")
keyboard.addButton("❓ Help")

# Send with message
bot.replyText(user_id, "Main Menu:", reply_markup=keyboard)
```

### Reply Keyboard with Multiple Rows — Alternative Pattern

```python
keyboard = ReplyKeyboardMarkup()
# Row 1
keyboard.addButton("🛍 Shop")
keyboard.addButton("📦 Orders")
keyboard.newRow()
# Row 2
keyboard.addButton("⚙️ Settings")
keyboard.addButton("❓ Help")
keyboard.newRow()
# Row 3
keyboard.addButton("💬 Contact Support")

bot.replyText(user_id, "Choose an option:", reply_markup=keyboard)
```

### Remove Reply Keyboard

```python
# Send a message that removes the keyboard
bot.replyText(user_id, "Keyboard removed.", remove_keyboard=True)
```

### Connecting Keyboard Buttons to Commands

Since tapping a button sends the button label as text, create a command with the exact button text as its trigger name:

- Button label: `💰 Balance`
- Command name: `💰 Balance`

When the user taps the button, TBC runs the `💰 Balance` command automatically.

### Inline Keyboard (Buttons Inside Message)

Inline keyboards are attached to a specific message and use callback data.

```python
keyboard = InlineKeyboardMarkup()
keyboard.addButton("✅ Yes", callback_data="/confirm_yes")
keyboard.addButton("❌ No", callback_data="/confirm_no")
keyboard.newRow()
keyboard.addButton("🌐 Visit Website", url="https://example.com")

bot.replyText(user_id, "Are you sure?", reply_markup=keyboard)
```

### Inline Keyboard — Two-Column Layout

```python
keyboard = InlineKeyboardMarkup()
# Row 1: two buttons side by side
keyboard.addButton("Option A", callback_data="/opt_a")
keyboard.addButton("Option B", callback_data="/opt_b")
keyboard.newRow()
# Row 2: one wide button
keyboard.addButton("🔙 Back to Menu", callback_data="/main_menu")

bot.replyText(user_id, "Select an option:", reply_markup=keyboard)
```

---

## 9. Command Chaining — `Bot.handleNextCommand`

`Bot.handleNextCommand` is used to **chain commands** — after the current command finishes, TBC runs another command automatically.

### Basic Usage

```python
# Command: /start
bot.replyText(user_id, "Welcome! Let's set up your account.")
Bot.handleNextCommand("/ask_name")
```

### `keep=True` Parameter

When `keep=True`, TBC remembers the next command even if the user sends other messages before arriving there. When `keep=False` (default), the chain is cleared after one interaction.

```python
# Direct to next step and keep the chain active
Bot.handleNextCommand("/enter_amount", keep=True)
```

### Multi-Step Command Chain Example

```python
# Command: /withdraw
bot.replyText(user_id, "💸 Withdrawal Process Started")
Bot.handleNextCommand("/withdraw_step1", keep=True)

# Command: /withdraw_step1
# Answer field: "Enter the amount you want to withdraw:"
# (Wait for Answer: ON)
amount = message
if not amount.isdigit():
    bot.replyText(user_id, "❌ Please enter a valid number.")
    Bot.handleNextCommand("/withdraw_step1", keep=True)
    return
User.saveData("withdraw_amount", amount)
bot.replyText(user_id, "Enter your wallet address:")
Bot.handleNextCommand("/withdraw_step2", keep=True)

# Command: /withdraw_step2
wallet = message
amount = User.getData("withdraw_amount")
bot.replyText(user_id,
    "✅ Withdrawal Request:\n"
    "Amount: " + amount + "\n"
    "Wallet: " + wallet + "\n\n"
    "Processing..."
)
User.deleteData("withdraw_amount")
```

---

## 10. HTTP Requests — `HTTP.get` & `HTTP.post`

Use TBC's built-in `HTTP` object to call external APIs and web services.

### GET Request

```python
# Simple GET request
response = HTTP.get("https://api.example.com/data")

# Access response text
bot.replyText(user_id, "Response: " + response.text)

# Parse JSON response
data = response.json()
price = data["price"]
bot.replyText(user_id, "Current price: " + str(price))
```

### GET Request with Headers

```python
response = HTTP.get(
    "https://api.example.com/user",
    headers={"Authorization": "Bearer YOUR_API_TOKEN"}
)

if response.status_code == 200:
    data = response.json()
    bot.replyText(user_id, "Name: " + data["name"])
else:
    bot.replyText(user_id, "❌ Failed to fetch data.")
```

### POST Request (JSON Body)

```python
payload = {
    "user_id": user_id,
    "amount": 100,
    "currency": "USDT"
}

response = HTTP.post(
    "https://api.payment-gateway.com/create",
    data=payload,
    headers={"Content-Type": "application/json"}
)

result = response.json()
if result.get("status") == "success":
    bot.replyText(user_id, "✅ Payment link: " + result["link"])
else:
    bot.replyText(user_id, "❌ Payment creation failed.")
```

### POST Request (Form Data)

```python
response = HTTP.post(
    "https://api.example.com/login",
    data={"username": "admin", "password": "secret123"}
)

bot.replyText(user_id, "Status: " + str(response.status_code))
```

### Response Object Properties

| Property | Description |
|----------|-------------|
| `response.text` | Raw response body as string |
| `response.json()` | Parse response body as JSON (dict) |
| `response.status_code` | HTTP status code (e.g., 200, 404) |
| `response.headers` | Response headers dict |

### Handling HTTP Errors Gracefully

```python
try:
    response = HTTP.get("https://api.example.com/prices")
    if response.status_code != 200:
        bot.replyText(user_id, "❌ API error: " + str(response.status_code))
        return
    data = response.json()
    bot.replyText(user_id, "BTC Price: $" + str(data["bitcoin"]["usd"]))
except:
    bot.replyText(user_id, "❌ Failed to connect to API. Try again later.")
```

---

## 11. Wait for Answer — Input Collection System

**Wait for Answer** is one of TBC's most powerful features. It pauses the bot and waits for the user's next message before executing the command's code.

### How It Works

1. User triggers a command (e.g., `/setname`).
2. TBC sends the **Answer** text (the prompt message) immediately.
3. TBC **pauses** and waits for the user's next reply.
4. When the user replies, TBC runs the **command's code**.
5. Inside the code, the user's reply is available as the `message` variable.

### Setting Up Wait for Answer

In the TBC command editor:
- Set **Answer** field to your prompt text (e.g., "What is your name?")
- Enable the **Wait for Answer** toggle
- Write your processing code in the script section

### Example: Collecting User Name

```python
# Command: /setname
# Answer: "Please enter your name:"
# Wait for Answer: ON

name = message.strip()
if not name:
    bot.replyText(user_id, "❌ Name cannot be empty. Try again.")
    Bot.handleNextCommand("/setname", keep=True)
    return

User.saveData("name", name)
bot.replyText(user_id, "✅ Your name has been saved as: " + name)
```

### Example: Collecting a Number with Validation

```python
# Command: /deposit
# Answer: "Enter the amount you want to deposit (minimum 10):"
# Wait for Answer: ON

try:
    amount = int(message)
    if amount < 10:
        bot.replyText(user_id, "❌ Minimum deposit is 10. Try again.")
        Bot.handleNextCommand("/deposit", keep=True)
        return
    User.saveData("pending_deposit", amount)
    bot.replyText(user_id, "✅ Deposit amount set: " + str(amount))
except:
    bot.replyText(user_id, "❌ Invalid amount. Please enter a number.")
    Bot.handleNextCommand("/deposit", keep=True)
```

### Cancel Pattern

Always provide a `/cancel` command so users can exit a wait loop:

```python
# Command: /cancel
# Answer: (empty)
# Wait for Answer: OFF

User.deleteData("pending_deposit")
User.deleteData("pending_action")
bot.replyText(user_id, "❌ Action cancelled. Returning to main menu.")
Bot.handleNextCommand("/start")
```

---

## 12. Inline Keyboards & Callback Queries

### Creating Inline Keyboards

```python
keyboard = InlineKeyboardMarkup()
keyboard.addButton("✅ Confirm Payment", callback_data="/pay_confirm")
keyboard.addButton("❌ Cancel", callback_data="/pay_cancel")
keyboard.newRow()
keyboard.addButton("📞 Contact Support", url="https://t.me/support_username")

bot.replyText(user_id, "💳 Payment Details:\nAmount: $10\nPlan: Premium", reply_markup=keyboard)
```

### Handling Callback — the Callback Command

When a user taps an inline button with `callback_data="/pay_confirm"`, TBC runs the `/pay_confirm` command:

```python
# Command: /pay_confirm

balance = int(User.getData("balance") or 0)
cost = 100

if balance < cost:
    bot.replyText(user_id, "❌ Insufficient balance. You need " + str(cost) + " coins.")
    return

User.saveData("balance", balance - cost)
User.saveData("is_premium", "true")
bot.replyText(user_id, "🎉 Premium activated! Your new balance: " + str(balance - cost) + " coins.")
```

### Inline Keyboard with URL Buttons

```python
keyboard = InlineKeyboardMarkup()
keyboard.addButton("🌐 Visit Website", url="https://telebotcreator.com")
keyboard.newRow()
keyboard.addButton("📢 Join Channel", url="https://t.me/your_channel")
keyboard.newRow()
keyboard.addButton("🔙 Back", callback_data="/main_menu")

bot.replyText(user_id, "Choose an option:", reply_markup=keyboard)
```

### Dynamic Inline Keyboard (Generated from Data)

```python
# Command: /shop

items = [
    {"name": "Basic Plan", "price": 100, "cmd": "/buy_basic"},
    {"name": "Pro Plan", "price": 250, "cmd": "/buy_pro"},
    {"name": "VIP Plan", "price": 500, "cmd": "/buy_vip"},
]

keyboard = InlineKeyboardMarkup()
for item in items:
    keyboard.addButton(
        item["name"] + " — " + str(item["price"]) + " coins",
        callback_data=item["cmd"]
    )
    keyboard.newRow()

keyboard.addButton("🔙 Back", callback_data="/start")

bot.replyText(user_id, "🛍 <b>Shop</b>\nChoose a plan:", reply_markup=keyboard, parse_mode="HTML")
```

---

## 13. Sending Media — Photos, Documents, Videos

### Send a Photo

```python
# Send by URL
bot.sendPhoto(user_id, "https://example.com/image.jpg")

# Send with caption
bot.sendPhoto(user_id, "https://example.com/banner.png", caption="Welcome to our bot! 🎉")

# Send with HTML caption
bot.sendPhoto(
    user_id,
    "https://example.com/profile.jpg",
    caption="<b>Your Profile Picture</b>",
    parse_mode="HTML"
)
```

### Send a Document / File

```python
# Send a PDF file
bot.sendDocument(user_id, "https://example.com/guide.pdf", caption="📄 User Guide")

# Send a ZIP file
bot.sendDocument(user_id, "https://example.com/assets.zip", caption="📦 Download Assets")
```

### Send a Video

```python
bot.sendVideo(user_id, "https://example.com/tutorial.mp4", caption="🎬 Watch the tutorial")
```

### Send an Animation (GIF)

```python
bot.sendAnimation(user_id, "https://example.com/welcome.gif", caption="👋 Welcome!")
```

### Send Audio

```python
bot.sendAudio(user_id, "https://example.com/music.mp3", caption="🎵 Enjoy the music")
```

### Send a Voice Message

```python
bot.sendVoice(user_id, "https://example.com/voice.ogg")
```

### Send Photo with Inline Keyboard

```python
keyboard = InlineKeyboardMarkup()
keyboard.addButton("🛒 Buy Now", callback_data="/buy")
keyboard.addButton("ℹ️ More Info", callback_data="/info")

bot.sendPhoto(
    user_id,
    "https://example.com/product.jpg",
    caption="🔥 Special Offer — Only $9.99!",
    reply_markup=keyboard
)
```

---

## 14. Editing & Deleting Messages

### Save a Message ID First

Before you can edit or delete a message, you must save its `message_id` after sending it.

```python
# TBC returns the sent message object
sent = bot.replyText(user_id, "⏳ Processing your request...")
msg_id = sent.message_id
User.saveData("last_msg_id", str(msg_id))
```

### Edit a Sent Message

```python
msg_id = User.getData("last_msg_id")
bot.editMessage(
    user_id,
    int(msg_id),
    "✅ Your request has been completed!"
)
```

### Edit a Message with HTML Formatting

```python
msg_id = User.getData("last_msg_id")
bot.editMessage(
    user_id,
    int(msg_id),
    "<b>✅ Done!</b>\n\n<i>Transaction ID: TXN-12345</i>",
    parse_mode="HTML"
)
```

### Edit a Message's Inline Keyboard

```python
msg_id = User.getData("last_msg_id")
new_keyboard = InlineKeyboardMarkup()
new_keyboard.addButton("✅ Completed", callback_data="/done")

bot.editMessageReplyMarkup(user_id, int(msg_id), reply_markup=new_keyboard)
```

### Delete a Message

```python
msg_id = User.getData("last_msg_id")
try:
    bot.deleteMessage(user_id, int(msg_id))
except:
    pass  # Message may already be deleted or too old
```

### Pattern: Processing Message That Updates Itself

```python
# Step 1: Send the initial "processing" message
sent = bot.replyText(user_id, "⏳ Processing payment...")
msg_id = sent.message_id
User.saveData("proc_msg_id", str(msg_id))

# Step 2: Make an API call (e.g., check payment)
response = HTTP.get("https://api.payment.com/check?id=XYZ")
result = response.json()

# Step 3: Edit the message with the result
msg_id = User.getData("proc_msg_id")
if result.get("status") == "paid":
    bot.editMessage(user_id, int(msg_id), "✅ Payment confirmed!")
else:
    bot.editMessage(user_id, int(msg_id), "❌ Payment not found. Try again.")
```

---

## 15. Channel Membership Check

A common requirement is verifying that the user has joined your Telegram channel before they can use the bot.

### Basic Membership Check

```python
# Command: /start

def check_membership():
    channels = ["@your_channel1", "@your_channel2"]
    for channel in channels:
        member_info = bot.getChatMember(channel, user_id)
        if member_info.status in ["left", "kicked"]:
            return False
    return True

if not check_membership():
    keyboard = InlineKeyboardMarkup()
    keyboard.addButton("📢 Join Channel 1", url="https://t.me/your_channel1")
    keyboard.addButton("📢 Join Channel 2", url="https://t.me/your_channel2")
    keyboard.newRow()
    keyboard.addButton("✅ I've Joined — Check Again", callback_data="/start")

    bot.replyText(
        user_id,
        "⚠️ <b>Join Required!</b>\n\nPlease join all our channels to use this bot.",
        reply_markup=keyboard,
        parse_mode="HTML"
    )
    return

# User has joined — proceed with bot
bot.replyText(user_id, "✅ Welcome! You are verified.")
```

### Membership Status Values

| Status | Meaning |
|--------|---------|
| `"member"` | Active member |
| `"administrator"` | Channel admin |
| `"creator"` | Channel owner |
| `"left"` | Not a member |
| `"kicked"` | Was banned |
| `"restricted"` | Restricted member |

---

## 16. Admin-Specific Functions

### Restricting Commands to Admins

```python
# At the top of any admin-only command
ADMIN_IDS = ["123456789", "987654321"]  # Replace with real admin Telegram IDs

if str(u) not in ADMIN_IDS:
    bot.replyText(user_id, "❌ Access denied. This command is for admins only.")
    return

# Admin-only code continues here
bot.replyText(user_id, "✅ Welcome, Admin!")
```

### Using `u` for User ID Check

In TBC, `u` is a shorthand for the current user's Telegram ID:

```python
# Check if user is admin
if u != "123456789":
    bot.replyText(user_id, "⛔ You don't have permission.")
    return
```

### Admin Command: View Bot Stats

```python
# Command: /stats (admin only)
ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    return

total_users = Bot.getData("total_users") or 0
total_withdrawals = Bot.getData("total_withdrawals") or 0

bot.replyText(user_id,
    "📊 <b>Bot Statistics</b>\n\n"
    "👥 Total Users: <code>" + str(total_users) + "</code>\n"
    "💸 Total Withdrawals: <code>" + str(total_withdrawals) + "</code>",
    parse_mode="HTML"
)
```

### Admin Command: Send Announcement to a User

```python
# Command: /message_user (admin only)
# Wait for Answer: ON
# Answer: "Enter: telegramId|message"

parts = message.split("|", 1)
if len(parts) != 2:
    bot.replyText(user_id, "❌ Format: telegramId|Your message here")
    return

target_id = parts[0].strip()
msg_text = parts[1].strip()

try:
    bot.replyText(int(target_id), "📨 Message from Admin:\n\n" + msg_text)
    bot.replyText(user_id, "✅ Message sent to user " + target_id)
except:
    bot.replyText(user_id, "❌ Failed to send message. Check the user ID.")
```

---

## 17. Error Handling

### Basic try/except

```python
try:
    amount = int(message)
    balance = int(User.getData("balance") or 0)

    if balance < amount:
        bot.replyText(user_id, "❌ Insufficient balance!")
        return

    User.saveData("balance", balance - amount)
    bot.replyText(user_id, "✅ Deducted " + str(amount) + " coins.")

except ValueError:
    bot.replyText(user_id, "❌ Please enter a valid number.")
except Exception as e:
    bot.replyText(user_id, "⚠️ An error occurred. Please try again.")
```

### Validating HTTP Responses

```python
try:
    response = HTTP.get("https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd")
    if response.status_code != 200:
        bot.replyText(user_id, "❌ API error. Status: " + str(response.status_code))
        return
    data = response.json()
    price = data["bitcoin"]["usd"]
    bot.replyText(user_id, "₿ Bitcoin Price: $" + str(price))
except:
    bot.replyText(user_id, "❌ Could not fetch price. Try again later.")
```

### Input Validation Patterns

```python
# Validate numeric input
amount_str = message.strip()
if not amount_str.isdigit():
    bot.replyText(user_id, "❌ Invalid input. Please enter a number only.")
    Bot.handleNextCommand("/ask_amount", keep=True)
    return

amount = int(amount_str)
if amount <= 0 or amount > 100000:
    bot.replyText(user_id, "❌ Amount must be between 1 and 100,000.")
    Bot.handleNextCommand("/ask_amount", keep=True)
    return
```

---

## 18. Properties & Data Storage Patterns

### Storing Complex Data (Using String Serialization)

Since TBC stores data as strings, you need to handle lists and dicts carefully:

```python
# Save a list (convert to comma-separated string)
items = ["apple", "banana", "cherry"]
User.saveData("cart", ",".join(items))

# Read it back
cart_str = User.getData("cart")
if cart_str:
    cart = cart_str.split(",")
else:
    cart = []
```

### Storing Numeric Data

```python
# Always convert to int/float when reading back
User.saveData("balance", "500")     # save as string
balance = int(User.getData("balance") or 0)

# Or store numbers directly
User.saveData("score", 1500)
score = int(User.getData("score") or 0)
```

### Global Counter Pattern (e.g., Total Users)

```python
# In /start command — increment user counter
existing = User.getData("registered")
if not existing:
    User.saveData("registered", "true")
    total = int(Bot.getData("total_users") or 0)
    Bot.saveData("total_users", total + 1)
```

### Storing Timestamps

```python
import_not_needed = ""  # TBC doesn't need import

# Save current time marker (use a sequential counter or passed value)
User.saveData("last_active", "2026-09-12")

# Check stored date
last_active = User.getData("last_active")
bot.replyText(user_id, "Last active: " + str(last_active))
```

---

## 19. Referral System Pattern

A complete referral system that tracks who referred whom and rewards the referrer.

### Step 1 — Generate & Show Referral Link

```python
# Command: /refer or "🔗 My Referral Link" button

ref_link = "https://t.me/YourBotUsername?start=ref_" + str(user_id)
ref_count = int(User.getData("ref_count") or 0)
earnings = int(User.getData("ref_earnings") or 0)

bot.replyText(
    user_id,
    "🔗 <b>Your Referral Link</b>\n\n"
    "<code>" + ref_link + "</code>\n\n"
    "👥 Total Referrals: <b>" + str(ref_count) + "</b>\n"
    "💰 Total Earned: <b>" + str(earnings) + " coins</b>\n\n"
    "📢 Share your link and earn <b>100 coins</b> per referral!",
    parse_mode="HTML"
)
```

### Step 2 — Handle New User Starting via Referral Link

```python
# Command: /start
# params will contain "ref_123456789" if user came via referral link

existing = User.getData("registered")

if not existing:
    # New user — register them
    User.saveData("registered", "true")
    User.saveData("balance", 0)

    # Increment global user count
    total = int(Bot.getData("total_users") or 0)
    Bot.saveData("total_users", total + 1)

    # Check if they came via referral
    start_param = params or ""
    if start_param.startswith("ref_"):
        referrer_id = start_param.replace("ref_", "")

        # Make sure the referrer is not self-referring
        if referrer_id != str(user_id):
            # Check this referrer hasn't already been credited for this user
            already_credited = User.getData("ref_credited_by")
            if not already_credited:
                User.saveData("ref_credited_by", referrer_id)

                # Reward the referrer
                # (TBC limitation: we must read/update referrer data via Bot global storage with key per user)
                ref_balance_key = "ref_bal_" + referrer_id
                ref_count_key = "ref_count_" + referrer_id
                ref_earn_key = "ref_earn_" + referrer_id

                ref_balance = int(Bot.getData(ref_balance_key) or 0)
                ref_count = int(Bot.getData(ref_count_key) or 0)
                ref_earn = int(Bot.getData(ref_earn_key) or 0)

                Bot.saveData(ref_balance_key, ref_balance + 100)
                Bot.saveData(ref_count_key, ref_count + 1)
                Bot.saveData(ref_earn_key, ref_earn + 100)

                # Notify the referrer
                try:
                    bot.replyText(
                        int(referrer_id),
                        "🎉 Someone joined using your referral link!\n"
                        "✅ +100 coins added to your balance!"
                    )
                except:
                    pass  # Referrer might have blocked the bot

        bot.replyText(user_id, "✅ Welcome! You joined via a referral link. 🎁")
    else:
        bot.replyText(user_id, "✅ Welcome to the bot!")

else:
    # Existing user
    bot.replyText(user_id, "Welcome back! 👋")

# Show main menu keyboard
keyboard = ReplyKeyboardMarkup()
keyboard.addButton("💰 Balance")
keyboard.addButton("🔗 My Referral Link")
keyboard.newRow()
keyboard.addButton("📋 Tasks")
keyboard.addButton("❓ Help")
bot.replyText(user_id, "Main Menu:", reply_markup=keyboard)
```

### Step 3 — User Checks Their Referral Stats

```python
# Command: /refstats or "🔗 My Referral Link" (reuse from Step 1)

ref_count_key = "ref_count_" + str(user_id)
ref_earn_key = "ref_earn_" + str(user_id)

ref_count = int(Bot.getData(ref_count_key) or 0)
ref_earn = int(Bot.getData(ref_earn_key) or 0)
ref_link = "https://t.me/YourBotUsername?start=ref_" + str(user_id)

bot.replyText(
    user_id,
    "📊 <b>Your Referral Stats</b>\n\n"
    "👥 Referrals: <b>" + str(ref_count) + "</b>\n"
    "💰 Earned: <b>" + str(ref_earn) + " coins</b>\n\n"
    "🔗 Your Link:\n<code>" + ref_link + "</code>",
    parse_mode="HTML"
)
```

---

## 20. Economy System Pattern

A full economy bot using balance, daily bonus, transfers, and admin top-up.

### Check Balance

```python
# Command: /balance or "💰 Balance" button

balance = int(User.getData("balance") or 0)
bot.replyText(
    user_id,
    "💰 <b>Your Balance</b>\n\n"
    "Coins: <code>" + str(balance) + "</code>",
    parse_mode="HTML"
)
```

### Daily Bonus

```python
# Command: /daily or "🎁 Daily Bonus" button

import_note = ""  # No imports needed in TBC

# Check if already claimed today
last_claim = User.getData("last_daily")
today = "2026-09-12"  # In real use, pass date via params or Bot.getData("today_date")

if last_claim == today:
    bot.replyText(user_id, "⏳ You already claimed your daily bonus today!\nCome back tomorrow.")
    return

# Give bonus
BONUS_AMOUNT = 100
balance = int(User.getData("balance") or 0)
User.saveData("balance", balance + BONUS_AMOUNT)
User.saveData("last_daily", today)

bot.replyText(
    user_id,
    "✅ <b>Daily Bonus Claimed!</b>\n\n"
    "+ " + str(BONUS_AMOUNT) + " coins added!\n"
    "New Balance: <code>" + str(balance + BONUS_AMOUNT) + "</code> coins",
    parse_mode="HTML"
)
```

### Transfer Coins Between Users

```python
# Command: /send
# Wait for Answer: ON
# Answer: "Enter: ReceiverTelegramID Amount\nExample: 123456789 100"

parts = message.strip().split()
if len(parts) != 2:
    bot.replyText(user_id, "❌ Invalid format.\nUse: <ReceiverID> <Amount>")
    Bot.handleNextCommand("/send", keep=True)
    return

receiver_id = parts[0]
try:
    amount = int(parts[1])
except:
    bot.replyText(user_id, "❌ Invalid amount. Please enter a number.")
    Bot.handleNextCommand("/send", keep=True)
    return

# Validations
if receiver_id == str(user_id):
    bot.replyText(user_id, "❌ You cannot send coins to yourself.")
    return

if amount <= 0:
    bot.replyText(user_id, "❌ Amount must be greater than 0.")
    return

my_balance = int(User.getData("balance") or 0)
if my_balance < amount:
    bot.replyText(user_id, "❌ Insufficient balance.\nYour balance: " + str(my_balance))
    return

# Execute transfer
User.saveData("balance", my_balance - amount)

# Update receiver balance (stored per user via Bot global with per-user key)
recv_bal_key = "balance_" + receiver_id
recv_balance = int(Bot.getData(recv_bal_key) or 0)
Bot.saveData(recv_bal_key, recv_balance + amount)

# Notify both parties
bot.replyText(user_id, "✅ Sent " + str(amount) + " coins to user " + receiver_id + "\nYour new balance: " + str(my_balance - amount))
try:
    bot.replyText(int(receiver_id), "💰 You received " + str(amount) + " coins from user " + str(user_id) + "!")
except:
    pass
```

### Admin Top-Up

```python
# Command: /topup (admin only)
# Wait for Answer: ON
# Answer: "Enter: telegramId amount"

ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    bot.replyText(user_id, "❌ Access denied.")
    return

parts = message.strip().split()
if len(parts) != 2:
    bot.replyText(user_id, "❌ Format: <telegramId> <amount>")
    return

target_id = parts[0]
try:
    amount = int(parts[1])
except:
    bot.replyText(user_id, "❌ Invalid amount.")
    return

bal_key = "balance_" + target_id
current = int(Bot.getData(bal_key) or 0)
Bot.saveData(bal_key, current + amount)

bot.replyText(user_id, "✅ Added " + str(amount) + " coins to user " + target_id)
try:
    bot.replyText(int(target_id), "💰 Admin credited " + str(amount) + " coins to your account!")
except:
    pass
```

---

## 21. Broadcast System

Send a message to all users of your bot.

### Register Users in Broadcast List

```python
# In /start command — add user to broadcast list
all_users_str = Bot.getData("broadcast_users") or ""
if str(user_id) not in all_users_str:
    if all_users_str:
        all_users_str += "," + str(user_id)
    else:
        all_users_str = str(user_id)
    Bot.saveData("broadcast_users", all_users_str)
```

### Send Broadcast (Admin Only)

```python
# Command: /broadcast
# Wait for Answer: ON
# Answer: "Enter your broadcast message:"

ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    bot.replyText(user_id, "❌ Access denied.")
    return

broadcast_text = message.strip()
if not broadcast_text:
    bot.replyText(user_id, "❌ Message cannot be empty.")
    return

# Get all users
all_users_str = Bot.getData("broadcast_users") or ""
if not all_users_str:
    bot.replyText(user_id, "❌ No users found.")
    return

user_ids = all_users_str.split(",")
sent = 0
failed = 0

for uid in user_ids:
    uid = uid.strip()
    if not uid:
        continue
    try:
        bot.replyText(int(uid), "📢 <b>Announcement</b>\n\n" + broadcast_text, parse_mode="HTML")
        sent += 1
    except:
        failed += 1

bot.replyText(user_id,
    "📊 <b>Broadcast Complete</b>\n\n"
    "✅ Sent: " + str(sent) + "\n"
    "❌ Failed: " + str(failed),
    parse_mode="HTML"
)
```

---

## 22. HTML & Markdown Formatting

### HTML Formatting (Recommended)

Use `parse_mode="HTML"` in your `bot.replyText()` calls.

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
bot.replyText(
    user_id,
    "<b>💰 Your Account</b>\n\n"
    "Name: <code>" + str(User.getData("name")) + "</code>\n"
    "Balance: <b>" + str(User.getData("balance")) + " coins</b>\n"
    "<i>Last updated: Today</i>",
    parse_mode="HTML"
)
```

### Markdown Formatting

Use `parse_mode="Markdown"` in your `bot.replyText()` calls.

| Syntax | Result |
|--------|--------|
| `*text*` | **Bold** |
| `_text_` | *Italic* |
| `` `text` `` | `Monospace` |
| ` ```text``` ` | Code block |
| `[text](URL)` | [Hyperlink] |

```python
bot.replyText(
    user_id,
    "*💰 Your Balance*\n\n"
    "Coins: `" + str(balance) + "`\n"
    "_Updated just now_",
    parse_mode="Markdown"
)
```

---

## 23. Security Best Practices

### Always Validate User Input

```python
# Never trust raw message input
amount_str = (message or "").strip()

# Validate it's a number
if not amount_str.isdigit():
    bot.replyText(user_id, "❌ Please enter a valid number.")
    return

amount = int(amount_str)

# Validate range
if amount < 1 or amount > 1000000:
    bot.replyText(user_id, "❌ Amount must be between 1 and 1,000,000.")
    return
```

### Restrict Admin Commands Using ID Check

```python
ADMIN_IDS = ["123456789", "987654321"]

if str(u) not in ADMIN_IDS:
    bot.replyText(user_id, "⛔ Access denied.")
    return
```

### Prevent Double Claiming (Idempotency)

```python
# Prevent claiming a bonus twice
already_claimed = User.getData("bonus_claimed_today")
if already_claimed == "yes":
    bot.replyText(user_id, "⏳ Already claimed. Come back tomorrow!")
    return

# Give bonus
User.saveData("bonus_claimed_today", "yes")
# ... proceed with bonus logic
```

### Validate Webhook or External Callbacks

```python
# Always check expected data structure before processing
response = HTTP.get("https://api.gateway.com/check?txn=" + txn_id)
try:
    data = response.json()
except:
    bot.replyText(user_id, "❌ Invalid payment response.")
    return

# Only proceed if status is explicitly "paid"
if data.get("status") != "paid":
    bot.replyText(user_id, "❌ Payment not confirmed.")
    return

# Safe to proceed
bot.replyText(user_id, "✅ Payment confirmed!")
```

### Keep Sensitive Keys in Bot Data, Not Hardcoded

```python
# ✅ Store API keys via Bot.saveData (set once from admin command)
# Then read them at runtime:
api_key = Bot.getData("payment_api_key")

# ❌ Never hardcode in script
api_key = "sk_live_hardcoded_key_here"   # Bad practice
```

---

## 24. Complete Syntax Rules & Common Mistakes

### ✅ Correct Patterns

```python
# ✅ Correct: Convert message to int safely
try:
    amount = int(message)
except:
    bot.replyText(user_id, "Enter a valid number.")
    return

# ✅ Correct: Always check if data exists before using
balance = User.getData("balance")
if balance is None:
    balance = 0
else:
    balance = int(balance)

# ✅ Correct: Use str() when concatenating numbers with strings
bot.replyText(user_id, "Balance: " + str(balance) + " coins")

# ✅ Correct: Guard against missing params
start_param = params or ""

# ✅ Correct: Use .strip() on user input
name = message.strip()
if not name:
    bot.replyText(user_id, "Name cannot be empty.")
    return

# ✅ Correct: Try/except around delete operations
try:
    bot.deleteMessage(user_id, int(msg_id))
except:
    pass

# ✅ Correct: Check membership status properly
member = bot.getChatMember("@channel", user_id)
if member.status in ["left", "kicked"]:
    bot.replyText(user_id, "Please join first.")
    return
```

### ❌ Incorrect Patterns (Will Cause Errors)

```python
# ❌ Never use import or from
import requests          # Not allowed in TBC
from json import loads   # Not allowed in TBC

# ❌ Never use async/await
async def handle():      # Not supported
    await bot.send(...)

# ❌ Never use standard Python libraries
with open("file.txt") as f:    # Not available
    content = f.read()

# ❌ Never concatenate int directly with str
bot.replyText(user_id, "Balance: " + balance)   # Crashes if balance is int
# ✅ Fix:
bot.replyText(user_id, "Balance: " + str(balance))

# ❌ Never assume User.getData returns a non-None value
balance = int(User.getData("balance"))   # Crashes if never set (None)
# ✅ Fix:
balance = int(User.getData("balance") or 0)

# ❌ Never ignore type mismatch
amount = User.getData("balance") + 100   # String + int crash
# ✅ Fix:
amount = int(User.getData("balance") or 0) + 100

# ❌ Never use == to check None in Python-like code
if balance == None:     # Works but bad style
# ✅ Fix:
if balance is None:

# ❌ Never send an int as chat_id without int()
bot.replyText("123456789", "Hello")   # May cause type error
# ✅ Fix:
bot.replyText(int("123456789"), "Hello")
```

---

## 25. Full Example — /start Command with All Features

This is a complete, production-ready `/start` command that includes channel verification, referral handling, and a full main menu keyboard.

```python
# Command: /start
# Wait for Answer: OFF

# ─────────────────────────────────────────────
# STEP 1: Channel Membership Check
# ─────────────────────────────────────────────
REQUIRED_CHANNELS = ["@your_channel1", "@your_channel2"]

def is_member():
    for ch in REQUIRED_CHANNELS:
        try:
            info = bot.getChatMember(ch, user_id)
            if info.status in ["left", "kicked"]:
                return False
        except:
            return False
    return True

if not is_member():
    keyboard = InlineKeyboardMarkup()
    for ch in REQUIRED_CHANNELS:
        keyboard.addButton("📢 Join " + ch, url="https://t.me/" + ch.replace("@", ""))
        keyboard.newRow()
    keyboard.addButton("✅ I've Joined — Verify", callback_data="/start")

    bot.replyText(
        user_id,
        "⚠️ <b>Channel Membership Required</b>\n\n"
        "Please join all our channels to use this bot.\n"
        "After joining, tap the verify button below.",
        reply_markup=keyboard,
        parse_mode="HTML"
    )
    return

# ─────────────────────────────────────────────
# STEP 2: New User Registration
# ─────────────────────────────────────────────
existing = User.getData("registered")

if not existing:
    User.saveData("registered", "true")
    User.saveData("balance", 0)

    # Increment total user count
    total = int(Bot.getData("total_users") or 0)
    Bot.saveData("total_users", total + 1)

    # Add to broadcast list
    all_users = Bot.getData("broadcast_users") or ""
    if str(user_id) not in all_users:
        Bot.saveData("broadcast_users",
            all_users + ("," if all_users else "") + str(user_id))

    # Handle referral
    start_param = params or ""
    if start_param.startswith("ref_"):
        referrer_id = start_param.replace("ref_", "")
        if referrer_id != str(user_id):
            already_credited = User.getData("ref_credited")
            if not already_credited:
                User.saveData("ref_credited", "true")

                # Reward referrer
                ref_bal = int(Bot.getData("ref_bal_" + referrer_id) or 0)
                ref_cnt = int(Bot.getData("ref_cnt_" + referrer_id) or 0)
                Bot.saveData("ref_bal_" + referrer_id, ref_bal + 100)
                Bot.saveData("ref_cnt_" + referrer_id, ref_cnt + 1)

                try:
                    bot.replyText(int(referrer_id),
                        "🎉 Someone joined using your referral link!\n"
                        "✅ +100 coins added to your account!")
                except:
                    pass

    welcome_msg = (
        "🎉 <b>Welcome to the Bot!</b>\n\n"
        "Your account has been created successfully.\n"
        "Starting balance: <code>0 coins</code>\n\n"
        "Use the menu below to get started!"
    )
    bot.replyText(user_id, welcome_msg, parse_mode="HTML")

else:
    # Returning user
    balance = int(User.getData("balance") or 0)
    bot.replyText(
        user_id,
        "👋 <b>Welcome back!</b>\n\n"
        "💰 Balance: <code>" + str(balance) + " coins</code>",
        parse_mode="HTML"
    )

# ─────────────────────────────────────────────
# STEP 3: Show Main Menu Keyboard
# ─────────────────────────────────────────────
keyboard = ReplyKeyboardMarkup()
keyboard.addButton("💰 Balance")
keyboard.addButton("🔗 Referral Link")
keyboard.newRow()
keyboard.addButton("📋 Tasks")
keyboard.addButton("💸 Withdraw")
keyboard.newRow()
keyboard.addButton("👤 My Profile")
keyboard.addButton("❓ Help")

bot.replyText(user_id, "📌 Main Menu:", reply_markup=keyboard)
```

---

## 26. Quick Reference Card

### Send a Message
```python
bot.replyText(user_id, "text")
bot.replyText(user_id, "<b>bold</b>", parse_mode="HTML")
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

### Wait for Answer Flow
1. Create command with **Answer** as prompt
2. Enable **Wait for Answer: ON**
3. In BJS: read `message` for the user's reply

### Reply Keyboard
```python
keyboard = ReplyKeyboardMarkup()
keyboard.addButton("Button 1")
keyboard.addButton("Button 2")
keyboard.newRow()
keyboard.addButton("Button 3")
bot.replyText(user_id, "Choose:", reply_markup=keyboard)
```

### Inline Keyboard
```python
keyboard = InlineKeyboardMarkup()
keyboard.addButton("Click Me", callback_data="/my_command")
keyboard.addButton("Visit", url="https://example.com")
bot.replyText(user_id, "Choose:", reply_markup=keyboard)
```

### HTTP GET Request
```python
response = HTTP.get("https://api.example.com/data")
data = response.json()
```

### HTTP POST Request
```python
response = HTTP.post("https://api.example.com/create",
    data={"key": "value"},
    headers={"Content-Type": "application/json"})
result = response.json()
```

### Chain Commands
```python
Bot.handleNextCommand("/next_command")
Bot.handleNextCommand("/next_command", keep=True)
```

### Send Media
```python
bot.sendPhoto(user_id, "https://example.com/image.jpg", caption="Caption")
bot.sendDocument(user_id, "https://example.com/file.pdf", caption="Document")
bot.sendVideo(user_id, "https://example.com/video.mp4", caption="Video")
```

### Channel Check
```python
member = bot.getChatMember("@channel", user_id)
if member.status in ["left", "kicked"]:
    bot.replyText(user_id, "Please join first!")
    return
```

### Admin Check
```python
ADMIN_IDS = ["123456789"]
if str(u) not in ADMIN_IDS:
    bot.replyText(user_id, "Access denied.")
    return
```

### Edit a Sent Message
```python
sent = bot.replyText(user_id, "Loading...")
msg_id = sent.message_id
User.saveData("msg_id", str(msg_id))
# Later:
bot.editMessage(user_id, int(User.getData("msg_id")), "Done!")
```

### Delete a Message
```python
try:
    bot.deleteMessage(user_id, int(User.getData("msg_id")))
except:
    pass
```

### Referral Link Generation
```python
ref_link = "https://t.me/YourBot?start=ref_" + str(user_id)
bot.replyText(user_id, "Your link: " + ref_link)
```

### Error Handling
```python
try:
    amount = int(message)
except:
    bot.replyText(user_id, "❌ Invalid input!")
    return
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
| **YouTube** | [youtube.com/@Sky-Pay-BD](https://youtube.com/@Sky-Pay-BD) |
| **Facebook** | [facebook.com/siyamahmedjsx](https://facebook.com/siyamahmedjsx) |

---

*This document is the official TeleBot Creator (TBC) AI Agent training guide, created by MD Jaid Bin Siyam (Siyam) to help developers and bot creators build powerful Telegram bots on the TBC platform. All content, code examples, and explanations are original. © 2026–2027 SkyPayBD. All rights reserved.*
