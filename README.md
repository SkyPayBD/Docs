# SkyPay Payment Gateway — Complete API & Integration Documentation

<div align="center">

![SkyPay Banner](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)

**Automated Multi-Channel MFS Payment Infrastructure for Bangladesh**  
*Accept automated **bKash**, **Nagad**, **Rocket**, and **Upay** payments directly on websites, Telegram bots, and mobile apps.*

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Documentation](https://img.shields.io/badge/Read%20Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Gateway](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/API%20Version-v2.0%20Live-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
[![Last Updated](https://img.shields.io/badge/Updated-12%20September%202026-10b981?style=for-the-badge&logo=clock&logoColor=white)](https://skypaybd.top/docs)

</div>

---

> 📖 **Online Documentation:** You can read the live, interactive version of this documentation anytime at [https://skypaybd.top/docs](https://skypaybd.top/docs).  
> 🌐 **Official Website:** [https://skypaybd.top](https://skypaybd.top)  
> ⚡ **API Core Endpoint Domain:** `https://core.skypaybd.top`  
> 🔐 **Simplified Authentication:** All endpoints require only your **`BRAND-KEY`** header. No `SECRET-KEY` is needed.

---

## 🛠️ Technology Stack & Architecture Badges

SkyPay's payment gateway, hosted checkout engines, headless APIs, and merchant automation bridge are built using industry-standard enterprise technologies:

<div align="center">

| Layer | Supported Technologies & Badges |
|---|---|
| **Core Gateway & Backend Engine** | [![PHP 8.x](https://img.shields.io/badge/PHP%208.x-777BB4?style=for-the-badge&logo=php&logoColor=white)](https://php.net) [![CodeIgniter 4](https://img.shields.io/badge/CodeIgniter%204-EF4223?style=for-the-badge&logo=codeigniter&logoColor=white)](https://codeigniter.com) [![RESTful API](https://img.shields.io/badge/RESTful%20API-005571?style=for-the-badge&logo=fastapi&logoColor=white)](https://restfulapi.net) |
| **Frontend & UI Templates** | [![Blade Views](https://img.shields.io/badge/Blade%20Views-F05340?style=for-the-badge&logo=laravel&logoColor=white)](https://laravel.com) [![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)](https://w3.org) [![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)](https://w3.org) [![JavaScript ES6+](https://img.shields.io/badge/JavaScript%20ES6+-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org) |
| **Data Interchange & Network** | [![JSON](https://img.shields.io/badge/JSON%20Standard-000000?style=for-the-badge&logo=json&logoColor=white)](https://json.org) [![cURL](https://img.shields.io/badge/cURL-073551?style=for-the-badge&logo=curl&logoColor=white)](https://curl.se) [![Python Requests](https://img.shields.io/badge/Python%20Requests-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://requests.readthedocs.io) [![HTTPS / TLS](https://img.shields.io/badge/TLS%20%2F%20HTTPS-10b981?style=for-the-badge&logo=letsencrypt&logoColor=white)](https://letsencrypt.org) |
| **Hardware & SMS Synchronization** | [![Android SMS Gateway](https://img.shields.io/badge/Android%20SMS%20Bridge-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://developer.android.com) [![GSM MFS Network](https://img.shields.io/badge/MFS%20Telco%20Bridge-0284c7?style=for-the-badge&logo=cellular&logoColor=white)](https://skypaybd.top) |
| **E-Commerce & Client Ecosystem** | [![WooCommerce](https://img.shields.io/badge/WooCommerce-96588A?style=for-the-badge&logo=woocommerce&logoColor=white)](https://woocommerce.com) [![WordPress](https://img.shields.io/badge/WordPress-21759B?style=for-the-badge&logo=wordpress&logoColor=white)](https://wordpress.org) [![WHMCS](https://img.shields.io/badge/WHMCS-535353?style=for-the-badge&logo=serverfault&logoColor=white)](https://whmcs.com) [![Telegram Bot API](https://img.shields.io/badge/Telegram%20Bot-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://core.telegram.org/bots/api) [![Sketchware](https://img.shields.io/badge/Sketchware%20SWB-EA4335?style=for-the-badge&logo=googleplay&logoColor=white)](https://skypaybd.top) |

</div>

---

## 📋 Table of Contents

- [1. Executive Overview](#1-executive-overview)
- [2. Global Endpoints & Simplified Authentication](#2-global-endpoints--simplified-authentication)
- [3. Dual Architecture Philosophy: Hosted vs Headless](#3-dual-architecture-philosophy-hosted-vs-headless)
  - [3.1 When to Choose Hosted Gateway](#31-when-to-choose-hosted-gateway)
  - [3.2 When to Choose Headless API v2](#32-when-to-choose-headless-api-v2)
- [4. API 1: Hosted Checkout Gateway (Web Redirect Flow)](#4-api-1-hosted-checkout-gateway-web-redirect-flow)
  - [4.1 Architecture & End-to-End Workflow](#41-architecture--end-to-end-workflow)
  - [4.2 Create Hosted Payment URL Endpoint](#42-create-hosted-payment-url-endpoint)
  - [4.3 Customer Return Callback & Query Parameters](#43-customer-return-callback--query-parameters)
  - [4.4 Verify Hosted Payment Endpoint](#44-verify-hosted-payment-endpoint)
- [5. API 2: Headless Payment API v2 (Zero Redirect / In-App / Bot)](#5-api-2-headless-payment-api-v2-zero-redirect--in-app--bot)
  - [5.1 3-Step Headless Architecture](#51-3-step-headless-architecture)
  - [5.2 Step 1: Initiate Headless Payment Session](#52-step-1-initiate-headless-payment-session)
  - [5.3 Step 2: Telegram Bot & Mobile App Wallet Presentation](#53-step-2-telegram-bot--mobile-app-wallet-presentation)
  - [5.4 Step 3: Real-Time Transaction Verification](#54-step-3-real-time-transaction-verification)
  - [5.5 Headless Integration Best Practices & Rules](#55-headless-integration-best-practices--rules)
- [6. Architectural Comparison Matrix](#6-architectural-comparison-matrix)
- [7. Merchant Mobile App & Android SMS Synchronization Engine](#7-merchant-mobile-app--android-sms-synchronization-engine)
- [8. Pre-Built CMS Modules & Module Downloads](#8-pre-built-cms-modules--module-downloads)
- [9. HTTP Status Codes, Error Dictionary & Troubleshooting](#9-http-status-codes-error-dictionary--troubleshooting)
- [10. Production Security & Deployment Checklist](#10-production-security--deployment-checklist)

---

## 1. Executive Overview

**SkyPay BD** is an automated payment infrastructure designed specifically for businesses, online shops, software platforms, and developers in Bangladesh.

Traditionally, accepting mobile financial payments (**bKash**, **Nagad**, **Rocket**, **Upay**) required either:
1. Contracting with expensive corporate aggregators that take 2.5%–4% cut per transaction and take weeks for merchant approval, or
2. Manual verification, where merchants manually check SMS logs on personal Android phones and match numbers one by one before activating customer accounts or shipping goods.

SkyPay completely automates this process. By pairing an **Android SMS synchronization bridge** on your merchant device with our **high-speed cloud API at `core.skypaybd.top`**, incoming payment SMS notifications are encrypted and matched in real-time. Transactions are verified and completed in **5 to 20 seconds** with zero human intervention.

### Supported Channels & Payment Modes

| Channel | Personal (Send Money) | Agent (Cash In) | Merchant (Payment) |
|---|:---:|:---:|:---:|
| **bKash** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Nagad** | ✅ Yes | ✅ Yes | ❌ Under Review |
| **Rocket** | ✅ Yes | ✅ Yes | ❌ Under Review |
| **Upay** | ✅ Yes | ❌ No | ❌ Under Review |

---

## 2. Global Endpoints & Simplified Authentication

### Unified Endpoint Domain
All API calls across both v1 (Hosted) and v2 (Headless) use the centralized API core domain:
```
https://core.skypaybd.top
```

### Simplified Authentication Header
To make development as simple and fast as possible, **SkyPay does NOT require a `SECRET-KEY`**.  
Every API call — whether creating a payment or verifying a transaction — requires **only one header**: **`BRAND-KEY`**.

```http
BRAND-KEY: your_unique_brand_key_here
Content-Type: application/json
Accept: application/json
```

### Master Endpoints Directory

| API Model | Action | Method | Full Endpoint URL |
|---|---|:---:|---|
| **Hosted Gateway** | Create Payment URL | `POST` | `https://core.skypaybd.top/api/payment/create` |
| **Hosted Gateway** | Verify Payment Order | `POST` | `https://core.skypaybd.top/api/payment/verify` |
| **Headless API v2** | Initiate Payment Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| **Headless API v2** | Verify Transaction (SMS Sync) | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

> 🔒 **Security Rule:** Keep your `BRAND-KEY` strictly on your backend server (in environment variables or `.env`). Never expose it in client-side JavaScript, public mobile code, or GitHub repositories.

---

## 3. Dual Architecture Philosophy: Hosted vs Headless

### 3.1 When to Choose Hosted Gateway
- **User Experience:** Customer clicks "Pay Now", browser redirects to `core.skypaybd.top` secure checkout page. Customer selects bKash/Nagad/Rocket/Upay, sends money, inputs TrxID, and is redirected back to your website callback.
- **Best Use Cases:**
  - Standard websites, WordPress WooCommerce, WHMCS web hosting billing, OpenCart, PrestaShop, PHP/Laravel e-Commerce carts.
  - Platforms where you do not want to design any payment frontend, modal, or selection screens.
- **Development Time:** Under 15 minutes.

### 3.2 When to Choose Headless API v2
- **User Experience:** **Zero redirection.** The customer never leaves your Telegram Bot chat, Discord server, native mobile app (Flutter / React Native / Swift / Kotlin), or modern Single-Page Application (Next.js / React / Vue).

---

## 4. API 1: Hosted Checkout Gateway (Web Redirect Flow)

### 4.1 Architecture & End-to-End Workflow

Generates a hosted payment URL where customers choose their preferred MFS channel on a pre-built secure payment page.

**Flow:** Your Backend → `POST /api/payment/create` → Receive `payment_url` → Redirect Customer → Customer Pays on SkyPay Page → Customer Redirected to `success_url` → Your Backend verifies via `POST /api/payment/verify`

---

### 4.2 Create Hosted Payment URL Endpoint

**`POST https://core.skypaybd.top/api/payment/create`**

#### Request Parameters

| Parameter    | Type          | Status      | Description |
|---|---|---|---|
| `amount`       | Numeric       | Required    | Payable amount in BDT (Integer or Decimal, range 1 to 1,000,000). |
| `success_url`  | String (URL)  | Required    | Valid callback URL where customer is redirected after successful payment. |
| `cancel_url`   | String (URL)  | Required    | Valid callback URL where customer is redirected if they cancel payment. |
| `meta_data`    | Object / JSON | Recommended | Custom metadata payload (e.g. `order_id`, `user_id`). Supported alias: `metadata`. Must be an object, array, or JSON string. |
| `cus_name`     | String        | Optional    | Customer's full name. Supported aliases: `customer_name`, `c_name`, `name`. Defaults to `'Default Name'`. |
| `cus_email`    | String        | Optional    | Customer email address. Supported aliases: `customer_email`, `c_email`, `email`. Defaults to `'default@gmail.com'`. |
| `webhook_url`  | String (URL)  | Optional    | Webhook notification URL. When configured, an automated server-to-server POST request is dispatched instantly upon payment completion. |
| `return_type`  | String        | Optional    | HTTP redirect method for callback URLs (`GET` or `POST`). Defaults to `GET`. |

#### Example Request Body

```json
{
  "amount": 250.00,
  "success_url": "https://mystore.com/payment/success",
  "cancel_url": "https://mystore.com/payment/cancel",
  "webhook_url": "https://mystore.com/api/payment-webhook",
  "cus_name": "John Doe",
  "cus_email": "customer@gmail.com",
  "meta_data": {
    "order_id": "ORD-10928",
    "user_id": 4821
  }
}
```

#### Success Response (HTTP 200)

```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/api/execute/f7b3a9c2d1e0f8"
}
```

#### Error Response

```json
{
  "status": false,
  "message": "The success_url field is required."
}
```

---

### 4.3 Customer Return Callback & Query Parameters

After the customer completes (or cancels) payment on the SkyPay hosted page, they are redirected back to your `success_url` or `cancel_url` with the following query parameters appended automatically by the gateway.

#### Example Redirect URLs

**Successful Payment:**
```
https://mystore.com/payment/success?paymentMethod=bkash&transactionId=KUCSPL777353&paymentAmount=250&paymentFee=0&status=completed
```

**Failed / Cancelled Payment (no payment made):**
```
https://mystore.com/payment/cancel?paymentMethod=undetected&transactionId=KUCSPL777353&paymentAmount=100&paymentFee=0&status=failed
```

#### Callback Query Parameters

| Parameter       | Type    | Description |
|---|---|---|
| `paymentMethod`  | String  | The MFS channel used: `bkash`, `nagad`, `rocket`, `upay`. Returns `undetected` if the customer did not complete payment. |
| `transactionId`  | String  | The gateway-assigned transaction identifier. Always present regardless of payment outcome. Use this to call the verify endpoint. |
| `paymentAmount`  | Numeric | The payment amount in BDT as submitted during session creation. |
| `paymentFee`     | Numeric | Gateway fee charged for the transaction. Returns `0` if no fee applies. |
| `status`         | String  | Final payment outcome. Values: `completed` (successful), `failed` (payment not completed or cancelled). |

#### `status` Values

| Value       | Meaning | Action Required |
|---|---|---|
| `completed` | Customer successfully sent the payment and it was verified by SMS sync. | Proceed to backend verification via `/api/payment/verify` before fulfilling the order. |
| `failed`    | Customer cancelled, did not pay, or payment could not be verified. | Redirect customer to an error/retry page. Do not fulfill the order. |

> ⚠️ **Critical Security Rule:** Never fulfill an order based on redirect parameters alone. The `status=completed` in the URL is a **hint only** — always confirm by calling `POST /api/payment/verify` from your backend server with the `transactionId` before activating accounts, shipping goods, or crediting balances.

---

### 4.4 Verify Hosted Payment Endpoint

**`POST https://core.skypaybd.top/api/payment/verify`**

Verify transaction status on your server using the unique transaction ID. The system validates the ID against connected device SMS logs.

#### Request Parameters

| Parameter        | Type   | Status   | Description |
|---|---|---|---|
| `transaction_id` | String | Required | The transaction identifier to verify. Supported aliases: `transactionId`, `transactionid`, `trx_id`, `trx`, `transaction`. |

#### Example Request Body

```json
{
  "transaction_id": "TRX12345678"
}
```

#### Success Response (HTTP 200)

```json
{
  "status": true,
  "data": {
    "cus_name": "John Doe",
    "cus_email": "customer@gmail.com",
    "amount": 250.00,
    "transaction_id": "TRX12345678",
    "meta_data": {
      "order_id": "ORD-10928",
      "user_id": 4821
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

#### Transaction `data.status` Values

| Status Value | Meaning | Action Required |
|---|---|---|
| `COMPLETED` | Payment matched against merchant device SMS and confirmed. | Safely deliver goods, credit account balance, or activate membership. |
| `PENDING`   | Payment initialized but not yet confirmed by SMS synchronization. | Do not fulfill yet; prompt customer to complete the transfer and wait for sync. |
| `FAILED`    | Transaction failed, expired, or invalid credentials. | Payment not successful. Inform customer to re-attempt checkout. |

---

## 5. API 2: Headless Payment API v2 (Zero Redirect / In-App / Bot)

### 5.1 3-Step Headless Architecture

A seamless, zero-redirect architecture designed for Telegram bots, native Android/iOS apps, and single-page applications.

**Flow:** Initiate Session → Present Wallet Numbers to Customer → Customer Sends Money & Submits TrxID → Verify via SMS Sync → Fulfill Order

---

### 5.2 Step 1: Initiate Headless Payment Session

**`POST https://core.skypaybd.top/api/v2/payment/create`**

Initializes a pending payment session and returns the active merchant wallet numbers currently synchronized on your connected Android devices.

#### Request Parameters

| Parameter   | Type          | Status   | Description |
|---|---|---|---|
| `cus_name`  | String        | Required | Full name of customer or Telegram username. |
| `amount`    | Numeric       | Required | Total payable amount in BDT (natural number or 2 decimal places). |
| `meta_data` | Object / JSON | Optional | Arbitrary custom metadata (e.g. Telegram user ID, plan ID, invoice ref). |

#### Example Request Body

```json
{
  "cus_name": "Siyam Ahmed",
  "amount": 500,
  "meta_data": {
    "telegram_user_id": 12345678,
    "product_id": "VIP_SUB_01"
  }
}
```

#### Success Response (HTTP 200)

```json
{
  "status": true,
  "id": "a1b2c3d4e5f6g7h8",
  "brand": {
    "name": "My Tech Store",
    "mobile": "017XXXXXXXX",
    "whatsapp": "017XXXXXXXX",
    "email": "support@mytechstore.com"
  },
  "methods": [
    {
      "name": "bkash",
      "active_payments": {
        "personal": true,
        "agent": false,
        "payment": true
      },
      "personal": "01761844968",
      "agent": "",
      "payment": "01761844968"
    },
    {
      "name": "nagad",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "01761844968",
      "agent": ""
    },
    {
      "name": "rocket",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "017257649946",
      "agent": ""
    },
    {
      "name": "upay",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "017284684646",
      "agent": ""
    }
  ]
}
```

---

### 5.3 Step 2: Telegram Bot & Mobile App Wallet Presentation

When your backend receives the response from `/create`, present the active wallet numbers to the customer directly in Telegram or your app screen.

```
💳 Order #VIP_SUB_01 | Payable: 500 BDT

Please send the exact amount to any of our official wallets:
• bKash (Personal): 01761844968 (Send Money)
• Nagad (Personal): 01761844968 (Send Money)
• Rocket (Personal): 017257649946 (Send Money)

After completing the payment in your MFS app, reply with your Transaction ID (TrxID):
```

---

### 5.4 Step 3: Real-Time Transaction Verification

**`POST https://core.skypaybd.top/api/v2/payment/verify`**

When the customer submits their TrxID into your bot or custom checkout screen, your backend sends a verification request to match incoming device SMS records.

#### Request Parameters

| Parameter        | Type   | Status   | Description |
|---|---|---|---|
| `id`             | String | Required | The payment session ID returned by `/create`. |
| `method`         | String | Required | Channel in strict lowercase: `bkash`, `nagad`, `rocket`, or `upay`. |
| `transaction_id` | String | Required | The SMS Transaction ID (TrxID) submitted by the customer. |

#### Example Request Body

```json
{
  "id": "a1b2c3d4e5f6g7h8",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

#### Success Response (HTTP 200)

```json
{
  "status": true,
  "amount": "500.00",
  "cus_name": "Siyam Ahmed",
  "id": "a1b2c3d4e5f6g7h8"
}
```

#### Headless Error Scenarios

| HTTP Status | Response Message | Root Cause & Recommendation |
|---|---|---|
| `400` | `Invalid transaction ID or transaction already used.` | TrxID does not match incoming SMS logs or was already claimed by another order. |
| `404` | `Payment session not found or expired.` | Session ID expired or does not exist. Prompt user to re-initiate order. |
| `400` | `This payment session has already been completed.` | Session already confirmed. Prevent double fulfillment. |
| `400` | `Unsupported payment method supplied.` | Ensure method is lowercase: `bkash`, `nagad`, `rocket`, or `upay`. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | Check your active BRAND-KEY in SkyPay Merchant Dashboard. |

---

### 5.5 Headless Integration Best Practices & Rules

> ⏱️ **5 to 20 Seconds SMS Bridge Latency Rule**  
> When a customer sends money via bKash or Nagad, the telecom GSM network delivers the SMS notification to your merchant Android phone. The SkyPay Android Sync App captures and pushes the notification to the server in real-time. This process takes **5 to 20 seconds**. Never reject an order permanently on the initial instant submit; allow the customer to retry after 10 seconds.

---

## 6. Architectural Comparison Matrix

| Dimension | Hosted Gateway (v1) | Headless API (v2) |
|---|---|---|
| **Checkout Flow** | Browser redirect to SkyPay hosted checkout page | 100% In-App / In-Bot (Zero external redirect) |
| **User Interface** | Pre-built responsive SkyPay hosted page | Custom interface designed by merchant / bot chat |
| **Telegram & Chatbot Support** | Opens external browser payment link | Native interactive chat buttons & text replies |
| **Session Identifier** | Unique checkout identifier returned in URL / response | Unique `session id` returned in JSON response |
| **Authentication Headers** | `BRAND-KEY` (or `API-KEY` / `SECRET-KEY`) | `BRAND-KEY` |
| **Active MFS Numbers** | Displayed automatically on hosted UI | Returned dynamically in `/create` JSON payload |
| **Required Input Data** | `amount`, `success_url`, `cancel_url` | `cus_name`, `amount` |
| **Best Suited For** | WordPress, WooCommerce, WHMCS, SMM Panels, Webstores | Telegram Bots, Discord Bots, Mobile Apps, SPAs |

---

## 7. Merchant Mobile App & Android SMS Synchronization Engine

SkyPay's real-time verification depends on your **Android Sync App** running on a dedicated merchant phone with active MFS SIM cards.

**Requirements:**
- Android phone with bKash / Nagad / Rocket SIM cards registered as merchant/personal accounts
- SkyPay Merchant Sync APK installed and running in background
- Battery optimization **disabled** for the SkyPay app
- Uninterrupted WiFi or cellular data connection
- Phone kept plugged into power 24/7

[Download Android APK](https://skypaybd.top/public/assets/downloads/SkyPay.apk)

---

## 8. Pre-Built CMS Modules & Module Downloads

| Module | Platform | Download |
|---|---|---|
| WordPress & WooCommerce Plugin | WordPress | [Download .zip](https://skypaybd.top/public/assets/downloads/WP.zip) |
| WHMCS Billing Module | WHMCS 7.x & 8.x | [Download .zip](https://skypaybd.top/public/assets/downloads/WHMCS.zip) |
| SMM Panel Auto-Deposit | SmartPanel / SMM Scripts | [Download .zip](https://skypaybd.top/public/assets/downloads/SMM.zip) |
| Sketchware SWB Project | Sketchware Mobile | [Download .swb](https://skypaybd.top/public/assets/downloads/Apps.swb) |
| SkyPay Merchant Sync APK | Android | [Download APK](https://skypaybd.top/public/assets/downloads/SkyPay.apk) |
| Developer Offline Docs & AI Specs | All Platforms | [Download .zip](https://skypaybd.top/public/assets/downloads/dev.zip) |

---

## 9. HTTP Status Codes, Error Dictionary & Troubleshooting

| HTTP Code | Status | Meaning & Resolution |
|---|---|---|
| `200` | OK | Request processed successfully. Check `status: true` in JSON payload. |
| `400` | Bad Request | Missing required parameter, invalid amount, or mismatched transaction ID. |
| `401` | Unauthorized | Missing or invalid `BRAND-KEY`, `API-KEY`, or `SECRET-KEY`. |
| `403` | Forbidden | Invalid or inactive brand credentials provided. |
| `404` | Not Found | Payment session or transaction record not found. |
| `422` | Unprocessable Entity | Validation error on specific field (e.g. invalid URL, amount out of range, malformed JSON). |
| `500` | Internal Error | Database insertion or gateway system error. Please contact technical support. |

---

## 10. Production Security & Deployment Checklist

- ✅ Store `BRAND-KEY` in server `.env` files — **never** in public client code.
- ✅ Implement idempotency checks in your database to prevent duplicate fulfillment.
- ✅ Provide a 10–20 second retry window for customer SMS matching.
- ✅ Keep the merchant Android sync smartphone plugged into power 24/7.
- ✅ Disable battery optimization for the SkyPay APK on the sync device.
- ✅ Always verify payment via backend `POST` to `/verify` — never trust browser redirect parameters alone.
- ✅ When passing metadata via `meta_data`, always provide a valid JSON object or JSON-encoded string to avoid `422` errors.
