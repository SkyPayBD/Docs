# SkyPay Payment Gateway — Telegram Bot Integration Guide for AI Agents

## Overview

This document provides complete instructions for integrating SkyPay BD into a **Telegram Bot**. Since a Telegram bot operates entirely within Telegram's chat interface, it is impossible to redirect users to an external browser page. Therefore, all Telegram bot integrations must use the **Headless API v2**, which delivers wallet numbers as JSON data that the bot renders as chat messages.

This guide is framework-agnostic. It applies to any Telegram bot implementation:
- Python: `python-telegram-bot`, `aiogram`, `pyTelegramBotAPI`
- Node.js: `grammy`, `telegraf`, `node-telegram-bot-api`
- PHP: `longman/telegram-bot`, custom webhook handlers
- Go: `telebot`, `tgbotapi`

Always read the main SkyPay `README.md` API documentation first to understand the complete API structure before implementing.

---

## API to Use

**Headless API v2 only:**
- Initiate session: `POST https://core.skypaybd.top/api/v2/payment/create`
- Verify transaction: `POST https://core.skypaybd.top/api/v2/payment/verify`

Never use the Hosted Gateway (v1) in a Telegram bot. It generates a redirect URL, which would require the user to leave Telegram and return — breaking the bot flow and making callback handling very complex.

---

## Authentication

Every request requires:
```
BRAND-KEY: <your_brand_key>
Content-Type: application/json
Accept: application/json
```

Store the BRAND-KEY in your bot's `.env` file or environment variables. Never hardcode it in source code or commit it to a repository.

---

## Config Module

Create a dedicated config file in your bot project:

Regardless of language, define these constants/variables:
- `SKYPAY_V2_CREATE_URL` → `https://core.skypaybd.top/api/v2/payment/create`
- `SKYPAY_V2_VERIFY_URL` → `https://core.skypaybd.top/api/v2/payment/verify`
- `SKYPAY_BRAND_KEY` → read from environment variable

---

## Database Setup

Your bot needs a database to track payment sessions between the create step and the verify step. The bot cannot store session data in memory alone because restarts would lose it.

### Table: `bot_payment_sessions`

| Column | Type | Description |
|---|---|---|
| `id` | INT AUTO_INCREMENT | Primary key |
| `chat_id` | BIGINT | Telegram chat ID of the user initiating payment |
| `user_id` | BIGINT | Telegram user ID |
| `username` | VARCHAR(255), nullable | Telegram username for reference |
| `skypay_session_id` | VARCHAR(255) | The `id` returned by `/api/v2/payment/create` |
| `amount` | DECIMAL(10,2) | Payment amount in BDT |
| `product_id` | VARCHAR(255), nullable | Internal product/plan reference |
| `selected_method` | VARCHAR(50), nullable | Which channel the user chose (bkash/nagad/etc) |
| `submitted_txn_id` | VARCHAR(100), nullable | TrxID submitted by the user |
| `status` | ENUM('pending','awaiting_txn','verifying','completed','failed','expired') | Session lifecycle state |
| `retry_count` | INT DEFAULT 0 | How many verify retries have been attempted |
| `expires_at` | TIMESTAMP | Session expiry (typically 15–30 minutes after creation) |
| `created_at` | TIMESTAMP | When the session was created |
| `completed_at` | TIMESTAMP, nullable | When verification succeeded |

### Table: `fulfilled_orders`

Track fulfilled orders to prevent double fulfillment:

| Column | Type | Description |
|---|---|---|
| `id` | INT AUTO_INCREMENT | Primary key |
| `chat_id` | BIGINT | Telegram chat ID |
| `skypay_session_id` | VARCHAR(255), unique | SkyPay session ID |
| `transaction_id` | VARCHAR(100), unique | Verified TrxID |
| `amount` | DECIMAL(10,2) | Amount paid |
| `payment_method` | VARCHAR(50) | Channel used |
| `fulfilled_at` | TIMESTAMP | When the order was processed |

---

## Bot State Machine

The payment flow is multi-step. The bot must track where each user is in the flow. Use the `bot_payment_sessions.status` column as the state machine:

```
[User triggers payment]
        ↓
    status: pending
        ↓ (call /api/v2/payment/create)
    status: awaiting_txn
        ↓ (user selects channel and submits TrxID)
    status: verifying
        ↓ (call /api/v2/payment/verify)
    status: completed  ←──── or ────→  status: failed
```

Incoming messages from the user must be routed based on the current session state. When a message arrives, look up the active session for that `chat_id` and determine what action to take.

---

## Step-by-Step Bot Flow

### Step 1 — User Triggers Payment

The user sends a command or taps a button (e.g., `/pay`, `/deposit 500`, or a callback button from an inline keyboard).

The bot handler:
1. Parses the amount from the command or the button callback data
2. Looks up if there is already a non-expired, non-completed session for this `chat_id` — if yes, ask the user to complete or cancel the existing session before starting a new one
3. Sends a "Processing..." message to give the user immediate feedback

### Step 2 — Create Payment Session (call SkyPay)

The bot backend calls `POST /api/v2/payment/create`:

Request body:
```json
{
  "cus_name": "Telegram user's first name or username",
  "amount": 500,
  "meta_data": {
    "telegram_user_id": 123456789,
    "chat_id": 123456789,
    "product_id": "VIP_PLAN_01"
  }
}
```

On success:
1. Save a new row in `bot_payment_sessions`:
   - `chat_id`, `user_id`, `username`
   - `skypay_session_id` = response `id`
   - `amount` = requested amount
   - `status = 'awaiting_txn'`
   - `expires_at` = now + 20 minutes
2. Parse the `methods` array from the response
3. Filter: only keep channels where `active_payments.personal === true` (or agent/payment if applicable) AND the number is not an empty string

