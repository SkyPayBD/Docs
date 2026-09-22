# SkyPay — Headless Payment API v2

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png" width="100" alt="SkyPay Logo" />

<br/>

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789098593_de7d9d238ad2e0178762.png" width="420" alt="SkyPay Banner" />

<br/><br/>

**Zero-Redirect Automated Payment Infrastructure for Bangladesh**

*Accept automated payments via **bKash**, **Nagad**, **Rocket**, **Upay**, and **Binance Pay** — directly inside your Telegram Bot, Mobile App, Discord Bot, or SPA — without ever redirecting the user to an external page.*

<br/>

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/API%20Version-v2.0%20Live-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
[![GitHub](https://img.shields.io/badge/GitHub-SkyPayBD-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/SkyPayBD)

</div>

---

> 📖 **এই ডকুমেন্টটি শুধুমাত্র Headless API v2 এর জন্য।**
> v1 Hosted Checkout এবং সম্পূর্ণ API Reference এর জন্য আলাদা ডকুমেন্ট দেখুন অথবা [skypaybd.top/docs](https://skypaybd.top/docs) ভিজিট করুন।

---

## 💳 Supported Payment Methods (v2)

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
| **Binance Pay** | `binance` | ✅ (USDT) | — | — | Binance Pay (Automatic) |

> **Note:** `binance` verification does **not** use the Android SMS bridge. SkyPay verifies Binance payments automatically on the server side — the customer only needs to provide their **Binance Order ID**. The Android device is only required for bKash, Nagad, Rocket, and Upay.

---

## 📋 Table of Contents

- [What Is This API?](#-what-is-this-api)
- [Prerequisites](#-prerequisites-before-you-start)
- [API Base URL](#-api-base-url)
- [Authentication](#-authentication)
- [Endpoint Directory](#-endpoint-directory)
- [Integration Flow (Overview)](#-step-by-step-integration-flow)
- [Real-World Example: Telegram Bot](#-real-world-example-telegram-bot-flow)
- [Step 1 — Create Payment Session](#step-1--create-payment-session)
- [Step 2 — Present Wallet Numbers to User](#step-2--present-wallet-numbers-to-user)
- [Step 3 — Verify Transaction](#step-3--verify-transaction)
  - [MFS Verification (bKash / Nagad / Rocket / Upay)](#mfs-verification-bkash--nagad--rocket--upay)
  - [Binance Pay Verification](#binance-pay-verification)
- [Step 4 — Fulfill the Order](#step-4--fulfill-the-order)
- [SMS Sync Latency — The 5–20 Second Rule](#-sms-sync-latency--the-520-second-rule)
- [Security Rules](#-security-rules)
- [HTTP Status Code Reference](#-http-status-code-reference)
- [Error Reference (v2 Verify)](#-error-reference-v2-verify)
- [Integration Checklist](#-integration-checklist)
- [Android Merchant Sync App Setup](#-android-merchant-sync-app-setup)
- [Quick Reference Summary](#-quick-reference-summary)
- [Quick Links](#-quick-links)
- [Contact & Support](#-contact--support)

---

## 🌐 What Is This API?

**SkyPay Headless API v2** is a zero-redirect payment system designed for Bangladesh's MFS ecosystem.

Instead of sending your user to an external payment page, your own platform:

1. Creates a payment session via API → receives active merchant wallet numbers
2. Shows those wallet numbers **directly inside your interface** (bot chat, app screen, modal)
3. User sends money via their MFS app and receives an SMS with a Transaction ID (TrxID)
4. User submits their TrxID back into your interface
5. Your backend calls the verify API → SkyPay matches it against incoming SMS on the merchant phone
6. On success → your platform fulfills the order — all without the user ever leaving your app

**This API is built for:** Telegram Bots · Discord Bots · Mobile Apps (Flutter, React Native) · Single Page Applications · Custom Headless Checkouts

---

## ⚙️ Prerequisites (Before You Start)

Two things must be in place before making any API call:

### 1. BRAND-KEY

- Log in to your SkyPay Merchant Dashboard
- Navigate to **[Brand Management](https://skypaybd.top/user/brands)**
- Create a Brand and copy the generated **BRAND-KEY**
- Store it securely in your server `.env` file or secret manager
- **Never expose it in frontend code, mobile APK bundles, or public repositories**

### 2. Merchant Android Device (SMS Sync Bridge)

Required for **bKash, Nagad, Rocket, Upay** verification. Not required for Binance.

- Install the **SkyPay Merchant Sync APK** on a dedicated Android smartphone
- The phone must contain your active merchant SIM cards (bKash / Nagad / Rocket / Upay)
- Grant **SMS Listener Permission** and **Notification Access** to the app
- Disable **Battery Optimization** for the SkyPay app (critical — Android will kill the service otherwise)
- Log in using your registered email and the **Device Key** from the Dashboard
- Enable **"Remember My Device"** before logging in
- Keep the phone **powered on and connected to internet 24/7**

> ⚠️ Without an active connected Android device, any call to `/api/v2/payment/create` for MFS gateways will return `403 Forbidden: No active SMS sync device found for this account.`

> 🔒 **IP Lock:** Each Device Key is locked to the IP address of the first login. If the IP changes and access is blocked, delete the device from [Device Management](https://skypaybd.top/user/devices) and create a new one.

---

## 🔗 API Base URL

All Headless API v2 calls use:

```
https://core.skypaybd.top
```

---

## 🔐 Authentication

Every v2 API request requires exactly **one header**:

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

> **Important:** For v2 endpoints, **only `BRAND-KEY` is accepted**. The header aliases `API-KEY`, `SECRET-KEY`, and the `?api_key=` query parameter are **not supported in v2** — they only work with the v1 Hosted Gateway. Always use `BRAND-KEY` for v2.

---

## 📡 Endpoint Directory

| Action | Method | Full Endpoint URL |
|---|:---:|---|
| Create Payment Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| Verify Transaction | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

---

## 🔄 Step-by-Step Integration Flow

```
User triggers payment (e.g. clicks "Deposit 500 BDT")
          │
          ▼
[Step 1] Your backend → POST /api/v2/payment/create
          Body: { cus_name, amount, meta_data }
          │
          ▼
SkyPay returns: session id + brand info + active methods[]
          │
          ├─ Save the session id immediately (in DB / bot context / session store)
          │
          ▼
[Step 2] Your platform displays wallet numbers to user
          Show ONLY channels where active_payments flag = true
          Show ONLY numbers that are not empty string ""
          │
          ▼
User selects payment method → sends money via MFS app
User receives SMS confirmation → gets TrxID
User submits: which method they used + their TrxID
          │
          ▼
[Step 3] Your backend → POST /api/v2/payment/verify
          Body: { id, method (lowercase), transaction_id }
          │
          ├─── For MFS: SkyPay matches TrxID against incoming Android SMS (5–20 seconds)
          └─── For Binance: SkyPay verifies Order ID automatically on the server (instant)
          │
          ▼
On success: { "status": true, "amount": "...", "cus_name": "...", "id": "..." }
          │
          ▼
[Step 4] Your platform fulfills the order
```

---

## 💬 Real-World Example: Telegram Bot Flow

```
1. User opens bot and clicks "💰 Deposit"

2. Bot asks: "Enter amount in BDT"
   User replies: 500

3. Bot's backend calls POST /api/v2/payment/create:
   Headers: BRAND-KEY, Content-Type: application/json
   Body:
   {
     "cus_name": "Siyam Ahmed",
     "amount": 500,
     "meta_data": { "telegram_id": 123456789, "plan": "VIP" }
   }

4. SkyPay returns session id + wallet numbers.
   Bot saves the session id mapped to telegram_id in database.

5. Bot sends message to user:
   ─────────────────────────────────────
   💳 Payment Request — 500 BDT

   Send exactly 500 BDT to any of these:

   🔵 bKash (Send Money): 01XXXXXXXX
   🟠 Nagad (Send Money): 01XXXXXXXX
   🟢 Rocket (Send Money): 01XXXXXXXX

   ⚠️ After paying, reply with:
      Method used + Transaction ID (TrxID from your SMS)
   Example: bkash BLA38KDK2M
   ─────────────────────────────────────

6. User pays 500 BDT via bKash, gets TrxID: BLA38KDK2M
   User replies: "bkash BLA38KDK2M"

7. Bot reads the reply, extracts method = "bkash" and transaction_id = "BLA38KDK2M"
   Retrieves the saved session id from database

8. Bot's backend calls POST /api/v2/payment/verify:
   {
     "id": "<session_id_from_step_4>",
     "method": "bkash",
     "transaction_id": "BLA38KDK2M"
   }

9. SkyPay responds: { "status": true, "amount": "500.00", "cus_name": "Siyam Ahmed", "id": "..." }

10. Bot credits 500 BDT to user's account and sends confirmation message.
```

> ⚠️ **Critical for Bot Integrations:** Always save the session `id` from Step 3 in your bot's conversation context (Redis, database, or in-memory store keyed by `telegram_user_id`). The `id` **is mandatory** in the verify call — without it, verification is impossible and you will need to create a new session.

---

## Step 1 — Create Payment Session

### Endpoint

```
POST https://core.skypaybd.top/api/v2/payment/create
```

### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Body Parameters

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `cus_name` | String | ✅ **Yes** | Full name or username of the customer. Used internally and returned in the verify response. |
| `amount` | Numeric | ✅ **Yes** | Total payable amount in BDT. Must be a positive number greater than `0`. Integer or decimal accepted. |
| `meta_data` | Object | ❌ Optional | Any custom JSON object you want to attach to the session (e.g. user ID, order reference, plan name, Telegram chat ID). Returned as-is in the verify response. Must be a valid JSON object — not a plain string. |

### Example Request Body

```json
{
  "cus_name": "Siyam Ahmed",
  "amount": 500,
  "meta_data": {
    "telegram_id": 123456789,
    "plan": "VIP_MONTHLY",
    "order_ref": "ORD-2026-001"
  }
}
```

### Success Response (HTTP 200)

```json
{
  "status": true,
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "brand": {
    "name": "My Store",
    "mobile": "017XXXXXXXX",
    "whatsapp": "017XXXXXXXX",
    "email": "support@mystore.com"
  },
  "methods": [
    {
      "name": "bkash",
      "active_payments": {
        "personal": true,
        "agent": false,
        "payment": true
      },
      "personal": "01XXXXXXXX",
      "agent": "",
      "payment": "01XXXXXXXX"
    },
    {
      "name": "nagad",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "01XXXXXXXX",
      "agent": ""
    },
    {
      "name": "rocket",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "01XXXXXXXX",
      "agent": ""
    },
    {
      "name": "upay",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "01XXXXXXXXX",
      "agent": ""
    },
    {
      "name": "binance",
      "active_payments": {
        "personal": true,
        "merchant": false
      },
      "personal": "XXXXXXXXXXXX",
      "currency": "USDT",
      "dollar_rate": "XXXX",
      "amount_usdt": "X.XXXX"
    }
  ]
}
```

### Response Fields — Detailed Explanation

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | `true` = session created successfully. `false` = error (check `message` field). |
| `id` | String | **Unique payment session ID. Save this immediately.** Required for the verify call. If lost, you must create a new session. |
| `brand` | Object | Your merchant brand information. |
| `brand.name` | String | Your brand's display name. |
| `brand.mobile` | String | Your brand's support phone number. |
| `brand.whatsapp` | String | Your brand's WhatsApp number. |
| `brand.email` | String | Your brand's support email. |
| `methods` | Array | List of active payment channels configured for your brand. **Only channels you have configured on your SkyPay dashboard will appear here.** |
| `methods[].name` | String | Channel identifier: `bkash`, `nagad`, `rocket`, `upay`, or `binance`. |
| `methods[].active_payments` | Object | Flags showing which sub-types are enabled for this channel. |
| `methods[].active_payments.personal` | Boolean | If `true`, the `personal` number is live for **Send Money** transactions. |
| `methods[].active_payments.agent` | Boolean | If `true`, the `agent` number is live for **Cash In** transactions. |
| `methods[].active_payments.payment` | Boolean | *(bKash only)* If `true`, the `payment` number is live for **Merchant Pay** transactions. |
| `methods[].active_payments.merchant` | Boolean | *(Binance only)* Merchant mode flag (currently unused, always `false`). |
| `methods[].personal` | String | Active phone number for Send Money. Empty string `""` if not active. |
| `methods[].agent` | String | Active phone number for Agent Cash In. Empty string `""` if not active. |
| `methods[].payment` | String | *(bKash only)* Active merchant payment number. Empty string `""` if not active. |
| `methods[].personal` *(Binance)* | String | The Binance Pay receiving ID — show this to the customer so they know where to send. |
| `methods[].currency` *(Binance)* | String | Always `"USDT"`. Only USDT is accepted. |
| `methods[].dollar_rate` *(Binance)* | Number | The BDT-to-USDT conversion rate set by the merchant in the Dashboard. |
| `methods[].amount_usdt` *(Binance)* | Number | The **exact USDT amount** the customer must send. Already calculated for you (= BDT amount ÷ rate, rounded to 4 decimal places). Show this directly to the customer. |

> **🚨 Critical Rule — Display Logic:**
> - Only show a channel to your user if it exists in `methods[]`
> - Within a channel, only show a wallet number if its corresponding `active_payments` flag is `true`
> - **Never show an empty string `""` as a wallet number**
> - A channel not returned in `methods[]` means it is not configured on your brand — you cannot verify payments for unconfigured channels

### Create — Error Responses

| HTTP Code | Message | Cause |
|---|---|---|
| `401` | `BRAND-KEY header is required.` | The `BRAND-KEY` header was not sent or is empty. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | The key does not exist or the brand is deactivated. |
| `403` | `Associated merchant account is inactive.` | The merchant user account linked to this brand is suspended. |
| `403` | `No active SMS sync device found for this account.` | No Android device is connected and active for this brand. |
| `400` | `Valid cus_name and numeric amount are required.` | `cus_name` is empty, or `amount` is missing / non-numeric / zero or negative. |
| `400` | `meta_data must be a valid JSON object.` | `meta_data` was sent as a string that is not valid JSON. |
| `400` | `No active payment gateways configured for this brand.` | Your brand has no payment gateways (bKash, Nagad, etc.) configured in the Dashboard. |
| `405` | `Method not allowed. Only POST requests are accepted.` | You sent a GET or other HTTP method instead of POST. |

---

## Step 2 — Present Wallet Numbers to User

After receiving the response from `/create`, display the active wallet numbers inside your own interface. Never show numbers that are empty or whose flags are `false`.

### For Chat Interfaces (Telegram Bot, Discord Bot)

```
💳 Payment Request — 500 BDT

Send the exact amount to one of these numbers:

🔵 bKash (Send Money): 01XXXXXXXX
🟠 Nagad (Send Money): 01XXXXXXXX
🟢 Rocket (Send Money): 01XXXXXXXX
🟡 Binance Pay (USDT): UID XXXXXXXXXXXX
   → Send exactly: X.XXXX USDT (from API response)

⚠️ Instructions:
1. Open your MFS app and send exactly 500 BDT (or the exact USDT for Binance)
2. After payment, you will receive an SMS with a Transaction ID (TrxID)
3. Reply with the method you used AND your TrxID
   Example: bkash BLA38KDK2M
            binance XXXXXXXXXXXXXXXXXX
```

### For Web Applications

Render a payment modal showing:
- Amount due (bold, prominent)
- Method tabs or cards — **only show active channels from API response**
- Wallet number with a **copy button** for each active channel
- A **dropdown/selector** for the user to choose which method they paid with
- A text input field: "Enter your Transaction ID (TrxID / Order ID)"
- A submit/verify button

### For Mobile Applications

Render a native payment screen with:
- Amount header
- Method selector (only show channels where `active_payments` flag is `true`)
- Selected channel's wallet number with one-tap copy
- Deep link to open the respective MFS app (optional)
- After user confirms payment: method confirmation + TrxID input field

### General Rules (All Platforms)

- Show **only active channels and numbers** from the API response
- Show the **exact amount** — no rounding, no modification
- For Binance: show the exact `amount_usdt` value from the response
- Make wallet numbers **easily copyable**
- Always ask the user **which method they used** before submitting TrxID
- Inform the user that SMS matching may take **5 to 20 seconds**
- Store the session `id` in your backend — needed for Step 3

---

## Step 3 — Verify Transaction

> ### ⚠️ THIS IS THE MOST CRITICAL STEP

When the user submits their TrxID, your backend **must** send all required fields to SkyPay. There are **two separate verification flows** depending on the payment method.

---

### MFS Verification (bKash / Nagad / Rocket / Upay)

**Endpoint:**

```
POST https://core.skypaybd.top/api/v2/payment/verify
```

**Request Headers:**

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

**Request Body Parameters:**

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `id` | String | ✅ **Yes** | The session ID returned by `/create` in Step 1. |
| `method` | String | ✅ **Yes** | The payment channel the user used. Must be **strict lowercase**: `bkash`, `nagad`, `rocket`, or `upay`. |
| `transaction_id` | String | ✅ **Yes** | The TrxID from the user's payment confirmation SMS. Also accepted as aliases: `order_id`, `order`, `trx_id`, `transactionId`, `transaction`. |

> **🚨 `method` Field Rules — Non-Negotiable:**
> - Must be **exactly one** of: `bkash` · `nagad` · `rocket` · `upay`
> - Must be **all lowercase** — no uppercase, no mixed case, no spaces
> - `"bKash"`, `"BKASH"`, `"Nagad"`, `"Rocket"` will **all fail** with `400 Unsupported payment method supplied`
> - Must **match the channel the user actually paid through**
> - Must be a channel that **exists in the `methods[]` array** returned from your `/create` call

**Example Request Body (bKash):**

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

**Example Request Body (Nagad):**

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "nagad",
  "transaction_id": "NGD123456789"
}
```

**Success Response (HTTP 200):**

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
      "telegram_id": 123456789,
      "plan": "VIP_MONTHLY"
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

> **Note:** For MFS (bKash, Nagad, Rocket, Upay) — the verify response wraps data inside a `data` object and includes `message`, `payment_method`, and `status: "COMPLETED"`. This is consistent with the API implementation.

---

### Binance Pay Verification

Binance verification কাজ করে MFS থেকে সম্পূর্ণ আলাদাভাবে।

**কীভাবে কাজ করে (সহজ ভাষায়):**
- MFS (bKash, Nagad ইত্যাদি) এ যেমন Android ফোনে SMS আসে এবং সেটা ম্যাচ করা হয়, Binance এ সেটা হয় না
- Binance এ পেমেন্ট করার পর কাস্টমার তাদের Binance অ্যাপ থেকে **Order ID** কপি করে দেয়
- SkyPay সেই Order ID দিয়ে **সার্ভার সাইডে স্বয়ংক্রিয়ভাবে** যাচাই করে নেয় — কাস্টমারকে বা আপনাকে কোনো বাড়তি কিছু করতে হয় না
- যাচাই হয় তাৎক্ষণিকভাবে — কোনো SMS latency নেই

> কাস্টমার Binance অ্যাপ খুলে → নির্দিষ্ট USDT পাঠায় → Order ID কপি করে আপনার ইন্টারফেসে দেয় → আপনি verify করেন → সম্পন্ন।

**Merchant এর পক্ষ থেকে কোনো Binance credential এই ডকুমেন্টে দেওয়ার প্রয়োজন নেই — সেগুলো শুধুমাত্র SkyPay Dashboard এ একবার সেটআপ করতে হয়।**

**Endpoint:**

```
POST https://core.skypaybd.top/api/v2/payment/verify
```

**Request Headers:**

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

**Request Body Parameters:**

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `id` | String | ✅ **Yes** | The session ID returned by `/create` in Step 1. |
| `method` | String | ✅ **Yes** | Must be exactly `binance` (lowercase). |
| `transaction_id` | String | ✅ **Yes** | The **Order ID** from the customer's Binance Pay transaction. Accepted as aliases: `order_id`, `order`, `trx_id`, `transactionId`, `transaction`. |

> **Binance-Specific Rules:**
> - কাস্টমারকে অবশ্যই `/create` response এ দেওয়া **`amount_usdt`** টি পাঠাতে হবে (tolerance: ±0.0005 USDT)
> - শুধুমাত্র **USDT** গ্রহণযোগ্য — অন্য কোনো currency কাজ করবে না
> - কাস্টমারকে `/create` response এ দেখানো **Binance Receiving ID** তে পাঠাতে হবে
> - একটি Order ID একবারের বেশি ব্যবহার করা যাবে না

**Example Request Body (Binance):**

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "binance",
  "transaction_id": "XXXXXXXXXXXXXXXXXX"
}
```

**Success Response (HTTP 200):**

```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Siyam Ahmed",
    "cus_email": "headless@skypaybd.top",
    "amount": 500,
    "transaction_id": "XXXXXXXXXXXXXXXXXX",
    "meta_data": {
      "telegram_id": 123456789,
      "plan": "VIP_MONTHLY"
    },
    "payment_method": "binance",
    "status": "COMPLETED"
  }
}
```

### Verify Response Fields — Explained

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | `true` = payment successfully verified and committed to database. |
| `message` | String | Human-readable confirmation message. |
| `data` | Object | Contains full payment details. |
| `data.cus_name` | String | The customer name provided during `/create`. |
| `data.cus_email` | String | Always `"headless@skypaybd.top"` for v2 sessions (internal placeholder). |
| `data.amount` | Number | The verified payment amount in BDT (as stored in the session). |
| `data.transaction_id` | String | The TrxID / Order ID that was verified. |
| `data.meta_data` | Object | The `meta_data` you attached during `/create`, returned as-is. |
| `data.payment_method` | String | The payment channel that was verified: `bkash`, `nagad`, `rocket`, `upay`, or `binance`. |
| `data.status` | String | Always `"COMPLETED"` on a successful verify response. |

---

## Step 4 — Fulfill the Order

Once you receive `"status": true` from the verify endpoint, execute your fulfillment logic immediately. This is entirely on your side — SkyPay only confirms the payment was received and matched.

**Examples of fulfillment actions:**
- Add the verified `amount` to the user's wallet balance in your database
- Activate or extend the user's subscription
- Mark the invoice as paid
- Unlock premium content or features
- Send a confirmation message to the user in your interface

**Always fulfill on your backend server.** Never trust any client-side signal — only a verified API response with `"status": true` from SkyPay authorizes fulfillment.

---

## ⏱️ SMS Sync Latency — The 5–20 Second Rule

When a customer pays via bKash, Nagad, Rocket, or Upay, the telecom network sends an SMS to your merchant Android phone. The SkyPay Sync App reads this SMS and pushes it to the cloud server via an encrypted connection. This entire round-trip takes **5 to 20 seconds**.

**What this means for your integration:**

- If a user submits their TrxID immediately after paying, the SMS may still be in transit
- The first verification attempt may return an error even though the payment was real
- **Do not permanently reject the user on the first failed attempt**

**Recommended Retry Flow:**

```
User submits TrxID + selected method
          │
          ▼
POST /api/v2/payment/verify
          │
    ┌─────┴─────┐
  success      error ("Transaction not found")
    │               │
    ▼               ▼
Fulfill          Show user:
order            "Payment matching in progress.
                 Please wait 10 seconds and tap Retry."
                      │
                      ▼
                User taps Retry
                      │
                      ▼
                POST /api/v2/payment/verify (again)
                Allow up to 2–3 minutes total retries
```

> এই latency শুধুমাত্র **MFS channels** (bKash, Nagad, Rocket, Upay) এর ক্ষেত্রে প্রযোজ্য। Binance verification তাৎক্ষণিক — কোনো SMS latency নেই।

---

## 🔒 Security Rules

| Rule | Description |
|---|---|
| **Server-side only** | All API calls must originate from your backend server. Never call from client-side JavaScript, mobile app code, or browser. |
| **Protect your BRAND-KEY** | Store in `.env` or a secrets manager. Never hardcode in source files or commit to version control. |
| **Idempotency** | Once a TrxID is verified and committed, SkyPay marks it as used. The same TrxID cannot be claimed again under any session. Implement your own DB check to prevent double-fulfillment on your side as well. |
| **Exact amount** | SkyPay verifies that the SMS amount matches the session amount. Partial payments will fail. For Binance: tolerance is ±0.0005 USDT only. |
| **Lowercase method names** | Always send `bkash`, `nagad`, `rocket`, `upay`, or `binance` — never capitalized or mixed case. |
| **Only use configured methods** | Only channels configured in your SkyPay brand dashboard can be verified. Attempting to verify a channel not in your `methods[]` response will result in a `400` error. |
| **Never trust user input alone** | Always verify via the SkyPay API before granting any benefit. |
| **Save session ID immediately** | The `id` from `/create` must be stored before presenting wallet numbers. If lost, you cannot verify the session — you must create a new one. |

---

## 🚦 HTTP Status Code Reference

| HTTP Code | Meaning | What To Do |
|---|---|---|
| `200 OK` | Request processed successfully | Check `"status": true` in the response body |
| `400 Bad Request` | Missing parameter, invalid TrxID, wrong method, or already-used session | Read the `"message"` field for exact detail |
| `401 Unauthorized` | Missing or invalid `BRAND-KEY` | Verify your key in the Brand Management Dashboard |
| `403 Forbidden` | Merchant account inactive, or no active Android device connected | Check your account status and verify the SkyPay APK is running on the merchant phone |
| `404 Not Found` | Session expired or does not exist | Call `/create` again to generate a new session |
| `405 Method Not Allowed` | Wrong HTTP method used (e.g. GET instead of POST) | Use `POST` for all v2 endpoints |
| `500 Internal Server Error` | Temporary cloud-side error | Wait briefly and retry; contact support if persistent |
| `502 Bad Gateway` | *(Binance only)* Binance service temporarily unreachable | Retry after a short delay; contact SkyPay support if persistent |

---

## ❌ Error Reference (v2 Verify)

| HTTP Code | Error Message | Cause | Fix |
|---|---|---|---|
| `400` | `id, method and transaction_id (or order_id) fields are required.` | One or more required fields are missing from the request body. | Include all three: `id`, `method`, `transaction_id`. |
| `400` | `Unsupported payment method supplied.` | `method` is not one of the valid options, or is not lowercase. | Use exactly `bkash`, `nagad`, `rocket`, `upay`, or `binance` — all lowercase. |
| `400` | `This payment session has already been completed.` | Session was already verified in a previous call. | Do not re-verify. Use your own DB to check order status. |
| `400` | `Transaction not found. Please check Order ID.` | TrxID / Order ID does not match any incoming SMS or Binance transaction. | Ask user to double-check TrxID. For MFS: wait 10–20 seconds and retry. |
| `400` | `Receiver UID does not match.` | *(Binance only)* Payment was sent to a different Binance UID. | Verify the merchant's Binance UID configuration in Dashboard. |
| `400` | `Only USDT payments are accepted.` | *(Binance only)* Customer sent a currency other than USDT. | Instruct customer to send USDT only. |
| `400` | `Insufficient amount received. Expected X USDT but received Y USDT.` | *(Binance only)* Customer sent less than required USDT (below ±0.0005 tolerance). | Ask customer to send the exact `amount_usdt` value shown. |
| `400` | `This Order ID has already been used.` | *(Binance only)* The Order ID was already committed to a previous session. | Do not accept duplicate Order IDs. |
| `401` | `BRAND-KEY header is required.` | The `BRAND-KEY` header is missing. | Add `BRAND-KEY` header to your request. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | Key is wrong or brand is deactivated. | Check your key in [Brand Management](https://skypaybd.top/user/brands). |
| `403` | `Associated merchant account is inactive.` | The merchant user account is suspended. | Contact SkyPay support. |
| `403` | `No active SMS sync device found for this account.` | Android phone is offline or APK service is stopped. | Restart the SkyPay APK and ensure the phone is online. |
| `404` | `Payment session not found or expired.` | Session `id` does not exist or belongs to another brand. | Call `/create` again for a fresh session. |
| `502` | *(Binance only)* `Failed to communicate with Binance. Please try again.` | Binance service is temporarily unreachable. | Retry after a short delay. Contact SkyPay support if issue persists. |

---

## ✅ Integration Checklist

Before going live, confirm all of the following:

**Security & Setup:**
- [ ] `BRAND-KEY` is stored on the server in `.env` — never exposed in frontend or mobile code
- [ ] Merchant Android phone is powered on, connected to internet, and SkyPay APK service is running
- [ ] Battery optimization is disabled for the SkyPay APK on the merchant phone
- [ ] All gateway wallets (bKash, Nagad, etc.) are correctly configured in the SkyPay Dashboard

**Create (`/create`) Flow:**
- [ ] Session `id` is saved to your database / bot context before presenting wallet numbers
- [ ] Only channels present in `methods[]` are displayed to the user
- [ ] Only wallet numbers where the corresponding `active_payments` flag is `true` are shown
- [ ] Empty string `""` wallet numbers are never displayed
- [ ] The exact session amount is shown to the user without modification
- [ ] For Binance: exact `amount_usdt` value is shown to the customer

**Verify (`/verify`) Flow:**
- [ ] The session `id` from `/create` is included in every `/verify` request
- [ ] The `method` field is always strict lowercase (`bkash`, `nagad`, `rocket`, `upay`, `binance`)
- [ ] The `method` matches one of the channels returned in the `/create` response
- [ ] A retry mechanism with a 10–20 second delay exists for SMS sync latency (MFS only)
- [ ] Order fulfillment only happens after receiving `"status": true` from `/verify`
- [ ] Your database prevents double-fulfillment (idempotency check) in case of duplicate verify calls

---

## 📱 Android Merchant Sync App Setup

*(Required for bKash, Nagad, Rocket, Upay — not required for Binance)*

<div align="center">

[![Download SkyPay APK](https://img.shields.io/badge/⬇%20Download%20SkyPay%20Merchant%20App-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://skypaybd.top/public/assets/downloads/SkyPay.apk)

</div>

### Requirements

- Android phone running Android 7.0 (Nougat) or higher
- Active merchant SIM cards (bKash / Nagad / Rocket / Upay) installed in the phone
- Phone kept plugged in to power 24/7
- Battery optimization **disabled** for the SkyPay app
- Uninterrupted WiFi or mobile data

### Setup Steps

**Step 1 — Download & Install**

Download `SkyPay.apk` from the link above and install on your Android device. Since this is sideloaded (not from Google Play), you may need to enable **"Install from Unknown Sources"** in your device settings.

**Step 2 — Create a Device in the Dashboard**

1. Log in to [Device Management](https://skypaybd.top/user/devices)
2. Ensure you have an active subscription ([Subscription Plans](https://skypaybd.top/user/plans))
3. Click **Create Device**, give it a name, and save
4. Copy the **Device Key** shown

**Step 3 — Log In to the App**

Open the SkyPay app and enter:

| Field | What to Enter |
|---|---|
| **Email** | Your registered email on skypaybd.top |
| **Device Key** | The Device Key copied from Device Management |

Enable **"Remember My Device"** before tapping Login. This keeps you logged in across phone restarts.

> ⚠️ **IP Lock:** Each Device Key is locked to the IP of the first device that logs in with it. If you try to log in from a different phone, or your IP changes and you get blocked, **delete the device** from [Device Management](https://skypaybd.top/user/devices) and create a new one.

**Step 4 — Grant SMS Permissions**

After login, a yellow warning banner appears asking for SMS permission. Tap **Grant** and allow all requested permissions.

If permissions don't work:
1. Long-press the SkyPay app icon → tap **App Info**
2. Go to **Permissions** and manually enable **SMS** access

**Step 5 — Keep the Service Running**

A toggle in the app lets you pause/resume the sync service. Keep it **ON** at all times. If paused, incoming SMS will not be forwarded and payments will not be verified.

---

## 📌 Quick Reference Summary

```
═══════════════════════════════════════════════════════════
  SKYPAY HEADLESS API v2 — QUICK REFERENCE
═══════════════════════════════════════════════════════════

BASE URL       : https://core.skypaybd.top
AUTH HEADER    : BRAND-KEY: <your_brand_key>
CONTENT TYPE   : Content-Type: application/json
HTTP METHOD    : POST (all endpoints)

───────────────────────────────────────────────────────────
ENDPOINT 1 — Create Payment Session
  POST /api/v2/payment/create

  Required Body Fields:
    cus_name    : String  — customer name or username
    amount      : Number  — amount in BDT (positive, non-zero)

  Optional Body Fields:
    meta_data   : Object  — any custom JSON data

  Success Response:
    {
      "status": true,
      "id": "<session_id>",       ← SAVE THIS
      "brand": { name, mobile, whatsapp, email },
      "methods": [
        {
          "name": "bkash",
          "active_payments": { personal, agent, payment },
          "personal": "01XXXXXXXX",
          "agent": "",
          "payment": "01XXXXXXXX"
        },
        {
          "name": "binance",
          "active_payments": { personal, merchant },
          "personal": "XXXXXXXXXXXX",   ← Binance receiving ID (show to customer)
          "currency": "USDT",
          "dollar_rate": "XXXX",
          "amount_usdt": "X.XXXX"       ← EXACT USDT to send (show to customer)
        }
      ]
    }

───────────────────────────────────────────────────────────
ENDPOINT 2 — Verify Transaction
  POST /api/v2/payment/verify

  Required Body Fields:
    id              : String — session ID from /create
    method          : String — lowercase: bkash|nagad|rocket|upay|binance
    transaction_id  : String — TrxID (MFS) or Order ID (Binance)
                              Aliases: order_id, order, trx_id, transactionId, transaction

  Success Response:
    {
      "status": true,
      "message": "Payment verified successfully.",
      "data": {
        "cus_name": "...",
        "cus_email": "headless@skypaybd.top",
        "amount": 500,
        "transaction_id": "...",
        "meta_data": { ... },
        "payment_method": "bkash",
        "status": "COMPLETED"
      }
    }

───────────────────────────────────────────────────────────
VALID METHODS  : bkash | nagad | rocket | upay | binance
                 (always strict lowercase)
SMS LATENCY    : 5–20 seconds for MFS (implement retry)
IDEMPOTENCY    : Each TrxID/Order ID can only be verified once
BINANCE        : Verifies automatically server-side — no SMS/Android needed
                 Customer provides Order ID from their Binance app
═══════════════════════════════════════════════════════════
```

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
| 📄 Privacy Policy | [skypaybd.top/legal#privacy-policy](https://skypaybd.top/legal#privacy-policy) |
| 📋 Terms of Service | [skypaybd.top/legal#terms](https://skypaybd.top/legal#terms) |
| 💰 Refund Policy | [skypaybd.top/legal#refund-policy](https://skypaybd.top/legal#refund-policy) |
| 💵 Pricing | [skypaybd.top/#pricing](https://skypaybd.top/#pricing) |
| ❓ FAQ | [skypaybd.top/#faq](https://skypaybd.top/#faq) |
| ⬇️ Merchant Sync APK | [skypaybd.top/public/assets/downloads/SkyPay.apk](https://skypaybd.top/public/assets/downloads/SkyPay.apk) |
| 📦 Offline Developer Docs | [skypaybd.top/public/assets/downloads/dev.zip](https://skypaybd.top/public/assets/downloads/dev.zip) |

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

<div align="center">

*© 2024–2026 SkyPay BD. All rights reserved.*

*SkyPay Headless API v2 — Automated MFS Payment Infrastructure for Bangladesh*

</div>
