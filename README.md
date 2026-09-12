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
Every API call &mdash; whether creating a payment or verifying a transaction &mdash; requires **only one header**: **`BRAND-KEY`**.

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
- **Workflow:** Your backend calls `/api/v2/payment/create` to fetch live merchant numbers, renders the instructions inside your own custom interface, receives the customer's TrxID, and posts it to `/api/v2/payment/verify` for instant confirmation.
- **Best Use Cases:**
  - Telegram Bots (automated subscription channels, digital goods, bot checkouts).
  - Discord Bots (server rank upgrades, gaming credits).
  - Mobile Applications (in-app wallet balance, digital recharges).
  - Modern Headless Checkouts (clean custom checkout modals on React/Vue without external redirects).
- **Development Time:** 1 to 2 hours.

---

## 4. API 1: Hosted Checkout Gateway (Web Redirect Flow)

### 4.1 Architecture & End-to-End Workflow

```
[Customer on Store] ──── (1. Click Checkout) ────> [Merchant Backend Server]
                                                            │
                                                            │ (2. POST /api/payment/create)
                                                            ▼
                                                [SkyPay Core API]
                                                            │
                                                            │ (3. Return { payment_url })
                                                            ▼
[Customer Browser] <── (4. HTTP 302 Redirect) ── [Merchant Backend Server]
        │
        ▼ (5. Customer lands on https://core.skypaybd.top/checkout/...)
[SkyPay Hosted Gateway Page]
        │
        ├─ Customer selects bKash / Nagad / Rocket / Upay
        ├─ Customer copies Merchant Number & sends money from their MFS app
        └─ Customer enters SMS Transaction ID (TrxID) & clicks "Verify"
        │
        ▼ (6. Instant SMS match via Merchant Android App)
[Automated Validation Completed]
        │
        ▼ (7. Browser redirected back to Merchant success_url)
[Merchant Store Callback] ── (8. Server POST /api/payment/verify) ──> [Confirm & Deliver Order]
```

---

### 4.2 Create Hosted Payment URL Endpoint

Generates a secure, temporary payment checkout session and returns a hosted URL where the customer completes the transaction.

- **Endpoint:** `https://core.skypaybd.top/api/payment/create`
- **Method:** `POST`
- **Headers:** `BRAND-KEY: your_brand_key`, `Content-Type: application/json`

#### Request Parameters Table

| Parameter | Type | Required | Description | Example |
|---|---|:---:|---|---|
| `cus_name` | `String` | **Yes** | Full name of the customer | `"John Doe"` |
| `cus_email` | `String` | **Yes** | Valid email address of the customer | `"customer@gmail.com"` |
| `amount` | `Numeric` | **Yes** | Total payable amount in BDT (natural number or decimal) | `100` or `100.50` |
| `success_url` | `String (URL)` | **Yes** | Full URL where customer is redirected on successful payment | `"https://mystore.com/checkout/success"` |
| `cancel_url` | `String (URL)` | **Yes** | Full URL where customer is redirected if they cancel | `"https://mystore.com/checkout/cancel"` |
| `metadata` | `Object / JSON` | **No** | Arbitrary merchant data (e.g. order ID, phone number, cart ID) | `{"order_id": 9821, "phone": "017XXXXXXXX"}` |

#### Request Examples (cURL & Python Requests)

<details open>
<summary><strong>Option A: cURL (Bash)</strong></summary>

```bash
curl -X POST https://core.skypaybd.top/api/payment/create   -H "BRAND-KEY: your_brand_key_here"   -H "Content-Type: application/json"   -d '{
    "cus_name": "John Doe",
    "cus_email": "customer@gmail.com",
    "amount": "250.00",
    "success_url": "https://mystore.com/payment/success",
    "cancel_url": "https://mystore.com/payment/cancel",
    "metadata": {
      "order_id": "ORD-10928",
      "user_id": 4821
    }
  }'
```

