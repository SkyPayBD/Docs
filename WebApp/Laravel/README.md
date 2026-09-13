# SkyPay Payment Gateway — Laravel Framework Integration Guide for AI Agents

## Overview

This document provides complete instructions for an AI agent integrating SkyPay BD into a **Laravel** application. Laravel is used for two different scenarios — each uses a different API:

- **Laravel serving a website (Blade views, web routes):** Use Hosted Gateway (v1) with browser redirect flow
- **Laravel as an API backend (serving a Flutter app, Telegram bot, or React SPA):** Use Headless API v2 with JSON responses

The AI agent must determine which scenario applies based on the user's request and follow the appropriate section below.

Always read the main SkyPay `README.md` API documentation first before writing any code.

---

## Endpoints Reference

### Hosted Gateway (v1) — for web applications
- Create: `POST https://core.skypaybd.top/api/payment/create`
- Verify: `POST https://core.skypaybd.top/api/payment/verify`

### Headless API (v2) — for API backends
- Create: `POST https://core.skypaybd.top/api/v2/payment/create`
- Verify: `POST https://core.skypaybd.top/api/v2/payment/verify`

---

## Authentication

All requests require only the `BRAND-KEY` HTTP header. No SECRET-KEY is needed.

Store the BRAND-KEY in the `.env` file:
```
SKYPAY_BRAND_KEY=your_brand_key_here
SKYPAY_CREATE_URL=https://core.skypaybd.top/api/payment/create
SKYPAY_VERIFY_URL=https://core.skypaybd.top/api/payment/verify
SKYPAY_V2_CREATE_URL=https://core.skypaybd.top/api/v2/payment/create
SKYPAY_V2_VERIFY_URL=https://core.skypaybd.top/api/v2/payment/verify
```

Add corresponding entries in `config/skypay.php`:
```
return [
    'brand_key'      => env('SKYPAY_BRAND_KEY'),
    'create_url'     => env('SKYPAY_CREATE_URL'),
    'verify_url'     => env('SKYPAY_VERIFY_URL'),
    'v2_create_url'  => env('SKYPAY_V2_CREATE_URL'),
    'v2_verify_url'  => env('SKYPAY_V2_VERIFY_URL'),
];
```

---

## Database Migrations

### Migration 1: `auto_payment_gateway` table

Create this migration to store the gateway configuration in the database, enabling admin panel control over the gateway.

Columns:
| Column | Type | Description |
|---|---|---|
| `id` | bigIncrements | Primary key |
| `brand_key` | string(255) | BRAND-KEY from SkyPay |
| `create_url` | string(255) | Hosted create endpoint |
| `verify_url` | string(255) | Hosted verify endpoint |
| `v2_create_url` | string(255) | Headless v2 create endpoint |
| `v2_verify_url` | string(255) | Headless v2 verify endpoint |
| `status` | enum: ['active','inactive'] | Gateway toggle |
| `timestamps` | — | created_at, updated_at |

### Migration 2: `payments` table

Track all payment sessions and their lifecycle.

Columns:
| Column | Type | Description |
|---|---|---|
| `id` | bigIncrements | Primary key |
| `order_id` | string(100) | Your internal order reference |
| `skypay_session_id` | string(255), nullable | The `id` returned by v2 create (for Headless) |
| `transaction_id` | string(100), nullable, unique | TrxID after payment |
| `cus_name` | string(255) | Customer name |
| `cus_email` | string(255), nullable | Customer email (v1 only) |
| `amount` | decimal(10,2) | Amount in BDT |
| `payment_method` | string(50), nullable | bkash/nagad/rocket/upay |
| `api_version` | enum: ['v1','v2'] | Which API was used |
| `status` | enum: pending/completed/failed/cancelled | Payment state |
| `metadata` | json, nullable | Extra data (user ID, product, etc.) |
| `verified_at` | timestamp, nullable | When verification succeeded |
| `timestamps` | — | created_at, updated_at |

### Seeder

Create a `AutoPaymentGatewaySeeder` that inserts one active row reading values from `.env`. Run this seeder after migration.

---

## Model

Create `app/Models/AutoPaymentGateway.php`:
- `$fillable` includes all columns
- Add a static scope `active()` → `where('status', 'active')`
- Add a static helper `getActive()` → returns the first active row or throws an exception if none found

Create `app/Models/Payment.php`:
- `$fillable` includes all columns
- Cast `metadata` as `array`
- Cast `verified_at` as `datetime`

---

## Service Class

Create `app/Services/SkyPayService.php`. Register it as a singleton in `AppServiceProvider`.

The constructor accepts the active `AutoPaymentGateway` model instance (or reads it from DB internally). It exposes the following public methods:

### `createHostedPayment(array $data): array`

For Hosted Gateway (v1). Accepts:
- `cus_name` (required)
- `cus_email` (required)
- `amount` (required, numeric)
- `success_url` (required)
- `cancel_url` (required)
- `metadata` (optional, array)

Sends POST to `create_url` with `BRAND-KEY` header. Returns decoded response array. On success, response contains `payment_url`.

### `verifyHostedPayment(string $transactionId): array`

For Hosted Gateway (v1) callback verification. Sends POST to `verify_url` with `{ "transaction_id": "..." }`. Returns decoded response. On success, `status` equals `"COMPLETED"`.

### `createHeadlessPayment(array $data): array`

For Headless API v2. Accepts:
- `cus_name` (required)
- `amount` (required, numeric)
- `meta_data` (optional, array)

Sends POST to `v2_create_url`. Returns full response including `id` (session ID) and `methods` array (active wallet channels and numbers).

### `verifyHeadlessPayment(string $sessionId, string $method, string $transactionId): array`

