# SkyPay - BD

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png" width="100" alt="SkyPay Logo" />

<br/>

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789098593_de7d9d238ad2e0178762.png" width="420" alt="SkyPay Banner" />

<br/><br/>

**Zero-Redirect Automated Payment Infrastructure for Telegram Bots — Bangladesh**

*Accept automated payments via **bKash**, **Nagad**, **Rocket**, **Upay**, and **Binance Pay** — directly inside your Telegram bot chat, without redirecting the user anywhere.*

<br/>

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/Headless%20API-v2.0%20Live-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
[![GitHub](https://img.shields.io/badge/GitHub-SkyPayBD-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/SkyPayBD)

</div>

---

> 📖 **Scope of this document:** This README covers **only the Headless Payment API v2**, and **only** how to integrate it into a **Telegram Bot**. It does not cover the v1 Hosted Checkout Gateway, WooCommerce/WHMCS plugins, or website redirect flows — those are documented separately. If your bot needs payments without ever leaving the Telegram chat window, this is the correct document.

---

## 💳 Supported Payment Methods

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1720001734_bed271b1089aa12b9887.png" height="36" alt="bKash" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717231942_d63fe41b5e42176d4936.png" height="36" alt="Nagad" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717239532_10348cc78dc0b990a8e5.png" height="36" alt="Rocket" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717239551_f0b3a097df92d481e17f.png" height="36" alt="Upay" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717513584_d7ff0294bf6b6c98db7b.png" height="36" alt="Binance" />

</div>

<br/>

| Channel | `method` Value | Personal (Send Money) | Agent (Cash In) | Merchant Pay | Verification Method |
|---|---|:---:|:---:|:---:|---|
| **bKash** | `bkash` | ✅ | ✅ | ✅ | Android SMS Sync |
| **Nagad** | `nagad` | ✅ | ✅ | 🔄 Under Review | Android SMS Sync |
| **Rocket** | `rocket` | ✅ | ✅ | 🔄 Under Review | Android SMS Sync |
| **Upay** | `upay` | ✅ | ❌ | 🔄 Under Review | Android SMS Sync |
| **Binance Pay** | `binance` | ✅ (USDT) | — | — | Automatic (Server-Side) |

> **Note:** `binance` is verified **automatically on SkyPay's server** — it does **not** use the Android SMS bridge at all. The customer only needs to provide their **Binance Order ID**. No merchant Android device is required for Binance transactions; the device is only required for bKash, Nagad, Rocket, and Upay.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#️-prerequisites)
- [Authentication](#-authentication)
- [API Endpoint Directory](#-api-endpoint-directory)
- [Integration Flow (Overview)](#-integration-flow-overview)
- [Telegram Bot Architecture](#-telegram-bot-architecture)
  - [Database Design](#database-design)
  - [Bot State Machine](#bot-state-machine)
  - [Step 1 — User Triggers Payment](#step-1--user-triggers-payment)
  - [Step 2 — Create Payment Session](#step-2--create-payment-session)
  - [Step 3 — Present Wallet Numbers to the User](#step-3--present-wallet-numbers-to-the-user)
  - [Step 4 — Receive Method + Transaction ID](#step-4--receive-method--transaction-id)
  - [Step 5 — Verify the Transaction](#step-5--verify-the-transaction)
- [Binance Pay — How It Works](#-binance-pay--how-it-works)
- [SMS Sync Latency & Retry Handling](#️-sms-sync-latency--retry-handling)
- [Session Expiry Handling](#-session-expiry-handling)
- [Concurrent Session Handling](#-concurrent-session-handling)
- [Anti-Fraud & Idempotency Rules](#-anti-fraud--idempotency-rules)
- [Method Name Normalization](#-method-name-normalization)
- [HTTP Status Codes & Error Reference](#-http-status-codes--error-reference)
- [Android Merchant Sync App Setup](#-android-merchant-sync-app-setup)
- [Production Security Checklist](#-production-security-checklist)
- [Contact & Support](#-contact--support)
- [Quick Links](#-quick-links)

---

## 🌐 Overview

**SkyPay BD Headless API v2** is a **zero-redirect** payment system built for Bangladesh's MFS ecosystem, purpose-fit for **Telegram bots**. Since a Telegram bot lives entirely inside the chat window, it is never acceptable to redirect the user to an external browser page — the Headless API solves this by returning wallet numbers as plain JSON, which your bot renders directly as chat messages.

The complete flow, at a glance:

1. Your bot backend calls `POST /api/v2/payment/create` and receives a session `id` plus the merchant's active wallet numbers.
2. Your bot displays those wallet numbers as a normal chat message (or inline keyboard).
3. The customer pays via their MFS app (or Binance app) and receives a Transaction ID / Order ID.
4. The customer sends that ID back to the bot.
5. Your bot backend calls `POST /api/v2/payment/verify`.
6. On `"status": true`, your bot fulfills the order — instantly, without the user ever leaving Telegram.

This guide is **framework-agnostic** and applies to any Telegram bot stack:

| Language | Common Frameworks |
|---|---|
| Python | `python-telegram-bot`, `aiogram`, `pyTelegramBotAPI` |
| Node.js | `grammy`, `telegraf`, `node-telegram-bot-api` |
| PHP | `longman/telegram-bot`, custom webhook handlers |
| Go | `telebot`, `tgbotapi` |

> ⚠️ **Never use the v1 Hosted Gateway inside a Telegram bot.** It generates a redirect URL that forces the user to leave the chat and open an external browser — this breaks the native bot experience and turns callback handling into an unnecessary complication. Telegram bots must always use Headless API v2.

---

## ⚙️ Prerequisites

Two things must be ready before your bot makes its first API call:

### 1. BRAND-KEY

- Log in to the **[SkyPay Merchant Dashboard](https://skypaybd.top/user/brands)**
- Create a Brand (or open an existing one) and copy the generated **BRAND-KEY**
- Store it in your bot's server-side `.env` file or secret manager
- **Never** hardcode it in source files, commit it to a public repository, or expose it in client-facing bot code

### 2. Merchant Android Device — Required for bKash / Nagad / Rocket / Upay only

- Install the **SkyPay Merchant Sync APK** on a dedicated Android phone
- The phone must carry your active merchant SIM cards (bKash / Nagad / Rocket / Upay)
- Grant **SMS Listener** and **Notification Access** permissions
- Disable **Battery Optimization** for the app
- Keep the device powered on and connected to the internet 24/7

> ⚠️ Without an active, connected Android device, any `/api/v2/payment/create` call for MFS channels will return `403 Forbidden: No active SMS sync device found for this account.` **This requirement does not apply to Binance Pay** — Binance is configured once in the Dashboard and verified automatically on SkyPay's server.

---

## 🔐 Authentication

Every Headless API v2 request requires exactly **one** header:

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
Accept: application/json
```

> **Important — v2 accepts `BRAND-KEY` only.** Unlike the v1 Hosted Gateway (which also accepts `API-KEY`, `SECRET-KEY`, and a `?api_key=` query parameter as aliases), the **Headless API v2 endpoints do not read any header other than `BRAND-KEY`**. Sending `API-KEY` or `SECRET-KEY` to a v2 endpoint will simply be ignored and the request will fail authentication. Always use `BRAND-KEY` for every v2 call.

---

## 📡 API Endpoint Directory

| Action | Method | Full Endpoint URL |
|---|:---:|---|
| Create Payment Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| Verify Transaction | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

Both endpoints live under the single unified core domain:

```
https://core.skypaybd.top
```

---

## 🔄 Integration Flow (Overview)

```
Your Bot Backend
  → POST /api/v2/payment/create
  → Receive { id, brand, methods[] }
  → Save the session id (database, Redis, or bot context)
  → Render wallet numbers as a chat message inside Telegram
  → Customer pays via MFS app / Binance app
  → Customer sends back: method + TrxID (or Order ID for Binance)
  → Your Bot Backend → POST /api/v2/payment/verify
  → status: true → Fulfill order (balance, subscription, digital goods, etc.)
  → status: false → Retry (MFS) or show the exact error to the customer
```

---

## 🤖 Telegram Bot Architecture

The sections below describe the recommended way to structure a Telegram bot's payment logic around the Headless API v2. This part is **implementation guidance for your bot**, not part of the SkyPay API contract itself — you are free to adapt the schema and flow to your own stack, as long as the actual API calls (Steps 2 and 5) match the specification exactly.

### Database Design

Your bot needs persistent storage to track a payment session between the `/create` call and the `/verify` call. Do not rely on in-memory state alone — a bot restart would lose every pending payment.

#### Table: `bot_payment_sessions`

| Column | Type | Description |
|---|---|---|
| `id` | INT AUTO_INCREMENT | Primary key |
| `chat_id` | BIGINT | Telegram chat ID of the user initiating payment |
| `user_id` | BIGINT | Telegram user ID |
| `username` | VARCHAR(255), nullable | Telegram username, for reference only |
| `skypay_session_id` | VARCHAR(255) | The `id` returned by `/api/v2/payment/create` |
| `amount` | DECIMAL(10,2) | Payment amount in BDT |
| `product_id` | VARCHAR(255), nullable | Internal product / plan reference |
| `selected_method` | VARCHAR(50), nullable | `bkash`, `nagad`, `rocket`, `upay`, or `binance` |
| `submitted_txn_id` | VARCHAR(100), nullable | TrxID / Order ID submitted by the user |
| `status` | ENUM(`pending`,`awaiting_txn`,`verifying`,`completed`,`failed`,`expired`) | Session lifecycle state |
| `retry_count` | INT DEFAULT 0 | Number of verify retries attempted |
| `expires_at` | TIMESTAMP | Recommended: 15–30 minutes after creation |
| `created_at` | TIMESTAMP | Session creation time |
| `completed_at` | TIMESTAMP, nullable | Time verification succeeded |

#### Table: `fulfilled_orders`

Used to guard against double fulfillment, independently of SkyPay's own idempotency checks.

| Column | Type | Description |
|---|---|---|
| `id` | INT AUTO_INCREMENT | Primary key |
| `chat_id` | BIGINT | Telegram chat ID |
| `skypay_session_id` | VARCHAR(255), unique | SkyPay session ID |
| `transaction_id` | VARCHAR(100), unique | Verified TrxID / Order ID |
| `amount` | DECIMAL(10,2) | Amount paid |
| `payment_method` | VARCHAR(50) | Channel used |
| `fulfilled_at` | TIMESTAMP | When the order was processed |

### Bot State Machine

Drive the payment flow with the `bot_payment_sessions.status` column:

```
[User triggers payment]
        │
        ▼
   status: pending
        │  (call /api/v2/payment/create)
        ▼
   status: awaiting_txn
        │  (user selects channel and submits TrxID / Order ID)
        ▼
   status: verifying
        │  (call /api/v2/payment/verify)
        ▼
 status: completed  ◄────  or  ────►  status: failed
```

Every incoming message from the user should be routed based on the current session state for that `chat_id`.

### Step 1 — User Triggers Payment

The user sends a command or taps a button (`/pay`, `/deposit 500`, or an inline-keyboard callback).

1. Parse the amount from the command or callback data.
2. Check for an existing non-expired, non-completed session for this `chat_id`. If one exists, ask the user to finish or cancel it before starting a new one.
3. Send a "Processing…" message for immediate feedback.

### Step 2 — Create Payment Session

Call `POST https://core.skypaybd.top/api/v2/payment/create`.

**Request Headers**

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

**Request Body Parameters**

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `cus_name` | String | ✅ | Full name or Telegram username of the customer. |
| `amount` | Numeric | ✅ | Total payable amount in BDT. Must be a positive number greater than `0`. |
| `meta_data` | Object / JSON | ❌ | Any custom JSON object — e.g. Telegram user ID, chat ID, plan/product reference. Returned as-is in the verify response. Must be a valid JSON object, not a plain string. |

**Example Request Body**

```json
{
  "cus_name": "Siyam Ahmed",
  "amount": 500,
  "meta_data": {
    "telegram_user_id": 123456789,
    "chat_id": 123456789,
    "product_id": "VIP_PLAN_01"
  }
}
```

**Success Response (HTTP 200)**

```json
{
  "status": true,
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "brand": {
    "name": "Your Brand Name",
    "mobile": "017XXXXXXXX",
    "whatsapp": "017XXXXXXXX",
    "email": "support@yourdomain.com"
  },
  "methods": [
    {
      "name": "bkash",
      "active_payments": { "personal": true, "agent": false, "payment": true },
      "personal": "01XXXXXXXX",
      "agent": "",
      "payment": "01XXXXXXXX"
    },
    {
      "name": "nagad",
      "active_payments": { "personal": true, "agent": false },
      "personal": "01XXXXXXXX",
      "agent": ""
    },
    {
      "name": "binance",
      "active_payments": { "personal": true, "merchant": false },
      "personal": "XXXXXXXXXXXX",
      "currency": "USDT",
      "dollar_rate": "120.00",
      "amount_usdt": "4.1667"
    }
  ]
}
```

**Response Fields Explained**

| Field | Description |
|---|---|
| `id` | Unique payment session ID. **Save this immediately** — required for `/verify`. If lost, you must create a new session. |
| `brand.name` / `brand.mobile` / `brand.whatsapp` / `brand.email` | Your merchant brand's display details, for showing support info to the customer. |
| `methods[]` | Only the channels **configured and active** on your SkyPay dashboard are returned here. |
| `methods[].name` | Channel identifier: `bkash`, `nagad`, `rocket`, `upay`, or `binance`. |
| `methods[].active_payments.personal` | If `true`, `personal` is live for **Send Money**. |
| `methods[].active_payments.agent` | If `true`, `agent` is live for **Cash In**. |
| `methods[].active_payments.payment` | *(bKash only)* If `true`, `payment` is live for **Merchant Pay**. |
| `methods[].active_payments.merchant` | *(Binance only)* Reserved flag, currently always `false`. |
| `methods[].personal` / `agent` / `payment` | The actual wallet numbers. Empty string `""` means that sub-type is not active. |
| `methods[].personal` *(Binance)* | The Binance **receiving UID** — show this to the customer. |
| `methods[].currency` *(Binance)* | Always `"USDT"`. Only USDT is accepted. |
| `methods[].dollar_rate` *(Binance)* | The BDT → USDT conversion rate configured by the merchant. |
| `methods[].amount_usdt` *(Binance)* | The **exact** USDT amount the customer must send (= `amount ÷ dollar_rate`, rounded to 4 decimal places). Display this value directly — do not recalculate it yourself. |

> **🚨 Display Rule:** Never show a channel that is missing from `methods[]`. Never show a wallet number that is an empty string `""` or whose `active_payments` flag is `false`. A channel absent from `methods[]` is not configured on your brand and cannot be verified.

On a successful call, immediately save a row to `bot_payment_sessions`:
- `chat_id`, `user_id`, `username`
- `skypay_session_id` = response `id`
- `amount` = requested amount
- `status = 'awaiting_txn'`
- `expires_at` = now + 20 minutes (recommended)

**Create — Error Responses**

| HTTP Code | Message | Cause |
|---|---|---|
| `401` | `BRAND-KEY header is required.` | The `BRAND-KEY` header was not sent or is empty. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | The key does not exist, or the brand is deactivated. |
| `403` | `Associated merchant account is inactive.` | The merchant account linked to this brand is suspended. |
| `403` | `No active SMS sync device found for this account.` | No Android device is connected/active for MFS channels. |
| `400` | `Valid cus_name and numeric amount are required.` | `cus_name` is empty, or `amount` is missing / non-numeric / zero or negative. |
| `400` | `meta_data must be a valid JSON object.` | `meta_data` was sent as a plain string instead of a JSON object. |
| `400` | `No active payment gateways configured for this brand.` | Your brand has no channels configured in the Dashboard. |
| `405` | `Method not allowed. Only POST requests are accepted.` | A GET or other HTTP verb was used instead of POST. |

If the call fails, inform the user and **do not** create a session row.

### Step 3 — Present Wallet Numbers to the User

Delete the "Processing…" message and send the instructions as a normal chat message:

```
💳 Payment Order — 500 BDT

Send the exact amount to any of the numbers below:

📱 bKash (Send Money): 01XXXXXXXX
📱 Nagad (Send Money): 01XXXXXXXX
📱 Rocket (Send Money): 01XXXXXXXX
🟡 Binance Pay (USDT): UID XXXXXXXXXXXX
   → Send exactly: 4.1667 USDT

⚠️ Rules:
• Send EXACTLY 500 BDT (or the exact USDT amount for Binance) — no more, no less
• After paying, copy the Transaction ID (TrxID) or Order ID from your confirmation
• Reply here with your method + ID to confirm payment

Session expires in 20 minutes.
```

Add an inline keyboard with one button per available channel (e.g. "I paid via bKash", "I paid via Binance") plus a "Cancel" button. Only render buttons for channels that passed the Display Rule above.

### Step 4 — Receive Method + Transaction ID

**Option A — Inline button for channel, then text for ID:**
1. User taps "I paid via bKash" → bot saves `selected_method = 'bkash'`, replies "Please enter your bKash TrxID:"
2. User sends the ID as a plain text message.
3. The bot reads the next text message from this `chat_id` while `status = 'awaiting_txn'` and treats it as the ID.

**Option B — Single-message format:**
User sends: `bkash BLA38KDK2M` (or `binance 443903031407804416`)
Bot parses: `method` = first word, `transaction_id` = second word.

Always lowercase the `method` value before storing or sending it to SkyPay (see [Method Name Normalization](#-method-name-normalization)). Validate that the ID is not empty before calling `/verify`.

### Step 5 — Verify the Transaction

Update the session `status = 'verifying'` and send "Verifying your payment, please wait…".

Call `POST https://core.skypaybd.top/api/v2/payment/verify`.

**Request Headers**

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

**Request Body Parameters**

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `id` | String | ✅ | The session ID returned by `/create` in Step 2. |
| `method` | String | ✅ | Strict lowercase: `bkash`, `nagad`, `rocket`, `upay`, or `binance`. |
| `transaction_id` | String | ✅ | The TrxID (MFS) or Order ID (Binance) submitted by the customer. |

> **Field aliases:** The verify endpoint also accepts `order_id`, `order`, `trx_id`, `transactionId`, and `transaction` as interchangeable alternate names for the identifier — all resolve to the same value as `transaction_id`.
>
> | Method | Recommended Field | Notes |
> |---|---|---|
> | `bkash` / `nagad` / `rocket` / `upay` | `transaction_id` | Standard field name for MFS TrxIDs. |
> | `binance` | `order_id` | **Recommended for Binance**, since Binance Pay natively returns an "Order ID." `transaction_id` is also fully accepted and works exactly the same way — `order_id` is simply the clearer, recommended field name for this channel. |

> **🚨 `method` Field Rules — Non-Negotiable:**
> - Must be **exactly one** of: `bkash` · `nagad` · `rocket` · `upay` · `binance`
> - Must be **strict lowercase** — `"bKash"`, `"BKASH"`, `"Binance"` will all fail with `400 Unsupported payment method supplied`
> - Must match the channel the user actually paid through
> - Must be a channel that exists in the `methods[]` array returned by `/create`

**Example — bKash**

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

**Example — Binance**

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "binance",
  "order_id": "443903031407804416"
}
```

**Success Response (HTTP 200)**

```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Siyam Ahmed",
    "cus_email": "headless@skypaybd.top",
    "amount": 500,
    "transaction_id": "BLA38KDK2M",
    "meta_data": {
      "telegram_user_id": 123456789,
      "product_id": "VIP_PLAN_01"
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

> **Note on the response shape:** For **all** channels (MFS and Binance alike), the verify response wraps its payload inside a `data` object and includes a top-level `message` string. `data.cus_email` is always the fixed placeholder `"headless@skypaybd.top"` for Headless v2 sessions — this is expected and not an error. `data.status` is always `"COMPLETED"` on a successful verify call.

**On success, in your bot:**
1. Check `fulfilled_orders` for this `skypay_session_id` / `transaction_id`. If it already exists, this is a duplicate — reply "Payment already processed" and stop.
2. Update `bot_payment_sessions`: `status = 'completed'`, `submitted_txn_id`, `selected_method`, `completed_at`.
3. Insert a row into `fulfilled_orders`.
4. Fulfill the order (credit balance, activate subscription, deliver digital goods, etc.).
5. Send a confirmation:

```
✅ Payment Confirmed!

Amount: 500 BDT
Channel: bKash
TrxID: BLA38KDK2M

Your subscription has been activated. Enjoy!
```

**Verify — Error Responses**

| HTTP Code | Message | Cause & Fix |
|---|---|---|
| `400` | `id, method and transaction_id (or order_id) fields are required.` | A required field is missing from the request body. |
| `400` | `Unsupported payment method supplied.` | `method` is not one of the five valid values, or is not lowercase. |
| `400` | `This payment session has already been completed.` | The session was already verified in a previous call. Do not re-verify. |
| `400` | `Transaction not found. Please check Order ID.` | The TrxID/Order ID does not match any incoming SMS or Binance transaction yet. Wait and retry for MFS. |
| `400` | `Receiver UID does not match.` | *(Binance only)* Payment was sent to a different Binance UID than configured. |
| `400` | `Only USDT payments are accepted.` | *(Binance only)* Customer sent a currency other than USDT. |
| `400` | `Insufficient amount received. Expected X USDT but received Y USDT.` | *(Binance only)* Customer sent less than required (tolerance ±0.0005 USDT). |
| `400` | `This Order ID has already been used.` | *(Binance only)* Order ID was already committed to a previous session. |
| `401` | `BRAND-KEY header is required.` | Missing authentication header. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | Check your key in Brand Management. |
| `403` | `Associated merchant account is inactive.` | The merchant account is suspended — contact support. |
| `403` | `No active SMS sync device found for this account.` | *(MFS only)* Android device is offline. Restart the SkyPay APK. |
| `404` | `Payment session not found or expired.` | Session `id` does not exist, or belongs to another brand — call `/create` again. |
| `502` | `Failed to communicate with Binance. Please try again.` | *(Binance only)* Binance service temporarily unreachable — retry shortly. |

---

## 🪙 Binance Pay — How It Works

Binance Pay verification works fundamentally differently from bKash / Nagad / Rocket / Upay, and it is important your bot handles it as a separate branch of logic rather than reusing the MFS retry loop.

| | MFS (bKash / Nagad / Rocket / Upay) | Binance Pay |
|---|---|---|
| **What the customer provides** | Transaction ID (TrxID) from SMS | Order ID from the Binance app |
| **How it's matched** | Incoming SMS on the merchant Android phone | Automatically, server-side, against Binance's own records |
| **Android device required?** | ✅ Yes | ❌ No |
| **Verification speed** | 5–20 seconds (SMS bridge latency) | Instant — no waiting |
| **Currency** | BDT | USDT only |
| **Amount check** | Exact BDT match required | `amount_usdt` match, tolerance ±0.0005 USDT |
| **Receiving address shown to customer** | Wallet phone number | Binance receiving UID |

**How the flow works, in simple terms:**

1. At `/create`, if `binance` is a configured channel on your brand, the `methods[]` array includes a `binance` entry with a receiving `personal` UID, the `currency` (always `"USDT"`), the `dollar_rate`, and the exact `amount_usdt` the customer must send.
2. Show the customer that exact `amount_usdt` value and the receiving UID — never let them calculate it themselves, and never round it.
3. The customer opens their Binance app, sends the specified USDT to that UID, and copies their **Order ID**.
4. The customer sends that Order ID back to your bot.
5. Your bot calls `/verify` with `method: "binance"` and the Order ID as `transaction_id`.
6. SkyPay checks the payment against Binance's own transaction records automatically — there is **no SMS bridge involved**, so there is no 5–20 second wait and no need for a retry-button UX (though you should still handle a `502` from Binance's side gracefully, as that indicates a temporary Binance-side outage, not a missing payment).
7. On `"status": true`, `data.payment_method` will be `"binance"` and `data.status` will be `"COMPLETED"` — fulfill exactly as you would for any other channel.

> No Binance credentials are required in your bot code. The merchant's Binance receiving configuration is set up once in the SkyPay Dashboard — your bot only ever talks to the standard `/api/v2/payment/create` and `/api/v2/payment/verify` endpoints, exactly as it does for MFS.

---

## ⏱️ SMS Sync Latency & Retry Handling

This section applies **only to MFS channels** (bKash, Nagad, Rocket, Upay). Binance is instant and does not need this retry logic.

When a customer pays via bKash, Nagad, Rocket, or Upay, the telecom operator sends an SMS to your merchant Android phone. The SkyPay Sync App reads it and relays it to the cloud over an encrypted connection. This round-trip takes **5 to 20 seconds**.

**Recommended retry flow:**

```
User submits method + TrxID
          │
          ▼
POST /api/v2/payment/verify
          │
    ┌─────┴─────┐
  success     error ("Transaction not found")
    │               │
    ▼               ▼
Fulfill         Show user:
order           "Payment matching in progress.
                 Please wait 10 seconds and tap Retry."
                      │
                      ▼
                User taps 🔄 Retry
                      │
                      ▼
                POST /api/v2/payment/verify (again)
```

- Track `retry_count` on the session row.
- Maximum retries: **5 attempts**.
- Total retry window: **3 minutes** from the first attempt.
- After the limit is reached, mark the session `status = 'failed'` and direct the user to contact support with their TrxID.
- **Do not retry** on permanent errors — TrxID already claimed, session expired, or wrong amount. These will never succeed on retry.

---

## 🕐 Session Expiry Handling

Sessions should not stay open forever. Implement one of the following:

- A periodic background job that marks sessions `'expired'` where `expires_at < NOW()` and `status = 'awaiting_txn'`.
- Or check expiry inline at the start of each handler: if `expires_at < NOW()`, update to `'expired'` and inform the user before proceeding.

If a session has expired, the user must call `/pay` again to generate a fresh session `id` — an expired `id` cannot be reused against `/verify`.

---

## 🔁 Concurrent Session Handling

A single user should only ever have one active payment session. At the start of every payment trigger, check whether an `'awaiting_txn'` or `'verifying'` session already exists for that `chat_id`. If so, offer:

- **"Continue existing payment"** — resume the existing session.
- **"Cancel and start new"** — mark the old one `'failed'`/cancelled and create a fresh session.

---

## 🛡️ Anti-Fraud & Idempotency Rules

- Check `fulfilled_orders` **before** any fulfillment logic runs.
- SkyPay itself blocks re-use of a TrxID / Order ID once claimed, but your bot must enforce its own guard as a second layer of defense.
- Enforce a **unique constraint** on `transaction_id` in `fulfilled_orders`.
- Log every API call (request + full response) with a timestamp, `chat_id`, and session ID for audit and support purposes.

---

## 🔡 Method Name Normalization

Always convert the `method` value to lowercase **before** sending it to SkyPay, regardless of how the user typed it:

| Language | Normalization |
|---|---|
| Python | `method.lower()` |
| Node.js | `method.toLowerCase()` |
| PHP | `strtolower($method)` |
| Go | `strings.ToLower(method)` |

**Accepted values only:** `bkash`, `nagad`, `rocket`, `upay`, `binance`. Anything else (including correctly-cased variants like `"Bkash"`) will be rejected with `400 Unsupported payment method supplied`.

---

## 🚦 HTTP Status Codes & Error Reference

These are the HTTP status codes actually returned by the **Headless API v2** endpoints:

| HTTP Code | Meaning | What To Do |
|---|---|---|
| `200 OK` | Request processed successfully | Check `"status": true` inside the JSON body |
| `400 Bad Request` | Missing parameter, invalid method, or already-used session/TrxID | Read the `"message"` field for exact detail |
| `401 Unauthorized` | Missing or invalid `BRAND-KEY` | Verify your key in Brand Management |
| `403 Forbidden` | Merchant account inactive, or no active Android device (MFS only) | Check account status and the SkyPay APK on the merchant phone |
| `404 Not Found` | Session expired or does not exist | Call `/create` again for a fresh session |
| `405 Method Not Allowed` | Wrong HTTP verb used (e.g. GET instead of POST) | Always use `POST` on both v2 endpoints |
| `500 Internal Server Error` | Temporary cloud-side error | Retry shortly; contact support if persistent |
| `502 Bad Gateway` | *(Binance only)* Binance service temporarily unreachable | Retry after a short delay |

> For the full field-level error messages (create and verify), see the tables in [Step 2](#step-2--create-payment-session) and [Step 5](#step-5--verify-the-transaction) above.

---

## 📱 Android Merchant Sync App Setup

*(Required for bKash, Nagad, Rocket, Upay — not required for Binance Pay)*

<div align="center">

[![Download SkyPay APK](https://img.shields.io/badge/⬇%20Download%20SkyPay%20Merchant%20App-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://skypaybd.top/public/assets/downloads/SkyPay.apk)

</div>

### Requirements
- Android phone running Android 7.0 (Nougat) or higher
- Active merchant SIM cards (bKash / Nagad / Rocket / Upay) inserted in the phone
- Phone kept plugged into power 24/7
- Battery optimization **disabled** for the SkyPay app
- Uninterrupted WiFi or mobile data

### Setup Steps

**Step 1 — Download & Install**
Download `SkyPay.apk` from the link above. Since it is sideloaded (not from Google Play), enable **"Install from Unknown Sources"** in your device settings if prompted.

**Step 2 — Create a Device in the Dashboard**
1. Log in to [Device Management](https://skypaybd.top/user/devices)
2. Ensure you have an active subscription ([Subscription Plans](https://skypaybd.top/user/plans))
3. Click **Create Device**, give it a name, and save
4. Copy the **Device Key** shown

**Step 3 — Log In to the App**

| Field | What to Enter |
|---|---|
| **Email** | Your registered email on skypaybd.top |
| **Device Key** | The Device Key copied from Device Management |

Enable **"Remember My Device"** before tapping Login, so the device stays logged in across restarts.

> ⚠️ **IP Lock:** Each Device Key is locked to the IP address of the first device that logs in with it. If you log in from a different phone, or your IP changes and access is blocked, **delete the device** from [Device Management](https://skypaybd.top/user/devices) and create a new one.

**Step 4 — Grant SMS Permissions**
A yellow warning banner will appear asking for SMS permission. Tap **Grant** and allow all requested permissions.

If permissions don't take effect: long-press the app icon → **App Info** → **Permissions** → manually enable **SMS**.

**Step 5 — Keep the Service Running**
A toggle in the app pauses/resumes the sync service. Keep it **ON** at all times — if paused, incoming SMS will not be forwarded and MFS payments cannot be verified.

---

## ✅ Production Security Checklist

- [ ] `BRAND-KEY` is stored on the server in `.env` — never exposed in bot source code, logs, or repositories
- [ ] Only `POST /api/v2/payment/create` and `POST /api/v2/payment/verify` are ever called — no v1 hosted endpoints
- [ ] Merchant Android phone is powered on, connected, and the SkyPay APK service is running (for MFS)
- [ ] Battery optimization is disabled for the SkyPay APK
- [ ] Session `id` is saved to your database **before** presenting wallet numbers to the user
- [ ] Only channels present in `methods[]`, with a non-empty active number, are ever displayed
- [ ] The exact session `amount` (or `amount_usdt` for Binance) is shown to the user, unmodified
- [ ] The `method` field is always strict lowercase in every `/verify` call
- [ ] A 10–20 second retry window exists for MFS SMS sync latency
- [ ] Binance verification is treated as instant — no unnecessary retry delay is added
- [ ] Fulfillment only ever happens after `"status": true` is received from `/verify`
- [ ] `fulfilled_orders` enforces a unique constraint on `transaction_id` to block double fulfillment
- [ ] `meta_data` is always sent as a valid JSON object, never a plain string

---

## 📞 Contact & Support

<div align="center">

| Channel | Link |
|---|---|
| 🌐 **Website** | [skypaybd.top](https://skypaybd.top) |
| 📧 **Email** | [support@skypaybd.top](mailto:support@skypaybd.top) |
| 📞 **Phone / Call** | [+880 9696 014968](tel:+8809696014968) |
| 💬 **WhatsApp** | [+880 1761 844968](https://wa.me/8801761844968) |
| ✈️ **Telegram** | [@BD_Prime_Minister](https://t.me/BD_Prime_Minister) |
| 📘 **Facebook** | [facebook.com/hyper.10.squad](https://www.facebook.com/hyper.10.squad) |
| 🐙 **GitHub** | [github.com/SkyPayBD](https://github.com/SkyPayBD) |
| 📺 **YouTube** | [youtube.com/@Sky-Pay-BD](https://youtube.com/@Sky-Pay-BD) |

🕐 **Operating Hours:** Saturday – Thursday, 09:00 AM – 10:00 PM (BST)

📍 **Location:** Panchagarh, Rangpur, Dhaka, Bangladesh

</div>

---

## 🔗 Quick Links

| Resource | URL |
|---|---|
| 🔑 Login | [skypaybd.top/sign-in](https://skypaybd.top/sign-in) |
| 📝 Register | [skypaybd.top/sign-up](https://skypaybd.top/sign-up) |
| 🔒 Forgot Password | [skypaybd.top/password-reset](https://skypaybd.top/password-reset) |
| 🏷️ Brand Management | [skypaybd.top/user/brands](https://skypaybd.top/user/brands) |
| 💳 Wallet Management | [skypaybd.top/user/user-settings/wallets](https://skypaybd.top/user/user-settings/wallets) |
| 📱 Device Management | [skypaybd.top/user/devices](https://skypaybd.top/user/devices) |
| 📦 Subscription Plans | [skypaybd.top/user/plans](https://skypaybd.top/user/plans) |
| ⬇️ Merchant Sync APK | [skypaybd.top/public/assets/downloads/SkyPay.apk](https://skypaybd.top/public/assets/downloads/SkyPay.apk) |
| 📄 Privacy Policy | [skypaybd.top/legal#privacy-policy](https://skypaybd.top/legal#privacy-policy) |
| 📋 Terms of Service | [skypaybd.top/legal#terms](https://skypaybd.top/legal#terms) |
| 💰 Refund Policy | [skypaybd.top/legal#refund-policy](https://skypaybd.top/legal#refund-policy) |
| 💵 Pricing | [skypaybd.top/#pricing](https://skypaybd.top/#pricing) |
| ❓ FAQ | [skypaybd.top/#faq](https://skypaybd.top/#faq) |

---

<div align="center">

*© 2024–2026 SkyPay BD. All rights reserved.*

**SkyPay Headless API v2 — Zero-Redirect Payment Infrastructure for Telegram Bots**

</div>
