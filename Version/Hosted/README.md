# SkyPay Hosted Checkout Gateway — API v1 Complete Integration Guide

> **For Developers, Web Builders & Platform Integrators**
> This document is the complete reference for integrating SkyPay's **Hosted Checkout Gateway (API v1)** into any website, web application, or Telegram bot — where the user is redirected to SkyPay's secure hosted payment page to complete their transaction.

---

## What Is the Hosted Gateway?

SkyPay Hosted Checkout Gateway (v1) is a **redirect-based payment flow** for Bangladesh.

Instead of building any payment UI yourself, your platform:
1. Creates a payment session via API and receives a **hosted payment URL**
2. Redirects the customer's browser to that SkyPay-hosted payment page
3. The customer selects their payment channel (bKash / Nagad / Rocket / Upay), sends money, and enters their TrxID on SkyPay's page
4. SkyPay verifies the payment and redirects the customer back to your website with a result
5. Your backend verifies the transaction via API and fulfills the order

**Supported Payment Channels:** bKash · Nagad · Rocket · Upay

---

## When Should You Use This API?

Use the **Hosted Gateway (v1)** when:

- You are building a **website, online store, or web application** (WordPress, Laravel, CodeIgniter, raw PHP, React, Node.js, etc.)
- You want to accept payments **without designing any payment UI** — SkyPay handles the entire payment interface for you
- You have a **WHMCS billing panel**, **WooCommerce store**, **SMM panel**, or similar web-based platform
- Development speed is the priority — this can be integrated in **under 15 minutes**

> ⚠️ **Note for Android / Mobile App Developers:**
> The Hosted Gateway relies on browser redirects. This works inside a WebView but is not ideal for native Android or iOS apps where there is no browser context. For native mobile apps, consider using [SkyPay Headless API v2](./SkyPay_Headless_API_v2.md) instead, which gives you complete control over the in-app payment experience without any redirects.

---

## How Is This Different from API v2?

| Feature | Hosted Gateway (v1) | Headless API (v2) |
|---|---|---|
| User Experience | Redirected to SkyPay's page | Stays inside your app/bot |
| Payment UI | Built by SkyPay | You build it yourself |
| Best For | Websites, Web Stores, WHMCS | Telegram Bots, Mobile Apps, SPAs |
| Setup Time | ~15 minutes | 1–2 hours |
| Telegram Bot Support | Needs to open browser link | Native in-chat flow |
| Android App Support | Requires WebView | Native in-app UI |

---

## Prerequisites (Before You Start)

### 1. BRAND-KEY
- Log into your **SkyPay Merchant Dashboard**
- Create a **Brand** under your merchant account
- Copy the generated **BRAND-KEY**
- This key ties all payment sessions to your merchant account and connected Android device
- **Store it securely in your `.env` file or secret manager — never expose it in frontend code or commit it to any repository**

### 2. Merchant Android Device (SMS Sync Bridge)
- Install the **SkyPay Merchant Sync APK** on a dedicated Android smartphone (Android 7.0+)
- The phone must contain your active merchant SIM cards (bKash, Nagad, Rocket, Upay)
- Grant **SMS Listener Permission** and **Notification Access**
- Disable **Battery Optimization** for the SkyPay app so it stays alive in the background
- Enter your **BRAND-KEY** inside the app and tap **Connect Device**
- Keep this phone **powered on and connected to the internet 24/7**

> Without the connected Android device, all payment sessions will return `403 Forbidden`.
> The Android phone reads your incoming MFS payment SMS notifications and pushes them to SkyPay's cloud server in real-time. This is what makes automated verification possible.

---

## API Base URL

All v1 Hosted Gateway calls go to:

```
https://core.skypaybd.top
```

---

## Authentication

Every API request requires these headers:

```
BRAND-KEY: your_unique_brand_key_here
Content-Type: application/json
```

> **No `SECRET-KEY` is required.** SkyPay v1 and v2 both use only your `BRAND-KEY` for authentication.

---

## Full Endpoint Directory

| Action | Method | Full Endpoint URL |
|---|---|---|
| Create Hosted Payment URL | `POST` | `https://core.skypaybd.top/api/payment/create` |
| Verify Payment Order | `POST` | `https://core.skypaybd.top/api/payment/verify` |

---

## End-to-End Integration Flow

