# SkyPay Payment Gateway — Python Backend Integration Guide for Headless API v2

## Overview

This document is a complete instruction set for an AI agent integrating the SkyPay BD payment gateway into a **Python backend** application using the **Headless API (v2)**. The Headless API does not redirect the user to any external page. Your Python backend acts as a bridge — it creates a payment session with SkyPay, fetches live merchant wallet numbers, presents them through your own interface, collects the user's Transaction ID (TrxID), and verifies the payment entirely within your application.

This approach is ideal for **Telegram bots**, **Discord bots**, **FastAPI-powered SPAs**, **custom payment dashboards**, or any Python-based platform where you need full control over the payment interface and the user must never be redirected elsewhere.

All code patterns use the standard `requests` library and are framework-agnostic. Adaptations for Flask, Django, FastAPI, or async Python (`httpx`, `aiohttp`) are noted where relevant.

Always read the main SkyPay Headless API documentation first to understand the full API structure, session lifecycle, `methods[]` response format, and all available fields before writing any code.

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

Do NOT use the Hosted Gateway v1 for this integration. The Headless API is specifically for scenarios where you build and control your own payment UI and the user stays inside your application or bot.

---

## Authentication

Every request to the SkyPay API requires these HTTP headers:

```
BRAND-KEY: <merchant_brand_key>
Content-Type: application/json
```

There is no SECRET-KEY. Only BRAND-KEY is required. Store it in environment variables loaded via `python-dotenv` or your deployment environment's secret manager. Never hardcode it in source files or commit it to any repository.

Load it with:

```python
import os
BRAND_KEY = os.environ.get("SKYPAY_BRAND_KEY")
```

---

## Dependencies

Install the following Python packages:

```
pip install requests python-dotenv sqlalchemy
```

For async frameworks (FastAPI, async Telegram bots):

```
pip install httpx python-dotenv sqlalchemy
```

- `requests` / `httpx` — HTTP client for all SkyPay API calls
- `python-dotenv` — loads environment variables from `.env`
- `sqlalchemy` — ORM for database interaction

---

## How the Headless Flow Works in Python

Because the Headless API requires the session `id` from `/create` to be sent back in the `/verify` call, your application must persist this ID between the two operations. The exact storage mechanism depends on your platform:

- **Web apps (Flask/Django/FastAPI):** Store `session_id` in the database tied to the order row. Include `order_id` as a hidden field or route parameter so your verify handler can look it up.
- **Telegram/Discord bots:** Store `session_id` in a Redis key, in-memory dict, or database row keyed by the user's chat/user ID. Retrieve it when the user replies with their TrxID.
- **Long-running async services:** Use an async-safe store (Redis, Postgres) since in-memory dicts are not safe across restarts.

If the `session_id` is lost before verification, the session cannot be recovered. A new session must be created with `/create`.

---

## Database Setup

### Table: `auto_payment_gateway`

Create this table to store gateway configuration. This allows switching API endpoints or toggling the gateway from an admin panel without code changes.

| Column       | Type         | Nullable | Description                                                      |
| ------------ | ------------ | -------- | ---------------------------------------------------------------- |
| `id`         | INTEGER PK   | No       | Primary key, auto-increment                                      |
| `brand_key`  | VARCHAR(255) | No       | The merchant BRAND-KEY from SkyPay dashboard                     |
| `create_url` | VARCHAR(255) | No       | Full URL: `https://core.skypaybd.top/api/v2/payment/create`      |
| `verify_url` | VARCHAR(255) | No       | Full URL: `https://core.skypaybd.top/api/v2/payment/verify`      |
| `status`     | VARCHAR(20)  | No       | `active` or `inactive`                                           |
| `created_at` | DATETIME     | No       | Set on insert                                                    |
| `updated_at` | DATETIME     | Yes      | Set on update                                                    |

Insert a default row during setup with `status = 'active'`, the two v2 endpoint URLs, and the merchant's BRAND-KEY.

### Table: `payments`

Track every payment session, the wallet numbers shown, and the final verification outcome.

