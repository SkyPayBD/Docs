# SkyPay - BD · Hosted Checkout Gateway

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png" width="100" alt="SkyPay Logo" />

<br/>

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789098593_de7d9d238ad2e0178762.png" width="420" alt="SkyPay Banner" />

<br/><br/>

**API v1 — Hosted Checkout Gateway**

*Complete integration reference for websites, web stores, WHMCS, WooCommerce, and web-based platforms.*

<br/>

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![GitHub](https://img.shields.io/badge/GitHub-SkyPayBD-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/SkyPayBD)

</div>

---

## 💳 Supported Payment Methods

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1720001734_bed271b1089aa12b9887.png" height="36" alt="bKash" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717231942_d63fe41b5e42176d4936.png" height="36" alt="Nagad" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717239532_10348cc78dc0b990a8e5.png" height="36" alt="Rocket" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717239551_f0b3a097df92d481e17f.png" height="36" alt="Upay" />&nbsp;&nbsp;&nbsp;
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1717513584_d7ff0294bf6b6c98db7b.png" height="36" alt="Binance" />

</div>

<br/>

| Channel | Personal (Send Money) | Agent (Cash In) | Merchant Payment |
|---|:---:|:---:|:---:|
| **bKash** | ✅ | ✅ | ✅ |
| **Nagad** | ✅ | ✅ | 🔄 Under Review |
| **Rocket** | ✅ | ✅ | 🔄 Under Review |
| **Upay** | ✅ | ❌ | 🔄 Under Review |
| **Binance Pay** | ✅ (USDT) | — | — |

---

## 📋 Table of Contents

- [What Is the Hosted Gateway?](#-what-is-the-hosted-gateway)
- [When to Use API v1](#-when-to-use-api-v1)
- [v1 vs v2 Comparison](#-v1-vs-v2-comparison)
- [Prerequisites](#-prerequisites)
- [Authentication](#-authentication)
- [API Endpoints](#-api-endpoints)
- [Integration Flow](#-end-to-end-integration-flow)
- [Step 1 — Create Payment URL](#step-1--create-hosted-payment-url)
- [Step 2 — Redirect Customer](#step-2--redirect-customer-to-payment-page)
- [Step 3 — Handle Callback](#step-3--handle-the-callback-return-url)
- [Step 4 — Verify Payment](#step-4--verify-the-payment-mandatory)
- [Step 5 — Fulfill the Order](#step-5--fulfill-the-order)
- [Code Examples](#-code-examples)
- [HTTP Status Codes](#-http-status-code-reference)
- [Security Rules](#-security-rules)
- [SMS Sync Latency](#-sms-synchronization-latency)
- [Integration Checklist](#-integration-checklist)
- [Pre-Built Modules](#-pre-built-cms-modules)
- [Contact & Support](#-contact--support)
- [Quick Links](#-quick-links)

---

## 🌐 What Is the Hosted Gateway?

**SkyPay Hosted Checkout Gateway (API v1)** is a redirect-based payment flow for Bangladesh.

Instead of building any payment UI yourself, your platform:

1. Creates a payment session via API → receives a **hosted payment URL**
2. Redirects the customer's browser to that SkyPay-hosted page
3. Customer selects their channel (bKash / Nagad / Rocket / Upay / Binance), sends money, enters their TrxID
4. SkyPay verifies the payment via SMS sync and redirects the customer back to your site
5. Your backend calls the verify endpoint and fulfills the order

---

## 🎯 When to Use API v1

Use the **Hosted Gateway (v1)** when:

- You are building a **website, online store, or web application** (WordPress, Laravel, CodeIgniter, PHP, React, Node.js, etc.)
- You want to accept payments **without building any payment UI** — SkyPay handles the full interface
- You have a **WHMCS billing panel**, **WooCommerce store**, or **SMM panel**
- Development speed is the priority — integration takes **under 15 minutes**

> ⚠️ **For Telegram bots or native mobile apps:** The Hosted Gateway uses browser redirects. For a fully in-chat or in-app experience without any redirect, use [SkyPay Headless API v2](./README_v2.md) instead.

---

## 🔄 v1 vs v2 Comparison

| Feature | Hosted Gateway (v1) | Headless API (v2) |
|---|---|---|
| User Experience | Redirected to SkyPay's hosted page | Stays inside your app / bot |
| Payment UI | Built and handled by SkyPay | You build it yourself |
| Best For | Websites, Web Stores, WHMCS, SMM Panels | Telegram Bots, Mobile Apps, SPAs |
| Setup Time | ~15 minutes | ~1–2 hours |
| Telegram Bot Support | Opens external browser link | Native in-chat flow |
| Android App Support | Requires WebView | Native in-app UI |
| Auth Header | `BRAND-KEY`, `API-KEY`, `SECRET-KEY`, or `?api_key=` | `BRAND-KEY` only |

---

## 📦 Prerequisites

### 1. Get Your BRAND-KEY

- Log in to your **SkyPay Merchant Dashboard**
- Go to **[Brand Management](https://skypaybd.top/user/brands)** → create a Brand
- Copy the generated **BRAND-KEY**
- Store it securely in your `.env` file — **never expose it in frontend code or commit it to any repository**

### 2. Connect Your Android Device

- Download and install the **[SkyPay Merchant Sync APK](https://skypaybd.top/public/assets/downloads/SkyPay.apk)** on a dedicated Android phone
- The phone must have active MFS SIM cards (bKash / Nagad / Rocket personal or agent accounts)
- Log in with your registered email and **Device Key** from [Device Management](https://skypaybd.top/user/devices)
- Grant all SMS permissions when prompted
- Disable **Battery Optimization** for the SkyPay app
- Keep this phone **powered on and connected to internet 24/7**

> Without an active connected Android device, all payment sessions will return `403 Forbidden`.

---

## 🔐 Authentication

Every API request requires the following headers:

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Accepted Auth Header Aliases (v1 only)

| Header / Parameter | Accepted? |
|---|:---:|
| `BRAND-KEY` | ✅ Recommended |
| `API-KEY` | ✅ Alias |
| `SECRET-KEY` | ✅ Alias |
| `?api_key=` (URL query param) | ✅ Alias |

> All four resolve to the same credential. Use `BRAND-KEY` as the recommended header name.

> ⚠️ **Note:** These aliases apply to **v1 only**. The v2 Headless API accepts `BRAND-KEY` header exclusively.

---

## 📡 API Endpoints

| Action | Method | Full Endpoint URL |
|---|:---:|---|
| Create Hosted Payment URL | `POST` | `https://core.skypaybd.top/api/payment/create` |
| Verify Payment | `POST` | `https://core.skypaybd.top/api/payment/verify` |

---

## 🔁 End-to-End Integration Flow

```
Customer clicks "Pay Now" on your website
        │
        ▼
[Step 1] Your backend calls:
         POST https://core.skypaybd.top/api/payment/create
         Body: { amount, success_url, cancel_url, ... }
        │
        ▼
SkyPay returns: { "status": true, "payment_url": "https://core.skypaybd.top/api/execute/..." }
        │
        ▼
[Step 2] Your backend redirects the customer's browser to payment_url
        │
        ▼
Customer lands on SkyPay's secure hosted page
Customer selects bKash / Nagad / Rocket / Upay / Binance
Customer sends money and enters their TrxID
SkyPay verifies via SMS sync (5–20 seconds)
        │
        ▼
[Step 3] SkyPay redirects customer back to your success_url or cancel_url
         with query params: ?transactionId=XXX&paymentMethod=bkash&paymentAmount=500&paymentFee=0&status=completed
        │
        ▼
[Step 4] Your backend reads transactionId from callback URL
         and calls: POST https://core.skypaybd.top/api/payment/verify
         Body: { "transaction_id": "XXX" }
        │
        ▼
[Step 5] Verify response returns data.status = "COMPLETED"
         → Safely fulfill the order
```

---

## Step 1 — Create Hosted Payment URL

**`POST https://core.skypaybd.top/api/payment/create`**

### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Parameters

| Parameter | Type | Status | Description |
|---|---|---|---|
| `amount` | Numeric | **Required** | Payable amount in BDT. Must be numeric, between `1` and `1,000,000`. Integer or decimal accepted. |
| `success_url` | String (URL) | **Required** | Valid URL to redirect the customer after successful payment. Must be a valid URL format. |
| `cancel_url` | String (URL) | **Required** | Valid URL to redirect the customer if they cancel or fail to complete payment. Must be a valid URL format. |
| `cus_name` | String | Optional | Customer full name. Aliases: `customer_name`, `c_name`, `name`. Defaults to `Default Name` if not provided. |
| `cus_email` | String | Optional | Customer email address. Aliases: `customer_email`, `c_email`, `email`. Defaults to `default@gmail.com` if not provided. |
| `meta_data` | Object / JSON | Optional | Any JSON object with custom data (e.g. `order_id`, `user_id`). Alias: `metadata`. Returned as-is in the `/verify` response — use this to identify which order to fulfill. |
| `webhook_url` | String (URL) | Optional | If provided, SkyPay sends an automated server-to-server POST to this URL immediately upon payment completion. Must be a valid URL format. |
| `return_type` | String | Optional | HTTP method used for redirect callbacks. `GET` or `POST`. Defaults to `GET`. |

> **`meta_data` tip:** Pass your internal `order_id` or `user_id` in `meta_data` during `/create`. It will be returned in the `/verify` response inside `data.meta_data`, letting you instantly identify the order without any extra database lookup.

### Example Request Body

```json
{
  "amount": 500,
  "success_url": "https://mystore.com/payment/success",
  "cancel_url": "https://mystore.com/payment/cancel",
  "webhook_url": "https://mystore.com/api/payment-webhook",
  "cus_name": "John Doe",
  "cus_email": "john@example.com",
  "return_type": "GET",
  "meta_data": {
    "order_id": "ORD-10928",
    "user_id": "USR-4821",
    "plan": "PRO_MONTHLY"
  }
}
```

### Success Response (HTTP 200)

```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/api/execute/f89f359b182a25f58d57ae322c956c29"
}
```

### Response Fields

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | `true` = session created successfully |
| `message` | String | Human-readable status message |
| `payment_url` | String (URL) | SkyPay-hosted checkout URL. **Redirect your customer here immediately.** |

### Error Responses

| HTTP Code | Message | Cause |
|---|---|---|
| `422` | `The amount field is required.` | `amount` field missing |
| `422` | `The amount must be a valid numeric value.` | Non-numeric amount provided |
| `422` | `Amount out of range. Allowed limit is between 1 and 1,000,000.` | Amount outside valid range |
| `422` | `The success_url field is required.` | Missing `success_url` |
| `422` | `The success_url must be a valid URL format.` | Invalid URL format for `success_url` |
| `422` | `The cancel_url field is required.` | Missing `cancel_url` |
| `422` | `The cancel_url must be a valid URL format.` | Invalid URL format for `cancel_url` |
| `422` | `The webhook_url must be a valid URL format.` | Invalid URL format for `webhook_url` (if provided) |
| `422` | `The meta_data field must be a valid JSON object.` | Malformed `meta_data` JSON string |
| `401` | `API key is missing. Please provide API-KEY, BRAND-KEY or SECRET-KEY in headers or api_key parameter.` | No auth header provided |
| `403` | `Invalid or inactive API credentials provided.` | Wrong or inactive BRAND-KEY |
| `500` | `Failed to initialize transaction record in database.` | Server-side DB error |

---

## Step 2 — Redirect Customer to Payment Page

After receiving `payment_url` from Step 1, **immediately redirect the customer's browser** to that URL.

> Each `payment_url` is tied to a single payment session — do not store or reuse it.

### Redirect by Platform

**PHP**
```php
header("Location: " . $data['payment_url']);
exit;
```

**Laravel**
```php
return redirect($data['payment_url']);
```

**CodeIgniter 4**
```php
return redirect()->to($data['payment_url']);
```

**Node.js / Express**
```javascript
res.redirect(data.payment_url);
```

**JavaScript (Frontend)**
```javascript
window.location.href = data.payment_url;
```

**Telegram Bot (Inline Button)**
```
Send an inline keyboard button with payment_url as the URL:
[💳 Pay Now — Open Payment Page]
```

The customer will land on SkyPay's secure hosted page where they:
1. Choose their MFS channel (bKash / Nagad / Rocket / Upay / Binance)
2. Copy the merchant number shown on screen
3. Send the exact amount from their MFS app
4. Enter the TrxID (or Binance Order ID) on SkyPay's page and click Verify
5. SkyPay matches the TrxID against incoming SMS (5–20 seconds)
6. On successful match → customer is redirected to your `success_url`

---

## Step 3 — Handle the Callback (Return URL)

After payment is completed or cancelled, SkyPay redirects the customer back to your `success_url` or `cancel_url` with result details as URL query parameters.

### Example Redirect — Successful Payment

```
https://mystore.com/payment/success?paymentMethod=bkash&transactionId=TRXXXXXXXXX&paymentAmount=500&paymentFee=0&status=completed
```

### Example Redirect — Cancelled / Failed Payment

```
https://mystore.com/payment/cancel?paymentMethod=undetected&transactionId=TRXXXXXXXXX&paymentAmount=500&paymentFee=0&status=failed
```

### Callback Query Parameters

| Parameter | Type | Description |
|---|---|---|
| `transactionId` | String | The gateway-assigned transaction ID. **Always present** regardless of outcome. Use this to call `/verify`. |
| `paymentMethod` | String | Channel used: `bkash`, `nagad`, `rocket`, `upay`, `binance`. Returns `undetected` if customer did not complete payment. |
| `paymentAmount` | Numeric | The BDT amount set during session creation. |
| `paymentFee` | Numeric | Gateway fee. Returns `0` if no fee applies. |
| `status` | String | `completed` = payment verified. `failed` = cancelled or not completed. |

### `status` Values

| Value | Meaning | Action |
|---|---|---|
| `completed` | Customer paid and SkyPay verified via SMS sync | Proceed to Step 4 (verify via API) before fulfilling |
| `failed` | Customer cancelled or did not complete payment | Do not fulfill; redirect to error/retry page |

> ⚠️ **CRITICAL:** Never fulfill an order based on callback URL parameters alone. The `status=completed` in the URL can be manually crafted by anyone. **Always verify via `POST /api/payment/verify` from your backend (Step 4) before fulfilling any order.**

### Reading Callback Parameters

**PHP**
```php
$transactionId = $_GET['transactionId'] ?? null;
$status        = $_GET['status']        ?? null;

if ($status === 'completed' && $transactionId) {
    // Do NOT fulfill here — verify first (Step 4)
    verifyPayment($transactionId);
}
```

**Laravel**
```php
public function success(Request $request)
{
    $transactionId = $request->query('transactionId');
    $status        = $request->query('status');

    if ($status === 'completed' && $transactionId) {
        $this->verifyPayment($transactionId);
    }
}
```

**Node.js / Express**
```javascript
app.get('/payment/success', async (req, res) => {
    const { transactionId, status } = req.query;
    if (status === 'completed' && transactionId) {
        await verifyPayment(transactionId);
    }
});
```

---

## Step 4 — Verify the Payment (Mandatory)

> ### ⚠️ THIS STEP IS MANDATORY — SKIPPING IT IS A CRITICAL SECURITY VULNERABILITY

After receiving the callback, your backend must call the verify endpoint to confirm the transaction is genuinely completed in SkyPay's database.

**`POST https://core.skypaybd.top/api/payment/verify`**

### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Parameters

| Parameter | Type | Status | Description |
|---|---|---|---|
| `transaction_id` | String | **Required** | The `transactionId` from the callback URL query parameter. Aliases: `transactionId`, `transactionid`, `trx_id`, `trx`, `transaction`. |

### Example Request Body

```json
{
  "transaction_id": "TRXXXXXXXXX"
}
```

### Success Response — Payment Completed (HTTP 200)

```json
{
  "status": true,
  "data": {
    "cus_name": "John Doe",
    "cus_email": "john@example.com",
    "amount": "500.000",
    "transaction_id": "TRXXXXXXXXX",
    "meta_data": {
      "order_id": "ORD-10928",
      "user_id": "USR-4821",
      "plan": "PRO_MONTHLY"
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

### Success Response — Payment Failed (HTTP 200)

```json
{
  "status": true,
  "data": {
    "cus_name": "John Doe",
    "cus_email": "john@example.com",
    "amount": "500.000",
    "transaction_id": "TRXXXXXXXXX",
    "meta_data": {
      "order_id": "ORD-10928",
      "user_id": "USR-4821"
    },
    "payment_method": null,
    "status": "FAILED"
  }
}
```

### Response Fields

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | Top-level `true` = request processed. Check `data.status` for actual payment result. |
| `data.status` | String | Actual payment status: `COMPLETED`, `PENDING`, or `FAILED` |
| `data.cus_name` | String | Customer name as provided during `/create` |
| `data.cus_email` | String | Customer email as provided during `/create` |
| `data.amount` | String | Exact amount in BDT (returned as string with 3 decimal places) |
| `data.transaction_id` | String | The verified transaction ID |
| `data.payment_method` | String | Channel used: `bkash`, `nagad`, `rocket`, `upay`, `binance`. Returns `null` if payment was not completed. |
| `data.meta_data` | Object | The exact `meta_data` object passed during `/create` |

### `data.status` Values

| Value | Internal Code | Meaning | Action |
|---|:---:|---|---|
| `COMPLETED` | `2` | Payment confirmed via SMS sync | ✅ Safely fulfill the order |
| `PENDING` | `1` | Payment submitted but SMS not yet matched | ⏳ Retry after 10–15 seconds |
| `FAILED` | `0` or other | Not paid, cancelled, or session just initialized | ❌ Do not fulfill |

> **Important:** `FAILED` is returned for any internal status that is not `1` (PENDING) or `2` (COMPLETED). A newly created session that was never paid will also return `FAILED`.
>
> **Always check BOTH:** top-level `status === true` AND `data.status === "COMPLETED"` before fulfilling.

### Verify Error Responses

| HTTP Code | Message | Cause |
|---|---|---|
| `422` | `Transaction identifier is required. Provide transaction_id, transactionId, or trx_id.` | Missing transaction ID field |
| `404` | `No transaction record found with the provided transaction identifier.` | Transaction ID not found in database |
| `401` | `API key is missing. Please provide API-KEY, BRAND-KEY or SECRET-KEY in headers or api_key parameter.` | No auth header provided |
| `403` | `Invalid or inactive API credentials provided.` | Wrong or inactive BRAND-KEY |

---

## Step 5 — Fulfill the Order

Once verify returns `status: true` at the top level **AND** `data.status: "COMPLETED"`, safely fulfill the order.

**Examples:**
- Add verified `amount` to the user's wallet or credit balance
- Activate or extend the user's subscription plan
- Mark the invoice as Paid in your billing system
- Trigger server/hosting provisioning (WHMCS/cPanel)
- Unlock premium content or digital goods
- Send a payment confirmation email or Telegram message

**Always execute fulfillment on your backend server. Never grant benefits based on frontend data.**

---

## 💻 Code Examples

### PHP (cURL) — Create

```php
$ch = curl_init();
curl_setopt_array($ch, [
    CURLOPT_URL            => 'https://core.skypaybd.top/api/payment/create',
    CURLOPT_POST           => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER     => [
        'BRAND-KEY: ' . getenv('SKYPAY_BRAND_KEY'),
        'Content-Type: application/json',
    ],
    CURLOPT_POSTFIELDS => json_encode([
        'amount'      => 500,
        'success_url' => 'https://mystore.com/payment/success',
        'cancel_url'  => 'https://mystore.com/payment/cancel',
        'cus_name'    => 'John Doe',
        'cus_email'   => 'john@example.com',
        'meta_data'   => ['order_id' => 'ORD-10928'],
    ]),
]);
$response = curl_exec($ch);
curl_close($ch);
$data = json_decode($response, true);

if ($data['status']) {
    header("Location: " . $data['payment_url']);
    exit;
}
```

### PHP (cURL) — Verify

```php
$ch = curl_init();
curl_setopt_array($ch, [
    CURLOPT_URL            => 'https://core.skypaybd.top/api/payment/verify',
    CURLOPT_POST           => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER     => [
        'BRAND-KEY: ' . getenv('SKYPAY_BRAND_KEY'),
        'Content-Type: application/json',
    ],
    CURLOPT_POSTFIELDS => json_encode([
        'transaction_id' => $_GET['transactionId'],
    ]),
]);
$response = curl_exec($ch);
curl_close($ch);
$data = json_decode($response, true);

if ($data['status'] === true && $data['data']['status'] === 'COMPLETED') {
    $orderId = $data['data']['meta_data']['order_id'] ?? null;
    fulfillOrder($orderId, $data['data']['amount']);
} else {
    showError("Payment could not be verified.");
}
```

### Laravel — Create

```php
$response = Http::withHeaders([
    'BRAND-KEY'    => env('SKYPAY_BRAND_KEY'),
    'Content-Type' => 'application/json',
])->post('https://core.skypaybd.top/api/payment/create', [
    'amount'      => $order->total,
    'success_url' => route('payment.success'),
    'cancel_url'  => route('payment.cancel'),
    'cus_name'    => $user->name,
    'cus_email'   => $user->email,
    'meta_data'   => ['order_id' => $order->id],
]);

$data = $response->json();
if ($data['status']) {
    return redirect($data['payment_url']);
}
```

### Python — Create & Verify

```python
import requests

BRAND_KEY = "your_brand_key_here"
HEADERS   = {"BRAND-KEY": BRAND_KEY, "Content-Type": "application/json"}

# Step 1 — Create
res = requests.post(
    "https://core.skypaybd.top/api/payment/create",
    headers=HEADERS,
    json={
        "amount":      500,
        "success_url": "https://mystore.com/payment/success",
        "cancel_url":  "https://mystore.com/payment/cancel",
        "meta_data":   {"order_id": "ORD-10928"},
    }
)
data = res.json()
if data.get("status"):
    print("Redirect to:", data["payment_url"])

# Step 4 — Verify
res = requests.post(
    "https://core.skypaybd.top/api/payment/verify",
    headers=HEADERS,
    json={"transaction_id": "TRXXXXXXXXX"}
)
data = res.json()
if data.get("status") and data["data"]["status"] == "COMPLETED":
    print("Verified! Order:", data["data"]["meta_data"].get("order_id"))
```

### Node.js — Verify

```javascript
const axios = require('axios');

const headers = {
    'BRAND-KEY':    process.env.SKYPAY_BRAND_KEY,
    'Content-Type': 'application/json'
};

const { data } = await axios.post(
    'https://core.skypaybd.top/api/payment/verify',
    { transaction_id: req.query.transactionId },
    { headers }
);

if (data.status === true && data.data.status === 'COMPLETED') {
    const orderId = data.data.meta_data?.order_id;
    await fulfillOrder(orderId, data.data.amount);
}
```

---

## 🚦 HTTP Status Code Reference

| HTTP Code | Meaning | What To Do |
|---|---|---|
| `200` | Request processed | Check `status` and `data.status` in the response body |
| `400` | Bad request / invalid endpoint | Read the `message` field |
| `401` | Missing or invalid auth key | Check your BRAND-KEY |
| `403` | Invalid credentials or inactive account | Verify key in dashboard |
| `404` | Transaction not found | Transaction ID does not exist |
| `405` | Wrong HTTP method | Use `POST` for all endpoints |
| `422` | Validation error on a field | Check URL format, amount range, or meta_data JSON |
| `500` | Server-side database error | Retry or contact support |

---

## 🔒 Security Rules

| Rule | Description |
|---|---|
| **Never trust callback URL params** | `?transactionId=...&status=completed` can be faked by anyone. Always verify via `/api/payment/verify` before fulfilling. |
| **Server-side only** | All API calls (`/create` and `/verify`) must come from your backend server — never from client-side JavaScript. |
| **Protect your BRAND-KEY** | Store in `.env` or environment variables. Never hardcode in source code or commit to any repository. |
| **Idempotency** | Once a `transaction_id` is verified as `COMPLETED`, implement your own database check to prevent fulfilling the same order twice. |
| **Check both status fields** | Check `status === true` (top-level) AND `data.status === "COMPLETED"` before fulfilling. A `PENDING` or `FAILED` value means do NOT fulfill. |

---

## ⏱️ SMS Synchronization Latency

When a customer sends money and submits their TrxID on SkyPay's hosted page, SkyPay verifies it against incoming SMS on your merchant Android phone. This takes **5 to 20 seconds** and is handled entirely on SkyPay's hosted page.

By the time SkyPay redirects the customer back to your `success_url`, verification is usually already complete. However, in rare cases (network delays, slow SMS delivery), the transaction may still show `PENDING` when you call `/verify`:

- Retry the `/verify` call after 10–15 seconds
- Allow a maximum grace period of 2–3 minutes
- If still `PENDING` after 3 minutes, mark the order as pending and follow up manually or contact support

---

## ✅ Integration Checklist

Before going live, confirm all of the following:

- [ ] BRAND-KEY stored securely in `.env` — not in source code or frontend
- [ ] Merchant Android phone is online and SkyPay APK is running
- [ ] Battery optimization disabled for SkyPay APK on the merchant phone
- [ ] `success_url` and `cancel_url` are valid, publicly accessible URLs (not `localhost`)
- [ ] Customer is redirected to `payment_url` immediately after `/create` response
- [ ] Your `success_url` handler reads `transactionId` from the query parameters
- [ ] Backend `POST /api/payment/verify` is called before any order fulfillment
- [ ] Order is fulfilled only when `status === true` AND `data.status === "COMPLETED"`
- [ ] Database check prevents double-fulfillment for the same `transaction_id`
- [ ] `meta_data` includes your order/user reference so you know what to fulfill

---

## 📦 Pre-Built CMS Modules

| Module | Platform | Download |
|---|---|---|
| WordPress & WooCommerce Plugin | WordPress | [Download .zip](https://skypaybd.top/public/assets/downloads/WP.zip) |
| WHMCS Billing Module | WHMCS 7.x & 8.x | [Download .zip](https://skypaybd.top/public/assets/downloads/WHMCS.zip) |
| SMM Panel Auto-Deposit | SmartPanel / SMM Scripts | [Download .zip](https://skypaybd.top/public/assets/downloads/SMM.zip) |
| Sketchware SWB Project | Sketchware Mobile | [Download .swb](https://skypaybd.top/public/assets/downloads/Apps.swb) |
| SkyPay Merchant Sync APK | Android | [Download APK](https://skypaybd.top/public/assets/downloads/SkyPay.apk) |
| Developer Offline Docs | All Platforms | [Download .zip](https://skypaybd.top/public/assets/downloads/dev.zip) |

---

## 📞 Contact & Support

<div align="center">

| Channel | Link |
|---|---|
| 🌐 **Website** | [skypaybd.top](https://skypaybd.top) |
| 📧 **Email** | [support@skypaybd.top](mailto:support@skypaybd.top) |
| 📞 **Phone / Call** | [+880 9696 014968](tel:+8809696014968) |
| 💬 **WhatsApp** | [+880 1761 844968](https://wa.me/8801761844968) |
| ✈️ **Telegram** | [@BD_Prime_Minister](https://t.me/BD_Prime_Minister) |
| 📘 **Facebook** | [facebook.com/hyper.10.squad](https://www.facebook.com/hyper.10.squad) |
| 🐙 **GitHub** | [github.com/SkyPayBD](https://github.com/SkyPayBD) |
| 📺 **YouTube** | [youtube.com/@Sky-Pay-BD](https://youtube.com/@Sky-Pay-BD) |

🕐 **Operating Hours:** Saturday – Thursday, 09:00 AM – 10:00 PM (BST)

📍 **Location:** Panchagarh, Rangpur, Dhaka, Bangladesh

</div>

---

## 🔗 Quick Links

| Resource | URL |
|---|---|
| 🔑 Login | [skypaybd.top/sign-in](https://skypaybd.top/sign-in) |
| 📝 Register | [skypaybd.top/sign-up](https://skypaybd.top/sign-up) |
| 🔒 Forgot Password | [skypaybd.top/password-reset](https://skypaybd.top/password-reset) |
| 🏷️ Brand Management | [skypaybd.top/user/brands](https://skypaybd.top/user/brands) |
| 💳 Wallet Management | [skypaybd.top/user/user-settings/wallets](https://skypaybd.top/user/user-settings/wallets) |
| 📱 Device Management | [skypaybd.top/user/devices](https://skypaybd.top/user/devices) |
| 📦 Subscription Plans | [skypaybd.top/user/plans](https://skypaybd.top/user/plans) |
| 📄 Privacy Policy | [skypaybd.top/legal#privacy-policy](https://skypaybd.top/legal#privacy-policy) |
| 📋 Terms of Service | [skypaybd.top/legal#terms](https://skypaybd.top/legal#terms) |
| 💰 Refund Policy | [skypaybd.top/legal#refund-policy](https://skypaybd.top/legal#refund-policy) |
| 💵 Pricing | [skypaybd.top/#pricing](https://skypaybd.top/#pricing) |
| ❓ FAQ | [skypaybd.top/#faq](https://skypaybd.top/#faq) |

---

<div align="center">

*© 2024–2026 SkyPay BD. All rights reserved.*

</div>