```
Customer on your website clicks "Pay Now" or "Checkout"
          │
          ▼
[Step 1] Your backend server calls:
         POST https://core.skypaybd.top/api/payment/create
         with: cus_name, cus_email, amount, success_url, cancel_url, metadata
          │
          ▼
SkyPay returns: { "status": true, "payment_url": "https://core.skypaybd.top/checkout/..." }
          │
          ▼
[Step 2] Your server redirects the customer's browser to that payment_url
          │
          ▼
Customer lands on SkyPay's secure hosted payment page
Customer selects bKash / Nagad / Rocket / Upay
Customer sends money from their MFS app
Customer enters their SMS Transaction ID (TrxID) on the SkyPay page
SkyPay verifies the TrxID automatically (5–20 seconds)
          │
          ▼
[Step 3] SkyPay redirects the customer back to your success_url
         with result query parameters in the URL:
         ?transactionId=BLA38KDK2M&paymentMethod=bkash&paymentAmount=500.00&status=completed
          │
          ▼
[Step 4] Your backend reads the transactionId from the callback URL
         and calls POST https://core.skypaybd.top/api/payment/verify
         to confirm the transaction is genuinely completed
          │
          ▼
[Step 5] Verification confirmed → Fulfill the order
         (activate subscription, add balance, ship product, etc.)
```

---

## Real-World Integration Examples

### Example 1 — Standard Website / PHP Store

```
1. Customer adds items to cart and proceeds to checkout.
2. Customer selects "SkyPay" as payment method and clicks "Place Order".
3. Your PHP backend sends a POST to /api/payment/create with order details.
4. SkyPay returns a payment_url.
5. Your backend does: header("Location: " . $payment_url); exit;
6. Customer lands on SkyPay's hosted payment page.
7. Customer pays via bKash and enters TrxID on SkyPay's page.
8. SkyPay verifies and redirects customer to your success_url:
   https://yourstore.com/checkout/success?transactionId=BLA38KDK2M&status=completed
9. Your success_url handler calls /api/payment/verify with the transactionId.
10. Verified → order marked as paid, customer sees success screen.
```

### Example 2 — Telegram Bot (with browser redirect)

```
1. User sends /deposit 500 to the bot.
2. Bot's backend calls /api/payment/create and gets a payment_url.
3. Bot sends the user an inline button: [💳 Pay 500 BDT — Click Here]
   The button opens the payment_url in the user's phone browser.
4. User completes payment on SkyPay's hosted page.
5. SkyPay redirects the user to your success_url.
6. Your success_url backend verifies the transaction and credits the user's bot account.
7. (Optional) Bot sends a confirmation message to the user's Telegram chat.
```

> **Note:** For a fully in-chat Telegram bot experience without any browser redirect, use [Headless API v2](./SkyPay_Headless_API_v2.md).

### Example 3 — Laravel Application

```php
// In your PaymentController.php
$response = Http::withHeaders([
    'BRAND-KEY'    => env('SKYPAY_BRAND_KEY'),
    'Content-Type' => 'application/json',
])->post('https://core.skypaybd.top/api/payment/create', [
    'cus_name'    => $user->name,
    'cus_email'   => $user->email,
    'amount'      => $order->total,
    'success_url' => route('payment.success'),
    'cancel_url'  => route('payment.cancel'),
    'metadata'    => ['order_id' => $order->id],
]);

$data = $response->json();

if ($data['status']) {
    return redirect($data['payment_url']);
} else {
    return back()->withErrors(['payment' => 'Payment initiation failed.']);
}
```

### Example 4 — CodeIgniter 4

```php
// In your Payment.php controller
$client = \Config\Services::curlrequest();

$response = $client->post('https://core.skypaybd.top/api/payment/create', [
    'headers' => [
        'BRAND-KEY'    => getenv('SKYPAY_BRAND_KEY'),
        'Content-Type' => 'application/json',
    ],
    'json' => [
        'cus_name'    => $this->request->getPost('name'),
        'cus_email'   => $this->request->getPost('email'),
        'amount'      => $this->request->getPost('amount'),
        'success_url' => base_url('payment/success'),
        'cancel_url'  => base_url('payment/cancel'),
        'metadata'    => ['user_id' => session()->get('user_id')],
    ],
]);

$data = json_decode($response->getBody(), true);

if ($data['status']) {
    return redirect()->to($data['payment_url']);
}
```

---

## Step 1 — Create Hosted Payment URL

