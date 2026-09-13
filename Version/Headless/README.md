# SkyPay Headless Payment API v2 — Master Integration Guide

> **For AI Agents, Developers & Platform Builders**
> This document is a complete reference for integrating SkyPay's Headless Payment API (v2) into any platform — website, mobile app, Telegram bot, Discord bot, or custom application — without redirecting the user to any external page.

---

## What Is This API?

SkyPay Headless API v2 is a **zero-redirect payment infrastructure** for Bangladesh.

Instead of sending your user to an external payment page, your own platform:
1. Creates a payment session via API
2. Shows the merchant wallet numbers directly inside your interface
3. Asks the user for their Transaction ID (TrxID) after they pay
4. Verifies the TrxID via API in real-time
5. Fulfills the order — all without the user ever leaving your app

**Supported Payment Channels:** bKash · Nagad · Rocket · Upay

---

## Prerequisites (Before You Start)

Before making any API call, the following two things must be in place:

### 1. BRAND-KEY
- Go to your SkyPay Merchant Dashboard
- Create a **Brand** under your account
- Copy the generated **BRAND-KEY**
- This key identifies your merchant account and which connected phone handles the payment
- Store it securely in your server environment (`.env` file or secret manager)
- **Never expose it in frontend code, mobile bundles, or public repositories**

### 2. Merchant Android Device (SMS Sync Bridge)
- Install the **SkyPay Merchant Sync APK** on a dedicated Android smartphone
- The phone must contain your active merchant SIM cards (bKash, Nagad, Rocket, Upay)
- Grant **SMS Listener Permission** and **Notification Access** to the app
- Disable **Battery Optimization** for the SkyPay app
- Enter your **BRAND-KEY** inside the app and tap **Connect Device**
- Keep this phone **powered on and connected to internet 24/7**

> Without the connected Android device, all payment creation calls will return `403 Forbidden`.
> The Android phone is the hardware bridge — it reads incoming payment SMS and forwards them to the SkyPay cloud server in real-time.

---

## API Base URL

All Headless API v2 calls go to this domain:

```
https://core.skypaybd.top
```

---

## Authentication

Every API request requires exactly **one header**:

```
BRAND-KEY: your_unique_brand_key_here
Content-Type: application/json
```

No `SECRET-KEY` is required for v2.

---

## Full Endpoint Directory

| Action | Method | Full Endpoint URL |
|---|---|---|
| Create Payment Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| Verify Transaction | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

---

## Step-by-Step Integration Flow

```
User triggers payment (e.g. clicks "Deposit 500 BDT")
          │
          ▼
[Step 1] Your backend calls POST /api/v2/payment/create
          │
          ▼
SkyPay returns: session ID + active merchant wallet numbers
          │
          ▼
[Step 2] Your platform shows wallet numbers to the user
         inside your own interface (app screen, chat message, webpage modal)
         ⚠️ Show ONLY the channels returned in `methods[]` where active_payments flag = true
          │
          ▼
User selects a payment method (bKash / Nagad / Rocket / Upay)
User sends money via their MFS app
User receives an SMS with a Transaction ID (TrxID)
User selects which method they used + submits TrxID back into your interface
          │
          ▼
[Step 3] Your backend calls POST /api/v2/payment/verify
         ⚠️ MUST include: session id + method (lowercase) + transaction_id
          │
          ▼
SkyPay matches the TrxID against incoming SMS on the Android device (5–20 seconds)
          │
          ▼
[Step 4] Verification success → your platform fulfills the order
         (add balance, activate subscription, unlock content, etc.)
```

---

## Real-World Integration Example: Telegram Bot Flow

Below is a concrete, step-by-step walkthrough of how this API works inside a **Telegram Bot**. The same logic applies to websites, Discord bots, and any other platform.

