# SkyPay Payment Gateway — Telegram Mini App Integration Guide for Headless API v2

## Overview

This document is a complete instruction set for an AI agent integrating the SkyPay BD payment gateway into a **Telegram Mini App (Web App)** using the **Headless API (v2)**. In this flow, the entire payment experience happens inside the Mini App's WebView — the user never leaves Telegram, never opens an external browser, and never sees any SkyPay-branded page. Your Mini App frontend displays the merchant wallet numbers, collects the user's TrxID, and your backend calls SkyPay to verify the payment — all without any redirect.

This is the premium integration path. It delivers a seamless, native-feeling payment experience fully embedded within Telegram.

> **When to use this version:** Use the Headless API when you want the user to stay entirely within your Mini App. The payment UI is yours to design. This is ideal for wallet top-up screens, in-app checkout flows, and subscription panels where a redirect would disrupt the experience.
>
> **Looking for a simpler integration with redirect?** See the [Hosted Gateway WebApp guide](https://github.com/SkyPayBD/Docs/blob/main/Telegram/WebApp/Hosted/README.md) for a redirect-based integration that requires less frontend work.

> **Related Documentation**
> - Full Headless API v2 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md)
> - Full Hosted Gateway v1 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md)
> - Telegram Bot Integration Guide: [https://github.com/SkyPayBD/Docs/blob/main/Telegram/Bot/README.md](https://github.com/SkyPayBD/Docs/blob/main/Telegram/Bot/README.md)
> - Telegram Mini Apps Official Docs: [https://core.telegram.org/bots/webapps](https://core.telegram.org/bots/webapps)
> - Interactive SkyPay Documentation: [https://skypaybd.top/docs](https://skypaybd.top/docs)

---

## Which API to Use

Use the **Headless API (v2)**:

- Create Session URL: `POST https://core.skypaybd.top/api/v2/payment/create`
- Verify Transaction URL: `POST https://core.skypaybd.top/api/v2/payment/verify`

All API calls must come from your **backend server**, never from the Mini App's frontend JavaScript. The BRAND-KEY must never be exposed to the client.

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

When the Mini App opens, Telegram provides `tg.initData` — a URL-encoded string containing the user's Telegram identity and session context. This is the only trusted source of user identity in a Mini App. You must send `initData` to your backend with every request and **validate it server-side** before trusting any user identity claim.

`tg.initDataUnsafe` gives you a parsed JavaScript object of the same data, useful for reading values on the frontend. However, never use `initDataUnsafe` alone for any security or business logic decision — always validate the raw `initData` on your server.

Key fields available in `initDataUnsafe`:

| Field                | Type   | Description                                                    |
| -------------------- | ------ | -------------------------------------------------------------- |
| `user.id`            | Number | Telegram user ID — permanent, unique identifier                |
| `user.first_name`    | String | User's first name                                              |
| `user.last_name`     | String | User's last name (may be empty)                                |
| `user.username`      | String | Telegram @username (may be absent)                             |
| `user.language_code` | String | User's Telegram language setting                               |
| `chat_instance`      | String | Unique ID of this Mini App session                             |
| `hash`               | String | HMAC-SHA256 signature used to validate the entire initData     |

### Validating initData on Your Backend

Send the raw `tg.initData` string to your backend in every request that involves user identity. On your backend, validate it using your Telegram bot token:

1. Parse the `initData` URL-encoded string into key-value pairs
2. Extract the `hash` field and remove it from the set
3. Sort the remaining fields alphabetically and join them as `key=value\n` lines (newline-separated)
4. Compute `HMAC-SHA256` of the joined string using a secret key derived from `HMAC-SHA256("WebAppData", bot_token)`
5. Compare your computed hash to the extracted `hash` — they must match exactly

If the hashes do not match, reject the request with `401 Unauthorized`. Never proceed with unvalidated user data.

For the official validation algorithm and examples:
[https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app](https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app)

### Closing the Mini App and Sending Data Back to the Bot

The Headless flow completes entirely inside the Mini App. Once payment is verified by your backend, you can:

- Update the Mini App UI to show a success screen (stay in the app)
- Call `tg.close()` to close the Mini App and return the user to the chat
- Call `tg.sendData(jsonString)` to send a small data payload from the Mini App to your bot's webhook — useful for triggering a bot action after payment

For `sendData` to work, the Mini App must have been opened via a `KeyboardButton` with `web_app` type (not an inline button). If opened via an inline button, use a backend bot message instead of `sendData`.

Reference: [https://core.telegram.org/bots/webapps#initializing-mini-apps](https://core.telegram.org/bots/webapps#initializing-mini-apps)

### Back Button and MainButton

Use Telegram's native UI components for a consistent experience:

```javascript
// Show a native "Back" button in the Mini App header
tg.BackButton.show();
tg.BackButton.onClick(() => { /* go back to previous screen */ });

// Show a native action button at the bottom
tg.MainButton.setText('Confirm Payment');
tg.MainButton.show();
tg.MainButton.onClick(handleConfirm);

// Show loading state on the MainButton
tg.MainButton.showProgress();  // spinner
tg.MainButton.hideProgress();  // remove spinner
```

Reference: [https://core.telegram.org/bots/webapps#main-button](https://core.telegram.org/bots/webapps#main-button)

### Theme and Colors

Always use `tg.themeParams` to style your Mini App to match the user's Telegram theme:

```javascript
const theme = tg.themeParams;
// Available: bg_color, text_color, hint_color, link_color,
//            button_color, button_text_color, secondary_bg_color
document.body.style.backgroundColor = theme.bg_color;
```

Reference: [https://core.telegram.org/bots/webapps#themeparams](https://core.telegram.org/bots/webapps#themeparams)

### Common Frontend Frameworks

Telegram Mini Apps are standard web pages and work with any frontend framework:

- **Vanilla JavaScript + HTML/CSS** — no build step, simplest to deploy
- **React + Vite** — fast development, component-based UI; pairs well with `@twa-dev/sdk`
- **Vue.js** — lightweight reactive framework
- **Svelte** — minimal bundle size, good for performance-sensitive Mini Apps

Use the `@twa-dev/sdk` npm package for TypeScript types and a developer-friendly wrapper:
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

Track every payment session, the wallet numbers shown, and the verification outcome.

| Column              | Type                                                                   | Nullable | Description                                                          |
| ------------------- | ---------------------------------------------------------------------- | -------- | -------------------------------------------------------------------- |
| `id`                | INT UNSIGNED AUTO_INCREMENT                                            | No       | Primary key                                                          |
| `telegram_user_id`  | BIGINT                                                                 | No       | Validated Telegram user ID from initData                             |
| `order_id`          | VARCHAR(100)                                                           | No       | Your internal order reference                                        |
| `skypay_session_id` | VARCHAR(100)                                                           | No       | The `id` returned by SkyPay `/create` — required for verify          |
| `transaction_id`    | VARCHAR(100)                                                           | Yes      | TrxID submitted by the user                                          |
| `cus_name`          | VARCHAR(255)                                                           | No       | Customer name (from initData)                                        |
| `amount`            | DECIMAL(10,2)                                                          | No       | Payment amount in BDT                                                |
| `payment_method`    | VARCHAR(50)                                                            | Yes      | `bkash` / `nagad` / `rocket` / `upay` (submitted by user)           |
| `methods_json`      | JSON                                                                   | Yes      | Full `methods[]` array from `/create` stored for audit               |
| `status`            | ENUM('pending','awaiting_trxid','verifying','completed','failed','expired') | No  | Payment session lifecycle state                                      |
| `retry_count`       | INT DEFAULT 0                                                          | No       | Number of verify retry attempts made                                 |
| `expires_at`        | TIMESTAMP                                                              | Yes      | Session expiry (typically 20 minutes after creation)                 |
| `meta_data`         | JSON                                                                   | Yes      | Extra data (product ID, plan, etc.)                                  |
| `verified_at`       | TIMESTAMP                                                              | Yes      | Timestamp of successful verification                                 |
| `created_at`        | TIMESTAMP                                                              | No       | When the payment session was created                                 |

> **Critical:** Save `skypay_session_id` to the database immediately after `/create` returns, before rendering wallet numbers in the Mini App. If this is skipped, verification will be impossible.

### Table: `fulfilled_orders`

Prevent double fulfillment with a separate fulfilled orders log.

| Column              | Type                  | Nullable | Description                  |
| ------------------- | --------------------- | -------- | ---------------------------- |
| `id`                | INT UNSIGNED          | No       | Primary key                  |
| `telegram_user_id`  | BIGINT                | No       | Telegram user ID             |
| `skypay_session_id` | VARCHAR(100) UNIQUE   | No       | SkyPay session ID            |
| `transaction_id`    | VARCHAR(100) UNIQUE   | No       | Verified TrxID               |
| `amount`            | DECIMAL(10,2)         | No       | Verified amount paid         |
| `payment_method`    | VARCHAR(50)           | Yes      | Channel used                 |
| `fulfilled_at`      | TIMESTAMP             | No       | When order was fulfilled     |

---

## Architecture Overview

```
[Telegram Client]
      │
      │ Opens Mini App via inline button / menu
      ▼
[Mini App Frontend]  ←── tg.initData sent with every backend request
      │
      │ User selects amount → taps "Proceed"
      │ POST /api/webapp/initiate  (initData + amount)
      ▼
[Your Backend Server]
      │ Validates initData hash
      │ POST /api/v2/payment/create  →  SkyPay
      │ Saves skypay_session_id + methods_json to DB
      │ Returns { session_id, methods } to Mini App
      ▼
[Mini App Frontend]
      │ Renders wallet numbers inside the Mini App
      │ User selects channel, sends money via MFS app
      │ User gets TrxID via SMS
      │ User selects method + enters TrxID → taps "Verify"
      │ POST /api/webapp/verify  (initData + method + trxid)
      ▼
[Your Backend Server]
      │ Validates initData hash
      │ POST /api/v2/payment/verify  →  SkyPay
      │ If status: true → fulfill order, update DB
      │ Returns result to Mini App
      ▼
[Mini App Frontend]
      │ Shows success screen
      │ tg.MainButton.setText("Close") → tg.close()
```

---

## File/Folder Architecture

```
/webapp-backend/
    config/
        gateway.js (or .py / .php)    ← Gateway config loader
    routes/
        payment.js                    ← /initiate, /verify, /status, /cancel
    services/
        skypayService.js              ← SkyPay v2 API communication
        telegramAuth.js               ← initData validation
    models/
        PaymentSession.js             ← DB model for webapp_payment_sessions
        FulfilledOrder.js             ← DB model for fulfilled_orders

/webapp-frontend/
    index.html                        ← Entry point, loads Telegram.WebApp SDK
    app.js (or App.jsx / App.vue)     ← Main screen (amount input)
    payment.js                        ← Payment screen (wallet numbers + TrxID input)
    success.js                        ← Success screen
```

---

## Frontend — Initializing the Mini App

The very first thing your Mini App page must do on load:

```javascript
const tg = window.Telegram.WebApp;
tg.ready();
tg.expand();

// Apply Telegram theme colors
document.body.style.backgroundColor = tg.themeParams.bg_color || '#ffffff';
document.body.style.color = tg.themeParams.text_color || '#000000';

// Read user info for display (not for trust — backend validates initData)
const user = tg.initDataUnsafe?.user;
const displayName = user
    ? `${user.first_name} ${user.last_name || ''}`.trim()
    : 'User';

// Raw initData to send with every backend request
const initData = tg.initData;
```

Always pass `initData` with every backend call:

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

## Frontend — Payment Initiation Screen

Present a clean payment form inside the Mini App:

- Amount input (number, min 1)
- Display the user's name from `tg.initDataUnsafe.user.first_name` (read-only)
- Optional: product or plan selector
- Use `tg.MainButton` as the primary action button for a native feel:

```javascript
tg.MainButton.setText('Proceed to Payment');
tg.MainButton.show();
tg.MainButton.onClick(handleProceed);

async function handleProceed() {
    const amount = parseFloat(document.getElementById('amount').value);
    if (!amount || amount <= 0) {
        tg.showAlert('Please enter a valid amount.');
        return;
    }

    tg.MainButton.showProgress();
    tg.MainButton.disable();

    const result = await callBackend('/api/webapp/initiate', { amount });

    tg.MainButton.hideProgress();
    tg.MainButton.enable();

    if (result.success) {
        showPaymentScreen(result.methods, result.session_id, amount);
    } else {
        tg.showAlert('Payment initiation failed. Please try again.');
    }
}
```

---

## Frontend — Wallet Number Display Screen

After the backend returns the `methods[]` array, render the payment screen:

```javascript
function showPaymentScreen(methods, sessionId, amount) {
    // Filter: only show active numbers
    const activeMethods = methods.filter(m =>
        m.active_payments.personal || m.active_payments.agent || m.active_payments.payment
    );

    // Render wallet numbers
    // For each active method, show only the number types where the flag is true
    // Label them: personal = "Send Money", agent = "Cash In", payment = "Merchant Pay"

    // Save sessionId in a variable/closure for the verify call
    currentSessionId = sessionId;

    // Show method selector (buttons or dropdown)
    // Show TrxID input field
    // Update MainButton
    tg.MainButton.setText('Confirm Payment');
    tg.MainButton.onClick(handleVerify);
}
```

Display rules:

- Show only channels returned in `methods[]` where at least one `active_payments` flag is `true`
- For each channel, show only the number type (personal/agent/payment) where the flag is `true` and the number string is not empty
- Make each wallet number copyable — on tap, copy to clipboard and show a brief confirmation
- Display the exact amount prominently — the user must send this exact figure
- Instruct the user to note their TrxID from the payment SMS before tapping Confirm

---

## Frontend — TrxID Submission and Verify

```javascript
async function handleVerify() {
    const method = getSelectedMethod(); // e.g. 'bkash'
    const trxId = document.getElementById('trxid-input').value.trim();

    if (!method) {
        tg.showAlert('Please select the payment method you used.');
        return;
    }
    if (!trxId) {
        tg.showAlert('Please enter your Transaction ID (TrxID).');
        return;
    }

    tg.MainButton.showProgress();
    tg.MainButton.disable();

    const result = await callBackend('/api/webapp/verify', {
        session_id: currentSessionId,
        method: method.toLowerCase(),
        transaction_id: trxId
    });

    tg.MainButton.hideProgress();
    tg.MainButton.enable();

    if (result.success) {
        showSuccessScreen(result.amount, result.method);
    } else if (result.retry) {
        tg.showAlert('Payment is still being matched. Please wait 10 seconds and try again.');
    } else {
        tg.showAlert(result.message || 'Verification failed. Please contact support.');
    }
}
```

---

## Frontend — Success Screen

```javascript
function showSuccessScreen(amount, method) {
    // Render a success screen with amount and method
    // Use tg.MainButton for the close action
    tg.MainButton.setText('Close');
    tg.MainButton.show();
    tg.MainButton.enable();
    tg.MainButton.onClick(() => tg.close());

    // Optionally show a Telegram popup
    tg.showPopup({
        title: 'Payment Confirmed!',
        message: `${amount} BDT via ${method} has been verified. Your account has been updated.`,
        buttons: [{ type: 'close' }]
    });
}
```

---

## Backend — Initiate Endpoint

`POST /api/webapp/initiate`

Steps:

1. Extract and validate `initData` from the `X-Telegram-Init-Data` header. Return 401 if invalid.
2. Parse `telegram_user_id` and `first_name` from the validated initData.
3. Extract `amount` from the request body. Validate it is a positive number.
4. Generate a unique `order_id`.
5. Call SkyPay `POST /api/v2/payment/create`:
   ```json
   {
     "cus_name": "<first_name from initData>",
     "amount": <amount>,
     "meta_data": {
       "order_id": "<order_id>",
       "telegram_user_id": "<telegram_user_id>"
     }
   }
   ```
6. If SkyPay returns `status: true`:
   - Save to `webapp_payment_sessions`: `skypay_session_id = response.id`, `methods_json = response.methods`, `status = 'pending'`, `expires_at = NOW + 20 minutes`
   - Return to frontend: `{ "success": true, "session_id": "<order_id>", "methods": <filtered active methods array> }`
7. If SkyPay returns an error, return `{ "success": false, "message": "..." }`

> **Important:** Return the `order_id` (not the `skypay_session_id`) to the frontend as the session reference. The `skypay_session_id` is a server-side secret used only in the verify call — the frontend only needs a reference key to look up the session.

---

## Backend — Verify Endpoint

`POST /api/webapp/verify`

Steps:

1. Extract and validate `initData` from the header. Return 401 if invalid.
2. Parse `telegram_user_id` from validated initData.
3. Extract `session_id` (your `order_id`), `method`, and `transaction_id` from request body.
4. Look up `webapp_payment_sessions` by `order_id` AND `telegram_user_id`. Return 404 if not found.
5. If `status` is already `completed`, return `{ "success": true, "already_completed": true }` (idempotency guard).
6. If `expires_at < NOW()`, update status to `expired` and return `{ "success": false, "message": "Session expired. Please start a new payment." }`.
7. Check `fulfilled_orders` for this `skypay_session_id` — if exists, return already completed.
8. Sanitize `method`: apply lowercase + trim. Validate it is one of `bkash`, `nagad`, `rocket`, `upay`.
9. Increment `retry_count` on the session row.
10. Call SkyPay `POST /api/v2/payment/verify`:
    ```json
    {
      "id": "<skypay_session_id from DB>",
      "method": "<method>",
      "transaction_id": "<transaction_id>"
    }
    ```
11. If SkyPay returns `status: true`:
    - Update session: `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = NOW()`
    - Insert row into `fulfilled_orders`
    - Fulfill the order (credit balance, activate plan, etc.)
    - Optionally send a bot confirmation message: `bot.sendMessage(telegram_user_id, "✅ Payment confirmed!")`
    - Return `{ "success": true, "amount": "500.00", "method": "bkash" }`
12. If SkyPay returns an error:
    - If error indicates TrxID not found (SMS in transit): return `{ "success": false, "retry": true, "message": "Payment matching in progress." }`
    - If error is `session already completed`: return `{ "success": true, "already_completed": true }`
    - If `retry_count >= 6` or definitive error: update status to `failed`, return `{ "success": false, "retry": false, "message": "..." }`

---

## Backend — Payment Status Endpoint (Optional)

`POST /api/webapp/status`

Used when the Mini App reopens and needs to check if a previous payment was completed.

Steps:

1. Validate `initData`. Parse `telegram_user_id`.
2. Look up the most recent `webapp_payment_sessions` row for this `telegram_user_id` where `status IN ('pending', 'awaiting_trxid', 'completed')`.
3. Return the current status, amount, and payment method.
4. The Mini App frontend can use this to auto-show the success screen if the user reopens the app after completing payment.

---

## Security Rules the AI Agent Must Follow

- Validate `initData` hash on every backend request that involves user identity or business logic. Never skip this step.
- Never expose `skypay_session_id` or BRAND-KEY to the Mini App frontend. The frontend only needs an `order_id` reference to poll status.
- Always sanitize `method` to lowercase before sending to SkyPay. Use `strtolower()` in PHP, `.lower()` in Python, `.toLowerCase()` in JavaScript.
- Check `fulfilled_orders` before every fulfillment action. Use a unique constraint on `transaction_id` in the database.
- Set a session expiry (`expires_at`) and check it at the start of the verify handler. Do not allow verification on expired sessions.
- Log all SkyPay API calls with full request and response, timestamp, and `telegram_user_id`.
- Use parameterized queries or ORM methods for all database operations.
- The BRAND-KEY must never appear in any HTTP response, log file visible to users, or frontend code.

---

## Complete Payment Flow Summary

```
User opens Mini App → tg.ready() + tg.expand()
User enters amount → taps MainButton "Proceed to Payment"
    │
    ▼
Frontend → POST /api/webapp/initiate  (with initData)
    │
    ▼
Backend: validate initData → POST /api/v2/payment/create
→ Save skypay_session_id to DB
→ Return active methods[] to frontend

Frontend renders wallet numbers inside Mini App
User selects channel, sends money via MFS app
User gets TrxID via SMS
User selects method + enters TrxID → taps MainButton "Confirm Payment"
    │
    ▼
Frontend → POST /api/webapp/verify  (with initData + method + trxid)
    │
    ▼
Backend: validate initData → load skypay_session_id from DB
→ POST /api/v2/payment/verify with { id, method, transaction_id }
→ If status: true → fulfill order, insert fulfilled_orders
    → Return { success: true } to Mini App
    → Frontend shows success screen → tg.MainButton "Close" → tg.close()
→ If TrxID not found → Return { success: false, retry: true }
    → Frontend shows retry prompt
→ If definitive error → Return { success: false, retry: false }
    → Frontend shows error message
```

---

## HTTP Status Code Reference

| HTTP Code                   | Meaning                                               | What To Do                                              |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| `200 OK`                    | Request processed successfully                        | Check `"status": true` in the response body             |
| `400 Bad Request`           | Missing field, invalid TrxID, wrong method, or reuse  | Read the `"message"` field; show retry or error         |
| `401 Unauthorized`          | Missing or invalid BRAND-KEY                          | Verify BRAND-KEY in the Dashboard                       |
| `403 Forbidden`             | No active Android device connected                    | Check that the SkyPay APK phone is online               |
| `404 Not Found`             | Session ID not found or expired                       | Call `/create` again for a new session                  |
| `405 Method Not Allowed`    | Wrong HTTP method                                     | Use POST for all SkyPay endpoints                       |
| `500 Internal Server Error` | Temporary cloud-side error                            | Wait briefly and retry, or contact SkyPay support       |

---

## Official Resources

| Resource                           | Link                                                                                    |
| ---------------------------------- | --------------------------------------------------------------------------------------- |
| Official Website                   | https://skypaybd.top                                                                    |
| Interactive Documentation          | https://skypaybd.top/docs                                                               |
| API Core Domain                    | https://core.skypaybd.top                                                               |
| GitHub Documentation Repo          | https://github.com/SkyPayBD/Docs                                                        |
| Headless API v2 Full Docs          | https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md                  |
| Hosted Gateway v1 Full Docs        | https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md                    |
| Telegram Bot Integration Guide     | https://github.com/SkyPayBD/Docs/blob/main/Telegram/Bot/README.md                      |
| WebApp Hosted Guide                | https://github.com/SkyPayBD/Docs/blob/main/Telegram/WebApp/Hosted/README.md            |
| Telegram Mini Apps Docs            | https://core.telegram.org/bots/webapps                                                  |
| Telegram initData Validation       | https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app        |
| Telegram MainButton Docs           | https://core.telegram.org/bots/webapps#main-button                                     |
| Telegram ThemeParams Docs          | https://core.telegram.org/bots/webapps#themeparams                                     |
| TWA Dev SDK (npm)                  | https://github.com/twa-dev/SDK                                                          |
| Merchant Sync Android APK          | https://skypaybd.top/public/assets/downloads/SkyPay.apk                                 |
| WhatsApp Support                   | https://wa.me/+8801761844968                                                             |
| Telegram                           | https://t.me/BD_Prime_Minister                                                           |

---

*SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