### Endpoint

```
POST https://core.skypaybd.top/api/payment/create
```

### Request Headers

```
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Body Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `cus_name` | String | ✅ Yes | Full name of the customer |
| `cus_email` | String | ✅ Yes | Valid email address of the customer |
| `amount` | Numeric | ✅ Yes | Payable amount in BDT (integer or up to 2 decimal places, must be greater than 0) |
| `success_url` | String (URL) | ✅ Yes | The full URL where the customer will be redirected after successful payment |
| `cancel_url` | String (URL) | ✅ Yes | The full URL where the customer will be redirected if they cancel or abandon payment |
| `metadata` | Object | ❌ Optional | Any custom key-value data you want to attach (order ID, user ID, cart reference, plan name, etc.) |

> **Tip on `metadata`:** Whatever you pass in `metadata` during `/create` will be returned back to you in the `/verify` response. Use it to store your internal order or user reference so you can identify which order to fulfill after verification — without storing the session in a database.

### Example Request Body

```json
{
  "cus_name": "Siyam Ahmed",
  "cus_email": "siyam@example.com",
  "amount": 500,
  "success_url": "https://mystore.com/payment/success",
  "cancel_url": "https://mystore.com/payment/cancel",
  "metadata": {
    "order_id": "ORD-10928",
    "user_id": "USR-4821",
    "plan": "PRO_MONTHLY"
  }
}
```

### Example Request (cURL)

```bash
curl -X POST https://core.skypaybd.top/api/payment/create \
  -H "BRAND-KEY: your_brand_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "cus_name": "Siyam Ahmed",
    "cus_email": "siyam@example.com",
    "amount": 500,
    "success_url": "https://mystore.com/payment/success",
    "cancel_url": "https://mystore.com/payment/cancel",
    "metadata": {
      "order_id": "ORD-10928",
      "user_id": "USR-4821"
    }
  }'
```

### Example Request (Python)

```python
import requests

url = "https://core.skypaybd.top/api/payment/create"

headers = {
    "BRAND-KEY": "your_brand_key_here",
    "Content-Type": "application/json"
}

payload = {
    "cus_name": "Siyam Ahmed",
    "cus_email": "siyam@example.com",
    "amount": 500,
    "success_url": "https://mystore.com/payment/success",
    "cancel_url": "https://mystore.com/payment/cancel",
    "metadata": {
        "order_id": "ORD-10928",
        "user_id": "USR-4821"
    }
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data.get("status"):
    print("Redirect customer to:", data.get("payment_url"))
else:
    print("Error:", data.get("message"))
```

### Success Response (HTTP 200)

```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/checkout/order/f7b3a9c2d1e04856"
}
```

### Response Fields Explained

| Field | Type | Description |
|---|---|---|
| `status` | Boolean | `true` = payment session created successfully; `false` = something went wrong |
| `message` | String | Human-readable status message |
| `payment_url` | String (URL) | The secure SkyPay-hosted checkout URL — **redirect your customer to this URL immediately** |

### Error Responses

| HTTP Code | Response | Cause |
|---|---|---|
| `400` | `{"status": false, "message": "..."}` | Missing required field (`cus_name`, `cus_email`, `amount`, `success_url`, or `cancel_url`) |
| `401` | `{"status": false, "message": "Invalid or inactive BRAND-KEY provided."}` | Wrong or expired BRAND-KEY |
| `403` | (Device not connected) | Merchant Android phone is offline or APK is not running |

---

## Step 2 — Redirect Customer to Payment Page

After receiving the `payment_url` from Step 1, **immediately redirect the customer's browser** to that URL. Do not store or reuse the URL — each URL is tied to a single payment session.

### Redirect Methods by Platform

**PHP (Standard)**
```php
header("Location: " . $data['payment_url']);
exit;
```

**Laravel**
```php
return redirect($data['payment_url']);
```

**Node.js / Express**
```javascript
res.redirect(data.payment_url);
```

**HTML / JavaScript (Frontend, only if backend passed URL)**
```javascript
window.location.href = data.payment_url;
```

**Telegram Bot (Inline Button)**
```
Send an inline keyboard button with the payment_url as the URL:
[💳 Pay Now — Click to Open Payment Page]
```

The customer will land on SkyPay's secure, hosted payment page where they:
1. Choose their preferred MFS channel (bKash / Nagad / Rocket / Upay)
2. Copy the merchant number shown on screen
3. Open their MFS app and send the exact amount
4. Receive an SMS with a Transaction ID (TrxID)
5. Enter that TrxID on the SkyPay page and click Verify
6. SkyPay automatically matches the TrxID against incoming SMS (5–20 seconds)
7. On successful match, customer is redirected to your `success_url`

---

## Step 3 — Handle the Callback (Customer Return URL)

After the payment is completed, SkyPay redirects the customer back to your `success_url` with result details appended as **URL query parameters**.

### Example Callback URL

```
https://mystore.com/payment/success?transactionId=BLA38KDK2M&paymentMethod=bkash&paymentAmount=500.00&paymentFee=0.00&status=completed
```

### Callback Query Parameters

| Parameter | Type | Example | Description |
|---|---|---|---|
| `transactionId` | String | `BLA38KDK2M` | The SMS Transaction ID that was verified |
| `paymentMethod` | String | `bkash` | The MFS channel used (`bkash`, `nagad`, `rocket`, `upay`) |
| `paymentAmount` | Numeric | `500.00` | The net amount paid in BDT |
| `paymentFee` | Numeric | `0.00` | Gateway fee applied (if any) |
| `status` | String | `completed` | Payment outcome: `completed`, `pending`, or `failed` |

### How to Read Callback Parameters

**PHP**
```php
$transactionId  = $_GET['transactionId']  ?? null;
$paymentMethod  = $_GET['paymentMethod']  ?? null;
$paymentAmount  = $_GET['paymentAmount']  ?? null;
$status         = $_GET['status']         ?? null;