```
1. User opens the bot and clicks the "💰 Deposit" button.

2. Bot asks: "How much do you want to deposit? (in BDT)"
   User replies: 500

3. Bot reads user's Telegram name (e.g. "Siyam Ahmed") automatically.

4. Bot's backend sends a POST request to /api/v2/payment/create:
   - Header: BRAND-KEY, Content-Type
   - Body: { "cus_name": "Siyam Ahmed", "amount": 500, "meta_data": { "telegram_id": 123456789 } }

5. SkyPay returns a session ID and active wallet numbers for bKash, Nagad, etc.

6. Bot sends a formatted message to the user:
   ────────────────────────
   💳 Payment Request — 500 BDT

   Send exactly 500 BDT to any of these numbers:

   📱 bKash (Send Money): 01XXXXXXXX
   📱 Nagad (Send Money): 01XXXXXXXX
   📱 Rocket (Send Money): 01XXXXXXXX

   ⚠️ After payment, you'll receive an SMS with a TrxID.
   Reply here with:
   1️⃣ Which method you used (bKash / Nagad / Rocket)
   2️⃣ Your Transaction ID (e.g. BLA38KDK2M)
   ────────────────────────

7. User pays 500 BDT via bKash, gets TrxID: BLA38KDK2M
   User replies: "bKash - BLA38KDK2M"

8. Bot's backend sends POST to /api/v2/payment/verify:
   - Body: { "id": "<session_id>", "method": "bkash", "transaction_id": "BLA38KDK2M" }

9. SkyPay confirms: { "status": true, "amount": "500.00" }

10. Bot credits 500 BDT to the user's account and sends confirmation.
```

> **⚠️ Critical for Telegram Integration:** Always save the session `id` from step 4 in your bot's conversation context (e.g., Redis, database, or in-memory store keyed by `telegram_user_id`). You MUST send this `id` in the verify call. Without it, verification is impossible.

---

## Step 1 — Create Payment Session

### Endpoint

```
POST https://core.skypaybd.top/api/v2/payment/create
```

### Request Headers

```
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Body Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `cus_name` | String | ✅ Yes | Full name or username of the customer |
| `amount` | Numeric | ✅ Yes | Amount in BDT (integer or up to 2 decimal places, must be greater than 0) |
| `meta_data` | Object | ❌ Optional | Any custom data you want to attach (user ID, order reference, plan name, etc.) |

### Example Request Body

```json
{
  "cus_name": "Siyam Ahmed",
  "amount": 500,
  "meta_data": {
    "user_id": "USR-4821",
    "plan": "PRO_MONTHLY",
    "reference": "INV-2026-001"
  }
}
```

### Success Response (HTTP 200)

```json
{
  "status": true,
  "id": "a1b2c3d4e5f6g7h8",
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
    }
  ]
}
```

### Response Fields Explained

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | `true` = session created successfully |
| `id` | String | Unique session ID — **save this immediately**, required for verification |
| `brand` | Object | Your merchant brand info (name, contact) |
| `methods` | Array | List of active payment channels with live wallet numbers |
| `methods[].name` | String | Channel name: `bkash`, `nagad`, `rocket`, `upay` |
| `methods[].active_payments.personal` | Boolean | If `true`, show the `personal` number for Send Money |
| `methods[].active_payments.agent` | Boolean | If `true`, show the `agent` number for Cash In |
| `methods[].active_payments.payment` | Boolean | If `true`, show the `payment` number for Merchant Pay (bKash only) |
| `methods[].personal` | String | Active phone number for Send Money transactions |
| `methods[].agent` | String | Active phone number for Agent Cash In (empty if inactive) |
| `methods[].payment` | String | Active merchant payment number (bKash only, empty if inactive) |

> **Critical:** Only show numbers where the corresponding `active_payments` flag is `true`. If a number is `""` or its flag is `false`, do not display that option to the user.

> **Important:** The `methods[]` array returned from `/create` represents ONLY the wallets your merchant has configured and enabled in your Brand setup on the SkyPay Dashboard. If bKash is not in the response, it means it is not set up on your merchant account — you cannot verify a bKash payment for a brand that has no bKash wallet connected. Always display and accept only the methods returned in this response.

---

## Step 2 — Presenting Payment Options to the User

After receiving the response from `/create`, your platform must display the active wallet numbers inside your own interface. The presentation format depends on your platform type.

### For Chat Interfaces (Telegram Bot, Discord Bot, etc.)

Display a formatted message with the active wallet numbers and instruct the user to reply with both their **chosen method** and their **TrxID**:

```
💳 Payment Request — 500 BDT

