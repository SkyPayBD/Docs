# SkyPay Payment Gateway — PHP (Vanilla) Integration Guide for AI Agents

## Overview

This document is a complete instruction set for an AI agent integrating the SkyPay BD payment gateway into a **vanilla PHP** web application (no framework). The integration uses the **Hosted Gateway (v1)** because PHP websites serve pages in a browser, and the customer can be redirected to SkyPay's hosted checkout page and returned afterward.

Always read the main SkyPay `README.md` API documentation first to understand the full API structure, response formats, and all available fields before writing any code.

---

## Which API to Use

Use the **Hosted Gateway (v1)**:
- Create URL: `POST https://core.skypaybd.top/api/payment/create`
- Verify URL: `POST https://core.skypaybd.top/api/payment/verify`

Do NOT use the Headless v2 API for standard PHP websites. The Headless API is for mobile apps and bots where redirect is impossible.

---

## Authentication

Every request to the SkyPay API requires one HTTP header:

```
BRAND-KEY: <merchant_brand_key>
Content-Type: application/json
```

There is no SECRET-KEY. Only BRAND-KEY is required. The BRAND-KEY must be stored server-side in a `.env` file or PHP config constant — never in frontend HTML or JavaScript.

---

## Database Setup

### Table: `auto_payment_gateway`

Create this MySQL table to store gateway configuration. This allows the merchant to manage the gateway from an admin panel without touching code.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | INT UNSIGNED AUTO_INCREMENT | No | Primary key |
| `brand_key` | VARCHAR(255) | No | The merchant BRAND-KEY from SkyPay dashboard |
| `create_url` | VARCHAR(255) | No | Full URL: `https://core.skypaybd.top/api/payment/create` |
| `verify_url` | VARCHAR(255) | No | Full URL: `https://core.skypaybd.top/api/payment/verify` |
| `status` | ENUM('active', 'inactive') | No | Whether this gateway is currently enabled |
| `created_at` | TIMESTAMP | No | Auto-set on insert |
| `updated_at` | TIMESTAMP | Yes | Auto-set on update |

Insert a default row during setup with `status = 'active'`, the two endpoint URLs, and the merchant's BRAND-KEY.

### Table: `payments`

Track every payment attempt and its verification result.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | INT UNSIGNED AUTO_INCREMENT | No | Primary key |
| `order_id` | VARCHAR(100) | No | Your internal order/invoice reference |
| `transaction_id` | VARCHAR(100) | Yes | TrxID returned by SkyPay after payment |
| `cus_name` | VARCHAR(255) | No | Customer name sent to SkyPay |
| `cus_email` | VARCHAR(255) | No | Customer email sent to SkyPay |
| `amount` | DECIMAL(10,2) | No | Payment amount in BDT |
| `payment_method` | VARCHAR(50) | Yes | bkash / nagad / rocket / upay |
| `status` | ENUM('pending','completed','failed','cancelled') | No | Payment lifecycle state |
| `metadata` | JSON | Yes | Any extra data (cart ID, user ID, etc.) |
| `verified_at` | TIMESTAMP | Yes | Timestamp of successful backend verification |
| `created_at` | TIMESTAMP | No | When the payment session was initiated |

---

## File/Folder Architecture

The AI agent should create the following file structure inside the project:

```
/payment/
    config.php          ← Reads gateway config from DB
    SkyPayService.php   ← All API communication logic
    initiate.php        ← Entry point when user clicks "Pay"
    callback.php        ← SkyPay redirects user here after payment
    cancel.php          ← SkyPay redirects user here on cancel
```

---

## config.php

This file connects to the database and fetches the active gateway row. It exposes `$gateway` as an object/array with `brand_key`, `create_url`, and `verify_url`. If no active gateway is found, it stops execution with an error.

---

## SkyPayService.php

This is the core service class. It holds all SkyPay API communication. Instantiate it by passing the active gateway config.

### Method: `createPayment(array $data): array`

Accepts:
- `cus_name` (string, required)
- `cus_email` (string, required)
- `amount` (numeric, required)
- `success_url` (string, required) — full URL to your `callback.php`
- `cancel_url` (string, required) — full URL to your `cancel.php`
- `metadata` (array, optional) — pass `order_id`, `user_id`, etc.