if ($status === 'completed' && $transactionId) {
    // ⚠️ Do NOT fulfill the order yet!
    // Always verify via API first (Step 4)
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
        // ⚠️ Always verify via API — never trust callback params alone
        $this->verifyPayment($transactionId);
    }
}
```

**Node.js / Express**
```javascript
app.get('/payment/success', async (req, res) => {
    const { transactionId, status, paymentMethod, paymentAmount } = req.query;

    if (status === 'completed' && transactionId) {
        // ⚠️ Verify first, then fulfill
        await verifyPayment(transactionId);
    }
});
```

> ## 🚨 CRITICAL SECURITY WARNING — READ THIS
>
> **NEVER fulfill an order or credit a user's account based solely on the callback URL parameters.**
>
> URL query parameters (`?transactionId=...&status=completed`) are visible in the browser address bar and can be **manually crafted or tampered with** by malicious users. An attacker can simply type `?transactionId=FAKE123&status=completed` in their browser to trigger your success handler if you don't verify.
>
> **The only safe way to confirm a payment is to call `/api/payment/verify` from your backend server (Step 4) and check that the response returns `"status": "COMPLETED"` from SkyPay's database.**

---

## Step 4 — Verify the Payment (Backend API Call)

> ### ⚠️ THIS STEP IS MANDATORY — SKIPPING IT IS A CRITICAL SECURITY VULNERABILITY

After receiving the callback, your backend must **immediately call** the verify endpoint to confirm that the transaction is genuinely completed in SkyPay's database.

### Endpoint

```
POST https://core.skypaybd.top/api/payment/verify
```

### Request Headers

```
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Request Body Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `transaction_id` | String | ✅ Yes | The `transactionId` received from the callback URL query parameter |

### Example Request Body

```json
{
  "transaction_id": "BLA38KDK2M"
}
```

### Example Request (cURL)

```bash
curl -X POST https://core.skypaybd.top/api/payment/verify \
  -H "BRAND-KEY: your_brand_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction_id": "BLA38KDK2M"
  }'
```

### Example Request (Python)

```python
import requests

url = "https://core.skypaybd.top/api/payment/verify"

headers = {
    "BRAND-KEY": "your_brand_key_here",
    "Content-Type": "application/json"
}

payload = {
    "transaction_id": "BLA38KDK2M"
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data.get("status") == "COMPLETED":
    print("Payment verified!")
    print("Customer:", data.get("cus_name"))
    print("Amount:", data.get("amount"))
    print("Method:", data.get("payment_method"))
    print("Order ID from metadata:", data.get("metadata", {}).get("order_id"))
else:
    print("Payment not confirmed:", data.get("status"))
```

### Example Request (PHP)

