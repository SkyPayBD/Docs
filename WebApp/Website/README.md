# SkyPay Payment Gateway — Website Integration Guide for AI Agents

## Overview

This document provides complete instructions for integrating SkyPay BD into a **standard website**. A website runs in a browser, so the customer can be redirected to SkyPay's hosted checkout page, complete the payment there, and be sent back to the merchant's site automatically.

This guide covers any website stack: custom PHP, HTML+JS with a PHP backend, WordPress WooCommerce, WHMCS, OpenCart, PrestaShop, or any other browser-based platform.

Website integrations use the **Hosted Gateway (v1)**. Do not use Headless API v2 for websites unless you are building a fully custom checkout modal within a Single-Page Application (SPA) and want zero redirects — that is an advanced scenario covered at the end of this document.

Always read the main SkyPay `README.md` API documentation first to understand the full response formats and all available fields.

---

## Which API to Use

**Hosted Gateway (v1):**
- Create payment URL: `POST https://core.skypaybd.top/api/payment/create`
- Verify payment: `POST https://core.skypaybd.top/api/payment/verify`

SkyPay handles the entire checkout UI — the customer selects their wallet, sends money, enters the TrxID, and SkyPay verifies via SMS. Your website only needs to create the session, redirect the user, and verify the result on the callback.

---

## Authentication

Every request must include:
```
BRAND-KEY: <your_brand_key>
Content-Type: application/json
Accept: application/json
```

The BRAND-KEY must be stored server-side only. Acceptable storage methods:
- PHP `.env` file (e.g., using `vlucas/phpdotenv`)
- PHP `config.php` constant file located outside the web root
- Server environment variables set in Apache VirtualHost or Nginx config

Never put the BRAND-KEY in any JavaScript file, HTML template, or any file that is publicly accessible. Never commit it to a Git repository.

---

## Database Setup

### Table: `auto_payment_gateway`

This table stores the gateway configuration and allows enabling/disabling the gateway from an admin panel without touching code.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | INT UNSIGNED AUTO_INCREMENT | No | Primary key |
| `brand_key` | VARCHAR(255) | No | SkyPay BRAND-KEY |
| `create_url` | VARCHAR(255) | No | `https://core.skypaybd.top/api/payment/create` |
| `verify_url` | VARCHAR(255) | No | `https://core.skypaybd.top/api/payment/verify` |
| `status` | ENUM('active','inactive') | No | Enable or disable gateway |
| `created_at` | TIMESTAMP | No | Row creation time |
| `updated_at` | TIMESTAMP | Yes | Row update time |

At runtime, your payment initiation code queries this table for the first row where `status = 'active'`. If no active row is found, display an error and do not attempt to call SkyPay.

### Table: `payments`

Tracks every payment attempt initiated through your website.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | INT UNSIGNED AUTO_INCREMENT | No | Primary key |
| `order_id` | VARCHAR(100) | No | Your internal order/invoice number |
| `transaction_id` | VARCHAR(100) | Yes | TrxID returned from SkyPay callback (before verification) |
| `cus_name` | VARCHAR(255) | No | Customer full name sent to SkyPay |
| `cus_email` | VARCHAR(255) | No | Customer email sent to SkyPay |
| `amount` | DECIMAL(10,2) | No | Payment amount in BDT |
| `payment_method` | VARCHAR(50) | Yes | bkash / nagad / rocket / upay (from callback) |
| `status` | ENUM('pending','completed','failed','cancelled') | No | Payment state |
| `metadata` | JSON | Yes | Any extra data attached to the session |
| `skypay_payment_url` | TEXT | Yes | The payment_url returned by SkyPay (useful for logging) |
| `callback_raw_params` | JSON | Yes | Raw GET params received at callback URL (for audit trail) |
| `verified_at` | TIMESTAMP | Yes | Timestamp when backend verification succeeded |
| `created_at` | TIMESTAMP | No | When the payment session was initiated |

---

## Full Integration Architecture

### File Structure (Custom PHP Website)

```
/payment/
    GatewayConfig.php     ← Loads active gateway row from DB
    SkyPayService.php     ← All HTTP calls to SkyPay API
    initiate.php          ← Called when user clicks "Pay Now"
    callback.php          ← SkyPay redirects user here after payment
    cancel.php            ← SkyPay redirects user here on cancel
```

---

## GatewayConfig.php

Responsibilities:
- Connect to the database using PDO or MySQLi
- Query `SELECT * FROM auto_payment_gateway WHERE status = 'active' LIMIT 1`
- If no row found, throw an exception or redirect to an error page
- Expose the config values (`brand_key`, `create_url`, `verify_url`) to the service class