For Headless API v2. Sends POST to `v2_verify_url` with:
- `id` → the session ID from `createHeadlessPayment()`
- `method` → lowercase channel name
- `transaction_id` → TrxID submitted by user

Returns response. On success, `status` is `true`.

### HTTP Implementation

Use Laravel's `Http` facade:
```
Http::withHeaders([
    'BRAND-KEY'    => config('skypay.brand_key'),
    'Content-Type' => 'application/json',
    'Accept'       => 'application/json',
])->post($url, $payload)->json();
```

Wrap all calls in try-catch. Log both request and response using `Log::info()`. Throw a custom `SkyPayException` on failure for clean error handling in controllers.

---

## Hosted Gateway Flow (for Web Applications)

### Routes (web.php)
```
POST /payment/initiate   → PaymentController@initiate
GET  /payment/callback   → PaymentController@callback
GET  /payment/cancel     → PaymentController@cancel
```

### PaymentController@initiate

Steps:
1. Validate request: `cus_name`, `cus_email`, `amount`, and any order-specific fields
2. Create a `Payment` model record with `status = 'pending'` and `api_version = 'v1'`
3. Build `success_url` pointing to `/payment/callback` with `payment_id` as query param
4. Build `cancel_url` pointing to `/payment/cancel`
5. Call `SkyPayService::createHostedPayment()` with all fields. Include `payment_id` in `metadata`
6. If API returns `status: true` → redirect user to `payment_url`
7. If API returns error → return back with error message

### PaymentController@callback

Called by SkyPay after the user completes payment. Receives GET params: `transactionId`, `paymentMethod`, `paymentAmount`, `status`.

Steps:
1. Read `transactionId` from the request query
2. Look up the `Payment` record via `payment_id` from query param or `metadata`
3. **Check for duplicate:** if the payment record is already `completed`, stop and redirect to success page (idempotency)
4. Call `SkyPayService::verifyHostedPayment($transactionId)`
5. If response `status === 'COMPLETED'`:
   - Update `Payment` record: `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at = now()`
   - Trigger order fulfillment logic (event, job, or direct call)
   - Redirect to thank-you page
6. Otherwise: update record to `status = 'failed'`, redirect with error

### PaymentController@cancel

Steps:
1. Retrieve the payment record
2. Update status to `'cancelled'`
3. Redirect to cart or checkout with cancellation message

---

## Headless API Flow (for API Backends serving Mobile Apps or Bots)

### Routes (api.php)
```
POST /api/payment/initiate       → ApiPaymentController@initiate
POST /api/payment/verify         → ApiPaymentController@verify
```

### ApiPaymentController@initiate

Steps:
1. Validate JSON body: `cus_name`, `amount`, and any app-specific fields (user ID, product ID)
2. Create a `Payment` record: `status = 'pending'`, `api_version = 'v2'`
3. Call `SkyPayService::createHeadlessPayment()` with `cus_name`, `amount`, and `meta_data`
4. Save the returned `id` (SkyPay session ID) into the `Payment` record's `skypay_session_id` column
5. Return JSON to the app/bot:
   - `payment_id` (your DB record ID)
   - `skypay_session_id`
   - `amount`
   - `methods` array (filtered to only include channels where the active flag is true and number is not empty)

The app/bot uses the `methods` array to display wallet numbers to the user.

### ApiPaymentController@verify

Steps:
1. Validate JSON body: `payment_id`, `method` (must be lowercase), `transaction_id`
2. Look up the `Payment` record by `payment_id`
3. Check it's not already completed (idempotency guard)
4. Call `SkyPayService::verifyHeadlessPayment($skypaySessionId, $method, $transactionId)`
5. If `status === true`:
   - Update payment record: `status = 'completed'`, `transaction_id`, `payment_method`, `verified_at`
   - Fulfill the order (activate subscription, add balance, etc.)
   - Return success JSON to the app
6. If `status === false`:
   - If message indicates SMS not yet received → return a `pending` response with a message telling the app to retry in 10 seconds
   - Otherwise return an error response

---

## Binding in AppServiceProvider

In `register()`:
```php
$this->app->singleton(SkyPayService::class, function () {
    $gateway = AutoPaymentGateway::active()->firstOrFail();
    return new SkyPayService($gateway);
});
```

---

## Error Handling

Create a custom exception `App\Exceptions\SkyPayException`. Catch it in `Handler.php` and return a JSON error response for API routes, or redirect with a flash message for web routes.

---

## Logging

Log every API call to SkyPay:
```php
Log::channel('skypay')->info('Payment create request', ['payload' => $payload]);
Log::channel('skypay')->info('Payment create response', ['response' => $response]);
```

Add a `skypay` log channel in `config/logging.php` writing to `storage/logs/skypay.log`.

---

## Method Name Rule

Always ensure the `method` value is lowercase before sending to the API. Use `strtolower()` on any user-submitted or app-submitted method name. Accepted values: `bkash`, `nagad`, `rocket`, `upay`.

---

## Complete Flow Summary (Hosted)

```
POST /payment/initiate
    → Validate → Create Payment record (pending)
    → POST /api/payment/create → get payment_url
    → Redirect user to payment_url

SkyPay redirects back to /payment/callback?transactionId=...
    → Idempotency check
    → POST /api/payment/verify
    → If COMPLETED: update record, fulfill order, redirect to success
    → If failed: update record, show error
```

## Complete Flow Summary (Headless / API)

```
App/Bot → POST /api/payment/initiate
    → POST /api/v2/payment/create → get session id + methods
    → Return methods to app/bot

App/Bot renders wallet numbers → user pays → user submits TrxID

App/Bot → POST /api/payment/verify
    → POST /api/v2/payment/verify
    → If verified: fulfill → return success
    → If pending SMS: return retry signal
```
