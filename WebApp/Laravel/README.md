# SkyPay Payment Gateway — Laravel Integration Guide (Hosted Gateway v1)

> **Scope:** This guide covers the **SkyPay Hosted Checkout Gateway (v1)** for accepting payments in Laravel Blade and web applications through a browser redirect flow. It does **not** cover the Headless API v2 for Telegram bots, mobile apps, or other client applications.

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Endpoints and Authentication](#2-endpoints-and-authentication)
3. [Environment Configuration](#3-environment-configuration)
4. [Database Migrations](#4-database-migrations)
5. [Models](#5-models)
6. [Service Class — SkyPayService](#6-service-class--skypayservice)
7. [AppServiceProvider Binding](#7-appserviceprovider-binding)
8. [Routes](#8-routes)
9. [PaymentController](#9-paymentcontroller)
10. [Callback Parameters](#10-callback-parameters)
11. [Verify Endpoint](#11-verify-endpoint)
12. [Custom Exception](#12-custom-exception)
13. [Logging](#13-logging)
14. [Error Handling](#14-error-handling)
15. [Production Checklist](#15-production-checklist)
16. [Create Request Reference](#16-create-request-reference)
17. [HTTP Status Codes](#17-http-status-codes)

---

## 1. Architecture Overview

SkyPay Hosted Checkout v1 uses a **redirect-based payment flow**. Your Laravel application creates a payment session through the SkyPay Core API, redirects the customer to SkyPay’s hosted checkout page, and verifies the resulting transaction from your server before fulfilling the order.

### Payment lifecycle

1. **Customer initiates checkout:** The customer submits your Laravel checkout form.
2. **Create a local payment record:** Your application validates the order details and stores a pending payment record with a unique internal order ID.
3. **Create a SkyPay payment session:** Laravel sends a server-to-server `POST` request to the create endpoint using the `BRAND-KEY` header.
4. **Redirect to hosted checkout:** SkyPay returns a `payment_url`; Laravel redirects the customer’s browser to that URL.
5. **Customer pays:** On the SkyPay checkout page, the customer selects an available channel (such as bKash, Nagad, Rocket, or Upay), completes the payment steps, and submits the transaction ID as required.
6. **Return to the merchant:** SkyPay redirects the browser to the configured success or cancellation URL. The success callback may include transaction details as query parameters.
7. **Verify on the server:** Laravel sends the received transaction ID to SkyPay’s verify endpoint. The browser’s callback parameters must not be treated as proof of payment.
8. **Update and fulfill:** Only a verified `COMPLETED` response should trigger fulfillment. Pending or unsuccessful verification must not fulfill the order.

```text
Customer → Laravel checkout → SkyPay create API
         → Hosted checkout → Customer completes payment
         → Laravel callback → SkyPay verify API
         → Update payment → Fulfill verified order
```

**Important:** The customer’s browser is not a trusted payment authority. Always verify the transaction server-to-server before granting products, credits, subscriptions, or other benefits.

---

## 2. Endpoints and Authentication

### Base domain

`https://core.skypaybd.top`

### API endpoints

| Action | Method | Endpoint |
|---|---|---|
| Create a hosted payment session | `POST` | `/api/payment/create` |
| Verify a transaction | `POST` | `/api/payment/verify` |

Use the full endpoint URLs in your application configuration:

- Create: `https://core.skypaybd.top/api/payment/create`
- Verify: `https://core.skypaybd.top/api/payment/verify`

### Authentication

Every API request must include the following headers:

```http
BRAND-KEY: your_unique_brand_key
Content-Type: application/json
Accept: application/json
```

The integration uses **one credential: `BRAND-KEY`**. A separate `SECRET-KEY` is not required according to this v1 specification. `API-KEY` and `SECRET-KEY` are described as aliases for the same brand credential; send the documented `BRAND-KEY` header.

Keep the brand key exclusively on your server. Do not expose it in Blade templates, JavaScript, mobile code, public repositories, or client-side network requests. Store it in the environment configuration and restrict access to production environment files.

---

## 3. Environment Configuration

Add the following values to your Laravel `.env` file:

```env
SKYPAY_BRAND_KEY=your_brand_key_here
SKYPAY_CREATE_URL=https://core.skypaybd.top/api/payment/create
SKYPAY_VERIFY_URL=https://core.skypaybd.top/api/payment/verify
```

Create `config/skypay.php` to make these settings available through Laravel’s configuration system:

```php
<?php

return [
    'brand_key'  => env('SKYPAY_BRAND_KEY'),
    'create_url' => env('SKYPAY_CREATE_URL', 'https://core.skypaybd.top/api/payment/create'),
    'verify_url' => env('SKYPAY_VERIFY_URL', 'https://core.skypaybd.top/api/payment/verify'),
];
```

After changing environment values in a deployed application, refresh Laravel’s configuration cache as appropriate for your deployment process. Do not hard-code credentials in source files.

---

## 4. Database Migrations

The integration uses two tables: one for gateway configuration and another for individual payment records.

### 4.1 `auto_payment_gateway`

This table stores the brand credential, API endpoints, and activation state. It allows the application to manage the configured gateway without embedding credentials in controller logic.

Recommended fields:

| Field | Purpose |
|---|---|
| `id` | Primary key |
| `brand_key` | Server-side SkyPay brand credential |
| `create_url` | Payment creation endpoint |
| `verify_url` | Transaction verification endpoint |
| `status` | `active` or `inactive` |
| `created_at`, `updated_at` | Laravel timestamps |

Use a migration with the table name `auto_payment_gateway`. The supplied integration expects an active record to exist before the service is resolved.

### 4.2 `payments`

This table tracks the merchant’s payment lifecycle and connects a SkyPay transaction to a local order.

| Field | Purpose |
|---|---|
| `id` | Primary key and local payment identifier |
| `order_id` | Merchant-generated order reference |
| `transaction_id` | SkyPay/telecom transaction ID; nullable until available and unique when populated |
| `cus_name`, `cus_email` | Customer information |
| `amount` | Expected payment amount in BDT |
| `payment_method` | Channel reported by the verification response |
| `status` | Local state: `pending`, `completed`, `failed`, or `cancelled` |
| `metadata` | Optional JSON data such as user or product identifiers |
| `verified_at` | Time the payment was verified |
| `created_at`, `updated_at` | Laravel timestamps |

Use a decimal column for money, not a floating-point column. The provided design uses `decimal(10, 2)`. Ensure that `transaction_id` is nullable and unique so multiple not-yet-verified records can exist while duplicate transaction IDs are rejected.

### 4.3 Seeder

A seeder may initialize `auto_payment_gateway` from the environment values. Run migrations and then the seeder:

```bash
php artisan migrate
php artisan db:seed --class=AutoPaymentGatewaySeeder
```

Run the seeder only when appropriate for your deployment process. Avoid unintentionally creating duplicate gateway configuration records; use an update-or-create strategy if the seeder may be executed repeatedly.

---

## 5. Models

### 5.1 `AutoPaymentGateway`

Create `app/Models/AutoPaymentGateway.php` and map it to the `auto_payment_gateway` table. The model should allow the gateway fields to be mass-assigned, provide an `active` query scope, and expose a `getActive()` method that returns an active gateway record.

If no active record exists, `getActive()` should throw a clear runtime exception. This prevents payment creation from proceeding with missing or inactive configuration.

### 5.2 `Payment`

Create `app/Models/Payment.php` for the `payments` table. Its fillable fields should include the order/customer/payment attributes used by the controller. Cast `metadata` to an array, `verified_at` to a datetime, and `amount` to a two-decimal representation.

The local payment status is an application-side lifecycle state. It should be changed based on verified server responses and explicit cancellation handling—not merely because a customer returns to the callback URL.

---

## 6. Service Class — `SkyPayService`

Create `app/Services/SkyPayService.php` to keep SkyPay HTTP communication separate from controller logic. The service receives an active `AutoPaymentGateway` model and provides two operations.

### 6.1 Create a hosted payment

`createHostedPayment(array $data)` sends a JSON `POST` request to the configured create endpoint.

**Required fields:**
- `cus_name`
- `cus_email`
- `amount`
- `success_url`
- `cancel_url`

**Optional fields:**
- `meta_data`
- `webhook_url`
- `return_type` (`GET` by default, or `POST`)

On success, the service expects a response containing `status: true` and a `payment_url`. The controller uses that URL to redirect the customer to hosted checkout. If the request fails or the response indicates an error, the service should raise `SkyPayException`.

### 6.2 Verify a hosted payment

`verifyHostedPayment(string $transactionId)` sends a JSON `POST` request to the configured verify endpoint with the transaction ID.

The response’s `status` is a **string**, not a boolean. The integration should recognize `COMPLETED` as verified completion. `PENDING` means the transaction is not yet confirmed, while `ERROR` or `FAILED` indicates an unsuccessful or invalid transaction.

### 6.3 HTTP handling

Use Laravel’s HTTP client with the authentication headers described in Section 2. Handle connection failures and invalid responses explicitly. Consider setting a reasonable request timeout and checking HTTP success before interpreting the JSON body. Do not log the brand key or other secrets.

---

## 7. AppServiceProvider Binding

Bind `SkyPayService` in `app/Providers/AppServiceProvider.php` so Laravel can inject it into the controller.

The binding should resolve the active gateway through `AutoPaymentGateway::getActive()` and pass that model to the service. Ensure the database is available when the service is resolved, and configure the gateway record before using payment routes.

---

## 8. Routes

Register the following routes in `routes/web.php`:

| Method | URI | Route name | Purpose |
|---|---|---|---|
| `POST` | `/payment/initiate` | `payment.initiate` | Validate checkout and create a payment |
| `GET` | `/payment/callback` | `payment.callback` | Receive the customer’s return and verify the transaction |
| `GET` | `/payment/cancel` | `payment.cancel` | Handle customer cancellation |

The `success_url` should point to the callback route and include the local `payment_id`. The `cancel_url` should point to the cancellation route and include the same local identifier.

Protect payment initiation with the normal Laravel web security measures, including CSRF protection for form submissions and authorization checks to ensure a customer can only initiate or access their own order.

---

## 9. PaymentController

Create `app/Http/Controllers/PaymentController.php`. The controller coordinates local payment records, the service, callback verification, and the final redirect.

### 9.1 Initiate payment

The `initiate` action should:

1. Validate customer name, email, and amount on the server. The documented amount range is BDT 1 to BDT 1,000,000.
2. Create a unique merchant order ID and a local `payments` record with `pending` status.
3. Call `createHostedPayment()` with customer details, amount, success/cancel URLs, and metadata linking the SkyPay session to the local payment/order.
4. Redirect the customer to the returned `payment_url`.
5. If session creation fails, keep the failure visible to the customer and record enough diagnostic information for support without exposing secrets.

The amount must come from trusted server-side order pricing wherever possible. Do not rely on a client-submitted amount for products or services with a known price.

### 9.2 Process callback

The callback receives the local `payment_id` and the transaction ID returned by SkyPay. It may also receive `paymentMethod`, `paymentAmount`, `paymentFee`, and `status`.

The callback should:

1. Validate that the required identifiers are present.
2. Load the local payment record and verify that the current user/session is permitted to access it, where applicable.
3. If the payment is already completed, return the existing success result without fulfilling it again.
4. Call `verifyHostedPayment()` using the transaction ID.
5. Confirm that the response status is exactly `COMPLETED`.
6. Compare verified transaction details—especially amount and transaction identity—with the expected local payment before fulfillment. Do not treat a status alone as sufficient if the returned amount or transaction association does not match.
7. Persist the verified transaction ID, payment method, verification time, and completed status.
8. Trigger fulfillment in an idempotent way, preferably through a database transaction and/or a uniquely keyed event/job.

If verification returns `PENDING`, do not mark the payment as completed or failed immediately. Keep it pending and show a waiting/retry message or use a controlled server-side retry process. If verification definitively returns `ERROR` or `FAILED`, mark it failed according to your application’s policy.

**Reliability note:** A network timeout does not prove that a payment failed. Avoid permanently marking a payment failed solely because the verification request could not connect; preserve a retryable state and log the issue.

### 9.3 Handle cancellation

The cancellation action may mark a local payment as `cancelled` only while it is still pending. It should then return the customer to checkout with a clear message. A cancellation redirect is not proof that no payment occurred; if your business flow permits a customer to pay and then navigate back, provide a way to re-check the transaction before closing the order permanently.

---

## 10. Callback Parameters

After the customer completes checkout, SkyPay redirects the browser to the configured `success_url`. Under this v1 specification, the callback may contain parameters such as:

| Parameter | Type | Example | Meaning |
|---|---|---|---|
| `payment_id` | String/integer | `42` | Local payment record identifier included by your application |
| `transactionId` | String | `BLA38KDK2M` | Transaction ID to submit for server-side verification |
| `paymentMethod` | String | `bkash` | Reported payment channel |
| `paymentAmount` | Numeric | `250.00` | Reported amount in BDT |
| `paymentFee` | Numeric | `0.00` | Reported gateway fee |
| `status` | String | `completed` | Browser-return status, if supplied |

Example callback URL:

```text
https://mystore.com/payment/callback?payment_id=42&transactionId=BLA38KDK2M&paymentMethod=bkash&paymentAmount=250.00&paymentFee=0.00&status=completed
```

Treat these parameters as **untrusted display and lookup data only**. In particular, the callback `status`, amount, method, and fee must not be used as the sole basis for marking an order paid. Always call the verify endpoint from your Laravel backend and compare its response with the local order.

If the integration is configured with `return_type=POST`, implement and test the corresponding callback method and parameter handling rather than assuming query-string delivery.

---

## 11. Verify Endpoint

### Request

**Method:** `POST`  
**Endpoint:** `https://core.skypaybd.top/api/payment/verify`

Send the `BRAND-KEY` authentication header and a JSON body containing the transaction ID:

```json
{
  "transaction_id": "BLA38KDK2M"
}
```

The documented accepted aliases for the transaction ID are `transaction_id`, `transactionId`, `transactionid`, `trx_id`, `trx`, and `transaction`. Prefer the canonical `transaction_id` field in new integrations.

### Successful response

A completed response is documented in this shape:

```json
{
  "status": "COMPLETED",
  "cus_name": "John Doe",
  "cus_email": "customer@gmail.com",
  "amount": "250.00",
  "transaction_id": "BLA38KDK2M",
  "payment_method": "bkash",
  "meta_data": {
    "payment_id": 42,
    "order_id": "ORD-ABC123"
  }
}
```

### Response fields

| Field | Type | Meaning |
|---|---|---|
| `status` | String | `COMPLETED`, `PENDING`, or `ERROR` (some integrations may also report `FAILED`) |
| `cus_name` | String | Customer name associated with the payment |
| `cus_email` | String | Customer email |
| `amount` | String/numeric | Verified payment amount in BDT |
| `transaction_id` | String | Verified transaction identifier |
| `payment_method` | String | Channel, such as `bkash`, `nagad`, `rocket`, or `upay` |
| `meta_data` | Object | Metadata supplied when the payment was created |

### Status handling

| Verify status | Meaning | Application behavior |
|---|---|---|
| `COMPLETED` | Payment is confirmed | Validate transaction association and amount, then fulfill once |
| `PENDING` | Payment has not yet been confirmed | Keep pending; do not fulfill |
| `ERROR` / `FAILED` | Invalid or unsuccessful transaction | Do not fulfill; handle as failed when definitive |

The status comparison must be string-based, for example: `$verified['status'] === 'COMPLETED'`.

---

## 12. Custom Exception

Create `app/Exceptions/SkyPayException.php` as a dedicated exception type for SkyPay request, response, and configuration failures. Catch it where the application can provide a useful customer-facing message, while keeping technical details in restricted logs.

Avoid showing raw exception messages, credentials, internal URLs, or sensitive response data to customers in production.

---

## 13. Logging

Configure a dedicated `skypay` channel in `config/logging.php`, writing to `storage/logs/skypay.log`.

Log useful operational details such as:
- Request operation (create or verify)
- Local payment ID or order reference
- HTTP status and sanitized response status
- Connection errors and retry outcomes

Do **not** log the `BRAND-KEY`, full sensitive customer information, or unnecessary payment metadata. Restrict access to log files and define retention/rotation appropriate for production. Ensure Laravel’s storage directory is writable by the application process.

---

## 14. Error Handling

Handle `SkyPayException` centrally or at the controller boundary.

For browser requests, return the customer to a safe checkout or status page with a clear, non-sensitive message. For JSON requests, return a consistent JSON error structure and an appropriate HTTP status.

Differentiate between:
- **Validation errors:** Correct the submitted fields.
- **Authentication/configuration errors:** Check the configured brand key and gateway activation.
- **Definitive gateway rejection:** Display a safe failure message.
- **Connection timeout or uncertain response:** Preserve a retryable payment state and allow verification to be attempted again.
- **Pending verification:** Keep the order pending and do not grant the purchased benefit.

Do not use `back()` blindly when the callback request may not have a valid browser history. Redirect to a known payment-status or checkout route.

---

## 15. Production Checklist

Before enabling live payments, verify each item:

- [ ] `SKYPAY_BRAND_KEY` is correctly configured in the production environment.
- [ ] The brand key is never exposed in client-side code, public repositories, or logs.
- [ ] Create and verify requests use the intended HTTPS Core API endpoints.
- [ ] An active gateway configuration exists in `auto_payment_gateway`.
- [ ] Payment initiation validates the customer and uses a trusted server-side amount.
- [ ] A local pending payment record is created before redirecting to SkyPay.
- [ ] Callback parameters are treated as untrusted and server-side verification is mandatory.
- [ ] Verified amount and transaction identity are checked against the local order.
- [ ] Fulfillment is idempotent and cannot run twice for the same payment.
- [ ] The `transaction_id` column has a unique constraint.
- [ ] `PENDING` and temporary network errors remain retryable and do not trigger fulfillment.
- [ ] Cancellation only updates eligible pending records and does not incorrectly override a completed payment.
- [ ] The merchant Android phone running SkyPay is operational, with battery optimization configured as required and reliable power/network access.
- [ ] The dedicated log channel works, is access-restricted, and does not record secrets.
- [ ] Exceptions and customer-facing errors are handled safely.
- [ ] Test successful, pending, failed, cancelled, duplicate-callback, and verification-timeout scenarios before launch.

---

## 16. Create Request Reference

**Method:** `POST`  
**Endpoint:** `https://core.skypaybd.top/api/payment/create`

### Parameters

| Parameter | Type | Required | Description | Example |
|---|---|---|---|---|
| `cus_name` | String | Yes | Customer’s full name | `John Doe` |
| `cus_email` | String | Yes | Customer email; documented aliases include `customer_email`, `c_email`, and `email` | `john@gmail.com` |
| `amount` | Numeric | Yes | Payment amount in BDT; documented range is 1–1,000,000 | `250` or `250.50` |
| `success_url` | URL | Yes | Return URL after successful checkout | `https://mystore.com/payment/callback` |
| `cancel_url` | URL | Yes | Return URL when checkout is cancelled | `https://mystore.com/payment/cancel` |
| `meta_data` | Object/JSON | No | Merchant-defined order or user references | `{"order_id":"ORD-123"}` |
| `webhook_url` | URL | No | Optional server-to-server payment notification endpoint | `https://mystore.com/api/webhook` |
| `return_type` | String | No | Callback method: `GET` (default) or `POST` | `GET` |

### Create success response

```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/checkout/order/f7b3a9c2d1e04856"
}
```

The application should redirect only when the response indicates success and contains a valid `payment_url`.

### Create error response

```json
{
  "status": false,
  "message": "The success_url field is required."
}
```

Treat the returned message as diagnostic input; show a safe, understandable message to the customer and keep detailed diagnostics in protected logs.

---

## 17. HTTP Status Codes

The following codes are described by the v1 integration specification. Actual response bodies and behavior should be confirmed in your SkyPay environment during testing.

| HTTP code | Meaning | Recommended action |
|---|---|---|
| `200 OK` | Request processed | Inspect the JSON `status` field; HTTP 200 alone does not mean a payment is completed |
| `400 Bad Request` | Missing or invalid request parameter | Check the request body and amount |
| `401 Unauthorized` | Missing or invalid `BRAND-KEY` | Verify the brand credential |
| `403 Forbidden` | Brand credentials or account are inactive/restricted | Check the brand’s activation state |
| `404 Not Found` | Payment session or resource not found | Check identifiers and create a new session if appropriate |
| `422 Unprocessable Entity` | Validation failure, such as invalid URL, amount, or JSON | Correct the submitted data |
| `500 Internal Server Error` | Gateway-side server error | Retry cautiously; verify transaction state before creating another payment |

---

**SkyPay BD — Automated Payment Infrastructure for Bangladesh**

- Website: https://skypaybd.top
- Documentation: https://skypaybd.top/docs
- API Core: https://core.skypaybd.top
