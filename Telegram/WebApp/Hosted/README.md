# SkyPay Payment Gateway — Telegram Mini App Integration Guide for Hosted Gateway v1

## Overview

This document is a complete instruction set for an AI agent integrating the SkyPay BD payment gateway into a **Telegram Mini App (Web App)** using the **Hosted Gateway (v1)**. In this flow, the Mini App opens inside Telegram, collects the payment amount and customer details, calls your backend to create a SkyPay payment session, and then opens the SkyPay hosted checkout page — either by navigating the WebView to the payment URL or by opening it in an external browser via `Telegram.WebApp.openLink()`.

This is the simpler integration path. Your Mini App frontend handles the UI up to the point of payment initiation, and SkyPay's hosted page handles the entire payment process from there.

> **When to use this version:** Use the Hosted Gateway if you want the fastest integration path, do not want to build a custom payment UI inside your Mini App, and are comfortable with the user briefly leaving the Mini App to complete payment on SkyPay's page before returning.
>
> **Looking for a fully in-app experience?** See the [Headless API v2 WebApp guide](https://github.com/SkyPayBD/Docs/blob/main/Telegram/WebApp/Headless/README.md) for an integration where the user never leaves your Mini App.

> **Related Documentation**
> - Full Hosted Gateway v1 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md)
> - Full Headless API v2 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md)
> - Telegram Bot Integration Guide: [https://github.com/SkyPayBD/Docs/blob/main/Telegram/Bot/README.md](https://github.com/SkyPayBD/Docs/blob/main/Telegram/Bot/README.md)
> - Telegram Mini Apps Official Docs: [https://core.telegram.org/bots/webapps](https://core.telegram.org/bots/webapps)
> - Interactive SkyPay Documentation: [https://skypaybd.top/docs](https://skypaybd.top/docs)

---

## Which API to Use

Use the **Hosted Gateway (v1)**:

- Create URL: `POST https://core.skypaybd.top/api/payment/create`
- Verify URL: `POST https://core.skypaybd.top/api/payment/verify`

All API calls must come from your **backend server**, never directly from the Mini App's frontend JavaScript. The BRAND-KEY must never be exposed to the client.

---

## How Telegram Mini Apps Work — Key Concepts for This Integration

Before writing any code, understand these Telegram Mini App fundamentals:

### Telegram.WebApp Object

When a Mini App is opened, Telegram injects a global `window.Telegram.WebApp` object into the page. This object is the primary bridge between your web page and the Telegram client. Always initialize it first:

```javascript
const tg = window.Telegram.WebApp;
tg.ready(); // Must be called to signal the app is ready
tg.expand(); // Expand to full screen height
```

For the full API reference of `Telegram.WebApp`, see: [https://core.telegram.org/bots/webapps#initializing-mini-apps](https://core.telegram.org/bots/webapps#initializing-mini-apps)

### initData and User Identity

When the Mini App opens, Telegram provides `tg.initData` — a URL-encoded string containing the user's identity and session context. This is the only trusted source of user identity in a Mini App. You must send `initData` to your backend and **validate it server-side** before trusting any user identity claim.

`tg.initDataUnsafe` gives you a parsed JavaScript object of the same data for convenience on the client side, but never use `initDataUnsafe` alone for security decisions — always validate the raw `initData` string on your server.

Key fields available in `initDataUnsafe`:

| Field             | Type   | Description                                         |
| ----------------- | ------ | --------------------------------------------------- |
| `user.id`         | Number | Telegram user ID (unique, permanent identifier)     |
| `user.first_name` | String | User's first name                                   |
| `user.last_name`  | String | User's last name (may be empty)                     |
| `user.username`   | String | Telegram @username (may be absent)                  |
| `user.language_code` | String | User's Telegram language setting                 |
| `chat_instance`   | String | Unique ID of the Mini App session                   |
| `hash`            | String | HMAC signature used to validate the entire initData |

### Validating initData on Your Backend

Send the raw `tg.initData` string to your backend in every request that needs user identity. On your backend, validate it using your bot token:

1. Parse the `initData` URL-encoded string into key-value pairs
2. Extract the `hash` field and remove it from the set
3. Sort the remaining fields alphabetically and join them as `key=value\n` lines
4. Compute `HMAC-SHA256` of the joined string using a key derived from `HMAC-SHA256("WebAppData", bot_token)`
5. Compare your computed hash to the extracted `hash` — they must match

If the hash does not match, reject the request. Never trust a `user.id` value that has not been validated this way.

For language-specific validation examples:
- Python: [https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app](https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app)
- Node.js and other languages follow the same HMAC-SHA256 logic described in the official docs above.

### Opening External URLs

Since the Mini App renders inside Telegram's internal browser, redirecting to an external URL requires using Telegram's API methods rather than `window.location.href`. Use:

```javascript
tg.openLink(url);           // Opens URL in the system browser (outside Telegram)
tg.openTelegramLink(url);   // Opens a t.me link inside Telegram
```

For the SkyPay hosted payment page, use `tg.openLink(paymentUrl)` to open it in the device's default browser. The user completes payment there and returns to Telegram.

### Common Frontend Frameworks

Telegram Mini Apps are standard web pages and work with any frontend framework. Common choices:

- **Vanilla JavaScript + HTML/CSS** — simplest, no build step required
- **React** — widely used; pairs well with Vite for fast builds
- **Vue.js** — lightweight alternative to React
- **Next.js** — if you need server-side rendering alongside the Mini App frontend

Use the `@twa-dev/sdk` npm package for TypeScript type definitions and a more ergonomic wrapper around `Telegram.WebApp`:
[https://github.com/twa-dev/SDK](https://github.com/twa-dev/SDK)

---

## Authentication

Every backend API call to SkyPay requires:

```
BRAND-KEY: <merchant_brand_key>
Content-Type: application/json
```

Store BRAND-KEY in your backend environment variables. Never send it to the Mini App frontend. All SkyPay API calls must originate from your server.

---

## Database Setup

### Table: `webapp_payment_sessions`

Track every payment session initiated from the Mini App.

| Column           | Type                                             | Nullable | Description                                                       |
| ---------------- | ------------------------------------------------ | -------- | ----------------------------------------------------------------- |
| `id`             | INT UNSIGNED AUTO_INCREMENT                      | No       | Primary key                                                       |
| `telegram_user_id` | BIGINT                                         | No       | Validated Telegram user ID from initData                          |
| `order_id`       | VARCHAR(100)                                     | No       | Your internal order reference                                     |
| `transaction_id` | VARCHAR(100)                                     | Yes      | TrxID returned by SkyPay after successful payment                |
| `cus_name`       | VARCHAR(255)                                     | No       | Customer name (from initData or user input)                       |
| `cus_email`      | VARCHAR(255)                                     | Yes      | Customer email (collected in the Mini App UI)                     |
| `amount`         | DECIMAL(10,2)                                    | No       | Payment amount in BDT                                             |
| `payment_method` | VARCHAR(50)                                      | Yes      | `bkash` / `nagad` / `rocket` / `upay`                            |
| `status`         | ENUM('pending','completed','failed','cancelled') | No       | Payment lifecycle state                                           |
| `metadata`       | JSON                                             | Yes      | Extra data (product ID, plan name, etc.)                          |
| `verified_at`    | TIMESTAMP                                        | Yes      | Timestamp of successful backend verification                      |
| `created_at`     | TIMESTAMP                                        | No       | When the payment session was initiated                            |

---

## Architecture Overview

```
[Telegram Client]
      │
      │ Opens Mini App via inline button or menu
      ▼
[Mini App Frontend]  ←── tg.initData sent with every backend request
      │
      │ POST /api/webapp/initiate  (user fills amount + email)
      ▼
[Your Backend Server]
      │
      │ Validates initData hash
      │ POST /api/payment/create  →  SkyPay
      │ Returns payment_url to frontend
      ▼
[Mini App Frontend]
      │
      │ tg.openLink(payment_url)
      ▼
[User's System Browser]  ←── SkyPay hosted checkout page
      │
      │ User pays via bKash/Nagad/Rocket/Upay
      │ SkyPay redirects to success_url
      ▼
[Your Backend — success_url handler]
      │
      │ POST /api/payment/verify  →  SkyPay
      │ If COMPLETED → fulfill order, update DB
      ▼
[Notify user in Telegram]
      │ Bot sends confirmation message to the user's chat
```

---

## File/Folder Architecture

```
/webapp-backend/
    config/
        gateway.js (or .py / .php)  ← Gateway config loader
    routes/
        payment.js                  ← /initiate, /callback, /cancel endpoints
    services/
        skypayService.js            ← SkyPay API communication
        telegramAuth.js             ← initData validation logic
    models/
        PaymentSession.js           ← DB model for webapp_payment_sessions

/webapp-frontend/
    index.html
    app.js (or App.jsx / App.vue)   ← Main Mini App page
    payment.js                      ← Payment initiation logic
    success.html                    ← Shown after return from SkyPay page
    cancel.html                     ← Shown on cancellation
```

---

## Frontend — Initializing the Mini App

The very first thing your Mini App page must do on load:

```javascript
const tg = window.Telegram.WebApp;
tg.ready();
tg.expand();

// Read user data (for display only — never trust without backend validation)
const user = tg.initDataUnsafe?.user;
const displayName = user ? `${user.first_name} ${user.last_name || ''}`.trim() : 'Guest';

// The raw initData string to send with every backend request
const initData = tg.initData;
```

Always pass `initData` as a header or body field in every request your frontend makes to your backend. Example using `fetch`:

```javascript
async function callBackend(endpoint, body) {
    const response = await fetch(endpoint, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-Telegram-Init-Data': tg.initData
        },
        body: JSON.stringify(body)
    });
    return response.json();
}
```

---

## Frontend — Payment Initiation UI

Your Mini App should present a simple form:

- An amount input field (number, min 1, no decimals or up to 2)
- An email input field (required for SkyPay hosted checkout)
- A "Pay Now" button styled with Telegram's native colors using `tg.themeParams`

When the user taps "Pay Now":

1. Validate the amount is a positive number and email is a valid format
2. Show a loading state (disable the button, show spinner)
3. Call your backend `/api/webapp/initiate` with `{ amount, email }` and include `tg.initData` in the request
4. If your backend returns a `payment_url`, call `tg.openLink(payment_url)` to open SkyPay's hosted page in the system browser
5. Show a "Waiting for payment confirmation..." state in the Mini App while the user completes payment in the browser

---

## Backend — Initiate Endpoint

`POST /api/webapp/initiate`

This endpoint is called by the Mini App frontend when the user submits the payment form.

Steps:

1. Extract `X-Telegram-Init-Data` from the request header (or from the request body if you prefer)
2. Validate the initData hash using your bot token (see initData validation section above). If validation fails, return 401.
3. Parse `telegram_user_id` and `first_name` from the validated initData
4. Extract `amount` and `email` from the request body. Validate both.
5. Generate a unique `order_id`
6. Insert a new row into `webapp_payment_sessions` with `status = 'pending'`
7. Build `success_url` pointing to your backend's `/api/webapp/callback` route, including `order_id` as a query parameter
8. Build `cancel_url` pointing to your backend's `/api/webapp/cancel` route
9. Call SkyPay `POST /api/payment/create` with:
   ```json
   {
     "cus_name": "<first_name from initData>",
     "cus_email": "<email from request body>",
     "amount": <amount>,
     "success_url": "<your success_url>",
     "cancel_url": "<your cancel_url>",
     "metadata": {
       "order_id": "<order_id>",
       "telegram_user_id": "<telegram_user_id>"
     }
   }
   ```
10. If SkyPay returns `status: true`, return `{ "payment_url": "..." }` to the Mini App frontend
11. If SkyPay returns an error, return an appropriate error response to the frontend

---

## Backend — Callback Endpoint (success_url)

`GET /api/webapp/callback`

SkyPay redirects the user's browser here after the payment is completed on the hosted page. This is a browser redirect, not a Mini App request — there is no `initData` available here. Identify the order using the `order_id` query parameter.

Query parameters received from SkyPay:

| Parameter       | Example      | Description                           |
| --------------- | ------------ | ------------------------------------- |
| `transactionId` | `BLA38KDK2M` | The TrxID that was verified           |
| `paymentMethod` | `bkash`      | Channel used                          |
| `paymentAmount` | `500.00`     | Amount paid                           |
| `paymentFee`    | `0.00`       | Gateway fee                           |
| `status`        | `completed`  | `completed`, `pending`, or `failed`   |

**Critical security rule:** Never fulfill an order based solely on these parameters. Always call `/api/payment/verify` from your server before taking any action.

Steps:

1. Read `transactionId` and `order_id` from query parameters
2. Look up the `webapp_payment_sessions` row by `order_id`. If already `completed`, stop.
3. Call SkyPay `POST /api/payment/verify` with `{ "transaction_id": "<transactionId>" }`
4. If verify response `status === "COMPLETED"`:
   - Update the session row: `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at`
   - Retrieve `telegram_user_id` from the session row or from the verify response `metadata`
   - Fulfill the order in your database (add balance, activate plan, etc.)
   - Use your Telegram bot to send a confirmation message to the user's chat: `bot.sendMessage(telegram_user_id, "✅ Payment confirmed! ...")`
   - Render a success HTML page that the browser displays, instructing the user to return to Telegram
5. If verification fails, render an error page and optionally notify the user via bot message

---

## Backend — Cancel Endpoint

`GET /api/webapp/cancel`

Called when the user cancels on the SkyPay hosted page.

Steps:

1. Read `order_id` from query parameters
2. Look up the session row. If `status` is `pending`, update to `cancelled`
3. Optionally send a Telegram bot message: "Payment was cancelled. Tap /pay to try again."
4. Render a cancellation page instructing the user to return to Telegram

---

## Notifying the User in Telegram After Payment

Because the user completes payment in an external browser and then returns to Telegram, the Mini App may no longer be active. Use your Telegram bot to send a message to the user's chat after fulfillment:

```
✅ Payment Confirmed!

Amount: 500 BDT
Method: bKash
TrxID: BLA38KDK2M

Your [subscription/balance/access] has been activated.
Return to the Mini App to continue.
```

Use `sendMessage` from your bot library with the `telegram_user_id` retrieved from the session record. This is the most reliable way to notify the user regardless of whether the Mini App is still open.

---

## Returning to the Mini App After Payment

After the user returns to Telegram from the system browser, they can reopen the Mini App. Your Mini App should check payment status on load:

```javascript
// On app load, check if there is a pending payment for this user
const status = await callBackend('/api/webapp/payment-status', {});
if (status.completed) {
    showSuccessScreen(status.amount, status.method);
}
```

Your backend `/api/webapp/payment-status` endpoint should validate initData, look up the most recent session for this `telegram_user_id`, and return the current status.

---

## Security Rules the AI Agent Must Follow

- Validate `initData` hash on every backend request that involves user identity. Never trust `initDataUnsafe` alone.
- Store BRAND-KEY only in backend environment variables. Never send it to the Mini App frontend.
- Always call `verify` before fulfilling any order. Never trust SkyPay's redirect parameters alone.
- Check for duplicate `transaction_id` in your database before fulfilling (idempotency guard).
- Use parameterized queries or ORM methods for all database operations.
- The `success_url` and `cancel_url` must be publicly accessible HTTPS endpoints — `localhost` URLs will not work.
- Log all SkyPay API calls server-side with timestamps and full responses.

---

## Complete Payment Flow Summary

```
User opens Mini App → tg.ready() + tg.expand()
User fills amount + email → taps "Pay Now"
    │
    ▼
Frontend → POST /api/webapp/initiate  (with initData)
    │
    ▼
Backend: validate initData → POST /api/payment/create → returns payment_url
    │
    ▼
Frontend: tg.openLink(payment_url)
    │
User completes payment in system browser on SkyPay's page
SkyPay redirects to your success_url
    │
    ▼
Backend /api/webapp/callback:
    → POST /api/payment/verify
    → If COMPLETED: update DB, fulfill order
    → Send confirmation via bot: bot.sendMessage(telegram_user_id, "✅ Payment confirmed!")
    → Render "Return to Telegram" page in browser
    │
User returns to Telegram and sees confirmation message
```

---

## HTTP Status Code Reference

| HTTP Code                   | Meaning                                               | What To Do                                              |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| `200 OK`                    | Request processed successfully                        | Check `"status"` value in the response body             |
| `400 Bad Request`           | Missing parameter or invalid data                     | Read the `"message"` field for details                  |
| `401 Unauthorized`          | Missing or invalid BRAND-KEY                          | Verify your key in the Dashboard                        |
| `403 Forbidden`             | No active Android device connected                    | Check that the SkyPay APK phone is online               |
| `404 Not Found`             | Endpoint not found                                    | Ensure correct URL and HTTP method                      |
| `405 Method Not Allowed`    | Wrong HTTP method                                     | Use POST for all SkyPay endpoints                       |
| `500 Internal Server Error` | Temporary cloud-side error                            | Wait briefly and retry, or contact SkyPay support       |

---

## Official Resources

| Resource                       | Link                                                                                    |
| ------------------------------ | --------------------------------------------------------------------------------------- |
| Official Website               | https://skypaybd.top                                                                    |
| Interactive Documentation      | https://skypaybd.top/docs                                                               |
| API Core Domain                | https://core.skypaybd.top                                                               |
| GitHub Documentation Repo      | https://github.com/SkyPayBD/Docs                                                        |
| Hosted Gateway v1 Full Docs    | https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md                    |
| Headless API v2 Full Docs      | https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md                  |
| Telegram Bot Integration Guide | https://github.com/SkyPayBD/Docs/blob/main/Telegram/Bot/README.md                      |
| WebApp Headless Guide          | https://github.com/SkyPayBD/Docs/blob/main/Telegram/WebApp/Headless/README.md          |
| Telegram Mini Apps Docs        | https://core.telegram.org/bots/webapps                                                  |
| Telegram initData Validation   | https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app        |
| TWA Dev SDK (npm)              | https://github.com/twa-dev/SDK                                                          |
| Merchant Sync Android APK      | https://skypaybd.top/public/assets/downloads/SkyPay.apk                                 |
| WhatsApp Support               | https://wa.me/+8801761844968                                                             |
| Telegram                       | https://t.me/BD_Prime_Minister                                                           |

---

*SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