| Column           | Type         | Nullable | Description                                                       |
| ---------------- | ------------ | -------- | ----------------------------------------------------------------- |
| `id`             | INTEGER PK   | No       | Primary key, auto-increment                                       |
| `order_id`       | VARCHAR(100) | No       | Your internal order or invoice reference                          |
| `session_id`     | VARCHAR(100) | No       | The `id` returned by SkyPay `/create` — required for verify       |
| `transaction_id` | VARCHAR(100) | Yes      | TrxID submitted by the user after payment                         |
| `cus_name`       | VARCHAR(255) | No       | Customer name sent to SkyPay                                      |
| `amount`         | NUMERIC      | No       | Payment amount in BDT                                             |
| `payment_method` | VARCHAR(50)  | Yes      | `bkash` / `nagad` / `rocket` / `upay` (submitted by user)        |
| `methods_json`   | JSON / TEXT  | Yes      | Full `methods[]` array from `/create` stored for audit/display    |
| `status`         | VARCHAR(20)  | No       | `pending`, `completed`, `failed`, or `cancelled`                  |
| `meta_data`      | JSON / TEXT  | Yes      | Any extra data (user ID, plan, Telegram chat ID, reference, etc.) |
| `verified_at`    | DATETIME     | Yes      | Timestamp of successful backend verification                      |
| `created_at`     | DATETIME     | No       | When the payment session was initiated                            |

> **Critical:** Save `session_id` to the database or your persistent store immediately after the `/create` call returns, before displaying wallet numbers to the user. If this step is skipped and the process restarts, verification will be impossible.

---

## File/Folder Architecture

The AI agent should create the following file structure inside the project:

```
/payment/
    __init__.py
    config.py             ← Reads gateway config from DB or environment
    skypay_service.py     ← All API communication logic (v2)
    handlers.py           ← Route or bot command handlers: initiate, verify, cancel
    models.py             ← ORM models for gateway and payments tables
    utils.py              ← Helpers: order ID generation, method validation, logging
```

---

## config.py

This module loads the active gateway configuration. It queries the `auto_payment_gateway` table for the row with `status = 'active'` and returns a dictionary with `brand_key`, `create_url`, and `verify_url`. If no active gateway is found, it raises a `RuntimeError`. The `brand_key` value must never be logged or exposed in any user-facing output.

---

## skypay_service.py

This is the core service module. It contains all functions that communicate with the SkyPay Headless API v2.

### Function: `create_session(gateway: dict, data: dict) -> dict`

Accepts:

- `gateway` — the active gateway config dictionary (from `config.py`)
- `data` — a dictionary with the following keys:
  - `cus_name` (str, required) — customer's full name or username
  - `amount` (float or int, required) — amount in BDT, must be greater than 0
  - `meta_data` (dict, optional) — pass `order_id`, `user_id`, `telegram_id`, `plan`, or any reference data you need returned at verify time

Builds request headers:

```python
headers = {
    "BRAND-KEY": gateway["brand_key"],
    "Content-Type": "application/json"
}
```

Sends a POST request using `requests.post()` to `gateway["create_url"]` with `headers=headers`, `json=data`, and `timeout=30`.

Returns the full parsed JSON response as a dictionary. On success, the response contains:

- `status` — `True` if session was created
- `id` — the unique session ID **(save this to the database immediately)**
- `brand` — merchant brand info object (name, mobile, whatsapp, email)
- `methods` — list of active payment channel objects

Each object in `methods` contains:

- `name` — channel name (`bkash`, `nagad`, `rocket`, `upay`)
- `active_payments` — dict with boolean flags: `personal`, `agent`, `payment`
- `personal` — phone number for Send Money (use only if `active_payments["personal"]` is `True`)
- `agent` — phone number for Cash In (use only if `active_payments["agent"]` is `True`)
- `payment` — merchant payment number, bKash only (use only if `active_payments["payment"]` is `True`)

**Only present wallet numbers where the corresponding `active_payments` flag is `True` and the number string is not empty.**

### Function: `verify_transaction(gateway: dict, session_id: str, method: str, transaction_id: str) -> dict`

Accepts:

- `gateway` — the active gateway config dictionary
- `session_id` — the `id` from the `/create` response, retrieved from your database or session store
- `method` — the payment channel the user selected, in **strict lowercase** (`bkash`, `nagad`, `rocket`, or `upay`)
- `transaction_id` — the TrxID from the user's payment SMS