```php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://core.skypaybd.top/api/payment/verify');
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'BRAND-KEY: ' . getenv('SKYPAY_BRAND_KEY'),
    'Content-Type: application/json',
]);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode([
    'transaction_id' => $_GET['transactionId'],
]));

$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response, true);

if ($data['status'] === 'COMPLETED') {
    // ✅ Safe to fulfill the order
    $orderId = $data['metadata']['order_id'] ?? null;
    fulfillOrder($orderId, $data['amount']);
} else {
    // ❌ Do NOT fulfill
    showError("Payment could not be verified.");
}
```

### Success Response (HTTP 200)

```json
{
  "status": "COMPLETED",
  "cus_name": "Siyam Ahmed",
  "cus_email": "siyam@example.com",
  "amount": "500.00",
  "transaction_id": "BLA38KDK2M",
  "payment_method": "bkash",
  "metadata": {
    "order_id": "ORD-10928",
    "user_id": "USR-4821",
    "plan": "PRO_MONTHLY"
  }
}
```

### Success Response Fields Explained

| Field | Type | Description |
|---|---|---|
| `status` | String | `"COMPLETED"` = payment is fully verified and confirmed. Other values: `"PENDING"`, `"ERROR"` |
| `cus_name` | String | Customer name as provided during `/create` |
| `cus_email` | String | Customer email as provided during `/create` |
| `amount` | String | Exact amount confirmed and paid in BDT |
| `transaction_id` | String | The verified SMS Transaction ID |
| `payment_method` | String | Channel used (`bkash`, `nagad`, `rocket`, `upay`) |
| `metadata` | Object | The exact metadata object you passed during `/create` — use this to identify which order to fulfill |

> **Pro Tip:** Use the `metadata` field to pass your internal order ID or user ID during `/create`. It will be returned here in the verify response, so you can instantly know which order to activate without any extra database lookup.

### Error Responses

| HTTP Code | Response | Cause | Action |
|---|---|---|---|
| `400` | `{"status": false, "message": "Invalid transaction ID."}` | TrxID does not exist in SkyPay's verified records | Do not fulfill; transaction may be fake |
| `400` | `{"status": false, "message": "Transaction already used."}` | This TrxID was already used to verify a previous order | Do not fulfill again; idempotency protection |
| `401` | `{"status": false, "message": "Invalid or inactive BRAND-KEY provided."}` | Wrong BRAND-KEY | Check your Dashboard |
| `403` | (Device not connected) | Merchant Android phone is offline | Check your SkyPay APK phone |

---

## Step 5 — Fulfill the Order

Once `/api/payment/verify` returns `"status": "COMPLETED"`, your platform can safely fulfill the order.

**Examples of fulfillment actions:**
- Add the verified `amount` to the user's wallet or credit balance in your database
- Activate or extend the user's subscription plan
- Mark the invoice as Paid in your billing system
- Trigger server/hosting provisioning (for WHMCS/cPanel integrations)
- Unlock premium content, features, or digital goods
- Send a payment confirmation email or Telegram message to the customer

**Always execute fulfillment on your backend server.** Never grant benefits based on frontend data alone.

---

## SMS Synchronization Latency

When a customer pays via bKash, Nagad, Rocket, or Upay on SkyPay's hosted page and submits their TrxID, SkyPay internally verifies it against incoming SMS on your merchant Android phone. This process takes **5 to 20 seconds**.

This is handled entirely on SkyPay's hosted page — the customer sees a "Verifying your payment..." indicator while this happens. Your `/api/payment/verify` call only happens **after** SkyPay has already completed this match and redirected the customer back to your `success_url`. So by the time you call `/verify`, the transaction should already be fully recorded.

However, in rare edge cases (network delays, slow SMS delivery), the transaction may still show `"PENDING"` status. In that case:

- Retry the `/verify` call after 10–15 seconds
- Allow a maximum grace period of 2–3 minutes
- If still not confirmed after 3 minutes, mark the order as pending and follow up manually or via support

---

## Security Rules

| Rule | Description |
|---|---|
| **Never trust callback URL params** | The `?transactionId=...&status=completed` in your `success_url` is for display only. Always verify via `/api/payment/verify` before fulfilling |
| **Server-side only** | All API calls (`/create` and `/verify`) must come from your backend server, never from client-side JavaScript or frontend code |
| **Protect your BRAND-KEY** | Store it in `.env` files or environment variables. Never hardcode it in source code or commit it to version control |
| **Idempotency** | Once a TrxID is verified, it is marked as claimed. Implement your own database check to prevent fulfilling the same order twice |
| **Check `status` value** | Always check that `status === "COMPLETED"` — a `"PENDING"` or `"ERROR"` response means do NOT fulfill |

