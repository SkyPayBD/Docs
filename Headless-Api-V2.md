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
          │
          ▼
User sends money via bKash/Nagad/Rocket/Upay app
User receives an SMS with a Transaction ID (TrxID)
User submits that TrxID back into your interface
          │
          ▼
[Step 3] Your backend calls POST /api/v2/payment/verify
          │
          ▼
SkyPay matches the TrxID against incoming SMS on the Android device (5–20 seconds)
          │
          ▼
[Step 4] Verification success → your platform fulfills the order
         (add balance, activate subscription, unlock content, etc.)
```

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

---

## Step 2 — Presenting Payment Options to the User

After receiving the response from `/create`, your platform must display the active wallet numbers inside your own interface. The presentation format depends on your platform type.

### For Chat Interfaces (Telegram Bot, Discord Bot, etc.)

Display a formatted message with the active wallet numbers and instruct the user to reply with their TrxID:

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
4. Reply to this message with your TrxID

Example TrxID format: BLA38KDK2M
```

Add buttons or prompts for the user to submit their TrxID when ready.

### For Web Applications & Dashboards

Render a payment modal or dedicated section showing:
- The amount due (bold, prominent)
- Channel tabs or cards (bKash / Nagad / Rocket / Upay) — only show active ones
- The wallet number for each active channel (with a copy button)
- A text input field labeled "Enter your Transaction ID (TrxID)"
- A submit/verify button

### For Mobile Applications

Display a native bottom sheet or payment screen with:
- Amount header
- Payment method selector (show only channels where `active_payments` flag is true)
- Selected channel's wallet number with a one-tap copy functionality
- Deep link or redirect to open the respective MFS app if possible
- A TrxID input field that appears after the user confirms they have paid

### General Rules for All Platforms

- Display **only the channels and numbers that are currently active**
- Show the **exact amount** the user must send — partial payments will fail verification
- Make the wallet number **easily copyable** to avoid typos
- Inform the user that payment SMS may take **5 to 20 seconds** to be detected
- Store the session `id` from Step 1 in your session, database, or bot context — you need it for Step 3

---

## Step 3 — Verify the Transaction

When the user submits their TrxID, your backend immediately sends it to SkyPay for verification.

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
| `method` | String | ✅ Yes | The payment channel the user used — must be **strict lowercase** |
| `transaction_id` | String | ✅ Yes | The TrxID from the user's payment SMS |

**Valid values for `method`:** `bkash` · `nagad` · `rocket` · `upay`

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

When `status` is `true`, the payment is fully verified. Proceed to fulfill the order immediately.

### Error Responses

| HTTP Code | Response | Cause | Action |
|---|---|---|---|
| `400` | `Invalid transaction ID or transaction already used.` | TrxID not found in SMS logs, or already claimed by another session | Ask user to double-check TrxID, or wait 10 seconds and retry |
| `400` | `This payment session has already been completed.` | Session was already verified — prevents double fulfillment | Do not fulfill again |
| `400` | `Unsupported payment method supplied.` | Method string is not lowercase or not one of the four valid values | Fix the method string |
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
User submits TrxID
        │
        ▼
Call /verify endpoint
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
| **Do not trust user input alone** | Always verify via API before granting any benefit |

---

## HTTP Status Code Reference

| HTTP Code | Meaning | What To Do |
|---|---|---|
| `200 OK` | Request processed successfully | Check `"status": true` in the response body |
| `400 Bad Request` | Missing parameter, invalid TrxID, or already-used session | Read the `"message"` field for exact detail |
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

ENDPOINT 2 — Verify Payment
  POST /api/v2/payment/verify
  Body: { "id": "...", "method": "bkash", "transaction_id": "..." }
  Returns: { "status": true, "amount": "500.00", "cus_name": "..." }

VALID METHODS  : bkash | nagad | rocket | upay  (always lowercase)
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
- [ ] A retry mechanism with a 10-second delay exists for SMS sync latency
- [ ] Your database prevents double-fulfillment if the same session is verified twice
- [ ] `method` field is always sent in strict lowercase
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