Send the exact amount to any of these numbers:

📱 bKash (Send Money): 01XXXXXXXX
📱 Nagad (Send Money): 01XXXXXXXX
📱 Rocket (Send Money): 01XXXXXXXX

⚠️ Instructions:
1. Open your MFS app (bKash / Nagad / Rocket)
2. Send exactly 500 BDT to one of the numbers above
3. After payment, you will receive an SMS with a Transaction ID (TrxID)
4. Reply to this message with the method you used AND your TrxID

Example reply: bkash BLA38KDK2M
```

Add buttons or prompts for the user to:
- **Select which method they used** (bKash / Nagad / Rocket / Upay)
- **Submit their TrxID**

Both pieces of information are **mandatory** for verification.

### For Web Applications & Dashboards

Render a payment modal or dedicated section showing:
- The amount due (bold, prominent)
- Channel tabs or cards (bKash / Nagad / Rocket / Upay) — **only show active ones from the API response**
- The wallet number for each active channel (with a copy button)
- A **dropdown or tab selector** for the user to choose which method they paid with
- A text input field labeled "Enter your Transaction ID (TrxID)"
- A submit/verify button that sends both the method and TrxID to your backend

### For Mobile Applications

Display a native bottom sheet or payment screen with:
- Amount header
- Payment method selector (show only channels where `active_payments` flag is true)
- Selected channel's wallet number with a one-tap copy functionality
- Deep link or redirect to open the respective MFS app if possible
- After the user confirms they have paid: a method confirmation step + a TrxID input field

### General Rules for All Platforms

- Display **only the channels and numbers that are currently active** (from the API response)
- Show the **exact amount** the user must send — partial payments will fail verification
- Make the wallet number **easily copyable** to avoid typos
- **Always ask the user to confirm which method they used** before submitting the TrxID
- Inform the user that payment SMS may take **5 to 20 seconds** to be detected
- Store the session `id` from Step 1 in your session, database, or bot context — you need it for Step 3

---

## Step 3 — Verify the Transaction

> ### ⚠️ THIS IS THE MOST CRITICAL STEP — READ CAREFULLY

When the user submits their TrxID, your backend **must** send **all three of the following** to SkyPay for verification:

1. **`id`** — The session ID you received from `/create` in Step 1
2. **`method`** — The exact payment channel the user used, in **strict lowercase** (e.g. `bkash`, not `Bkash`, not `BKASH`)
3. **`transaction_id`** — The TrxID from the user's payment SMS

**If any one of these three fields is missing, wrong, or not in the correct format — the verification WILL fail. There are no exceptions.**

Specifically:
- If `method` is not lowercase → `400 Unsupported payment method supplied`
- If `method` is a channel not configured in your brand → `400` error
- If `id` is wrong or expired → `404 Payment session not found or expired`
- If `transaction_id` doesn't match any incoming SMS → `400 Invalid transaction ID`

### Endpoint

```
POST https://core.skypaybd.top/api/v2/payment/verify
```

### Request Headers

```
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Body Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | String | ✅ Yes | The session ID received from `/create` in Step 1 |
| `method` | String | ✅ Yes | The payment channel the user used — **must be strict lowercase** |
| `transaction_id` | String | ✅ Yes | The TrxID from the user's payment SMS |

> **🚨 `method` Field Rules — Non-Negotiable:**
> - Must be **exactly one** of: `bkash` · `nagad` · `rocket` · `upay`
> - Must be **all lowercase** — no uppercase, no mixed case, no spaces
> - Must **match the channel the user actually paid through**
> - Must be a channel that **exists in the `methods[]` array** returned from your `/create` call
> - **Wrong method = failed verification, even if the TrxID is 100% valid**