</details>

<details>
<summary><strong>Option B: Python (Requests)</strong></summary>

```python
import requests

url = "https://core.skypaybd.top/api/payment/create"

headers = {
    "BRAND-KEY": "your_brand_key_here",
    "Content-Type": "application/json"
}

payload = {
    "cus_name": "John Doe",
    "cus_email": "customer@gmail.com",
    "amount": "250.00",
    "success_url": "https://mystore.com/payment/success",
    "cancel_url": "https://mystore.com/payment/cancel",
    "metadata": {
        "order_id": "ORD-10928",
        "user_id": 4821
    }
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data.get("status"):
    print("Redirect customer to:", data.get("payment_url"))
else:
    print("Error:", data.get("message"))
```

</details>

#### Success Response (`200 OK`)

```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/checkout/order/f7b3a9c2d1e04856"
}
```

#### Response Fields Breakdown

| Field | Type | Description |
|---|---|---|
| `status` | `Boolean` | `true` indicates the session was successfully created; `false` on failure. |
| `message` | `String` | Informational status message describing the outcome. |
| `payment_url` | `String (URL)` | Secure checkout URL to which you must redirect the customer's browser. |

#### Error Responses

```json
{
  "status": false,
  "message": "BRAND-KEY header is required or invalid."
}
```

---

### 4.3 Customer Return Callback & Query Parameters

When the customer completes the transaction on SkyPay, they are automatically returned to your `success_url` (or `cancel_url`) with the following URL query parameters:

```
https://mystore.com/payment/success?transactionId=BLA38KDK2M&paymentMethod=bkash&paymentAmount=250.00&paymentFee=0.00&status=completed
```

#### Callback Parameters Breakdown

| Query Parameter | Type | Example | Purpose |
|---|---|---|---|
| `transactionId` | `String` | `BLA38KDK2M` | SMS Transaction ID verified from the telecom network. |
| `paymentMethod` | `String` | `bkash` | Payment channel utilized (`bkash`, `nagad`, `rocket`, `upay`). |
| `paymentAmount` | `Numeric` | `250.00` | Net amount paid in BDT. |
| `paymentFee` | `Numeric` | `0.00` | Gateway fee applied. |
| `status` | `String` | `completed` | Outcome status: `completed`, `pending`, or `failed`. |

> ⚠️ **CRITICAL SECURITY WARNING:** Never fulfill an order or grant account balance purely from GET parameters in the browser URL, as query strings can be manipulated by malicious users. **Always execute a backend POST to `/api/payment/verify`** to double-check that the transaction is recorded as completed in SkyPay's database.

---

### 4.4 Verify Hosted Payment Endpoint

Validates that a transaction ID returned in the callback URL is genuinely completed and recorded in the database.

- **Endpoint:** `https://core.skypaybd.top/api/payment/verify`
- **Method:** `POST`
- **Headers:** `BRAND-KEY: your_brand_key`, `Content-Type: application/json`

#### Request Parameters Table

| Parameter | Type | Required | Description | Example |
|---|---|:---:|---|---|
| `transaction_id` | `String` | **Yes** | Transaction ID received from the return callback URL | `"BLA38KDK2M"` |

#### Request Examples (cURL & Python Requests)

<details open>
<summary><strong>Option A: cURL (Bash)</strong></summary>

```bash
curl -X POST https://core.skypaybd.top/api/payment/verify   -H "BRAND-KEY: your_brand_key_here"   -H "Content-Type: application/json"   -d '{
    "transaction_id": "BLA38KDK2M"
  }'
```

</details>

<details>
<summary><strong>Option B: Python (Requests)</strong></summary>

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
    print("Order verified successfully for:", data.get("cus_name"))
    print("Amount paid:", data.get("amount"))
else:
    print("Verification failed or pending.")