Sanitize `method` before calling by applying `.lower().strip()` to whatever the user submitted.

Sends a POST request to `gateway["verify_url"]` with:

```python
headers = {
    "BRAND-KEY": gateway["brand_key"],
    "Content-Type": "application/json"
}
payload = {
    "id": session_id,
    "method": method,
    "transaction_id": transaction_id
}
```

Returns the parsed JSON response. On success:

- `status` — `True` if payment is verified
- `amount` — verified payment amount in BDT
- `cus_name` — customer name from the session
- `id` — the verified session ID

**If any of `id`, `method`, or `transaction_id` is missing, incorrect, or `method` is not lowercase — the API returns a 400 error and verification fails. There are no exceptions to this rule.**

### HTTP Implementation Notes

- Always use `timeout=30` in all `requests.post()` calls
- Wrap API calls in `try/except requests.RequestException as e` to handle timeouts and connection errors
- For async frameworks, use `httpx.AsyncClient` with `await client.post(...)` and the same headers and payload structure
- Use Python's `logging` module to log all request payloads and responses at DEBUG level to a server-side log file
- Never log the `brand_key` value or customer personally identifiable information to shared or public log outputs

---

## handlers.py

This module contains the handler functions for your routes or bot commands. Adapt signatures to your framework.

### Handler: `initiate_payment(cus_name, amount, user_reference)`

Called when the user triggers a payment (submits a form, sends `/deposit 500` to a bot, clicks "Add Funds", etc.).

Steps the AI agent must implement:

1. Validate `cus_name` is a non-empty string and `amount` is a positive number. Return an error if either is invalid.
2. Load gateway config via `config.py`. Return an error if gateway is inactive.
3. Build the `meta_data` dict with your internal references: `order_id`, `user_id`, `telegram_id`, or any identifier needed at verify time.
4. Call `skypay_service.create_session(gateway, data)`.
5. If `response["status"]` is `True`:
   - Generate a unique `order_id` (e.g. `"ORD-" + uuid.uuid4().hex[:10].upper()`)
   - Save to the `payments` table: `status = 'pending'`, `session_id = response["id"]`, `methods_json = json.dumps(response["methods"])`
   - Extract only the active wallet numbers from `response["methods"]` — skip any method or number type where `active_payments` flag is `False` or the number string is empty
   - Present these wallet numbers to the user inside your interface (formatted message, UI card, modal, etc.)
   - Display the exact `amount` the user must send — do not modify or round it
   - Ask the user to select which method they used and submit their TrxID
   - Store `order_id` in your bot context, session, or URL so the verify handler can retrieve `session_id` later
6. If the API returns an error, present a friendly error message. Do not expose raw API errors.

### Handler: `verify_payment(order_id, method, transaction_id)`

Called when the user submits their TrxID (replies to a bot message, submits a form, hits an API endpoint).

Steps the AI agent must implement:

1. Validate that `order_id`, `method`, and `transaction_id` are all present and non-empty strings.
2. Sanitize `method` with `.lower().strip()`. Confirm it is one of: `bkash`, `nagad`, `rocket`, `upay`. Reject with a clear error if not.
3. Look up the `payments` row in the database by `order_id`. Retrieve `session_id` and current `status`.
4. If `status` is already `completed`, stop immediately — do not process again (idempotency guard). Inform the user the payment was already confirmed.
5. Load gateway config via `config.py`.
6. Call `skypay_service.verify_transaction(gateway, session_id, method, transaction_id)`.
7. If `response["status"]` is `True`:
   - Update the `payments` row: set `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = datetime.utcnow()`
   - Execute your fulfillment logic (credit wallet balance, activate subscription, unlock feature, send confirmation message, etc.)
   - Notify the user of success
8. If the API returns an error:
   - If the error indicates the TrxID was not found (SMS still in transit), show a retry prompt: *"Payment is being matched. Please wait 10 seconds and try again."*
   - If the error is definitive (session already completed, invalid method, session expired, TrxID already used), update `status = 'failed'` and inform the user clearly.
   - Allow retry attempts for up to 2–3 minutes before permanently marking as failed.
9. Never update `status` to `completed` unless `response["status"]` is explicitly `True`.

### Handler: `cancel_payment(order_id)`