### Example Request Body

```json
{
  "id": "a1b2c3d4e5f6g7h8",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

### Success Response (HTTP 200)

```json
{
  "status": true,
  "amount": "500.00",
  "cus_name": "Siyam Ahmed",
  "id": "a1b2c3d4e5f6g7h8"
}
```

### Success Response Fields Explained

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | `true` = payment verified successfully |
| `amount` | String | The verified payment amount in BDT (matches the session amount) |
| `cus_name` | String | The customer name provided during `/create` |
| `id` | String | The session ID that was verified |

When `status` is `true`, the payment is fully verified. Proceed to fulfill the order immediately.

### Error Responses

| HTTP Code | Response | Cause | Action |
|---|---|---|---|
| `400` | `Invalid transaction ID or transaction already used.` | TrxID not found in SMS logs, or already claimed by another session | Ask user to double-check TrxID, or wait 10 seconds and retry |
| `400` | `This payment session has already been completed.` | Session was already verified — prevents double fulfillment | Do not fulfill again |
| `400` | `Unsupported payment method supplied.` | `method` string is not lowercase, not one of the four valid values, or not configured in your brand | Fix the method string — must be `bkash`, `nagad`, `rocket`, or `upay` in strict lowercase |
| `401` | `Invalid or inactive BRAND-KEY provided.` | Wrong or deactivated BRAND-KEY | Check your Dashboard |
| `403` | (Device not connected) | Merchant Android phone is offline or APK is not running | Restart the SkyPay APK on your phone |
| `404` | `Payment session not found or expired.` | Session ID is wrong or the session has expired | Call `/create` again to generate a new session |

---

## Step 4 — Fulfill the Order After Verification

Once you receive `"status": true` from the verify endpoint, your platform should immediately execute the fulfillment logic. This is entirely on your side — SkyPay only confirms the payment was received and matched.

**Examples of fulfillment actions:**
- Add the verified `amount` to the user's wallet balance in your database
- Activate or extend the user's subscription
- Mark the invoice as paid
- Unlock premium content or features
- Send a confirmation message to the user

**Always do this on your backend server.** Never trust the user's claim of payment without a verified API response from SkyPay.

---

## SMS Synchronization Latency — The 5 to 20 Second Rule

When a customer pays via bKash, Nagad, Rocket, or Upay, the telecom network sends an SMS to your merchant Android phone. The SkyPay Sync App reads this SMS and pushes it to the cloud server via an encrypted connection. This entire process takes **5 to 20 seconds**.

**What this means for your integration:**

- If a user submits their TrxID immediately after paying, the SMS may still be in transit
- Your first verification attempt may return an error even though the payment was real
- **Do not permanently reject the user on the first failed attempt**

**Recommended retry flow:**

```
User submits TrxID + selected method
        │
        ▼
Call /verify endpoint
(with id + method + transaction_id)
        │
    ┌───┴───┐
  success  error (SMS not yet arrived)
    │           │
    ▼           ▼
Fulfill      Show message:
order        "Payment matching in progress.
             Please wait 10 seconds and tap Retry."
                  │
                  ▼
             User taps Retry
                  │
                  ▼
             Call /verify again
             (allow up to 2–3 minutes total)