```

</details>

#### Success Response (`200 OK`)

```json
{
  "status": "COMPLETED",
  "cus_name": "John Doe",
  "cus_email": "customer@gmail.com",
  "amount": "250.00",
  "transaction_id": "BLA38KDK2M",
  "payment_method": "bkash",
  "metadata": {
    "order_id": "ORD-10928",
    "user_id": 4821
  }
}
```

#### Response Fields Breakdown

| Field Name | Type | Description |
|---|---|---|
| `status` | `String` | Order status: `COMPLETED`, `PENDING`, or `ERROR`. |
| `cus_name` | `String` | Customer full name as submitted during order creation. |
| `cus_email` | `String` | Customer email address. |
| `amount` | `String` | Total amount confirmed and paid in BDT. |
| `transaction_id` | `String` | Unique verified telecom transaction ID. |
| `payment_method` | `String` | Channel used (`bkash`, `nagad`, `rocket`, `upay`). |
| `metadata` | `Object / JSON` | Custom metadata payload attached during order creation. |

---

## 5. API 2: Headless Payment API v2 (Zero Redirect / In-App / Bot)

### 5.1 3-Step Headless Architecture

The Headless API v2 gives you 100% control over the user experience. The customer never leaves your Telegram bot chat, Discord server, or mobile app screen.

```
[Customer inside Telegram Bot or Mobile App]
                      │
                      ▼
  [1. User triggers checkout: "Deposit 500 BDT"]
                      │
  [2. Your Bot calls: POST https://core.skypaybd.top/api/v2/payment/create]
                      │
  [3. SkyPay returns Active Merchant Numbers]
      - bKash: 01761844968 (Personal & Payment Active)
      - Nagad: 01761844968 (Personal Active)
      - Rocket: 017257649946 (Personal Active)
                      │
  [4. Bot renders message directly in chat:]
      "Please Send Money 500 BDT to bKash: 01761844968
       After paying, enter your SMS TrxID here:"
                      │
  [5. Customer sends money in bKash app & replies with TrxID: 'BLA38KDK2M']
                      │
  [6. Your Bot calls: POST https://core.skypaybd.top/api/v2/payment/verify]
                      │
  [7. SkyPay matches incoming Android SMS in real-time (5-20s)]
                      │
  [8. Verified! Bot immediately upgrades user account in chat!]
```

---

### 5.2 Step 1: Initiate Headless Payment Session

Initializes a pending payment session in SkyPay and dynamically returns all active merchant wallet numbers currently available on your connected Android devices.

- **Endpoint:** `https://core.skypaybd.top/api/v2/payment/create`
- **Method:** `POST`
- **Headers:** `BRAND-KEY: your_brand_key`, `Content-Type: application/json`

#### Request Parameters Table

| Parameter | Type | Required | Description | Example |
|---|---|:---:|---|---|
| `cus_name` | `String` | **Yes** | Customer name or handle | `"Siyam Ahmed"` |
| `amount` | `Numeric` | **Yes** | Amount in BDT (must be greater than 0) | `500` |
| `meta_data` | `Object / JSON` | **No** | Custom metadata (e.g. Telegram chat ID, product ID) | `{"telegram_user_id": 12345678, "plan": "PRO"}` |

#### Request Examples (cURL & Python Requests)

<details open>
<summary><strong>Option A: cURL (Bash)</strong></summary>

```bash
curl -X POST https://core.skypaybd.top/api/v2/payment/create   -H "BRAND-KEY: your_brand_key_here"   -H "Content-Type: application/json"   -d '{
    "cus_name": "Siyam Ahmed",
    "amount": 500,
    "meta_data": {
      "telegram_user_id": 12345678,
      "product_id": "VIP_SUB_01"
    }
  }'
```

</details>

<details>
<summary><strong>Option B: Python (Requests)</strong></summary>

```python
import requests

url = "https://core.skypaybd.top/api/v2/payment/create"

headers = {
    "BRAND-KEY": "your_brand_key_here",
    "Content-Type": "application/json"
}

payload = {
    "cus_name": "Siyam Ahmed",
    "amount": 500,
    "meta_data": {
        "telegram_user_id": 12345678,
        "product_id": "VIP_SUB_01"
    }
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data.get("status"):
    session_id = data.get("id")
    methods = data.get("methods")
    print(f"Session created: {session_id}")
    for m in methods:
        print(f"Gateway: {m['name']} | Personal: {m['personal']}")
else:
    print("Error:", data.get("message"))
```

</details>

#### Success Response (`200 OK`)

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

#### Detailed Explanation of Headless Response Fields

- **`id` (`String`):**  
  The unique payment session identifier. **You must save this ID** in your database, bot cache, or session store, as it is required when verifying the transaction in Step 3.
- **`brand` (`Object`):**  
  Contains your merchant brand name, support mobile, WhatsApp, and email to display support info to customers if needed.
- **`methods` (`Array of Objects`):**  
  Contains every supported gateway channel (`bkash`, `nagad`, `rocket`, `upay`) along with active status flags and active numbers:
  - `active_payments.personal`: If `true`, the `personal` number is active for **Send Money**.
  - `active_payments.agent`: If `true`, the `agent` number is active for **Cash Out**.
  - `active_payments.payment`: If `true`, the `payment` number is active for **Merchant Pay**.
  - If a number is empty `""` or its active flag is `false`, your UI must not present that option to the customer.

---

### 5.3 Step 2: Telegram Bot & Mobile App Wallet Presentation

Once your backend receives the active wallet numbers from `/create`, you present them to the customer inside your bot or app.

#### Recommended Telegram Bot Message Layout
```
💳 Order #VIP_SUB_01 — Payable: 500 BDT

Please send the exact amount to any of our official numbers:
📱 bKash (Personal): 01761844968 [Send Money]
📱 Nagad (Personal): 01761844968 [Send Money]
📱 Rocket (Personal): 017257649946 [Send Money]

⚠️ Important:
1. Pay the exact amount (500 BDT).
2. After completing payment in your MFS app, copy the Transaction ID (TrxID) from your SMS.
3. Reply to this bot with your TrxID below:
```

---

### 5.4 Step 3: Real-Time Transaction Verification

When the customer submits their TrxID in your bot chat or app input field, your backend immediately posts it to SkyPay to match incoming device SMS notifications.

- **Endpoint:** `https://core.skypaybd.top/api/v2/payment/verify`
- **Method:** `POST`
- **Headers:** `BRAND-KEY: your_brand_key`, `Content-Type: application/json`

#### Request Parameters Table

| Parameter | Type | Required | Description | Example |
|---|---|:---:|---|---|
| `id` | `String` | **Yes** | The payment session `id` received from the `/create` endpoint | `"a1b2c3d4e5f6g7h8"` |
| `method` | `String` | **Yes** | Channel name in **strict lowercase**: `bkash`, `nagad`, `rocket`, or `upay` | `"bkash"` |
| `transaction_id` | `String` | **Yes** | The SMS Transaction ID (TrxID) submitted by the customer | `"BLA38KDK2M"` |

#### Request Examples (cURL & Python Requests)

<details open>
<summary><strong>Option A: cURL (Bash)</strong></summary>

```bash
curl -X POST https://core.skypaybd.top/api/v2/payment/verify   -H "BRAND-KEY: your_brand_key_here"   -H "Content-Type: application/json"   -d '{
    "id": "a1b2c3d4e5f6g7h8",
    "method": "bkash",
    "transaction_id": "BLA38KDK2M"
  }'
```

</details>

<details>
<summary><strong>Option B: Python (Requests)</strong></summary>

```python
import requests

url = "https://core.skypaybd.top/api/v2/payment/verify"

headers = {
    "BRAND-KEY": "your_brand_key_here",
    "Content-Type": "application/json"
}

payload = {
    "id": "a1b2c3d4e5f6g7h8",
    "method": "bkash",
    "transaction_id": "BLA38KDK2M"
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data.get("status"):
    print("Payment successfully verified!")
    print(f"Customer: {data.get('cus_name')} | Amount: {data.get('amount')}")
else:
    print("Verification failed:", data.get("message"))
```

</details>

#### Success Response (`200 OK`)

```json
{
  "status": true,
  "amount": "500.00",
  "cus_name": "Siyam Ahmed",
  "id": "a1b2c3d4e5f6g7h8"
}
```

#### Field Explanation
- `status`: `true` confirms that the SMS Transaction ID matched an incoming notification on your merchant phone, the amount matched, and the session is now marked completed.
- `amount`: Verified amount credited in BDT.
- `cus_name`: Name of the customer.
- `id`: Session ID confirmed.

#### Error Responses

| HTTP Code | JSON Error Response | Cause & Resolution |
|---|---|---|
| `400` | `{"status": false, "message": "Invalid transaction ID or transaction already used."}` | The TrxID does not exist in incoming SMS logs, or has already been claimed by another order. |
| `404` | `{"status": false, "message": "Payment session not found or expired."}` | Session ID is invalid or older than session lifetime. Call `/create` again. |
| `400` | `{"status": false, "message": "This payment session has already been completed."}` | Prevents double fulfillment. The session was already verified earlier. |
| `400` | `{"status": false, "message": "Unsupported payment method supplied."}` | Ensure `method` is one of `bkash`, `nagad`, `rocket`, `upay`. |
| `401` | `{"status": false, "message": "Invalid or inactive BRAND-KEY provided."}` | Verify that your Brand Key matches your active merchant profile. |

---

### 5.5 Headless Integration Best Practices & Rules

1. **⏱️ Android SMS Bridge Latency & Grace Period (5 to 20 Seconds):**
   When a customer completes a transaction in their bKash or Nagad app, the telecom operator delivers an SMS notification to your physical merchant smartphone. The SkyPay Android Sync App intercepts this SMS and relays it to the database over an encrypted TLS stream. This round-trip requires between **5 to 20 seconds**.
   * **Best Practice:** If a customer inputs their TrxID within seconds of paying, the SMS may still be in transit. Do not permanently reject the transaction immediately. If verification returns `false`, reply:  
     *"Payment matching in progress. Please wait 10 seconds and click Retry."*  
     Allow users a grace period of 2 to 3 minutes with an interactive retry button.

2. **🔒 Strict Idempotency & Replay Attack Prevention:**
   Once a `transaction_id` matches an incoming SMS record, SkyPay immediately marks that SMS as `claimed = 1`. **A transaction ID cannot be claimed a second time under any circumstance.** Subsequent attempts will fail instantly.

3. **🔡 Strict Lowercase Method Names:**
   Always convert gateway names to lowercase before calling the API:
   - ✅ Correct: `"bkash"`, `"nagad"`, `"rocket"`, `"upay"`
   - ❌ Wrong: `"bKash"`, `"BKASH"`, `"Nagad"`, `"RocketPay"`

4. **💰 Exact Amount Verification:**
   SkyPay compares the amount in the received SMS notification against the amount submitted during `/create`. If a customer pays 400 BDT for a 500 BDT session, the verification will fail.

---

## 6. Architectural Comparison Matrix

| Feature / Dimension | Hosted Gateway (`v1`) | Headless API (`v2`) |
|---|---|---|
| **Base Endpoint Domain** | `https://core.skypaybd.top` | `https://core.skypaybd.top` |
| **Authentication Required** | `BRAND-KEY` only | `BRAND-KEY` only |
| **Customer Experience** | Redirects to `core.skypaybd.top` | **Zero Redirect** &mdash; 100% inside your Bot/App |
| **UI Responsibility** | Handled completely by SkyPay | You build your own UI/Chat flow |
| **Telegram / Bot Fit** | Requires opening external browser | Native chat buttons & prompt messages |
| **Mobile App (Flutter/RN) Fit** | Requires WebView / In-App Browser | Native in-app UI inputs and modals |
| **Session Identification** | Return URL query string (`transactionId`) | Session `id` returned via JSON payload |
| **Verification Method** | Backend calls `/api/payment/verify` | Backend calls `/api/v2/payment/verify` |
| **Setup Time** | 10 to 15 Minutes | 1 to 2 Hours |
| **Best Used For** | WooCommerce, WHMCS, standard web stores | Telegram Bots, Discord Bots, Mobile Apps, SPAs |

---

## 7. Merchant Mobile App & Android SMS Synchronization Engine

SkyPay’s real-time payment matching is powered by our official **SkyPay Merchant Sync Android Application**.

```
[Customer Pays in bKash/Nagad App]
               │
               ▼ (GSM SMS Network)
[Merchant Android Smartphone]
   └── SkyPay Merchant Sync APK (Background Service)
               │
               ▼ (Encrypted TLS API Bridge)
[https://core.skypaybd.top Cloud Matcher]
               │
               ▼ (Database Instant Match)
[Merchant Verification API Endpoint]
```

### Application Specifications & Setup Guide

- **Download APK:** `/public/assets/downloads/SkyPay.apk`
- **Android Version:** Android 7.0 (Nougat) or higher.
- **Hardware Requirement:** Any entry-level Android smartphone containing your merchant SIM cards (bKash, Nagad, Rocket, Upay).
- **Setup Instructions:**
  1. Download and install `SkyPay.apk` on your merchant Android phone.
  2. Grant **SMS Listener Permission** and **Notification Access** so the app can read incoming payment SMS alerts.
  3. Disable **Battery Optimization** for the SkyPay app so Android does not kill the background sync service.
  4. Open the SkyPay app, enter your merchant **`BRAND-KEY`**, and tap **Connect Device**.
  5. The device indicator on your merchant dashboard will turn **Active (Green)**.
  6. The sync bridge is live! All incoming MFS transactions will now automatically match customer orders.

---

## 8. Pre-Built CMS Plugins & Module Downloads

For merchants running existing content management, web hosting billing, or social panel scripts, SkyPay provides pre-configured, production-ready modules requiring zero programming:

### 1. WordPress WooCommerce Plugin
- **Download:** `/public/assets/downloads/WP.zip`
- **Compatibility:** WordPress 5.8+ &middot; WooCommerce 6.0+ &middot; PHP 7.4 to 8.2
- **Key Features:**
  - Adds SkyPay as a native checkout gateway in WooCommerce.
  - Automatically updates orders to `Processing` or `Completed` on verified payment.
  - Supports automatic cart cancellation if the user aborts payment.
- **Setup:** Upload `WP.zip` via WordPress Admin &rarr; *Plugins* &rarr; *Add New* &rarr; *Upload*, activate, and paste your `BRAND-KEY` in WooCommerce Payment Settings.

### 2. WHMCS Web Hosting Billing Gateway
- **Download:** `/public/assets/downloads/WHMCS.zip`
- **Compatibility:** WHMCS 7.x &amp; 8.x
- **Key Features:**
  - Automatically marks invoices as `Paid` upon instant SMS verification.
  - Instantly triggers server and hosting provisioning (cPanel, Plesk, VPS).
- **Setup:** Extract `WHMCS.zip` into `modules/gateways/` in your WHMCS root directory, activate *SkyPay* in Payment Gateways, and enter your `BRAND-KEY`.

### 3. SMM Panel Automated Balance Module
- **Download:** `/public/assets/downloads/SMM.zip`
- **Compatibility:** SmartPanel, PerfectPanel, and custom SMM architectures.
- **Key Features:**
  - Instant auto-balance addition 24/7 without manual admin verification.
  - Fraud prevention and duplicate TrxID rejection.

### 4. Sketchware SWB Mobile Project
- **Download:** `/public/assets/downloads/SWB.zip`
- **Key Features:** Full visual block project file for Sketchware Android developers to integrate SkyPay directly into mobile APKs.

---

## 9. HTTP Status Codes, Error Dictionary & Troubleshooting

### Master Status Codes

| HTTP Status | Meaning | Resolution |
|---|---|---|
| `200 OK` | Request succeeded. | Check `"status": true` in the JSON body. |
| `400 Bad Request` | Missing parameters, invalid amount, or TrxID already claimed. | Inspect the returned `"message"` string for exact validation details. |
| `401 Unauthorized` | Missing or invalid `BRAND-KEY` header. | Verify that your Brand Key matches your merchant profile. |
| `403 Forbidden` | No active Android SMS sync device found. | Check that your Android phone running `SkyPay.apk` is powered on and connected to WiFi/Data. |
| `404 Not Found` | Payment session not found or expired. | Re-create the payment session using `/create`. |
| `405 Method Not Allowed` | Invalid HTTP verb (e.g. GET instead of POST). | Ensure your client sends `POST` requests. |
| `500 Internal Error` | Temporary cloud server error. | Retry request or contact SkyPay technical support. |

### Top 4 Integration Pitfalls & Solutions

1. **"No active SMS sync device found for this account" (`403 Forbidden`):**
   * *Cause:* The merchant Android phone running `SkyPay.apk` has disconnected, lost internet access, or battery saver stopped the app.
   * *Fix:* Open `SkyPay.apk` on the phone, ensure battery optimization is disabled, and verify that the status indicator shows green.

2. **Customer Paid but Verification Returns "Invalid transaction ID":**
   * *Cause:* The customer submitted their TrxID before the telecom SMS reached the merchant Android phone.
   * *Fix:* Implement a 10-second retry delay in your bot or app.

3. **Wrong Payment Method String:**
   * *Cause:* Sending `"Bkash"` or `"bKash"` instead of lowercase `"bkash"`.
   * *Fix:* Always normalize gateway names with `.toLowerCase()`.

4. **Spoofed Callbacks on Hosted Gateway:**
   * *Cause:* Fulfilling orders purely based on browser URL query strings.
   * *Fix:* Always execute a backend call to `/api/payment/verify` to confirm that the transaction is marked completed in the database.

---

## 10. Production Security & Deployment Checklist

Before rolling out your integration to live customers, verify that your application meets all security benchmarks:

- [ ] **Secure Storage:** Your `BRAND-KEY` is stored strictly in server environment variables (`.env`) and never exposed to client browsers or mobile code.
- [ ] **No SECRET-KEY Needed:** Confirm that your integration code does not require or depend on `SECRET-KEY`.
- [ ] **Unified Core Domain:** Verify that all requests are pointed to `https://core.skypaybd.top`.
- [ ] **Server-Side Verification:** Hosted Gateway implementations verify transactions via backend POST requests to `/api/payment/verify`.
- [ ] **Retry Grace Period:** Headless bot/app integrations provide a 10–20 second retry window for SMS synchronization.
- [ ] **Idempotency Guard:** Your backend database checks that an order is not fulfilled twice if a customer double-submits.
- [ ] **Android Phone Maintenance:** Merchant device is connected to stable power, reliable high-speed WiFi, and has background execution enabled 24/7.

---

<div align="center">

**SkyPay BD — Automated Payment Infrastructure for Bangladesh**  
*Enterprise-Grade Reliability for High-Growth Merchants, Telegram Bots & Mobile Apps*

🌐 [Official Website](https://skypaybd.top) • 📖 [Online Documentation](https://skypaybd.top/docs) • ⚡ [API Core](https://core.skypaybd.top)

</div>
