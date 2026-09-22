# SkyPay - BD

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png" width="100" alt="SkyPay Logo" />

<br/>

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789098593_de7d9d238ad2e0178762.png" width="420" alt="SkyPay Banner" />

<br/><br/>

**Automated Multi-Channel Payment Infrastructure for Bangladesh**

*Accept automated payments via **bKash**, **Nagad**, **Rocket**, **Upay**, **Binance Pay**, and more — directly on websites, Telegram bots, and mobile apps.*

<br/>

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/API%20Version-v2.0%20Live-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
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
| **Sonali Bank** | 🔄 Coming Soon | — | — |
| **Islami Bank** | 🔄 Coming Soon | — | — |
| **PayPal & International** | 🔄 Planned | — | — |

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Authentication](#-authentication)
- [API Endpoints Directory](#-api-endpoints-directory)
- [API v1 — Hosted Checkout Gateway](#-api-v1--hosted-checkout-gateway-web-redirect-flow)
  - [Create Payment URL](#1-create-hosted-payment-url)
  - [Callback Redirect Parameters](#2-customer-return-callback--redirect-parameters)
  - [Verify Payment](#3-verify-hosted-payment)
- [API v2 — Headless API (Zero Redirect)](#-api-v2--headless-payment-api-zero-redirect--in-app--bot)
  - [Step 1: Initiate Session](#step-1-initiate-headless-payment-session)
  - [Step 2: Present Wallets to Customer](#step-2-present-wallet-numbers-to-customer)
  - [Step 3: Verify Transaction](#step-3-verify-transaction)
- [Comparison: v1 vs v2](#-architecture-comparison-v1-vs-v2)
- [Android SMS Sync App Setup](#-android-merchant-sync-app-setup)
- [Pre-Built CMS Modules](#-pre-built-cms-modules)
- [HTTP Status Codes & Errors](#-http-status-codes--error-reference)
- [Security Checklist](#-production-security-checklist)
- [Contact & Support](#-contact--support)

---

## 🌐 Overview

**SkyPay BD** is an automated payment infrastructure for businesses, developers, and online platforms in Bangladesh. It bridges your Android merchant SIM phone with a high-speed cloud API so that incoming MFS payment SMS notifications are verified in real-time — **5 to 20 seconds, zero human intervention.**

Two integration modes are available:

| Mode | Best For |
|---|---|
| **v1 Hosted Gateway** | Websites, WooCommerce, WHMCS, SMM Panels — browser redirect checkout |
| **v2 Headless API** | Telegram Bots, mobile apps, SPAs — zero redirect, full custom UI |

---

## 🔐 Authentication

Every API call requires a single header: **`BRAND-KEY`**.

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

### Where to find your BRAND-KEY

Log in to the dashboard → **[Brand Management](https://skypaybd.top/user/brands)** → copy the key for your brand.

### Accepted Header Aliases

The accepted header names differ between v1 and v2:

| Header Name | v1 Hosted | v2 Headless | Notes |
|---|:---:|:---:|---|
| `BRAND-KEY` | ✅ | ✅ **Only accepted** | Recommended for all |
| `API-KEY` | ✅ | ❌ | v1 only alias |
| `SECRET-KEY` | ✅ | ❌ | v1 only alias |
| `?api_key=` (query param) | ✅ | ❌ | v1 only, URL parameter |

> **Important:** For **v2 Headless API**, only `BRAND-KEY` is accepted. The header names `API-KEY` and `SECRET-KEY` are not read by v2 endpoints — always use `BRAND-KEY` header with v2.
>
> For **v1 Hosted**, all three header names (`BRAND-KEY`, `API-KEY`, `SECRET-KEY`) and the `?api_key=` query parameter resolve to the same credential. The key value is identical regardless of which name you use.

> ⚠️ **Never expose your BRAND-KEY in client-side JavaScript, public mobile code, or GitHub repositories. Always keep it in your server `.env` file.**

---

## 📡 API Endpoints Directory

| API | Action | Method | Endpoint |
|---|---|:---:|---|
| **v1 Hosted** | Create Payment URL | `POST` | `https://core.skypaybd.top/api/payment/create` |
| **v1 Hosted** | Verify Payment | `POST` | `https://core.skypaybd.top/api/payment/verify` |
| **v2 Headless** | Initiate Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| **v2 Headless** | Verify Transaction | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

---

## 🏦 API v1 — Hosted Checkout Gateway (Web Redirect Flow)

The Hosted Gateway generates a SkyPay-hosted payment page URL. Your customer is redirected to that page, selects their preferred payment method, sends money, enters their TrxID, and is redirected back to your `success_url` or `cancel_url`.

**Flow:**
```
Your Backend
  → POST /api/payment/create
  → Receive payment_url
  → Redirect Customer to payment_url
  → Customer Pays on SkyPay Hosted Page
  → Customer Redirected to success_url or cancel_url
  → Your Backend calls POST /api/payment/verify
  → Fulfill Order if status = COMPLETED
```

---

### 1. Create Hosted Payment URL

**`POST https://core.skypaybd.top/api/payment/create`**

#### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

#### Request Parameters

| Parameter | Type | Status | Description |
|---|---|---|---|
| `amount` | Numeric | **Required** | Payable amount in BDT. Must be numeric, between `1` and `1,000,000`. Integer or decimal. |
| `success_url` | String (URL) | **Required** | Valid HTTPS URL to redirect the customer after successful payment. |
| `cancel_url` | String (URL) | **Required** | Valid HTTPS URL to redirect the customer if they cancel or fail to complete payment. |
| `cus_name` | String | Optional | Customer full name. Aliases: `customer_name`, `c_name`, `name`. Defaults to `Default Name`. |
| `cus_email` | String | Optional | Customer email. Aliases: `customer_email`, `c_email`, `email`. Defaults to `default@gmail.com`. |
| `meta_data` | Object / JSON | Optional | Arbitrary JSON object to carry custom data (e.g. `order_id`, `user_id`). Returned as-is in verify response. Alias: `metadata`. |
| `webhook_url` | String (URL) | Optional | If provided, a server-to-server POST is fired immediately when payment completes. Must be a valid URL. |
| `return_type` | String | Optional | HTTP method for redirect callbacks. `GET` or `POST`. Defaults to `GET`. |

#### Example Request Body

```json
{
  "amount": 1535,
  "success_url": "https://mystore.com/payment/success",
  "cancel_url": "https://mystore.com/payment/cancel",
  "webhook_url": "https://mystore.com/api/webhook",
  "cus_name": "Ahmed",
  "cus_email": "ahmed@example.com",
  "meta_data": {
    "order_id": "ORD-10928",
    "user_id": 27727373
  }
}
```

#### Success Response (HTTP 200)

```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/api/execute/f89f359b182a25f58d57ae322c956c29"
}
```

Redirect your customer to `payment_url`. They will see the SkyPay hosted checkout page.

#### Error Response

```json
{
  "status": false,
  "message": "The success_url field is required."
}
```

---

### 2. Customer Return Callback & Redirect Parameters

After the customer completes or cancels payment, they are redirected back to your `success_url` or `cancel_url` with these query parameters appended automatically:

#### Successful Payment Redirect

```
https://mystore.com/payment/success?paymentMethod=bkash&transactionId=8TIMCL035056&paymentAmount=1535&paymentFee=0&status=completed
```

#### Cancelled / Failed Payment Redirect

```
https://mystore.com/payment/cancel?paymentMethod=undetected&transactionId=8TIMCL035056&paymentAmount=1535&paymentFee=0&status=failed
```

#### Redirect Parameter Reference

| Parameter | Type | Description |
|---|---|---|
| `paymentMethod` | String | The channel used: `bkash`, `nagad`, `rocket`, `upay`, `binance`. Returns `undetected` if customer did not complete payment. |
| `transactionId` | String | The gateway-assigned transaction ID. **Always present** regardless of payment outcome. Use this for the verify call. |
| `paymentAmount` | Numeric | The BDT amount set during session creation. |
| `paymentFee` | Numeric | Gateway fee. Returns `0` if no fee applies. |
| `status` | String | `completed` = paid and verified. `failed` = not completed or cancelled. |

> ⚠️ **Critical Security Rule:** Never fulfill an order based on the redirect URL parameters alone. The `status=completed` in the URL is a hint only. **Always verify by calling `POST /api/payment/verify` from your backend server** before activating accounts, shipping goods, or crediting balances.

---

### 3. Verify Hosted Payment

**`POST https://core.skypaybd.top/api/payment/verify`**

#### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

#### Request Parameters

| Parameter | Type | Status | Description |
|---|---|---|---|
| `transaction_id` | String | **Required** | The transaction ID from the redirect callback (`transactionId` query param). Aliases: `transactionId`, `transactionid`, `trx_id`, `trx`, `transaction`. |

#### Example Request Body

```json
{
  "transaction_id": "8TIMCL035056"
}
```

#### Success Response — Payment Completed (HTTP 200)

```json
{
  "status": true,
  "data": {
    "cus_name": "Ahmed",
    "cus_email": "ahmed@example.com",
    "amount": "1535.000",
    "transaction_id": "8TIMCL035056",
    "meta_data": {
      "order_id": "ORD-10928",
      "user_id": 27727373
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

#### Success Response — Payment Failed (HTTP 200)

```json
{
  "status": true,
  "data": {
    "cus_name": "Ahmed",
    "cus_email": "ahmed@example.com",
    "amount": "1535.000",
    "transaction_id": "8TIMCL035056",
    "meta_data": {
      "order_id": "ORD-10928",
      "user_id": 27727373
    },
    "payment_method": null,
    "status": "FAILED"
  }
}
```

#### `data.status` Values

| Value | Internal Code | Meaning | Action |
|---|:---:|---|---|
| `COMPLETED` | `2` | Payment confirmed via SMS sync. | Safely fulfill the order. |
| `PENDING` | `1` | Payment submitted but not yet confirmed by SMS sync. | Do not fulfill yet; wait for confirmation. |
| `FAILED` | `0` or other | Payment not made, session just initialized, cancelled, or failed. | Inform customer; do not fulfill. |

> **Note:** `FAILED` is returned for any internal status that is not `1` (PENDING) or `2` (COMPLETED). A newly created session that was never paid will also return `FAILED`.

#### v1 Verify Error Responses

```json
{
  "status": false,
  "message": "Transaction identifier is required. Provide transaction_id, transactionId, or trx_id."
}
```
*(HTTP 422 — missing transaction_id field)*

```json
{
  "status": false,
  "message": "No transaction record found with the provided transaction identifier."
}
```
*(HTTP 404 — transaction ID not found)*

---

## ⚡ API v2 — Headless Payment API (Zero Redirect / In-App / Bot)

The Headless API provides a fully custom, zero-redirect payment experience. Your customer never leaves your Telegram bot, mobile app, or website. You control the entire UI and flow.

**Flow:**
```
Your Backend
  → POST /api/v2/payment/create
  → Receive session id + wallet numbers
  → Show wallet numbers to customer in your UI / bot
  → Customer sends money and gives you their TrxID
  → Your Backend calls POST /api/v2/payment/verify
  → Fulfill Order if status = COMPLETED
```

---

### Step 1: Initiate Headless Payment Session

**`POST https://core.skypaybd.top/api/v2/payment/create`**

#### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

#### Request Parameters

| Parameter | Type | Status | Description |
|---|---|---|---|
| `cus_name` | String | **Required** | Customer full name or Telegram username. |
| `amount` | Numeric | **Required** | Total payable amount in BDT. Must be a positive number. |
| `meta_data` | Object / JSON | Optional | Any JSON object with custom data (e.g. Telegram user ID, plan name, order ref). |

#### Example Request Body

```json
{
  "cus_name": "Ahmed",
  "amount": 1535,
  "meta_data": {
    "telegram_user_id": 12345678,
    "plan": "VIP"
  }
}
```

#### Success Response (HTTP 200)

```json
{
  "status": true,
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "brand": {
    "name": "Your Brand Name",
    "mobile": "017XXXXXXXX",
    "whatsapp": "017XXXXXXXX",
    "email": "support@yourdomain.com"
  },
  "methods": [
    {
      "name": "bkash",
      "active_payments": {
        "personal": true,
        "agent": false,
        "payment": false
      },
      "personal": "017XXXXXXXX",
      "agent": "",
      "payment": ""
    },
    {
      "name": "nagad",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "017XXXXXXXX",
      "agent": ""
    },
    {
      "name": "binance",
      "active_payments": {
        "personal": true,
        "merchant": false
      },
      "personal": "XXXXXXXXXX",
      "currency": "USDT",
      "dollar_rate": 120,
      "amount_usdt": 12.5000
    }
  ]
}
```

#### Response Fields Explained

| Field | Description |
|---|---|
| `id` | The unique payment session ID. **Save this — required for the verify call.** |
| `brand.name` | Your brand's display name. |
| `brand.mobile` | Your brand's support phone number. |
| `brand.whatsapp` | Your brand's WhatsApp number. |
| `brand.email` | Your brand's support email. |
| `methods[]` | Array of active payment channels configured for your brand. Only active/configured channels are returned. |
| `methods[].name` | Channel identifier: `bkash`, `nagad`, `rocket`, `upay`, `binance`. |
| `methods[].active_payments` | Which sub-types are enabled (personal, agent, payment/merchant). |
| `methods[].personal` | Personal wallet number (for Send Money). Empty string if not active. |
| `methods[].agent` | Agent wallet number (for Cash In). Empty string if not active. |
| `methods[].payment` | Merchant payment number (bKash only). Empty string if not active. |
| `methods[].personal` *(Binance)* | Binance UID of the receiving account. |
| `methods[].dollar_rate` *(Binance)* | BDT per 1 USDT rate configured by merchant. |
| `methods[].amount_usdt` *(Binance)* | Exact USDT amount customer must send (= `amount ÷ dollar_rate`, rounded to 4 decimals). |

#### Error Response

```json
{
  "status": false,
  "message": "No active payment gateways configured for this brand."
}
```

---

### Step 2: Present Wallet Numbers to Customer

After receiving the `/create` response, display the active wallet numbers to your customer inside your Telegram bot, mobile app, or custom UI.

**Example Telegram Bot Message:**

```
💳 Payment Request

Amount: 1,535 BDT
Order Ref: VIP Plan

Please send the exact amount to one of the following:

🔵 bKash (Personal – Send Money): 017XXXXXXXX
🟠 Nagad (Personal – Send Money): 017XXXXXXXX
🟡 Binance Pay (USDT): UID XXXXXXXXXX
   → Send exactly: 12.5000 USDT

After completing the payment, reply with your Transaction ID (TrxID).
```

> **Note for Binance:** The customer must send exactly `amount_usdt` USDT to the Binance UID shown. A tolerance of ±0.0005 USDT is allowed. Only USDT currency is accepted.

---

### Step 3: Verify Transaction

**`POST https://core.skypaybd.top/api/v2/payment/verify`**

When the customer submits their TrxID / Order ID, call this endpoint from your backend to verify the transaction.

#### Request Headers

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
```

#### Request Parameters

| Parameter | Type | Status | Description |
|---|---|---|---|
| `id` | String | **Required** | The payment session ID returned by `/v2/payment/create`. |
| `method` | String | **Required** | The payment channel used. Must be lowercase: `bkash`, `nagad`, `rocket`, `upay`, or `binance`. |
| `transaction_id` | String | **Required** | The TrxID or Order ID submitted by the customer. See field naming guide below. |

#### Field Naming Guide: `transaction_id` vs `order_id`

The verify endpoint accepts multiple field name aliases for the transaction identifier. The correct field to use depends on the payment method:

| Method | Recommended Field | Why |
|---|---|---|
| `bkash` | `transaction_id` | bKash returns a TrxID (e.g. `BLA38KDK2M`) |
| `nagad` | `transaction_id` | Nagad returns a TrxID |
| `rocket` | `transaction_id` | Rocket returns a TrxID |
| `upay` | `transaction_id` | Upay returns a TrxID |
| `binance` | `order_id` | Binance returns an Order ID (e.g. `443903031407804416`) |

**All accepted field aliases** (any of these resolve to the transaction identifier):

| Field Name | Works? | Notes |
|---|---|---|
| `transaction_id` | ✅ | Recommended for MFS (bKash, Nagad, Rocket, Upay) |
| `order_id` | ✅ | Recommended for Binance |
| `order` | ✅ | Alias for Binance |
| `trx_id` | ✅ | Generic alias |
| `transactionId` | ✅ | camelCase alias |
| `transaction` | ✅ | Generic alias |

> ℹ️ For **Binance**, the system matches the submitted value against both the Binance `orderId` and `transactionId` fields — so either can technically work. However, `order_id` is the recommended field because Binance Pay natively returns an `orderId`.

> ⚠️ For **bKash, Nagad, Rocket, Upay** — always use `transaction_id`. Using `order_id` may cause issues as these gateways use TrxIDs, not order IDs.

#### Example Request — bKash

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "bkash",
  "transaction_id": "TRXXXXXXXXX"
}
```

#### Example Request — Binance

```json
{
  "id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "method": "binance",
  "order_id": "XXXXXXXXXXXXXXXXXX"
}
```

#### Success Response — Payment Verified (HTTP 200)

```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Ahmed",
    "cus_email": "headless@skypaybd.top",
    "amount": 1500,
    "transaction_id": "XXXXXXXXXXXXXXXXXX",
    "meta_data": {
      "telegram_user_id": 12345678,
      "plan": "VIP"
    },
    "payment_method": "binance",
    "status": "COMPLETED"
  }
}
```

#### `data.status` Values

| Value | Meaning | Action |
|---|---|---|
| `COMPLETED` | Transaction verified and committed. | Safely fulfill the order. |

#### Error Responses

```json
{
  "status": false,
  "message": "Payment session not found or expired."
}
```

```json
{
  "status": false,
  "message": "This payment session has already been completed."
}
```

```json
{
  "status": false,
  "message": "Transaction not found. Please check Order ID."
}
```

```json
{
  "status": false,
  "message": "Insufficient amount received. Expected 11.8077 USDT."
}
```

#### Common v2 Verify Errors Reference

| HTTP Status | Message | Cause & Fix |
|---|---|---|
| `400` | `id, method and transaction_id (or order_id) fields are required.` | Missing required field in request body. |
| `400` | `Unsupported payment method supplied.` | `method` must be lowercase: `bkash`, `nagad`, `rocket`, `upay`, or `binance`. |
| `400` | `This payment session has already been completed.` | Session already verified. Prevent duplicate processing. |
| `400` | `Transaction not found. Please check Order ID.` | Submitted TrxID/Order ID does not match any record in Binance transaction history. |
| `400` | `Receiver UID does not match.` | The Binance payment was sent to a different UID. Merchant configuration issue. |
| `400` | `Only USDT payments are accepted.` | Customer sent a currency other than USDT via Binance. |
| `400` | `Insufficient amount received. Expected X USDT.` | Customer sent less than required USDT amount (tolerance is ±0.0005). |
| `400` | `This Order ID has already been used.` | Duplicate submission of an already-committed Binance order. |
| `401` | `BRAND-KEY header is required.` | Missing authentication header. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | Check your key in Brand Management dashboard. |
| `403` | `Associated merchant account is inactive.` | Your SkyPay merchant account is suspended. Contact support. |
| `403` | `No active SMS sync device found for this account.` | No Android device connected and active. Set up device first. |
| `404` | `Payment session not found or expired.` | `id` does not exist or belongs to a different brand. |

---

### Headless API Best Practices

> ⏱️ **5–20 Second SMS Bridge Latency**
> After a customer sends money via bKash or Nagad, the MFS network delivers an SMS to your merchant Android phone. The SkyPay Sync App captures and pushes it to the server. This takes **5 to 20 seconds**. Always allow the customer a retry window — do not reject immediately on first attempt.

- Store the `id` from `/create` against the customer's order in your database.
- Only call `/verify` after the customer actively submits their TrxID.
- Implement idempotency: once you receive `COMPLETED`, mark the order in your DB and do not re-verify.
- For Binance: display the exact `amount_usdt` value to the customer. The tolerance is only ±0.0005 USDT.
- If verification fails with "Transaction not found", ask the customer to wait 10–20 seconds and try again — the SMS may not have synced yet.

---

## 🔄 Architecture Comparison: v1 vs v2

| Feature | v1 Hosted Gateway | v2 Headless API |
|---|---|---|
| **Checkout UI** | SkyPay pre-built hosted page | 100% custom (your bot / app / website) |
| **Redirect Required** | Yes — browser redirect | None — zero external redirect |
| **Telegram Bot Support** | Opens external browser link | Native inline chat experience |
| **Wallet Numbers** | Shown on hosted page automatically | Returned in `/create` JSON response |
| **Required Fields** | `amount`, `success_url`, `cancel_url` | `cus_name`, `amount` |
| **Customer Input** | TrxID entered on SkyPay page | TrxID submitted via your custom UI |
| **Auth Header** | `BRAND-KEY`, `API-KEY`, `SECRET-KEY`, or `?api_key=` | `BRAND-KEY` only |
| **Best For** | WordPress, WooCommerce, WHMCS, SMM Panels | Telegram Bots, Mobile Apps, SPAs |
| **Dev Time** | ~15 minutes | ~30–60 minutes |

---

## 📱 Android Merchant Sync App Setup

SkyPay's real-time MFS payment verification (bKash, Nagad, Rocket, Upay) depends on your **Android Merchant Sync App** reading incoming payment SMS notifications from your SIM card and forwarding them to the SkyPay cloud server.

### Requirements

- Android phone with active MFS SIM cards (bKash / Nagad / Rocket personal or agent accounts)
- SkyPay Merchant Sync APK installed
- Phone kept plugged in to power 24/7
- Battery optimization **disabled** for the SkyPay app
- Uninterrupted WiFi or mobile data

### Step 1: Download & Install the App

<div align="center">

[![Download SkyPay APK](https://img.shields.io/badge/⬇%20Download%20SkyPay%20Merchant%20App-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://skypaybd.top/public/assets/downloads/SkyPay.apk)

</div>

Download the APK from the link above and install it on your Android device. Since this is not from the Google Play Store, you may need to enable **"Install from Unknown Sources"** in your device settings.

### Step 2: Create a Device in the Dashboard

1. Log in to [Device Management](https://skypaybd.top/user/devices)
2. Make sure you have an active subscription plan ([Subscription Plans](https://skypaybd.top/user/plans))
3. Click **Create Device**, enter a name for your device, and save
4. Copy the **Device Key** shown (a copy button is provided)

### Step 3: Log In to the App

Open the SkyPay Merchant App. You will see a login screen with two fields:

| Field | What to Enter |
|---|---|
| **Email** | The email address you used to register on skypaybd.top |
| **Device Key** | The Device Key you copied from the Device Management page |

Enable **"Remember My Device"** (checkbox) before tapping Login. This keeps you logged in across restarts.

> ⚠️ **IP Lock Security:** Each Device Key is locked to the IP address of the first device that logs in with it. If you try to log in from a different phone or your IP changes and you get blocked, **delete the device** from [Device Management](https://skypaybd.top/user/devices) and create a new one. This is intentional — it prevents one key from being shared across multiple phones.

### Step 4: Grant SMS Permissions

After logging in, a **yellow warning banner** will appear at the top of the app asking for SMS permission. Tap **Grant** and allow all requested permissions (SMS access may also request call permissions — allow all of them).

**If permissions don't work:**
1. Long-press the SkyPay app icon → tap **App Info**
2. Look for **Restricted Permissions** or **Permissions** at the top
3. Manually enable **SMS** permission from there

This is required because the APK is sideloaded (not from Play Store), and some Android versions restrict SMS access by default for sideloaded apps.

### Step 5: Keep the Service Running

A toggle button at the top of the app lets you **pause** or **resume** the sync service. If the service is paused, incoming SMS notifications will not be forwarded — and payments will not be verified. Keep it running at all times.

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

## 🚦 HTTP Status Codes & Error Reference

| HTTP Code | Status | Meaning |
|---|---|---|
| `200` | OK | Request successful. Check `status: true` in JSON. |
| `400` | Bad Request | Missing/invalid parameter, amount out of range, or TrxID mismatch. |
| `401` | Unauthorized | Missing or invalid `BRAND-KEY`. |
| `403` | Forbidden | Inactive account or no connected Android device. |
| `404` | Not Found | Payment session or transaction record not found. |
| `405` | Method Not Allowed | Only `POST` requests are accepted on all endpoints. |
| `422` | Unprocessable Entity | Validation error on a specific field (invalid URL, invalid amount, malformed JSON). |
| `500` | Internal Server Error | Database or system error. Contact support. |
| `502` | Bad Gateway | Failed to communicate with an external service (e.g. Binance API timeout). |

---

## ✅ Production Security Checklist

- ✅ Store `BRAND-KEY` in server `.env` — never expose it in client-side code or repositories
- ✅ Always verify payment via backend `POST` to `/verify` — never trust browser redirect parameters alone
- ✅ Implement idempotency in your database to prevent duplicate order fulfillment
- ✅ Allow 10–20 second retry window for MFS SMS matching before rejecting
- ✅ Keep the Android sync phone plugged in to power 24/7
- ✅ Disable battery optimization for the SkyPay APK on the sync device
- ✅ Always pass `meta_data` as a valid JSON object, not a plain string
- ✅ For Binance: display exact `amount_usdt` to customer; the tolerance is only ±0.0005 USDT
- ✅ For Binance: use `order_id` field in verify request; for MFS use `transaction_id`
- ✅ Do not re-verify a session that already returned `COMPLETED`

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
