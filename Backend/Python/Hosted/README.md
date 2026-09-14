# SkyPay Payment Gateway — Python Backend Integration Guide for Hosted Gateway v1

## Overview

This document is a complete instruction set for an AI agent integrating the SkyPay BD payment gateway into a **Python backend** application using the **Hosted Gateway (v1)**. The integration uses the redirect-based flow where the customer is sent to SkyPay's secure hosted payment page, completes the transaction there, and is returned to your application afterward.

This guide covers Python backends in general — whether you are using **Flask**, **Django**, **FastAPI**, or any other Python web framework. All code patterns shown use the standard `requests` library for HTTP calls and are framework-agnostic. Framework-specific adaptations are noted where relevant.

Always read the main SkyPay Hosted Gateway documentation first to understand the full API structure, response formats, callback parameters, and all available fields before writing any code.

> **Related Documentation**
> - Full Hosted Gateway v1 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md)
> - Full Headless API v2 Reference: [https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md](https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md)
> - PHP Hosted Integration Guide: [https://github.com/SkyPayBD/Docs/blob/main/Backend/PHP/Hosted/README.md](https://github.com/SkyPayBD/Docs/blob/main/Backend/PHP/Hosted/README.md)
> - Interactive Documentation: [https://skypaybd.top/docs](https://skypaybd.top/docs)

---

## Which API to Use

Use the **Hosted Gateway (v1)**:

- Create URL: `POST https://core.skypaybd.top/api/payment/create`
- Verify URL: `POST https://core.skypaybd.top/api/payment/verify`

Do NOT use the Headless v2 API for standard web applications where a browser redirect is possible. The Headless API is for Telegram bots, mobile apps, and custom panels where a redirect is not possible or not desired.

---

## Authentication

Every request to the SkyPay API requires these HTTP headers:

```
BRAND-KEY: <merchant_brand_key>
Content-Type: application/json
```

There is no SECRET-KEY. Only BRAND-KEY is required. The BRAND-KEY must be stored in environment variables (`.env` file via `python-dotenv`, or system environment) — never hardcoded in source code or committed to any repository.

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

- `requests` — HTTP client for all SkyPay API calls
- `python-dotenv` — loads environment variables from `.env`
- `sqlalchemy` — ORM for database interaction (or use raw DB drivers if preferred)

---

## Database Setup

### Table: `auto_payment_gateway`

Create this table to store gateway configuration. This allows the merchant to update API endpoints or toggle the gateway from an admin interface without touching code.

| Column       | Type         | Nullable | Description                                              |
| ------------ | ------------ | -------- | -------------------------------------------------------- |
| `id`         | INTEGER PK   | No       | Primary key, auto-increment                              |
| `brand_key`  | VARCHAR(255) | No       | The merchant BRAND-KEY from SkyPay dashboard             |
| `create_url` | VARCHAR(255) | No       | Full URL: `https://core.skypaybd.top/api/payment/create` |
| `verify_url` | VARCHAR(255) | No       | Full URL: `https://core.skypaybd.top/api/payment/verify` |
| `status`     | VARCHAR(20)  | No       | `active` or `inactive`                                   |
| `created_at` | DATETIME     | No       | Set on insert                                            |
| `updated_at` | DATETIME     | Yes      | Set on update                                            |

Insert a default row during setup with `status = 'active'`, the two endpoint URLs, and the merchant's BRAND-KEY.

### Table: `payments`

Track every payment attempt and its verification result.

| Column           | Type         | Nullable | Description                                      |
| ---------------- | ------------ | -------- | ------------------------------------------------ |
| `id`             | INTEGER PK   | No       | Primary key, auto-increment                      |
| `order_id`       | VARCHAR(100) | No       | Your internal order or invoice reference         |
| `transaction_id` | VARCHAR(100) | Yes      | TrxID returned by SkyPay after payment           |
| `cus_name`       | VARCHAR(255) | No       | Customer name sent to SkyPay                     |
| `cus_email`      | VARCHAR(255) | No       | Customer email sent to SkyPay                    |
| `amount`         | NUMERIC      | No       | Payment amount in BDT                            |
| `payment_method` | VARCHAR(50)  | Yes      | `bkash` / `nagad` / `rocket` / `upay`            |
| `status`         | VARCHAR(20)  | No       | `pending`, `completed`, `failed`, or `cancelled` |
| `metadata`       | JSON / TEXT  | Yes      | Any extra data (cart ID, user ID, plan, etc.)    |
| `verified_at`    | DATETIME     | Yes      | Timestamp of successful backend verification     |
| `created_at`     | DATETIME     | No       | When the payment session was initiated           |

---

## File/Folder Architecture

The AI agent should create the following file structure inside the project:

```
/payment/
    __init__.py
    config.py           ← Reads gateway config from DB or environment
    skypay_service.py   ← All API communication logic
    views.py            ← Route handlers: initiate, callback, cancel
    models.py           ← SQLAlchemy models for gateway and payments tables
    utils.py            ← Helper functions (order ID generation, logging, etc.)
```

---

## config.py

This module loads the active gateway configuration. It queries the `auto_payment_gateway` table for the row with `status = 'active'` and returns a dictionary with `brand_key`, `create_url`, and `verify_url`. If no active gateway is found, it raises a `RuntimeError` with a descriptive message. Do not log or expose the `brand_key` value in any output visible to users.

---

## skypay_service.py

This is the core service module. It contains all functions that communicate with the SkyPay Hosted Gateway v1 API.

### Function: `create_payment(gateway: dict, data: dict) -> dict`

Accepts:

- `gateway` — the active gateway config dictionary (from `config.py`)
- `data` — a dictionary with the following keys:
  - `cus_name` (str, required) — customer's full name
  - `cus_email` (str, required) — customer's valid email address
  - `amount` (float or int, required) — payable amount in BDT, must be greater than 0
  - `success_url` (str, required) — the full public URL of your callback route
  - `cancel_url` (str, required) — the full public URL of your cancel route
  - `metadata` (dict, optional) — pass `order_id`, `user_id`, or any internal reference

Builds the request headers:

```python
headers = {
    "BRAND-KEY": gateway["brand_key"],
    "Content-Type": "application/json"
}
```

Sends a POST request using `requests.post()` to `gateway["create_url"]` with `headers=headers` and `json=data`. Sets `timeout=30`.

Returns the parsed JSON response as a dictionary. On success, the response includes:

- `status` — `True` if the session was created
- `message` — human-readable message
- `payment_url` — the SkyPay-hosted checkout URL to redirect the customer to

Raises an exception or returns an error dict on HTTP errors or connection failures.

### Function: `verify_payment(gateway: dict, transaction_id: str) -> dict`

Accepts:

- `gateway` — the active gateway config dictionary
- `transaction_id` — the TrxID from the callback URL's `transactionId` query parameter

Sends a POST request to `gateway["verify_url"]` with:

```python
headers = {
    "BRAND-KEY": gateway["brand_key"],
    "Content-Type": "application/json"
}
payload = {"transaction_id": transaction_id}
```

Returns the parsed JSON response. On success:

- `status` — `"COMPLETED"` if the payment is verified
- `cus_name` — customer name
- `cus_email` — customer email
- `amount` — verified amount in BDT
- `transaction_id` — the verified TrxID
- `payment_method` — channel used
- `metadata` — the metadata dict you passed during `/create`

**Always check that `status == "COMPLETED"` before fulfilling any order. A `"PENDING"` or `"ERROR"` status means do not fulfill.**

### HTTP Implementation Notes

- Always use `timeout=30` in `requests.post()` calls
- Wrap calls in `try/except requests.RequestException` to handle network failures gracefully
- Log all request payloads and responses to a server-side log file using Python's `logging` module
- Never print sensitive data (BRAND-KEY, customer email) to stdout or error pages

---

## views.py

This module contains the route handler functions. Adapt the function signatures to your framework (Flask uses `@app.route`, Django uses URL patterns, FastAPI uses `@router.post`).

### Handler: `initiate_payment(request)`

Called when the user submits the payment form or clicks "Pay Now".

Steps the AI agent must implement:

1. Extract `cus_name`, `cus_email`, `amount`, and `order_id` from the request (POST body or form data).
2. Validate all required fields. Reject if `amount` is not a positive number or if required strings are empty.
3. Load gateway config via `config.py`. If the gateway is inactive, return an error response.
4. Insert a new row into the `payments` table with `status = 'pending'` and save the record.
5. Build `success_url` and `cancel_url` as full public URLs pointing to your callback and cancel handlers. Include `order_id` as a query parameter so the callback can identify the order.
6. Call `skypay_service.create_payment()` with all required fields. Include `order_id` inside `metadata`.
7. If `response["status"]` is `True`, redirect the user's browser to `response["payment_url"]`.
8. If the response indicates an error, return a friendly error page or JSON error. Do not expose raw API error details.

### Handler: `payment_callback(request)`

SkyPay redirects the user to this handler after a successful payment. The URL will contain these query parameters:

- `transactionId`
- `paymentMethod`
- `paymentAmount`
- `paymentFee`
- `status`

**Critical security rule:** Never fulfill an order based solely on these query parameters. A malicious user can craft any URL with `status=completed` and a fake `transactionId`. Always verify server-side.

Steps the AI agent must implement:

1. Read `transactionId` from the query parameters.
2. Look up the `payments` table. If a row with this `transaction_id` already has `status = 'completed'`, stop and return without processing again (idempotency guard).
3. Call `skypay_service.verify_payment(gateway, transaction_id)`.
4. If `response["status"] == "COMPLETED"`:
   - Update the matching `payments` row: set `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = datetime.utcnow()`
   - Retrieve `order_id` from `response["metadata"]`
   - Fulfill the order (activate plan, add balance, mark invoice paid, send confirmation email, etc.)
   - Redirect the user to a success/thank-you page
5. If verification fails or status is not `COMPLETED`:
   - Update the payment row with `status = 'failed'`
   - Return a user-friendly error page

### Handler: `payment_cancel(request)`

Called when the user cancels on the SkyPay page.

Steps:

1. Read `order_id` from query parameters if available.
2. Look up the `payments` row. If `status` is `pending`, update it to `cancelled`.
3. Return a cancellation page with a message and a link to try again or return to the previous page.

---

## Security Rules the AI Agent Must Follow

- Store BRAND-KEY only in environment variables. Use `python-dotenv` to load from a `.env` file that is listed in `.gitignore`. Never hardcode it in source files.
- Always run `verify_payment()` before fulfilling any order. Never trust callback query parameters alone.
- Check for duplicate `transaction_id` in your database before fulfilling. SkyPay also blocks reuse on its side, but your application must guard against it independently.
- Use parameterized queries or your ORM's safe query methods for all database operations to prevent SQL injection.
- Log all API request and response data to a server-side log file. Do not expose these logs to end users.
- Validate all user inputs before sending them to the SkyPay API. Use Python's `isinstance()` checks and string `.strip()` to sanitize inputs.
- Run your verify call from the server process, never from a browser-side AJAX call that could be intercepted or replicated.

---

## Complete Payment Flow Summary

```
User clicks "Pay Now" on your web page
    → initiate_payment() handler
        → Validate inputs
        → INSERT pending payment row in DB
        → POST /api/payment/create  →  returns payment_url
        → HTTP 302 redirect to payment_url

User lands on SkyPay hosted page
User selects bKash / Nagad / Rocket / Upay
User sends money, enters TrxID on SkyPay page
SkyPay verifies TrxID (5–20 seconds)
SkyPay redirects user back to your success_url:

    → payment_callback() handler
        ?transactionId=BLA38KDK2M&paymentMethod=bkash&status=completed

        → Check DB: is this transactionId already completed? If yes, stop.
        → POST /api/payment/verify  →  returns { "status": "COMPLETED", ... }
        → If COMPLETED: UPDATE payment row, fulfill order, redirect to success page
        → If not COMPLETED: UPDATE to failed, show error

User cancels on SkyPay page
    → payment_cancel() handler
        → UPDATE payment row to cancelled
        → Show cancellation page
```

---

## HTTP Status Code Reference

| HTTP Code                   | Meaning                                               | What To Do                                              |
| --------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| `200 OK`                    | Request processed successfully                        | Check `"status"` value in the response body             |
| `400 Bad Request`           | Missing parameter or invalid data                     | Read the `"message"` field for details                  |
| `401 Unauthorized`          | Missing or invalid BRAND-KEY                          | Verify your key in the Dashboard                        |
| `403 Forbidden`             | No active Android device connected                    | Check that the SkyPay APK phone is online               |
| `404 Not Found`             | Endpoint not found                                    | Ensure you are using the correct URL and HTTP method    |
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
| Hosted Gateway v1 Docs    | https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md   |
| Headless API v2 Docs      | https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md |
| Merchant Sync Android APK | https://skypaybd.top/public/assets/downloads/SkyPay.apk   |
| WhatsApp Support          | https://wa.me/+8801761844968                              |
| Telegram                  | https://t.me/BD_Prime_Minister                            |

---

*SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