---

## SkyPayService.php

This class handles all HTTP communication with SkyPay. It is instantiated with the gateway config object.

### Method: `createPayment(array $data): array`

**Input parameters:**
- `cus_name` (string, required) — customer's full name
- `cus_email` (string, required) — customer's email address
- `amount` (numeric, required) — amount in BDT, must be greater than 0
- `success_url` (string, required) — absolute URL of your `callback.php`, with `payment_id` as a query parameter so the callback knows which DB row to update
- `cancel_url` (string, required) — absolute URL of your `cancel.php`
- `metadata` (array, optional) — include `order_id`, `user_id`, or any data you need returned at callback time

**Behavior:**
- Sends POST to `create_url` with `BRAND-KEY` header and JSON-encoded body
- Returns the decoded JSON array
- On success: response includes `status: true` and `payment_url`
- On failure: response includes `status: false` and a `message` string

**Implementation detail:**
Use PHP's cURL extension. Set `CURLOPT_RETURNTRANSFER` to true, `CURLOPT_TIMEOUT` to 30, `CURLOPT_POST` to true, and send the body as JSON. Decode with `json_decode($result, true)`. Always check `curl_errno()` and log any cURL errors.

### Method: `verifyPayment(string $transactionId): array`

**Input:**
- `transaction_id` (string, required) — the TrxID received at the callback URL

**Behavior:**
- Sends POST to `verify_url` with `BRAND-KEY` header and body `{ "transaction_id": "..." }`
- Returns decoded JSON array
- On success: `status` equals `"COMPLETED"`, and response includes `cus_name`, `cus_email`, `amount`, `payment_method`, and `metadata`
- On failure: `status` may be `"PENDING"` or `"ERROR"` with a `message`

---

## initiate.php — Payment Initiation Endpoint

This file is your entry point when the user clicks "Pay Now" or submits the checkout form. It must be a POST endpoint (the checkout form POSTs to it).

**Step-by-step logic:**

1. **Validate inputs:** Ensure `cus_name`, `cus_email`, and `amount` are present and valid. Validate that `amount` is a positive number. Validate that `cus_email` is a valid email format.

2. **Generate or retrieve `order_id`:** This should come from your order system. If creating a new order here, generate a unique order ID (e.g., `ORD-` + timestamp + random suffix).

3. **Insert pending payment row:** Insert a new row in the `payments` table with `status = 'pending'`, the `order_id`, `cus_name`, `cus_email`, `amount`, and any `metadata`. Save the inserted row's `id` as `$payment_id`.

4. **Build URLs:**
   - `success_url` → `https://yoursite.com/payment/callback.php?payment_id={$payment_id}`
   - `cancel_url` → `https://yoursite.com/payment/cancel.php?payment_id={$payment_id}`

5. **Call SkyPayService::createPayment()** with all required fields. Include `['order_id' => $order_id, 'payment_id' => $payment_id]` in `metadata`.

6. **If API returns `status: true`:**
   - Update the payment row: store `skypay_payment_url`
   - Redirect the user's browser to `payment_url` using `header("Location: " . $payment_url); exit;`

7. **If API returns an error:**
   - Update the payment row to `status = 'failed'`
   - Log the error
   - Show the user a friendly error message (e.g., "Payment gateway unavailable. Please try again.")
   - Do not redirect to SkyPay

---

## callback.php — Payment Return Handler

SkyPay redirects the customer here after they complete the payment on the hosted page. The URL will look like:

```
https://yoursite.com/payment/callback.php?payment_id=42&transactionId=BLA38KDK2M&paymentMethod=bkash&paymentAmount=250.00&paymentFee=0.00&status=completed
```

**⚠️ CRITICAL SECURITY WARNING:** These GET parameters are sent by the browser and can be forged by any user. A malicious actor could visit `callback.php?transactionId=FAKEID&status=completed` to try to trigger order fulfillment. You must NEVER fulfill an order based on these GET params alone. Always verify server-side.

**Step-by-step logic:**

1. **Read GET parameters:** Extract `payment_id` and `transactionId` from `$_GET`. If either is missing or empty, redirect to an error page.

2. **Load the payment row:** Query the `payments` table by `payment_id`. If no row found, redirect to an error page.

3. **Store raw callback params for audit:** Update the payment row's `callback_raw_params` column with `json_encode($_GET)`.