```

---

## Security Rules

| Rule | Description |
|---|---|
| **Server-side only** | All API calls must come from your backend server, never from client-side JavaScript or frontend mobile code |
| **Protect your BRAND-KEY** | Store it in environment variables (`.env`). Never hardcode it or commit it to version control |
| **Idempotency** | Once a TrxID is verified, SkyPay marks it as claimed. The same TrxID cannot be used again under any session. Implement your own database check to prevent fulfilling the same order twice |
| **Exact amount match** | SkyPay verifies that the SMS amount matches the session amount. Partial payments will fail |
| **Lowercase method names** | Always send `bkash`, `nagad`, `rocket`, or `upay` — never capitalized or mixed case |
| **Only use methods from your brand** | Only the payment channels configured in your SkyPay Brand dashboard can be used for verification. Using a method that isn't in your brand's setup will result in an error |
| **Do not trust user input alone** | Always verify via API before granting any benefit |
| **Save session ID immediately** | The `id` from `/create` must be stored on your backend before presenting wallet numbers. If lost, the session cannot be verified and a new one must be created |

---

## HTTP Status Code Reference

| HTTP Code | Meaning | What To Do |
|---|---|---|
| `200 OK` | Request processed successfully | Check `"status": true` in the response body |
| `400 Bad Request` | Missing parameter, invalid TrxID, wrong/missing method, or already-used session | Read the `"message"` field for exact detail |
| `401 Unauthorized` | Missing or invalid BRAND-KEY | Verify your key in the Dashboard |
| `403 Forbidden` | No active Android device connected | Check that your SkyPay APK phone is online |
| `404 Not Found` | Session expired or does not exist | Call `/create` again for a new session |
| `405 Method Not Allowed` | Wrong HTTP method used (e.g. GET instead of POST) | Use POST for all endpoints |
| `500 Internal Server Error` | Temporary cloud-side error | Wait briefly and retry, or contact SkyPay support |

---

## Quick Reference Summary

```
BASE URL       : https://core.skypaybd.top
AUTH HEADER    : BRAND-KEY: <your_key>
CONTENT TYPE   : Content-Type: application/json

ENDPOINT 1 — Create Session
  POST /api/v2/payment/create
  Body: { "cus_name": "...", "amount": 500, "meta_data": {} }
  Returns: { "status": true, "id": "...", "methods": [...] }
  → Save the "id" immediately. It is required for verification.
  → Only display methods where active_payments flag is true.

ENDPOINT 2 — Verify Payment
  POST /api/v2/payment/verify
  Body: { "id": "...", "method": "bkash", "transaction_id": "..." }
  Returns: { "status": true, "amount": "500.00", "cus_name": "..." }
  → "id"             : session ID from /create (required)
  → "method"         : lowercase channel name (required, must match what user paid with)
  → "transaction_id" : TrxID from user's payment SMS (required)

VALID METHODS  : bkash | nagad | rocket | upay  (always lowercase, always from your brand setup)
SMS LATENCY    : 5–20 seconds (implement retry logic)
IDEMPOTENCY    : each TrxID can only be verified once
```

---

## Integration Checklist

Before going live, confirm all of the following:

- [ ] BRAND-KEY is stored securely on the server (not exposed to frontend)
- [ ] Merchant Android phone is powered on, connected to internet, and SkyPay APK is running
- [ ] Battery optimization is disabled for SkyPay APK on the merchant phone
- [ ] Session `id` from `/create` is saved before presenting wallet numbers to the user
- [ ] Only active wallet numbers (where `active_payments` flag is `true`) are shown to the user
- [ ] The exact session amount is shown to the user — no rounding, no modification
- [ ] User is asked to select which payment method they used before submitting TrxID
- [ ] The `method` field sent to `/verify` is always in strict lowercase (`bkash`, `nagad`, `rocket`, `upay`)
- [ ] The `method` field matches one of the channels returned in the `/create` response
- [ ] The session `id` from `/create` is included in every `/verify` request
- [ ] A retry mechanism with a 10-second delay exists for SMS sync latency
- [ ] Your database prevents double-fulfillment if the same session is verified twice
- [ ] Order fulfillment only happens after receiving `"status": true` from `/verify`

---

## Official Resources

| Resource | Link |
|---|---|
| Official Website | https://skypaybd.top |
| Interactive Documentation | https://skypaybd.top/docs |
| API Core Domain | https://core.skypaybd.top |
| GitHub Documentation Repo | https://github.com/SkyPayBD/Docs |
| Merchant Sync Android APK | https://skypaybd.top/public/assets/downloads/SkyPay.apk |
| WhatsApp Support | https://wa.me/+8801761844968 |
| Telegram | https://t.me/BD_Prime_Minister |

---

*SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
