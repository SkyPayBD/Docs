# SkyPay Payment Gateway — PHP (Vanilla) Integration Guide for Headless API v2

## Overview

This document is a complete instruction set for an AI agent integrating the SkyPay BD payment gateway into a **vanilla PHP** backend application using the **Headless API (v2)**. Unlike the Hosted Gateway, the Headless API does not redirect the user to any external page. Instead, your PHP backend acts as a bridge — it fetches the merchant wallet numbers from SkyPay, presents them through your own interface (web dashboard, custom panel, or any frontend you control), collects the user's Transaction ID (TrxID), and verifies the payment entirely within your application.

This approach is ideal for building a custom payment bridge on top of SkyPay's infrastructure — for example, a white-label payment panel, an in-app wallet top-up system, or a self-hosted checkout interface where you need full control over the user experience.

Always read the main SkyPay Headless API documentation first to understand the full API structure, response formats, session lifecycle, and all available fields before writing any code.

> **Related Documentation**
> - Full Headless API v2 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md)
> - Full Hosted Gateway v1 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md)
> - PHP Hosted Integration Guide: [https://github.com/SkyPayBD/Docs/blob/main/Backend/PHP/Hosted/README.md](https://github.com/SkyPayBD/Docs/blob/main/Backend/PHP/Hosted/README.md)
> - Interactive Documentation: [https://skypaybd.top/docs](https://skypaybd.top/docs)

---

## Which API to Use

Use the **Headless API (v2)**:

- Create Session URL: `POST https://core.skypaybd.top/api/v2/payment/create`
- Verify Transaction URL: `POST https://core.skypaybd.top/api/v2/payment/verify`

Do NOT use the Hosted Gateway v1 for this integration. The Headless API is specifically for scenarios where you build your own payment UI and the user never leaves your application.

---

## Authentication

Every request to the SkyPay API requires these HTTP headers:

```
BRAND-KEY: <merchant_brand_key>
Content-Type: application/json
```

There is no SECRET-KEY. Only BRAND-KEY is required. The BRAND-KEY must be stored server-side in a `.env` file or PHP config constant — never in frontend HTML, JavaScript, or any client-facing code.

---

## How the Headless Flow Works in PHP

Because PHP is a server-side language without a persistent runtime, your application must manage the payment session `id` returned from `/create` between requests. Store it in the database (tied to the user's order or session) immediately after receiving it. This `id` is mandatory for the verify call — if it is lost, the session cannot be recovered and a new one must be created.

Your PHP backend serves as the bridge between your frontend interface and the SkyPay API. The user interacts entirely with your own pages; your PHP code makes all API calls silently in the background.

---

## Database Setup

### Table: `auto_payment_gateway`

Create this MySQL table to store gateway configuration. This allows the merchant to manage the gateway settings from an admin panel without touching code.

| Column       | Type                         | Nullable | Description                                                     |
| ------------ | ---------------------------- | -------- | --------------------------------------------------------------- |
| `id`         | INT UNSIGNED AUTO_INCREMENT  | No       | Primary key                                                     |
| `brand_key`  | VARCHAR(255)                 | No       | The merchant BRAND-KEY from SkyPay dashboard                    |
| `create_url` | VARCHAR(255)                 | No       | Full URL: `https://core.skypaybd.top/api/v2/payment/create`     |
| `verify_url` | VARCHAR(255)                 | No       | Full URL: `https://core.skypaybd.top/api/v2/payment/verify`     |
| `status`     | ENUM('active', 'inactive')   | No       | Whether this gateway is currently enabled                       |
| `created_at` | TIMESTAMP                    | No       | Auto-set on insert                                              |
| `updated_at` | TIMESTAMP                    | Yes      | Auto-set on update                                              |

Insert a default row during setup with `status = 'active'`, the two v2 endpoint URLs, and the merchant's BRAND-KEY.

### Table: `payments`

Track every payment session, the wallet numbers shown to the user, and the final verification result.

| Column           | Type                                             | Nullable | Description                                                       |
| ---------------- | ------------------------------------------------ | -------- | ----------------------------------------------------------------- |
| `id`             | INT UNSIGNED AUTO_INCREMENT                      | No       | Primary key                                                       |
| `order_id`       | VARCHAR(100)                                     | No       | Your internal order or invoice reference                          |
| `session_id`     | VARCHAR(100)                                     | No       | The `id` returned by SkyPay `/create` — required for verify       |
| `transaction_id` | VARCHAR(100)                                     | Yes      | TrxID submitted by the user after payment                        |
| `cus_name`       | VARCHAR(255)                                     | No       | Customer name sent to SkyPay                                     |
| `amount`         | DECIMAL(10,2)                                    | No       | Payment amount in BDT                                            |
| `payment_method` | VARCHAR(50)                                      | Yes      | bkash / nagad / rocket / upay (submitted by user)                |
| `methods_json`   | JSON                                             | Yes      | Full `methods[]` array from `/create` response, stored for display|
| `status`         | ENUM('pending','completed','failed','cancelled') | No       | Payment lifecycle state                                          |
| `meta_data`      | JSON                                             | Yes      | Any extra data (user ID, plan name, reference, etc.)             |
| `verified_at`    | TIMESTAMP                                        | Yes      | Timestamp of successful backend verification                     |
| `created_at`     | TIMESTAMP                                        | No       | When the payment session was initiated                           |

> **Critical:** The `session_id` column must be saved to the database immediately after calling `/create`, before you display anything to the user. If your process dies after showing wallet numbers but before saving the session ID, verification will be impossible.

---

## File/Folder Architecture

The AI agent should create the following file structure inside the project:

```
/payment/
    config.php            ← Reads gateway config from DB
    SkyPayService.php     ← All API communication logic (v2)
    initiate.php          ← Creates payment session, shows wallet numbers to user
    verify.php            ← Receives TrxID from user, verifies with SkyPay API
    status.php            ← (Optional) Polling endpoint for AJAX-based retry flow
    cancel.php            ← Handles user-initiated cancellation
```

---

## config.php

This file connects to the database and fetches the active gateway row from `auto_payment_gateway`. It exposes `$gateway` as an associative array with `brand_key`, `create_url`, and `verify_url`. If no active gateway row is found, it halts execution with an appropriate error. Do not output the `brand_key` value anywhere in the HTML response.

---

## SkyPayService.php

This is the core service class. It encapsulates all communication with the SkyPay Headless API v2. Instantiate it by passing the active gateway config array.

### Method: `createSession(array $data): array`

Accepts:

- `cus_name` (string, required) — full name or username of the customer
- `amount` (numeric, required) — amount in BDT, must be greater than 0
- `meta_data` (array, optional) — pass `order_id`, `user_id`, `plan`, or any reference data

Sends a POST request to `create_url` with the `BRAND-KEY` header and JSON body. Returns the full decoded JSON response from SkyPay.

On success, the response contains:

- `status` — `true` if session was created
- `id` — the unique session ID (save this immediately to the database)
- `brand` — merchant brand info (name, contact)
- `methods[]` — array of active payment channels with live wallet numbers

Each item in `methods[]` has:

- `name` — channel name (`bkash`, `nagad`, `rocket`, `upay`)
- `active_payments` — object with boolean flags: `personal`, `agent`, `payment`
- `personal` — phone number for Send Money (show only if `active_payments.personal` is `true`)
- `agent` — phone number for Cash In (show only if `active_payments.agent` is `true`)
- `payment` — merchant payment number, bKash only (show only if `active_payments.payment` is `true`)

Only display wallet numbers where the corresponding `active_payments` flag is `true`. Never show a number that has an empty string value or a `false` flag.

### Method: `verifyTransaction(string $sessionId, string $method, string $transactionId): array`

Accepts:

- `$sessionId` (string) — the `id` from the `/create` response, retrieved from your database
- `$method` (string) — the payment channel the user selected, in **strict lowercase** (`bkash`, `nagad`, `rocket`, or `upay`)
- `$transactionId` (string) — the TrxID from the user's payment SMS

Sends a POST request to `verify_url` with the `BRAND-KEY` header and JSON body `{ "id": "...", "method": "...", "transaction_id": "..." }`. Returns decoded JSON.

On success, the response contains:

- `status` — `true` if payment is verified
- `amount` — verified amount in BDT
- `cus_name` — customer name from the session
- `id` — the verified session ID

**If any of the three fields (`id`, `method`, `transaction_id`) is missing, incorrect, or `method` is not lowercase — the API will return a 400 error and verification will fail.**

### HTTP Implementation

Use PHP's `curl` for all API calls. Always:

- Set `CURLOPT_TIMEOUT` to 30 seconds
- Set `CURLOPT_RETURNTRANSFER` to `true`
- Send both `Content-Type: application/json` and `BRAND-KEY` headers as an array in `CURLOPT_HTTPHEADER`
- Encode the request body with `json_encode()`
- Decode the response with `json_decode($response, true)`
- Handle curl errors with `curl_error()` and log them server-side
- Always close the curl handle with `curl_close()`

---

## initiate.php

This is the entry point called when the user initiates a payment (submits an amount, clicks "Add Funds", etc.).

Steps the AI agent must implement here:

1. Validate required inputs: `cus_name` and `amount`. Reject if `amount` is not a positive number.
2. Load gateway config via `config.php`.
3. Build the `meta_data` array with your internal references: `order_id`, `user_id`, or any identifier you need returned at verify time.
4. Call `SkyPayService::createSession()` with `cus_name`, `amount`, and `meta_data`.
5. If the API returns `status: true`:
   - Generate a unique `order_id` for this payment (e.g. `ORD-` + `uniqid()`)
   - Insert a new row into the `payments` table immediately with `status = 'pending'`, `session_id = $response['id']`, and `methods_json = json_encode($response['methods'])`
   - Render your payment UI showing the active wallet numbers from `$response['methods']`
   - For each method in `methods[]`, only show channels where `active_payments` flags are `true`
   - Display the exact `amount` the user must send — do not round or modify it
   - Show a method selector (dropdown or buttons) and a TrxID input field
   - The form's submit action should POST to `verify.php`
   - Include the `order_id` as a hidden field so `verify.php` can look up the session
6. If the API returns an error, display a user-friendly error message. Do not expose raw API error details to the user.

---

## verify.php

This file is called when the user submits their TrxID after making the payment.

Steps the AI agent must implement here:

1. Read from POST: `order_id`, `payment_method`, and `transaction_id`.
2. Validate all three fields are present and non-empty. The `payment_method` must be one of: `bkash`, `nagad`, `rocket`, `upay` — enforce strict lowercase before sending to the API.
3. Look up the `payments` row in the database by `order_id`. Retrieve `session_id` and current `status`.
4. If `status` is already `completed`, stop immediately and do not process again (idempotency guard).
5. Load gateway config via `config.php`.
6. Call `SkyPayService::verifyTransaction($sessionId, $method, $transactionId)`.
7. If the API returns `status: true`:
   - Update the `payments` row: set `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = NOW()`
   - Fulfill the order (add wallet balance, activate subscription, mark invoice as paid, etc.)
   - Redirect the user to a success/confirmation page
8. If the API returns an error (TrxID not found, wrong method, session expired):
   - If the error indicates the SMS has not arrived yet (TrxID not found), show a retry message: "Payment is being verified. Please wait 10 seconds and try again." with a retry button that re-submits the same form.
   - If the error is definitive (session already completed, invalid method, expired session), update the payment row to `status = 'failed'` and show an appropriate error message.
   - Allow up to 2–3 minutes of retry attempts before marking as failed.
9. Never update payment status to `completed` unless the API explicitly returns `status: true`.

---

## status.php (Optional — AJAX Polling)

For a smoother user experience, this optional endpoint allows your frontend to poll for payment status using AJAX instead of requiring the user to manually click Retry.

Steps:

1. Accept `order_id` via GET or POST.
2. Look up the payment row in the database. If `status` is already `completed`, return `{"status": "completed"}`.
3. If still pending, call `SkyPayService::verifyTransaction()` with the stored `session_id`, `payment_method`, and `transaction_id` (if already submitted).
4. Return a JSON response your frontend JavaScript can act on.
5. The frontend should poll this endpoint every 10 seconds for up to 3 minutes before giving up.

---

## cancel.php

Called when the user explicitly cancels the payment from your interface.

Steps:

1. Read `order_id` from POST or GET.
2. Look up the payment row. If it is already `completed`, do not change its status.
3. If `status` is `pending`, update to `cancelled`.
4. Show a cancellation message with a link to start a new payment or return to the dashboard.

---

## Presenting Wallet Numbers — Display Rules

When rendering the wallet numbers returned from `/create`, always follow these rules:

- Show only methods where at least one `active_payments` flag is `true`
- For each active method, show only the number types (personal / agent / payment) where the flag is `true` and the value is not an empty string
- Label the number types correctly to the user: `personal` = "Send Money", `agent` = "Cash In", `payment` = "Merchant Pay" (bKash only)
- Make each wallet number easily copyable (copy-to-clipboard button or styled `<input readonly>`)
- Display the exact session amount prominently above the wallet numbers
- Remind the user to send the exact amount — partial payments will fail verification

---

## Security Rules the AI Agent Must Follow

- Store BRAND-KEY only in `.env` or a PHP constants file outside the web root. Never output it to the browser or logs visible to users.
- Always run `verifyTransaction()` before fulfilling any order. Never trust user-submitted form data alone.
- Save `session_id` to the database before rendering wallet numbers. If the session ID is lost before a verify call is made, a new session must be created.
- Check for duplicate `transaction_id` in your database before fulfilling. SkyPay also blocks reuse server-side, but your app must guard against it independently.
- Enforce lowercase method names before sending to the API. Use `strtolower()` on the user's selection.
- Use prepared statements (PDO or MySQLi) for all database queries to prevent SQL injection.
- Log all API request payloads and responses to a server-side log file for debugging. Never expose these logs to end users.
- Validate all user inputs before passing them to the SkyPay API.
- Do not expose the raw API error message to the user. Map error codes to friendly messages on your side.

---

## SMS Synchronization Latency — Retry Logic

When a user submits their TrxID immediately after paying, the telecom SMS may still be in transit to the merchant Android phone. The SkyPay Sync APK reads the SMS and pushes it to the cloud in 5 to 20 seconds.

This means the first verify call may return an error even though the payment was real. Do NOT permanently reject the user on the first failed attempt.

Implement the following retry flow:

```
User submits TrxID + selected method → POST to verify.php
        │
        ▼
Call SkyPayService::verifyTransaction()
        │
    ┌───┴────────────────┐
  status: true         Error: TrxID not found
    │                        │
    ▼                        ▼
Fulfill order         Show: "Verifying payment...
                       Please wait and try again in 10 seconds."
                             │
                             ▼
                       User clicks Retry (re-submits form)
                             │
                             ▼
                       Call verifyTransaction() again
                       (allow up to 2–3 minutes total)
                             │
                        Still failing after 3 min?
                             │
                             ▼
                       Mark as failed, contact support
```

---

## Complete Payment Flow Summary

```
User enters amount and name on your page
    → initiate.php
        → POST /api/v2/payment/create
        → Save session_id + methods_json to DB (status = pending)
        → Render wallet numbers from methods[] on your page

User selects a channel, sends money via MFS app
User gets TrxID via SMS
User selects method + enters TrxID on your page → submit

    → verify.php
        → Load session_id from DB using order_id
        → POST /api/v2/payment/verify with { id, method, transaction_id }
        → If status: true → update DB, fulfill order, redirect to success
        → If TrxID not found → show retry UI (SMS still in transit)
        → If definitive error → update DB to failed, show error
```

---

## HTTP Status Code Reference

| HTTP Code                   | Meaning                                               | What To Do                                              |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| `200 OK`                    | Request processed successfully                        | Check `"status": true` in the response body             |
| `400 Bad Request`           | Missing field, invalid TrxID, wrong method, or reuse  | Read the `"message"` field; show retry or error to user |
| `401 Unauthorized`          | Missing or invalid BRAND-KEY                          | Verify BRAND-KEY in the Dashboard                       |
| `403 Forbidden`             | No active Android device connected                    | Check that the SkyPay APK phone is online               |
| `404 Not Found`             | Session ID not found or expired                       | Call `/create` again for a new session                  |
| `405 Method Not Allowed`    | Wrong HTTP method (e.g. GET instead of POST)          | Use POST for all endpoints                              |
| `500 Internal Server Error` | Temporary cloud-side error                            | Wait briefly and retry, or contact SkyPay support       |

---

## Official Resources

| Resource                  | Link                                                      |
| ------------------------- | --------------------------------------------------------- |
| Official Website          | https://skypaybd.top                                      |
| Interactive Documentation | https://skypaybd.top/docs                                 |
| API Core Domain           | https://core.skypaybd.top                                 |
| GitHub Documentation Repo | https://github.com/SkyPayBD/Docs                          |
| Headless API v2 Full Docs | https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md |
| Hosted Gateway v1 Docs    | https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md   |
| Merchant Sync Android APK | https://skypaybd.top/public/assets/downloads/SkyPay.apk   |
| WhatsApp Support          | https://wa.me/+8801761844968                              |
| Telegram                  | https://t.me/BD_Prime_Minister                            |

---

*SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