---

## HTTP Status Code Reference

| HTTP Code | Meaning | What To Do |
|---|---|---|
| `200 OK` | Request processed successfully | Check `"status"` value in the response body |
| `400 Bad Request` | Missing parameter or invalid data | Read the `"message"` field for details |
| `401 Unauthorized` | Missing or invalid BRAND-KEY | Verify your key in the Dashboard |
| `403 Forbidden` | No active Android device connected | Check that your SkyPay APK phone is online |
| `404 Not Found` | Endpoint not found | Ensure you are using the correct URL and HTTP method |
| `405 Method Not Allowed` | Wrong HTTP method (e.g. GET instead of POST) | Use POST for all endpoints |
| `500 Internal Server Error` | Temporary cloud-side error | Wait briefly and retry, or contact SkyPay support |

---

## Quick Reference Summary

```
BASE URL           : https://core.skypaybd.top
AUTH HEADER        : BRAND-KEY: <your_key>
CONTENT TYPE       : Content-Type: application/json

ENDPOINT 1 — Create Hosted Payment URL
  POST /api/payment/create
  Body: {
    "cus_name":    "...",
    "cus_email":   "...",
    "amount":      500,
    "success_url": "https://yoursite.com/payment/success",
    "cancel_url":  "https://yoursite.com/payment/cancel",
    "metadata":    { "order_id": "..." }   ← optional, returned in verify
  }
  Returns: { "status": true, "payment_url": "https://core.skypaybd.top/checkout/..." }
  → Redirect the customer's browser to payment_url immediately.

ENDPOINT 2 — Verify Payment
  POST /api/payment/verify
  Body: { "transaction_id": "BLA38KDK2M" }
  Returns: { "status": "COMPLETED", "amount": "500.00", "payment_method": "bkash", ... }
  → Only fulfill the order when status === "COMPLETED"

SUPPORTED CHANNELS : bkash | nagad | rocket | upay
SMS LATENCY        : 5–20 seconds (handled by SkyPay's hosted page)
IDEMPOTENCY        : each transaction_id can only be verified once
```

---

## Integration Checklist

Before going live, confirm all of the following:

- [ ] BRAND-KEY is stored securely in `.env` or environment variables (not in source code)
- [ ] Merchant Android phone is powered on, online, and SkyPay APK is running with green status
- [ ] Battery optimization is disabled for SkyPay APK on the merchant phone
- [ ] `success_url` and `cancel_url` are valid, publicly accessible URLs (not `localhost`)
- [ ] Customer is redirected to `payment_url` immediately after `/create` response
- [ ] Your `success_url` handler reads `transactionId` from the query parameters
- [ ] A backend POST to `/api/payment/verify` is executed before any order fulfillment
- [ ] Order fulfillment only happens when verify response has `"status": "COMPLETED"`
- [ ] Your database prevents double-fulfillment if `/verify` is called more than once for the same `transaction_id`
- [ ] `metadata` includes your order/user reference so you can identify what to fulfill

---

## Official Resources

| Resource | Link |
|---|---|
| Official Website | https://skypaybd.top |
| Interactive Documentation | https://skypaybd.top/docs |
| API Core Domain | https://core.skypaybd.top |
| GitHub Documentation Repo | https://github.com/SkyPayBD/Docs |
| Merchant Sync Android APK | https://skypaybd.top/public/assets/downloads/SkyPay.apk |
| WordPress WooCommerce Plugin | https://skypaybd.top/public/assets/downloads/WP.zip |
| WHMCS Gateway Module | https://skypaybd.top/public/assets/downloads/WHMCS.zip |
| SMM Panel Module | https://skypaybd.top/public/assets/downloads/SMM.zip |
| WhatsApp Support | https://wa.me/+8801761844968 |
| Telegram | https://t.me/BD_Prime_Minister |

---

> **Looking for a zero-redirect, in-app payment experience?**
> For Telegram bots, Discord bots, mobile apps, and custom headless checkouts where you don't want to redirect the user anywhere — see [SkyPay Headless API v2](./SkyPay_Headless_API_v2.md).

---

*SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