Called when the user explicitly cancels the payment.

Steps:

1. Look up the `payments` row by `order_id`.
2. If `status` is `pending`, update it to `cancelled`.
3. If `status` is already `completed`, do not change it.
4. Inform the user the payment has been cancelled and provide an option to start a new payment.

---

## Presenting Wallet Numbers — Display Rules

When presenting the `methods[]` data to your users, always follow these rules regardless of your platform:

- Iterate through each item in `methods[]` from the API response
- For each method, only show number types where the `active_payments` flag is `True` AND the number string is not empty
- Label number types correctly: `personal` = "Send Money", `agent` = "Cash In", `payment` = "Merchant Pay" (bKash only)
- Display the exact session amount prominently — the user must send this exact figure
- Make each wallet number easy to copy (monospace font, copy button, or formatted for mobile)
- Remind the user to note their TrxID from the payment confirmation SMS
- For bots: send the wallet numbers as a formatted message and ask the user to reply with `<method> <TrxID>` or use inline buttons to select the method first

---

## Security Rules the AI Agent Must Follow

- Store BRAND-KEY only in environment variables. Add `.env` to `.gitignore`. Never hardcode it in any source file.
- Always call `verify_transaction()` before fulfilling any order. Never trust user-supplied data alone.
- Save `session_id` to persistent storage immediately after `/create`. If lost before `/verify`, the session is unrecoverable.
- Apply `.lower().strip()` to the `method` value before sending it to the API. Wrong casing causes a guaranteed 400 error.
- Check for duplicate `transaction_id` in your database before fulfilling. SkyPay blocks reuse on its side too, but your application must guard independently.
- Use parameterized queries or your ORM's query interface for all database operations to prevent SQL injection.
- Log all API interactions server-side using Python's `logging` module. Do not expose these logs to end users.
- Never expose the `brand_key` in HTTP responses, bot messages, or log outputs visible to users.
- Run all API calls from your backend process. Never call SkyPay APIs from client-side JavaScript or mobile app code directly.

---

## SMS Synchronization Latency — Retry Logic

When a user submits their TrxID immediately after paying, the SMS from the telecom network may still be in transit to the merchant Android phone. The SkyPay Sync APK reads it and pushes it to the cloud in 5 to 20 seconds.

This means the first verify call may return a "TrxID not found" error even though the payment was legitimate. Do NOT permanently reject the user on the first failed attempt.

Recommended retry flow for all platform types:

```
User submits TrxID + method
        │
        ▼
Call skypay_service.verify_transaction()
        │
    ┌───┴────────────────────┐
  status: True             Error: TrxID not found
    │                           │
    ▼                           ▼
Fulfill order            Inform user:
                         "Payment matching in progress.
                          Please wait 10 seconds and try again."
                              │
                              ▼
                         User retries (re-submits TrxID)
                              │
                              ▼
                         Call verify_transaction() again
                         (allow up to 2–3 minutes total)
                              │
                         Still failing after 3 min?
                              │
                              ▼
                         Mark as failed, suggest contacting support
```

For bots, implement an automatic retry loop with a 10-second `asyncio.sleep()` delay between attempts rather than making the user manually retry.

---

## Complete Payment Flow Summary

```
User initiates payment (form submit, bot command, API call)
    → initiate_payment()
        → POST /api/v2/payment/create
        → Save session_id + methods_json to DB (status = pending)
        → Present active wallet numbers to user inside your interface

User selects channel, sends money via MFS app
User receives TrxID via SMS
User submits method + TrxID back to your app

    → verify_payment()
        → Load session_id from DB using order_id
        → Sanitize method: method = method.lower().strip()
        → POST /api/v2/payment/verify with { id, method, transaction_id }
        → If status: True → UPDATE DB, fulfill order, notify user
        → If TrxID not found → show retry prompt (SMS still in transit)
        → If definitive error → UPDATE DB to failed, inform user

User cancels
    → cancel_payment()
        → UPDATE DB to cancelled
        → Inform user, offer to restart
```

---

## HTTP Status Code Reference

| HTTP Code                   | Meaning                                               | What To Do                                              |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| `200 OK`                    | Request processed successfully                        | Check `"status": True` in the response body             |
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