If the SkyPay API call fails, inform the user and do not create a session.

### Step 3 — Send Payment Instructions to User

Delete the "Processing..." message. Send a new message to the user with this structure:

```
💳 Payment Order — 500 BDT

Send the exact amount to any of the numbers below:

📱 bKash (Send Money): 01XXXXXXXX
📱 Nagad (Send Money): 01XXXXXXXX
📱 Rocket (Send Money): 01XXXXXXXX

⚠️ Rules:
• Send EXACTLY 500 BDT — no more, no less
• After sending, copy the Transaction ID (TrxID) from the SMS you receive
• Reply here with your TrxID to confirm payment

Session expires in 20 minutes.
```

Add an inline keyboard with:
- One button per available payment channel (e.g., "I paid via bKash", "I paid via Nagad")
- A "Cancel" button

The user must tap a channel button first (so the bot knows the `method`), and then the bot asks them to enter their TrxID.

Alternatively, ask the user to type the channel name along with the TrxID in a single message like: `bkash BLA38KDK2M`

### Step 4 — Receive Channel Selection and TrxID

#### Option A (Inline buttons for channel, then text for TrxID):
1. User taps "I paid via bKash" → bot saves `selected_method = 'bkash'` to the session row, sends "Please enter your bKash TrxID:"
2. User sends the TrxID as a plain text message
3. Bot reads the next incoming text message from this `chat_id` while session status is `awaiting_txn`, treats it as the TrxID

#### Option B (Single message format):
User sends: `bkash BLA38KDK2M`
Bot parses: method = first word, txnId = second word

Always call `.toLowerCase()` (or equivalent) on the method name before storing or sending it to SkyPay.

Validate the TrxID is not empty and not obviously malformed before calling the API.

### Step 5 — Verify the Transaction (call SkyPay)

Update session status to `'verifying'`. Send a "Verifying your payment, please wait..." message.

Call `POST /api/v2/payment/verify`:
```json
{
  "id": "<skypay_session_id>",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

**On success (`status: true`):**
1. Check `fulfilled_orders` table: if this `skypay_session_id` or `transaction_id` already exists, this is a duplicate — send "Payment already processed" and stop
2. Update `bot_payment_sessions`: `status = 'completed'`, `submitted_txn_id`, `selected_method`, `completed_at`
3. Insert a row into `fulfilled_orders`
4. Fulfill the order (add balance, activate subscription, send digital goods, etc.)
5. Send a success message to the user:
```
✅ Payment Confirmed!

Amount: 500 BDT
Channel: bKash
TrxID: BLA38KDK2M

Your [subscription/balance/access] has been activated. Enjoy!
```

**On failure (`status: false`):**

Check the `message` field from the API response:

- **SMS not yet received / TrxID not found:** This is a temporary state. Increment `retry_count` on the session row. Send:
```
⏳ Payment matching in progress...

Your SMS may not have arrived yet. Please wait 10 seconds and tap Retry.
```
With an inline "🔄 Retry" button. When the user taps Retry, repeat the verify call.

- **Session expired:** Tell the user to start over with `/pay`.

- **TrxID already used:** Tell the user this TrxID belongs to another order.

- **Wrong amount:** Tell the user the sent amount doesn't match. Ask them to contact support.

- **Retry limit exceeded (after 5–6 attempts):** Update session status to `'failed'`. Tell the user to contact support with their TrxID.

---

## Retry Handling

SkyPay's SMS bridge takes 5–20 seconds. Build a retry mechanism:

1. After the first failed verify, show a "Retry in 10 seconds" button (or auto-retry via a scheduled task/webhook timeout)
2. Track `retry_count` in the session table
3. Maximum retries: 5 attempts
4. Total retry window: 3 minutes from first attempt
5. After 3 minutes or 5 attempts, mark the session as failed

Do not retry if the error indicates the TrxID is already claimed, the session is expired, or the amount is wrong — those are permanent errors.

---

## Session Expiry Handling

Sessions should expire automatically. Either:
- Run a periodic task (cron job or background worker) that marks sessions as `'expired'` where `expires_at < NOW()` and `status = 'awaiting_txn'`
- Or check expiry at the start of each handler: if `expires_at < NOW()`, update to `'expired'` and inform the user

---

## Concurrent Session Handling

A user should only have one active payment session at a time. At the start of the payment trigger, check if there is already an `'awaiting_txn'` or `'verifying'` session for this `chat_id`. If yes:
- Offer the user options: "You have an incomplete payment. Continue it or cancel it before starting a new one."
- Show inline buttons: "Continue existing payment" or "Cancel and start new"

---

## Anti-Fraud and Idempotency Rules

- Check `fulfilled_orders` before processing any fulfillment
- SkyPay blocks TrxID reuse on its side, but the bot must also guard against it
- A `transaction_id` must be unique in your `fulfilled_orders` table (use a unique constraint)
- Log all API calls to a server-side log file with timestamp, chat_id, session ID, and full API response

---

## Method Name Rule

Always normalize the method to lowercase before sending to SkyPay. Regardless of how the user typed it ("bKash", "BKASH", "Bkash"), convert it:
- Python: `method.lower()`
- Node.js: `method.toLowerCase()`
- PHP: `strtolower($method)`
- Go: `strings.ToLower(method)`

Accepted values: `bkash`, `nagad`, `rocket`, `upay`

---

## Complete Flow Summary

```
User: /pay 500
    → Bot creates SkyPay session → saves to DB
    → Bot sends wallet numbers in chat

User: taps "I paid via bKash" → types TrxID "BLA38KDK2M"
    → Bot calls POST /api/v2/payment/verify
    → status: true → fulfill order → send success message
    → status: false (pending) → show Retry button
    → status: false (permanent error) → send error message
```