4. **Idempotency check:** If the payment row's `status` is already `'completed'`, this callback is a duplicate (e.g., the user refreshed the page). Do not process again. Redirect to the success page immediately.

5. **Call SkyPayService::verifyPayment($transactionId):**

6. **If verification response `status === 'COMPLETED'`:**
   - Check the `payments` table: is there any OTHER row (different `payment_id`) that already has this `transaction_id` with `status = 'completed'`? If yes, this TrxID is being reused — reject it and log a fraud alert.
   - Update the current payment row: `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = NOW()`
   - Fulfill the order: activate the user account, mark the order as paid, trigger shipping, send a confirmation email, add wallet balance — whatever applies to your platform
   - Redirect the user to a thank-you / order success page

7. **If verification response `status` is not `COMPLETED`:**
   - Update the payment row to `status = 'failed'`
   - Log the full API response for debugging
   - Show the user a clear error message with next steps (e.g., "Payment verification failed. If money was deducted from your account, please contact support with TrxID: {$transactionId}")

---

## cancel.php — Cancellation Handler

Called when the user cancels on the SkyPay page and is redirected back.

**Logic:**
1. Read `payment_id` from GET params
2. Load the payment row from DB
3. If `status` is still `'pending'`, update it to `'cancelled'`
4. Show the user a cancellation message with a link to return to the cart or try again
5. Do not delete the payment row — keep it for records

---

## Security Checklist for Website Integration

- BRAND-KEY is stored server-side only, never in frontend HTML or JS
- `initiate.php` validates all inputs before calling SkyPay
- `callback.php` always calls `verifyPayment()` before fulfilling any order
- Idempotency check prevents double fulfillment if user refreshes callback URL
- Duplicate TrxID check prevents one TrxID from fulfilling multiple orders
- All database queries use prepared statements (PDO bindParam or MySQLi bind_param)
- All SkyPay API responses and errors are logged to a server-side log file
- The `payments` table has a unique index on `transaction_id` to enforce database-level deduplication

---

## Pre-Built Plugin Option

For merchants using popular CMS platforms, SkyPay provides pre-built modules that handle the entire integration with zero custom code:

**WordPress WooCommerce:**
- Download `WP.zip` from the SkyPay merchant dashboard
- Upload via WordPress Admin → Plugins → Add New → Upload Plugin
- Activate the plugin
- Go to WooCommerce → Settings → Payments → SkyPay → enter BRAND-KEY
- The plugin handles create, redirect, callback, and verification automatically
- Orders are automatically updated to "Processing" on successful payment

**WHMCS Web Hosting Billing:**
- Download `WHMCS.zip` from the SkyPay merchant dashboard
- Extract into `<whmcs_root>/modules/gateways/`
- Go to WHMCS Admin → Setup → Payment Gateways → Activate SkyPay
- Enter BRAND-KEY in the gateway settings
- Invoices are automatically marked as Paid and hosting provisioning is triggered

**SMM Panels (SmartPanel, PerfectPanel):**
- Download `SMM.zip` from the SkyPay merchant dashboard
- Follow the module installation instructions included in the zip
- Supports instant balance auto-addition without manual admin approval

For custom websites, implement the full flow described above from scratch.

---

## Advanced: Headless Mode for SPAs (Optional)

If the website is a Single-Page Application (React, Vue, Next.js) and you want zero redirects, you can use the Headless API v2 instead. In this case:

- Your frontend calls your backend API to initiate a session
- Your backend calls `/api/v2/payment/create` and returns the wallet numbers to the frontend
- Your frontend renders a custom payment modal showing the wallet numbers
- The user pays and types their TrxID into the modal
- Your frontend sends the TrxID to your backend
- Your backend calls `/api/v2/payment/verify` and returns the result
- Your frontend shows success or a retry prompt

For this scenario, refer to the `flutter.md` guide for Headless v2 architecture patterns, and adapt them for a web frontend.

---

## Complete Flow Summary

```
User clicks "Pay Now" → form POST to initiate.php
    → Validate inputs
    → INSERT pending payment row in DB
    → POST /api/payment/create → get payment_url
    → Redirect user to payment_url (SkyPay hosted page)

User selects wallet, sends money, enters TrxID on SkyPay page
    → SkyPay verifies SMS in real-time
    → SkyPay redirects user back to callback.php?transactionId=...

callback.php
    → Idempotency check (already completed? → skip)
    → POST /api/payment/verify with transactionId
    → Response status === 'COMPLETED'?
        YES → Update payment row, fulfill order, redirect to success page
        NO  → Update payment row to failed, show error to user
```