Sends a POST request to `create_url` with `BRAND-KEY` header and JSON body. Returns the decoded JSON response. On success, the response includes `payment_url` which the user must be redirected to.

### Method: `verifyPayment(string $transactionId): array`

Accepts:
- `transaction_id` (string) — the TrxID from the callback URL query string

Sends a POST request to `verify_url` with `BRAND-KEY` header and JSON body `{ "transaction_id": "..." }`. Returns decoded JSON. On success, `status` equals `"COMPLETED"` and the response includes `cus_name`, `cus_email`, `amount`, `payment_method`, and `metadata`.

### HTTP Implementation

Use PHP's `curl` or `file_get_contents` with a stream context. Always:
- Set `CURLOPT_TIMEOUT` to 30 seconds
- Send `Content-Type: application/json` and `BRAND-KEY` headers
- Decode the response with `json_decode($response, true)`
- Handle curl errors and log them

---

## initiate.php

This is called when the user submits the checkout form or clicks "Pay Now".

Steps the AI agent must implement here:
1. Validate required inputs: `cus_name`, `cus_email`, `amount`, and the internal `order_id`
2. Load gateway config via `config.php`
3. Insert a new row into the `payments` table with `status = 'pending'` and save the row's ID
4. Build the `success_url` and `cancel_url` including the `order_id` as a query parameter so the callback can identify which order this is
5. Call `SkyPayService::createPayment()` with all required fields. Pass `order_id` inside `metadata`
6. If the API returns `status: true`, perform an HTTP 302 redirect to the returned `payment_url`
7. If the API returns an error, show the user a friendly error message and do NOT redirect

---

## callback.php

SkyPay redirects the user here after a successful payment. The URL will contain these GET parameters:
- `transactionId`
- `paymentMethod`
- `paymentAmount`
- `paymentFee`
- `status`

**Critical security rule:** Never trust these GET parameters to fulfill an order. A malicious user can manually craft this URL with `status=completed` and any `transactionId`. You must verify server-side.

Steps the AI agent must implement here:
1. Read `transactionId` from `$_GET`
2. Check the `payments` table: if a row with this `transaction_id` already has `status = 'completed'`, stop and do not process again (idempotency guard)
3. Call `SkyPayService::verifyPayment($transactionId)`
4. If the response `status === 'COMPLETED'`:
   - Update the matching `payments` row: set `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = NOW()`
   - Retrieve `order_id` from the response `metadata` field
   - Fulfill the order (activate account, mark order as paid, send confirmation email, etc.)
   - Redirect user to a thank-you/success page
5. If verification fails or status is not COMPLETED:
   - Update the payment row with `status = 'failed'`
   - Show an appropriate error message to the user

---

## cancel.php

Called when the user cancels on the SkyPay page.

Steps:
1. Optionally read `order_id` from the query string
2. Update the matching payment row to `status = 'cancelled'`
3. Show a cancellation message and a link to try again or return to cart

---

## Security Rules the AI Agent Must Follow

- Store BRAND-KEY only in `.env` or a PHP constants file outside the web root. Never output it to the browser.
- Always run `verifyPayment()` before fulfilling any order. Never rely on GET params alone.
- Check for duplicate `transaction_id` in your database before fulfilling. SkyPay blocks reuse on its side too, but your app must also guard against it.
- Use prepared statements (PDO or MySQLi) for all database queries to prevent SQL injection.
- Log all API request payloads and responses to a server-side log file for debugging.
- Validate all user inputs before sending them to the SkyPay API.

---

## Complete Payment Flow Summary

```
User clicks "Pay Now"
    → initiate.php
        → INSERT pending payment row in DB
        → POST /api/payment/create
        → Redirect to payment_url

User completes payment on SkyPay
    → SkyPay redirects to callback.php?transactionId=...&status=completed

callback.php
    → Check DB: is this transactionId already completed? If yes, stop.
    → POST /api/payment/verify with transactionId
    → If COMPLETED: UPDATE payment row, fulfill order, redirect to success page
    → If failed: UPDATE payment row, show error
```
