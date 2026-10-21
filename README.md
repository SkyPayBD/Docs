# SkyPay Developer Documentation

Direct MFS (bKash, Nagad, Rocket, Upay), Bangla QR, and decentralized crypto settlement rails across Bangladesh without intermediary cuts.

![SkyPay Architecture](https://skypaybd.top/public/assets/plat/mfs.png)

Direct-to-merchant SIM and non-custodial crypto collection engine with 5–20s automated SMS reconciliation and direct BSC blockchain node verification.



New to SkyPay? Review the **Authentication & Security** section below before dispatching API requests.

---

### Guide For LLM Models

For prompt context or autonomous agent integrations, copy the raw markdown documentation link:

```text
https://skypaybd.top/docs/llm.md
```

---

## Contents

- [Authentication & Security](#authentication--security)
- [Payment Creation Hub](#payment-creation-hub)
  - [Hosted Checkout (v1) — Create Payment](#hosted-checkout-v1--create-payment)
  - [Headless API (v2) — Initialize Payment Session](#headless-api-v2--initialize-payment-session)
- [Payment Verification Hub](#payment-verification-hub)
  - [Hosted Checkout (v1) — Verify Payment](#hosted-checkout-v1--verify-payment)
  - [Headless API (v2) — Multi-Rail Payment Verification](#headless-api-v2--multi-rail-payment-verification)
- [Programmatic SMS Gateway API](#programmatic-sms-gateway-api)
- [Device Health & Telemetry](#device-health--telemetry)
- [Webhooks & Instant Payment Notifications (IPN)](#webhooks--instant-payment-notifications-ipn)
- [Error Code Reference & Troubleshooting](#error-code-reference--troubleshooting)
- [Changelog & Versioning History](#changelog--versioning-history)

## Quick Start

1. Review [Authentication & Security](#authentication--security) and configure the appropriate credentials.
2. For hosted checkout integrations, follow [Hosted Checkout (v1) — Create Payment](#hosted-checkout-v1--create-payment), then [Hosted Checkout (v1) — Verify Payment](#hosted-checkout-v1--verify-payment).
3. For in-app or bot integrations, follow [Headless API (v2) — Initialize Payment Session](#headless-api-v2--initialize-payment-session), then [Headless API (v2) — Multi-Rail Payment Verification](#headless-api-v2--multi-rail-payment-verification).
4. Configure [Webhooks & IPN](#webhooks--instant-payment-notifications-ipn) for asynchronous payment notifications.
5. Use [Programmatic SMS Gateway API](#programmatic-sms-gateway-api) and [Device Health & Telemetry](#device-health--telemetry) for supported device-connected services.

---

## Solutions & Integration Architecture

Follow an integration route tailored to your platform architecture and customer checkout experience.

### Hosted Checkout (v1)
Generate secure checkout sessions and redirect customers to a responsive, pre-built payment interface. Supports automated MFS number copy, transaction ID input validation, and redirect webhooks.
* **Best for:** E-commerce stores, SaaS platforms, and quick plug-and-play integrations.
* **Reference:** [Read Hosted Checkout Guide →](#hosted-checkout-v1--create-payment)

### Headless In-App API (v2)
Build native, fully white-labeled payment flows inside your mobile applications, single-page web portals, and Telegram bots without browser redirection.
* **Best for:** Custom checkout UX, mobile apps (Flutter, React Native), and conversational bots.
* **Reference:** [Read Headless API Guide →](#headless-api-v2--initialize-payment-session)

### Multi-Rail Verification
Verify incoming transactions across multiple rails including MFS SMS proofs, Bangla QR push feeds, BEP20 EVM smart contract transfers, and Binance Pay orders.
* **Best for:** High-volume transaction reconciliation and automated order fulfillment.
* **Reference:** [Read Verification Hub Guide →](#headless-api-v2--multi-rail-payment-verification)

### Automated Webhooks (IPN)
Receive instant, cryptographically signed HTTP POST notifications to your backend whenever a transaction status changes. Includes HMAC-SHA256 signature verification and idempotency keys.
* **Best for:** Reliable asynchronous event handling and ledger synchronization.
* **Reference:** [Read Webhook Implementation →](#webhooks--instant-payment-notifications-ipn)

### Programmatic SMS Gateway
Dispatch OTPs, transactional alerts, and customer notifications via your own connected Android relay hardware without paying SMS aggregator fees.
* **Best for:** Low-cost OTP delivery, customer alerts, and system notifications.
* **Reference:** [Read SMS Gateway Documentation →](#programmatic-sms-gateway-api)

### Device Health & Telemetry
Monitor the hardware state of connected Android relay nodes, keepalive pings, battery vitals, and SIM network strength in real time.
* **Best for:** DevOps monitoring, uptime tracking, and fault detection.
* **Reference:** [Read Device Telemetry Guide →](#device-health--telemetry)

---

## Global Integration Prerequisites

> **Production API Base URLs**:
> * **Hosted & Core Gateway (v1)**: `https://core.skypaybd.top/api`
> * **Headless, Telemetry & SMS API (v2)**: `https://core.skypaybd.top/api/v2`
>
> All requests must be transmitted securely over HTTPS with TLS 1.3 encryption.

* **Transport Security**: Minimum TLS 1.2 enforced (TLS 1.3 strongly recommended). Plain HTTP connections will be rejected.
* **HTTP Headers**: All JSON payload requests must supply:
  ```http
  Content-Type: application/json
  Accept: application/json
  ```
* **Telecom Latency Window (5–20 Seconds)**:
  Mobile Financial Services transactions rely on cellular broadcast networks. Maintain an inherent processing buffer of 10–15 seconds before polling verification states.
* **Relay Hardware Power Management**:
  Android hardware running `SkyPay.apk` must remain connected to constant AC power with background battery optimization configured to **Unrestricted**.

---

## Developer Account & Support

For technical inquiries, contact support operations at `support@skypaybd.top` or open a ticket from your Merchant Portal.

---

# Authentication & Security

The SkyPay API implements a robust, multi-tiered authentication framework designed to secure merchant transactions, device communications, and programmatic services. Every request transmitted to SkyPay must provide valid credentials according to the specific endpoint category.

---

## 1. Header Specifications & Precedence

Authentication across SkyPay endpoints is handled via HTTP request headers. Depending on whether you are interacting with the Payment Gateway, SMS Gateway, or Device Telemetry endpoints, different headers are evaluated.

### Payment Gateway Authentication (Hosted v1 & Headless v2)

Merchant identity is validated via your unique **Brand Key** (generated in the SkyPay Merchant Portal).

| Header / Parameter | Location | Type | Status | Description |
|---|---|---|---|---|
| `BRAND-KEY` | HTTP Header | String | **Recommended** | Primary header evaluated across all payment endpoints. |
| `API-KEY` | HTTP Header | String | Supported Alias | Backward-compatible alias for `BRAND-KEY`. |
| `SECRET-KEY` | HTTP Header | String | Supported Alias | Alternative alias for legacy configurations. |
| `api_key` | Query / Body | String | Fallback Alias | Allowed in query string or JSON payload (not recommended for production). |

> **Evaluation Priority**:
> SkyPay checks for `BRAND-KEY` first, followed by `Authorization: Bearer`, `API-KEY`, `SECRET-KEY`, and finally `api_key` in the request body. Only one valid authentication credential is required per request.

### SMS Gateway Authentication

The SMS dispatching endpoint (`/api/v2/gateway/send`) utilizes a dedicated 18-character hardware gateway authorization key.

| Header | Type | Status | Description |
|---|---|---|---|
| `GATEWAY-KEY` | String | **Recommended** | Primary SMS gateway key assigned to your Android hardware bridge. |
| `gateway-key` | String | Supported Alias | Case-insensitive lowercase header. |
| `key` | String | Supported Alias | Short-form key header. |

### Device Telemetry Authentication

The Device Status endpoint (`/api/v2/device/status`) queries hardware operational states using comma-separated device keys.

| Header | Type | Required | Description |
|---|---|---|---|
| `DEVICE-KEYS` | String | **Required** | One or more comma-delimited device identifiers (e.g., `key_dev_01,key_dev_02`). |

---

## 2. Token Security Best Practices

To safeguard your merchant account and prevent unauthorized transaction execution, implement the following security controls:

### 1. Server-Side Environment Storage
Never hardcode API keys or Brand Keys into source code files. Store them in secure server-side environment variables:

```bash
# Production .env configuration
SKYPAY_BRAND_KEY="bk_live_9f83ac0194827dbe10928374a"
SKYPAY_GATEWAY_KEY="gw_a891029384758192"
SKYPAY_API_BASE="https://core.skypaybd.top/api"
```

### 2. Zero Client-Side Exposure
> **Never Expose Credentials in Frontend Code**:
> Never embed your `BRAND-KEY` or `GATEWAY-KEY` in client-facing applications such as browser single-page applications (React, Vue, Angular), mobile applications (Flutter, React Native, iOS, Android), or desktop clients. 
>
> All SkyPay API requests must originate from your secure backend server. Client applications should communicate with your own backend API, which securely signs and relays calls to SkyPay.

### 3. Key Rotation & Emergency Revocation
If a Brand Key or Gateway Key is accidentally exposed:
1. Log in to the SkyPay Merchant Portal immediately.
2. Navigate to **Settings > Brand**.
3. Generate a new Brand Key.
4. Update your production `.env` configuration and redeploy your service.
5. Deactivate the compromised key immediately.

---

## 3. Authorization Error Responses

If a request fails authentication, SkyPay immediately halts execution and returns an HTTP 401 Unauthorized or HTTP 403 Forbidden status code.

### 1. Missing Authentication Key (HTTP 401)
```json
{
  "status": false,
  "message": "BRAND-KEY header is required."
}
```

### 2. Invalid or Inactive Key (HTTP 401)
```json
{
  "status": false,
  "message": "Invalid or inactive BRAND-KEY provided."
}
```

### 3. Account Suspended (HTTP 403)
```json
{
  "status": false,
  "message": "Merchant account is suspended. Contact compliance support."
}
```

---

## 4. Multi-Language Authentication Examples

Switch between programming languages below to see how to transmit credentials across different HTTP clients:



### cURL


```bash
curl -X POST "https://core.skypaybd.top/api/v2/payment/create" \
  -H "BRAND-KEY: your_active_brand_key" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "cus_name": "Rahim Ahmed",
    "amount": 1200
  }'
```




### Node.js


```javascript
const axios = require('axios');

const skypayClient = axios.create({
  baseURL: 'https://core.skypaybd.top/api',
  headers: {
    'BRAND-KEY': process.env.SKYPAY_BRAND_KEY,
    'Content-Type': 'application/json',
    'Accept': 'application/json'
  },
  timeout: 10000
});

async function initiatePayment() {
  try {
    const response = await skypayClient.post('/v2/payment/create', {
      cus_name: 'Rahim Ahmed',
      amount: 1200
    });
    console.log('Session ID:', response.data.id);
  } catch (error) {
    console.error('Authentication or API error:', error.response?.data || error.message);
  }
}
```




### Python


```python
import os
import requests

API_URL = "https://core.skypaybd.top/api/v2/payment/create"
BRAND_KEY = os.getenv("SKYPAY_BRAND_KEY", "your_active_brand_key")

headers = {
    "BRAND-KEY": BRAND_KEY,
    "Content-Type": "application/json",
    "Accept": "application/json"
}

payload = {
    "cus_name": "Rahim Ahmed",
    "amount": 1200
}

response = requests.post(API_URL, json=payload, headers=headers, timeout=10)

if response.status_code == 200:
    data = response.json()
    print("Created Session ID:", data.get("id"))
else:
    print(f"Error {response.status_code}:", response.json())
```




### PHP


```php
<?php

$brandKey = getenv('SKYPAY_BRAND_KEY') ?: 'your_active_brand_key';
$url = 'https://core.skypaybd.top/api/v2/payment/create';

$payload = [
    'cus_name' => 'Rahim Ahmed',
    'amount'   => 1200
];

$ch = curl_init($url);
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST           => true,
    CURLOPT_HTTPHEADER     => [
        'BRAND-KEY: ' . $brandKey,
        'Content-Type: application/json',
        'Accept: application/json'
    ],
    CURLOPT_POSTFIELDS     => json_encode($payload),
    CURLOPT_TIMEOUT        => 10
]);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

$data = json_decode($response, true);
if ($httpCode === 200 && ($data['status'] ?? false)) {
    echo "Session ID: " . $data['id'];
} else {
    echo "Failed: " . ($data['message'] ?? 'Unknown error');
}
```




### Go


```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

type AuthRequest struct {
	CusName string  `json:"cus_name"`
	Amount  float64 `json:"amount"`
}

func main() {
	brandKey := os.Getenv("SKYPAY_BRAND_KEY")
	url := "https://core.skypaybd.top/api/v2/payment/create"

	payload, _ := json.Marshal(AuthRequest{
		CusName: "Rahim Ahmed",
		Amount:  1200.00,
	})

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(payload))
	req.Header.Set("BRAND-KEY", brandKey)
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("Accept", "application/json")

	client := &http.Client{Timeout: 10 * time.Second}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("HTTP Error: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Response (%d): %s\n", resp.StatusCode, string(body))
}
```




---

---

# Payment Creation Hub

The **Payment Creation Hub** introduces the two distinct payment initialization workflows supported by the SkyPay Core Engine:
1. **Version 1 — Hosted Checkout Gateway**: Generates a pre-built payment URL where customers are redirected to execute payments on a secure hosted UI.
2. **Version 2 — Headless In-App API**: Returns payment channel credentials (account numbers, Bangla QR image, Binance Pay UID, BEP20 receiver address) as structured JSON for rendering native screens in mobile apps and bots.

---

## Architectural Comparison: v1 Hosted vs. v2 Headless

| Integration Parameter | Hosted Checkout (v1) | Headless In-App API (v2) |
|---|---|---|
| **Primary Route** | `POST /api/payment/create` | `POST /api/v2/payment/create` |
| **User Journey** | Redirects customer browser to hosted payment URL | In-app native UI, modal dialog, or Telegram bot message |
| **Response Payload** | Returns `payment_url: "https://core.skypaybd.top/api/execute/{id}"` | Returns session `id`, merchant `brand`, and `methods` array |
| **Redirect Parameters** | Browser redirected to `success_url` or `cancel_url` | No browser redirection; customer submits TrxID in-app |
| **Webhook Delivery** | Dispatched automatically to `webhook_url` | Dispatched to merchant server upon backend verification |
| **Ideal For** | WooCommerce, WHMCS, WordPress, Laravel, PHP web stores | Flutter, React Native, Android/iOS, Telegram & Discord bots |

---

## 1. Version 1 — Hosted Checkout Overview

Hosted checkout allows you to initialize a session and redirect customers to a responsive, pre-built payment interface. After completion or cancellation, SkyPay redirects the user back to your site with transaction parameters appended as query strings.

### Quick Parameter Overview
* **Required Parameters**: `amount`, `success_url`, `cancel_url`
* **Optional Parameters**: `webhook_url`, `cus_name`, `cus_email`, `meta_data`, `return_type`
* **Key Response**: `payment_url` (hosted invoice URL)

> **Want full request schemas, redirect parameter lists, and complete code samples?**
>
> 👉 **[Read the Complete Hosted Checkout (v1) Guide](#hosted-checkout-v1--create-payment)**

---

## 2. Version 2 — Headless API Overview

The Headless API enables merchant applications to build custom payment UI screens natively without redirecting customers out of the app. SkyPay returns active payment methods, dynamic crypto conversion rates, and wallet addresses.

### Quick Parameter Overview
* **Required Parameters**: `cus_name`, `amount`
* **Optional Parameters**: `meta_data`
* **Key Response**: Temporary session `id`, `brand` profile, and dynamic `methods` array:
  * `bkash` (personal, agent, payment numbers)
  * `nagad` (personal, agent numbers)
  * `banglaqr` (interoperable QR code image URL)
  * `binance` (Binance UID, USDT exchange rate, converted `amount_usdt`)
  * `bep20` (BNB Smart Chain EVM receiver address, USDT rate, converted `amount_usdt`)

> **Want full JSON payloads, multi-rail response objects, and bot implementation guides?**
>
> 👉 **[Read the Complete Headless API (v2) Guide](#headless-api-v2--initialize-payment-session)**

---

## Navigation & Next Steps

Explore the detailed implementation guides for each version:

* [Hosted Checkout (v1) Create Guide](#hosted-checkout-v1--create-payment)
* [Headless API (v2) Create Guide](#headless-api-v2--initialize-payment-session)
* [Payment Verification Hub →](#payment-verification-hub)

---

# Hosted Checkout (v1) — Create Payment

The **Hosted Checkout (v1) Create Payment** endpoint initializes a customer payment session on the SkyPay platform and returns a pre-built, responsive hosted payment URL (`payment_url`). Customers are redirected to this URL to select their preferred payment channel (bKash, Nagad, Bank transfer) and execute payment within a managed interface.

---

## 1. Endpoint Specifications

* **Endpoint URL**: `https://core.skypaybd.top/api/payment/create`
* **HTTP Method**: `POST` (or `GET`)
* **Headers**:
  * `BRAND-KEY`: `<your_active_brand_key>` (Required)
  * `Content-Type`: `application/json`
  * `Accept`: `application/json`

---

## 2. Request Parameters

The payload can be delivered as raw JSON in the request body (recommended) or form-urlencoded parameters:

| Parameter | Type | Required | Default | Aliases | Description & Validation Constraints |
|---|---|---|---|---|---|
| `amount` | Float / Numeric | **Yes** | — | — | Bill amount in BDT. Constraints: Minimum `1.00`, Maximum `1,000,000.00`. |
| `success_url` | String (URL) | **Yes** | — | — | Absolute HTTPS URL where the customer's browser is redirected upon payment completion. |
| `cancel_url` | String (URL) | **Yes** | — | — | Absolute HTTPS URL where the customer's browser is redirected if payment is aborted. |
| `webhook_url` | String (URL) | No | Brand Default | — | Publicly accessible server endpoint for background IPN payment notifications. |
| `cus_name` | String | No | `"Default Name"` | `customer_name`, `c_name`, `name` | Customer's full name. Pre-filled on the hosted invoice. |
| `cus_email` | String | No | `"default@gmail.com"` | `customer_email`, `c_email`, `email` | Customer's email address for billing receipts. |
| `meta_data` | Object / JSON | No | `null` | `metadata` | Arbitrary JSON dictionary or valid JSON string echoed back in callbacks. |
| `return_type` | String | No | `"GET"` | — | Redirection HTTP verb (`GET` appends query strings to return URLs). |

---

## 3. Interactive Code Examples

Select your preferred language to view complete session initialization code:



### cURL


```bash
curl -X POST "https://core.skypaybd.top/api/payment/create" \
  -H "BRAND-KEY: your_active_brand_key" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "amount": 1500.00,
    "cus_name": "Rahim Ahmed",
    "cus_email": "rahim@example.com",
    "success_url": "https://merchant.com/payment/success",
    "cancel_url": "https://merchant.com/payment/cancel",
    "webhook_url": "https://merchant.com/api/payment-webhook",
    "meta_data": {
      "order_id": "ORD-99120",
      "user_id": 4821
    }
  }'
```




### Node.js


```javascript
const axios = require('axios');

async function createHostedCheckout(order) {
  const url = 'https://core.skypaybd.top/api/payment/create';
  
  const response = await axios.post(url, {
    amount: order.amount,
    cus_name: order.customerName,
    cus_email: order.customerEmail,
    success_url: 'https://merchant.com/payment/success',
    cancel_url: 'https://merchant.com/payment/cancel',
    webhook_url: 'https://merchant.com/api/payment-webhook',
    meta_data: {
      order_id: order.id
    }
  }, {
    headers: {
      'BRAND-KEY': process.env.SKYPAY_BRAND_KEY,
      'Content-Type': 'application/json'
    }
  });

  // Redirect customer browser to payment_url
  console.log('Redirect customer to:', response.data.payment_url);
  return response.data.payment_url;
}
```




### Python


```python
import os
import requests

def generate_hosted_checkout(amount: float, order_id: str, cus_name: str, cus_email: str):
    url = "https://core.skypaybd.top/api/payment/create"
    headers = {
        "BRAND-KEY": os.getenv("SKYPAY_BRAND_KEY"),
        "Content-Type": "application/json"
    }
    payload = {
        "amount": amount,
        "cus_name": cus_name,
        "cus_email": cus_email,
        "success_url": "https://merchant.com/payment/success",
        "cancel_url": "https://merchant.com/payment/cancel",
        "webhook_url": "https://merchant.com/api/payment-webhook",
        "meta_data": {"order_id": order_id}
    }

    response = requests.post(url, json=payload, headers=headers, timeout=10)
    response.raise_for_status()
    data = response.json()
    
    return data.get("payment_url")
```




### PHP


```php
<?php

function createHostedPayment($amount, $orderId, $brandKey) {
    $url = 'https://core.skypaybd.top/api/payment/create';
    
    $payload = [
        'amount'      => $amount,
        'success_url' => 'https://merchant.com/payment/success',
        'cancel_url'  => 'https://merchant.com/payment/cancel',
        'webhook_url' => 'https://merchant.com/api/payment-webhook',
        'meta_data'   => ['order_id' => $orderId]
    ];

    $ch = curl_init($url);
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST           => true,
        CURLOPT_HTTPHEADER     => [
            'BRAND-KEY: ' . $brandKey,
            'Content-Type: application/json'
        ],
        CURLOPT_POSTFIELDS     => json_encode($payload),
        CURLOPT_TIMEOUT        => 10
    ]);

    $response = curl_exec($ch);
    curl_close($ch);

    $result = json_decode($response, true);
    return $result['payment_url'] ?? null;
}
```




### Go


```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
)

type HostedPayload struct {
	Amount     float64                `json:"amount"`
	SuccessURL string                 `json:"success_url"`
	CancelURL  string                 `json:"cancel_url"`
	WebhookURL string                 `json:"webhook_url"`
	MetaData   map[string]interface{} `json:"meta_data"`
}

func CreateHostedCheckout(amount float64, orderID string) (string, error) {
	url := "https://core.skypaybd.top/api/payment/create"
	payload, _ := json.Marshal(HostedPayload{
		Amount:     amount,
		SuccessURL: "https://merchant.com/payment/success",
		CancelURL:  "https://merchant.com/payment/cancel",
		WebhookURL: "https://merchant.com/api/payment-webhook",
		MetaData:   map[string]interface{}{"order_id": orderID},
	})

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(payload))
	req.Header.Set("BRAND-KEY", os.Getenv("SKYPAY_BRAND_KEY"))
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return "", err
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	var res map[string]interface{}
	json.Unmarshal(body, &res)

	return fmt.Sprintf("%v", res["payment_url"]), nil
}
```




---

## 4. Response Payloads

### Success Response (HTTP 200 OK)
```json
{
  "status": true,
  "message": "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/api/execute/f7b3a9c2d1e0f8"
}
```

### Validation Error Responses

#### Missing Required Return URL (HTTP 422)
```json
{
  "status": false,
  "message": "The success_url field is required."
}
```

#### Amount Out of Allowed Bounds (HTTP 422)
```json
{
  "status": false,
  "message": "The amount must be between 1 and 1000000."
}
```

---

## 5. Hosted Checkout Redirection Lifecycle

Once the `payment_url` is generated, redirect the customer's browser. When the customer completes or cancels payment, SkyPay automatically appends transaction parameters to your provided return URLs:

#### 1. On Success / Settlement (`success_url`):
```http
GET https://merchant.com/payment/success?paymentMethod=bkash&transactionId=BLM37A9B1X&paymentAmount=1500.00&paymentFee=0.00&status=completed
```

#### 2. On Review / Pending Bank Settlement (`success_url`):
```http
GET https://merchant.com/payment/success?paymentMethod=bank&transactionId=FILE_OR_TRX&paymentAmount=1500.00&paymentFee=0.00&status=pending
```

#### 3. On Cancellation / Aborted Payment (`cancel_url`):
```http
GET https://merchant.com/payment/cancel?paymentMethod=bkash&transactionId=TRX_INIT_ID&paymentAmount=1500.00&paymentFee=0.00&status=failed
```

> **Never Fulfill Orders Based on URL Query Parameters Alone**:
> Browser query strings can be manipulated by malicious users. Always double-check transactions using the Hosted Verify endpoint or wait for cryptographically signed webhooks before crediting balances.

---

[← Back to Creation Hub](#payment-creation-hub) · [Proceed to Hosted Verify (v1) →](#hosted-checkout-v1--verify-payment)

---

# Headless API (v2) — Initialize Payment Session

The **Headless API (v2) Initialize Payment** endpoint enables mobile applications (Flutter, React Native, iOS, Android), custom web platforms, and automated chat-bots (Telegram, Discord) to initiate payment sessions and retrieve live channel credentials without redirecting users to an external website.

---

## 1. Endpoint Specifications

* **Endpoint URL**: `https://core.skypaybd.top/api/v2/payment/create`
* **HTTP Method**: `POST`
* **Headers**:
  * `BRAND-KEY`: `<your_active_brand_key>` (Required)
  * `Content-Type`: `application/json`
  * `Accept`: `application/json`

---

## 2. Request Parameters

| Parameter | Type | Required | Description & Constraints |
|---|---|---|---|
| `cus_name` | String | **Yes** | Full name or client username of the customer. |
| `amount` | Numeric / Float | **Yes** | Bill amount in BDT. Must be greater than `0.00`. |
| `meta_data` | Object | No | Arbitrary JSON dictionary stored with the session (e.g. `order_id`, `chat_id`). |

---

## 3. Interactive Code Examples



### cURL


```bash
curl -X POST "https://core.skypaybd.top/api/v2/payment/create" \
  -H "BRAND-KEY: your_active_brand_key" \
  -H "Content-Type: application/json" \
  -d '{
    "cus_name": "Tanvir Hasan",
    "amount": 2400,
    "meta_data": {
      "invoice_no": "INV-7731",
      "telegram_chat_id": 98127419
    }
  }'
```




### Node.js


```javascript
const axios = require('axios');

async function createHeadlessSession(customerName, amount, meta = {}) {
  const url = 'https://core.skypaybd.top/api/v2/payment/create';
  
  const response = await axios.post(url, {
    cus_name: customerName,
    amount: amount,
    meta_data: meta
  }, {
    headers: {
      'BRAND-KEY': process.env.SKYPAY_BRAND_KEY,
      'Content-Type': 'application/json'
    }
  });

  return response.data;
}
```




### Python


```python
import os
import requests

def init_headless_session(customer_name: str, amount_bdt: float, metadata: dict = None):
    url = "https://core.skypaybd.top/api/v2/payment/create"
    headers = {
        "BRAND-KEY": os.getenv("SKYPAY_BRAND_KEY"),
        "Content-Type": "application/json"
    }
    payload = {
        "cus_name": customer_name,
        "amount": amount_bdt,
        "meta_data": metadata or {}
    }

    response = requests.post(url, json=payload, headers=headers, timeout=10)
    response.raise_for_status()
    return response.json()
```




### PHP


```php
<?php
use GuzzleHttp\Client;

function initHeadlessPayment($name, $amount, $brandKey, $meta = []) {
    $client = new Client(['base_uri' => 'https://core.skypaybd.top/']);
    
    $response = $client->post('api/v2/payment/create', [
        'headers' => [
            'BRAND-KEY'    => $brandKey,
            'Content-Type' => 'application/json'
        ],
        'json' => [
            'cus_name'  => $name,
            'amount'    => $amount,
            'meta_data' => $meta
        ]
    ]);

    return json_decode($response->getBody(), true);
}
```




### Go


```go
package main

import (
	"bytes"
	"encoding/json"
	"io"
	"net/http"
	"os"
)

func InitHeadless(name string, amount float64, meta map[string]interface{}) ([]byte, error) {
	url := "https://core.skypaybd.top/api/v2/payment/create"
	payload, _ := json.Marshal(map[string]interface{}{
		"cus_name":  name,
		"amount":    amount,
		"meta_data": meta,
	})

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(payload))
	req.Header.Set("BRAND-KEY", os.Getenv("SKYPAY_BRAND_KEY"))
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	return io.ReadAll(resp.Body)
}
```




---

## 4. Multi-Rail Success Response (HTTP 200 OK)

The headless response returns all available payment rails configured for your merchant brand, including active numbers, exchange rates, and wallet addresses:

```json
{
  "status": true,
  "id": "ids_98fbc102e4aa",
  "brand": {
    "name": "SkyPay Official Store",
    "mobile": "+88017XXXXXXXX",
    "whatsapp": "+88017XXXXXXXX",
    "email": "support@skypaybd.top"
  },
  "methods": [
    {
      "name": "bkash",
      "active_payments": {
        "personal": true,
        "agent": false,
        "payment": true
      },
      "personal": "017XXXXXXXX",
      "agent": "",
      "payment": "017XXXXXXXX"
    },
    {
      "name": "nagad",
      "active_payments": {
        "personal": true,
        "agent": false
      },
      "personal": "018XXXXXXXX",
      "agent": ""
    },
    {
      "name": "banglaqr",
      "active_payments": {
        "qr": true
      },
      "qr_image": "https://core.skypaybd.top/uploads/qr/bangla_qr_sample.png"
    },
    {
      "name": "binance",
      "active_payments": {
        "personal": true,
        "merchant": false
      },
      "personal": "498217342",
      "currency": "USDT",
      "dollar_rate": 125.0,
      "amount_usdt": 19.2
    },
    {
      "name": "bep20",
      "active_payments": {
        "transfer": true
      },
      "network": "BNB Smart Chain (BEP20)",
      "currency": "USDT",
      "receiver_address": "0x51c7263b8be9238318e8df3c921312389102488a",
      "dollar_rate": 125.0,
      "amount_usdt": 19.2
    }
  ]
}
```

---

## 5. Building Bot & App UI from the Response

Your client application should inspect `methods` and render payment options:
* **MFS (bKash / Nagad)**: Display active wallet numbers and instruction (Send Money for personal, Payment for merchant).
* **Bangla QR**: Display the `qr_image` for instant scanning from any banking app.
* **Crypto (Binance Pay / BEP20)**: Display the target address or UID, along with the exact `amount_usdt` to send.

---

---

# Payment Verification Hub

The **Payment Verification Hub** introduces the verification and settlement workflows available across the SkyPay platform. After a customer executes payment via MFS, Bangla QR, or cryptocurrency rails, the merchant application must verify the transaction before fulfilling orders or crediting balances.

---

## Verification Architectures: v1 Query vs. v2 Multi-Rail

| Verification Dimension | Hosted Query (v1) | Headless Multi-Rail Verification (v2) |
|---|---|---|
| **Primary Route** | `POST /api/payment/verify` | `POST /api/v2/payment/verify` |
| **Verification Scope** | Server-to-server query of settled hosted transactions | Real-time verification across 4 distinct payment architectures |
| **Input Parameters** | `transaction_id` (settled SkyPay transaction ID) | `id` (session ID), `method`, `transaction_id` (proof string) |
| **Verification Logic** | Reads settled transaction record committed via hosted UI | Queries Android SMS relay, BSC RPC node, or Binance API |
| **Supported Rails** | bKash, Nagad, Bank transfer (hosted UI channels) | bKash, Nagad, Rocket, Upay, Bangla QR, BEP20 USDT, Binance Pay |
| **Ideal Use Case** | Double-checking callback parameters on web orders | Validating customer input in real time inside apps or bots |

---

## 1. Version 1 — Hosted Transaction Query Overview

Use `/api/payment/verify` from your backend server to double-check transaction states received from client redirection query parameters or webhooks.

### Key Features
* Accepts the settled `transaction_id` returned on `success_url`.
* Returns full transaction metadata, customer details, settlement amount, and final status (`COMPLETED`, `PENDING`, `FAILED`).
* Ensures web checkout orders are never fulfilled based on spoofed URL query parameters alone.

> **Want full request schemas, response formats, and status mapping tables?**
>
> 👉 **[Read the Complete Hosted Verification (v1) Guide](#hosted-checkout-v1--verify-payment)**

---

## 2. Version 2 — Headless Multi-Rail Verification Overview

The Headless verification endpoint validates client-submitted transaction proofs across four distinct payment architectures:

1. **Mobile Financial Services (bKash, Nagad, Rocket, Upay)**: Reconciles customer SMS Transaction IDs against incoming SMS relays on connected merchant Android phones.
2. **Bangla QR**: Matches customer paying mobile numbers (or last 4 digits) against real-time banking push notifications.
3. **BEP20 (USDT on BNB Smart Chain)**: Directly queries BSC decentralized RPC nodes (`eth_getTransactionReceipt`) to confirm smart contract token transfers.
4. **Binance Pay**: Direct server-to-server validation against Binance Open APIs using Binance Pay Order IDs.

> **Want rail-by-rail verification mechanics, smart contract topics, and polling strategies?**
>
> 👉 **[Read the Complete Headless Verification (v2) Guide](#headless-api-v2--multi-rail-payment-verification)**

---

## Navigation & Next Steps

* [Hosted Checkout (v1) Verify Guide](#hosted-checkout-v1--verify-payment)
* [Headless API (v2) Verify Guide](#headless-api-v2--multi-rail-payment-verification)
* [Webhooks & IPN Notifications →](#webhooks--instant-payment-notifications-ipn)

---

# Hosted Checkout (v1) — Verify Payment

The **Hosted Checkout (v1) Verify Payment** endpoint allows your backend server to query the definitive settlement state of any transaction processed through the hosted payment page.

---

## 1. Endpoint Specifications

* **Endpoint URL**: `https://core.skypaybd.top/api/payment/verify`
* **HTTP Method**: `POST` (or `GET`)
* **Headers**:
  * `BRAND-KEY`: `<your_active_brand_key>` (Required)
  * `Content-Type`: `application/json`
  * `Accept`: `application/json`

---

## 2. Request Parameters

| Parameter | Type | Required | Aliases | Description |
|---|---|---|---|---|
| `transaction_id` | String | **Yes** | `transactionId`, `transactionid`, `trx_id`, `trx`, `transaction` | Unique transaction ID returned on `success_url` or via webhook. |

---

## 3. Interactive Code Examples



### cURL


```bash
curl -X POST "https://core.skypaybd.top/api/payment/verify" \
  -H "BRAND-KEY: your_active_brand_key" \
  -H "Content-Type: application/json" \
  -d '{
    "transaction_id": "BLM37A9B1X"
  }'
```




### Node.js


```javascript
const axios = require('axios');

async function verifyHostedTransaction(trxId) {
  const url = 'https://core.skypaybd.top/api/payment/verify';
  const response = await axios.post(url, {
    transaction_id: trxId
  }, {
    headers: {
      'BRAND-KEY': process.env.SKYPAY_BRAND_KEY,
      'Content-Type': 'application/json'
    }
  });

  return response.data;
}
```




### Python


```python
import os
import requests

def query_hosted_payment(transaction_id: str):
    url = "https://core.skypaybd.top/api/payment/verify"
    headers = {
        "BRAND-KEY": os.getenv("SKYPAY_BRAND_KEY"),
        "Content-Type": "application/json"
    }
    payload = {"transaction_id": transaction_id}

    response = requests.post(url, json=payload, headers=headers, timeout=10)
    return response.json()
```




### PHP


```php
<?php

function verifyHostedPayment($trxId, $brandKey) {
    $url = 'https://core.skypaybd.top/api/payment/verify';
    $ch = curl_init($url);
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST           => true,
        CURLOPT_HTTPHEADER     => [
            'BRAND-KEY: ' . $brandKey,
            'Content-Type: application/json'
        ],
        CURLOPT_POSTFIELDS     => json_encode(['transaction_id' => $trxId]),
        CURLOPT_TIMEOUT        => 10
    ]);

    $response = curl_exec($ch);
    curl_close($ch);

    return json_decode($response, true);
}
```




### Go


```go
package main

import (
	"bytes"
	"encoding/json"
	"io"
	"net/http"
	"os"
)

func QueryTransaction(trxID string) ([]byte, error) {
	url := "https://core.skypaybd.top/api/payment/verify"
	payload, _ := json.Marshal(map[string]string{"transaction_id": trxID})

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(payload))
	req.Header.Set("BRAND-KEY", os.Getenv("SKYPAY_BRAND_KEY"))
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	return io.ReadAll(resp.Body)
}
```




---

## 4. Response Payloads

### Success Response (HTTP 200 OK)
```json
{
  "status": true,
  "data": {
    "cus_name": "Rahim Ahmed",
    "cus_email": "rahim@example.com",
    "amount": "1500.00",
    "transaction_id": "BLM37A9B1X",
    "meta_data": {
      "order_id": "ORD-99120"
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

### Status Mapping & Definitions:
* **`COMPLETED`**: Payment confirmed by SMS relay or banking node. Order can be safely fulfilled.
* **`PENDING`**: Payment under manual review (e.g. bank slip upload) or awaiting batch reconciliation.
* **`FAILED`**: Session cancelled, expired, or invalid.

---

---

# Headless API (v2) — Multi-Rail Payment Verification

The **Headless API (v2) Payment Verification** endpoint validates customer payments across four distinct payment architectures: Mobile Financial Services (bKash, Nagad, Rocket, Upay), Bangla QR, BEP20 (USDT on BNB Smart Chain), and Binance Pay.

---

## 1. Endpoint Specifications

* **Endpoint URL**: `https://core.skypaybd.top/api/v2/payment/verify`
* **HTTP Method**: `POST`
* **Headers**:
  * `BRAND-KEY`: `<your_active_brand_key>` (Required)
  * `Content-Type`: `application/json`
  * `Accept`: `application/json`

---

## 2. General Request Schema

| Field | Type | Required | Aliases | Description |
|---|---|---|---|---|
| `id` | String | **Yes** | — | Temporary session token returned by `/api/v2/payment/create`. |
| `method` | String | **Yes** | — | Selected payment channel: `bkash`, `nagad`, `rocket`, `upay`, `banglaqr`, `binance`, `bep20`. |
| `transaction_id` | String | **Yes** | `order_id`, `order`, `trx_id`, `transactionId`, `tx_hash`, `txhash`, `hash`, `number`, `mobile`, `trx`, `mobile_number` | Channel-specific transaction proof (SMS TrxID, sender mobile, EVM TxHash, or Binance Order ID). |

---

## 3. Channel-by-Channel Verification Mechanics

### Rail A: Mobile Financial Services (bKash, Nagad, Rocket, Upay)

Customer sends money via MFS app/USSD to your active number and inputs the provider's Transaction ID.

#### Verification Mechanics
* SkyPay queries the encrypted local database synchronized from your connected Android hardware relay phone.
* The system matches the SMS sender address (e.g., `bKash`, `16216`, `16167`), the recipient phone number, the parsed transaction amount, and the transaction ID.
* If a match is found and has not been credited previously, the transaction is marked `COMPLETED`.

> **Cellular SMS Latency Handling**:
> Telecommunication networks experience delivery latencies of **5 to 20 seconds**. If the customer submits their TrxID immediately, the SMS may still be in transit. Implement a client polling loop (every 5 seconds, up to 3 attempts) before displaying an error.

#### Request Payload
```json
{
  "id": "ids_98fbc102e4aa",
  "method": "bkash",
  "transaction_id": "CKL48A9D01"
}
```

#### Success Response (HTTP 200)
```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Tanvir Hasan",
    "cus_email": "headless@skypaybd.top",
    "amount": 2400,
    "transaction_id": "CKL48A9D01",
    "meta_data": {
      "invoice_no": "INV-7731"
    },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

---

### Rail B: Bangla QR

Customer scans your Bangla QR code via their banking application (Citytouch, MTBL, bKash, etc.) and completes payment.

#### Verification Mechanics
* Customer provides their **paying mobile number** (or the last 4 digits).
* SkyPay reconciles the payment against incoming real-time push data from the bank or acquirer gateway (TallyPay, MTB, etc.).
* Exact payment amount and sender MSISDN are matched.

#### Request Payload
```json
{
  "id": "ids_98fbc102e4aa",
  "method": "banglaqr",
  "transaction_id": "01712345678"
}
```

#### Success Response (HTTP 200)
```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Tanvir Hasan",
    "cus_email": "headless@skypaybd.top",
    "amount": 2400,
    "transaction_id": "BQR20491823",
    "meta_data": {
      "invoice_no": "INV-7731"
    },
    "payment_method": "banglaqr",
    "status": "COMPLETED"
  }
}
```

---

### Rail C: BEP20 (USDT on BNB Smart Chain)

Customer transfers USDT directly to your brand's designated EVM wallet on BNB Smart Chain.

#### Verification Mechanics
* The customer supplies the 66-character EVM Transaction Hash (`0x...`).
* SkyPay Core issues an authenticated JSON-RPC call (`eth_getTransactionReceipt`) to high-speed BSC nodes.
* The system verifies:
  1. Transaction execution status is `0x1` (Success).
  2. The contract called is the official BSC USDT Contract: `0x55d398326f99059ff775485246999027b3197955`.
  3. The topic hash is ERC20 `Transfer(address,address,uint256)`: `0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef`.
  4. The transfer destination address matches your merchant wallet address.
  5. The transferred amount equals or exceeds the converted `amount_usdt`.

#### Request Payload
```json
{
  "id": "ids_98fbc102e4aa",
  "method": "bep20",
  "transaction_id": "0x58cfa9d6bf303f8376997f7fa2032df362ab82cfb8ef10ad93144d67396a8498"
}
```

#### Success Response (HTTP 200)
```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Tanvir Hasan",
    "cus_email": "headless@skypaybd.top",
    "amount": 2400,
    "amount_usdt": 19.2,
    "transaction_id": "0x58cfa9d6bf303f8376997f7fa2032df362ab82cfb8ef10ad93144d67396a8498",
    "receiver": "0x51c7263b8be9238318e8df3c921312389102488a",
    "meta_data": {
      "invoice_no": "INV-7731"
    },
    "payment_method": "bep20",
    "status": "COMPLETED"
  }
}
```

---

### Rail D: Binance Pay

Customer sends USDT via Binance Pay to your Merchant Binance UID.

#### Verification Mechanics
* Customer inputs their Binance Pay Order ID or Transaction ID.
* SkyPay Core executes an authenticated server-to-server query against the Binance Pay Open API.
* The system verifies transaction settlement, recipient UID, currency (USDT), and paid volume.

#### Request Payload
```json
{
  "id": "ids_98fbc102e4aa",
  "method": "binance",
  "transaction_id": "2918471029384758"
}
```

#### Success Response (HTTP 200)
```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Tanvir Hasan",
    "cus_email": "headless@skypaybd.top",
    "amount": 2400,
    "transaction_id": "2918471029384758",
    "meta_data": {
      "invoice_no": "INV-7731"
    },
    "payment_method": "binance",
    "status": "COMPLETED"
  }
}
```

---

## 4. Interactive Multi-Language Code Examples



### cURL


```bash
curl -X POST "https://core.skypaybd.top/api/v2/payment/verify" \
  -H "BRAND-KEY: your_active_brand_key" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "ids_98fbc102e4aa",
    "method": "bkash",
    "transaction_id": "CKL48A9D01"
  }'
```




### Node.js


```javascript
const axios = require('axios');

async function verifyTrx(sessionId, method, transactionId) {
  const url = 'https://core.skypaybd.top/api/v2/payment/verify';
  const response = await axios.post(url, {
    id: sessionId,
    method: method,
    transaction_id: transactionId
  }, {
    headers: {
      'BRAND-KEY': process.env.SKYPAY_BRAND_KEY,
      'Content-Type': 'application/json'
    }
  });

  return response.data;
}
```




### Python


```python
import os
import requests

def verify_headless_trx(session_id: str, method: str, trx_id: str):
    url = "https://core.skypaybd.top/api/v2/payment/verify"
    headers = {
        "BRAND-KEY": os.getenv("SKYPAY_BRAND_KEY"),
        "Content-Type": "application/json"
    }
    payload = {
        "id": session_id,
        "method": method,
        "transaction_id": trx_id
    }

    response = requests.post(url, json=payload, headers=headers, timeout=15)
    return response.json()
```




### PHP


```php
<?php

function verifyHeadlessTrx($sessionId, $method, $trxId, $brandKey) {
    $url = 'https://core.skypaybd.top/api/v2/payment/verify';
    $payload = [
        'id'             => $sessionId,
        'method'         => $method,
        'transaction_id' => $trxId
    ];

    $ch = curl_init($url);
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST           => true,
        CURLOPT_HTTPHEADER     => [
            'BRAND-KEY: ' . $brandKey,
            'Content-Type: application/json'
        ],
        CURLOPT_POSTFIELDS     => json_encode($payload),
        CURLOPT_TIMEOUT        => 15
    ]);

    $response = curl_exec($ch);
    curl_close($ch);

    return json_decode($response, true);
}
```




### Go


```go
package main

import (
	"bytes"
	"encoding/json"
	"io"
	"net/http"
	"os"
)

func VerifyHeadless(sessionID, method, trxID string) ([]byte, error) {
	url := "https://core.skypaybd.top/api/v2/payment/verify"
	payload, _ := json.Marshal(map[string]string{
		"id":             sessionID,
		"method":         method,
		"transaction_id": trxID,
	})

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(payload))
	req.Header.Set("BRAND-KEY", os.Getenv("SKYPAY_BRAND_KEY"))
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	return io.ReadAll(resp.Body)
}
```




---

## 5. Failure & Error Responses

### Duplicate Transaction ID (HTTP 400 Bad Request)
```json
{
  "status": false,
  "message": "This transaction ID has already been verified and credited."
}
```

### Transaction Not Found (HTTP 404 Not Found)
```json
{
  "status": false,
  "message": "Transaction ID not found. Ensure customer has completed transfer."
}
```

### Upstream BSC RPC Timeout (HTTP 502 Bad Gateway)
```json
{
  "status": false,
  "message": "Upstream blockchain node timeout. Retry verification in 10 seconds."
}
```

---

---

# Programmatic SMS Gateway API

The **Programmatic SMS Gateway API** enables developers and merchant applications to dispatch real outgoing SMS messages (including OTPs, order updates, and marketing notifications) directly through their connected Android gateway smartphones.

---

## 1. Endpoint Specifications

* **Endpoint URL**: `https://core.skypaybd.top/api/v2/gateway/send`
* **HTTP Method**: `POST` (Strictly POST; `GET` requests return `405 Method Not Allowed`)
* **Content-Type**: `application/json` or `application/x-www-form-urlencoded`

### Authentication Headers

| Header | Type | Status | Description |
|---|---|---|---|
| `GATEWAY-KEY` | String | **Recommended** | Active 18-character SMS gateway authorization key. |
| `gateway-key` | String | Supported Alias | Case-insensitive header alias. |
| `key` | String | Supported Alias | Short-form key header. |

---

## 2. Request Parameters

| Parameter | Type | Required | Aliases | Description |
|---|---|---|---|---|
| `number` | String | **Yes** | `phone` | Destination mobile number (e.g. `01712345678` or `+88017XXXXXXXX`). |
| `text` | String | **Yes** | `message` | SMS body text. Full Unicode and native Bangla character support. |
| `id` | String | No | `message_id` | Optional client-side reference ID for tracking message delivery. |

---

## 3. Dispatch Flow Architecture

```text
Your Application (POST /api/v2/gateway/send)
       │
       ▼
SkyPay Core Engine ───► Validate GATEWAY-KEY & Active Device
       │
       ▼
Authenticate Service Account (OAuth2 JWT)
       │
       ▼
Fetch Target Device FCM Registration Token from Firestore
       │
       ▼
Dispatch Payload via Firebase Cloud Messaging (FCM HTTP v1)
       │
       ▼
Android Gateway Smartphone (SkyPay App) ───► SIM Card ───► Recipient Phone
```

---

## 4. Interactive Code Examples



### cURL


```bash
curl -X POST "https://core.skypaybd.top/api/v2/gateway/send" \
  -H "GATEWAY-KEY: your_18_char_gateway_key" \
  -H "Content-Type: application/json" \
  -d '{
    "number": "01712345678",
    "text": "Your verification code is 482910.",
    "id": "MSG_90218"
  }'
```




### Node.js


```javascript
const sendSms = async (phoneNumber, messageText, customId = '') => {
  const response = await fetch('https://core.skypaybd.top/api/v2/gateway/send', {
    method: 'POST',
    headers: {
      'GATEWAY-KEY': process.env.SKYPAY_GATEWAY_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      number: phoneNumber,
      text: messageText,
      id: customId
    })
  });

  const data = await response.json();
  return data;
};
```




### Python


```python
import os
import requests

def send_gateway_sms(phone: str, message: str, msg_id: str = ""):
    url = "https://core.skypaybd.top/api/v2/gateway/send"
    headers = {
        "GATEWAY-KEY": os.getenv("SKYPAY_GATEWAY_KEY"),
        "Content-Type": "application/json"
    }
    payload = {
        "number": phone,
        "text": message,
        "id": msg_id
    }

    response = requests.post(url, json=payload, headers=headers, timeout=10)
    return response.json()
```




### PHP


```php
<?php

$payload = [
    "number" => "01712345678",
    "text"   => "Your verification code is 482910.",
    "id"     => "MSG_90218"
];

$ch = curl_init('https://core.skypaybd.top/api/v2/gateway/send');
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST           => true,
    CURLOPT_HTTPHEADER     => [
        'GATEWAY-KEY: ' . getenv('SKYPAY_GATEWAY_KEY'),
        'Content-Type: application/json'
    ],
    CURLOPT_POSTFIELDS     => json_encode($payload)
]);

$response = curl_exec($ch);
curl_close($ch);
echo $response;
```




### Go


```go
package main

import (
	"bytes"
	"encoding/json"
	"io"
	"net/http"
	"os"
)

func SendSMS(phone, message, msgID string) ([]byte, error) {
	url := "https://core.skypaybd.top/api/v2/gateway/send"
	payload, _ := json.Marshal(map[string]string{
		"number": phone,
		"text":   message,
		"id":     msgID,
	})

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(payload))
	req.Header.Set("GATEWAY-KEY", os.Getenv("SKYPAY_GATEWAY_KEY"))
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	return io.ReadAll(resp.Body)
}
```




---

## 5. Responses & Error Reference

### Success Response (HTTP 200 OK)
```json
{
  "status": true,
  "message": "SMS queued successfully for delivery.",
  "data": {
    "id": "MSG_90218",
    "number": "01712345678",
    "text": "Your verification code is 482910.",
    "device_name": "Samsung Galaxy A07",
    "device_ip": "192.168.1.105",
    "timestamp": "2026-10-21 04:25:00"
  }
}
```

### Error Reference Table

| HTTP Status | Error Scenario | Response Message | Cause & Remediation |
|---|---|---|---|
| **400** | Device Offline | `Target device is offline or inactive.` | Android relay phone has disconnected or lost network IP. |
| **400** | Fields Missing | `Both number and text fields are required.` | Missing destination phone number or message body. |
| **401** | Key Missing | `GATEWAY-KEY header is required.` | No authentication header detected in request. |
| **401** | Invalid Key | `Invalid or inactive GATEWAY-KEY provided.` | Gateway key not found in active merchant registry. |
| **405** | Method Not Allowed | `Method not allowed. Only POST accepted.` | Request dispatched using `GET` instead of `POST`. |

---

---

# Device Health & Telemetry

SkyPay relies on a distributed cluster of dedicated Android relay smartphones running the `SkyPay.apk` synchronization service. These devices interface directly with telecommunication networks to capture real-time Mobile Financial Services (MFS) SMS transactions, monitor device connectivity, and report hardware vitals.

The **Device Status API** (`/api/v2/device/status`) allows merchants and platform administrators to inspect connection health, operational state, battery level, and last active intervals across single or multiple devices in real time.

---

## 1. Endpoint Details & Authentication

* **URL**: `https://core.skypaybd.top/api/v2/device/status`
* **Method**: `GET` | `POST`
* **Headers**:
  * `DEVICE-KEYS` (Required): Comma-separated list of active device keys (e.g., `dev_key_01,dev_key_02`).
  * `Content-Type`: `application/json`

---

## 2. Device State Machine & Lifecycle Logic

Every device connected to the SkyPay network transitions through an event-driven state machine based on ping acknowledgments, periodic heartbeat signals, and network availability:

```text
               ┌──────────────────────────────────────────────┐
               │              Device Registered               │
               └──────────────────────┬───────────────────────┘
                                      │
                         ┌────────────┴────────────┐
                         │ IP Assigned?            │
                         ├────────────┬────────────┤
                         │ No         │ Yes        │
                         ▼            ▼            │
                 [6: Not Connected]  [5: Never Active]
                         ▲            │ Valid Heartbeat
                         │            ▼
                         │     [1: Active (online)] ◄─────────────┐
                         │            │                           │
                         │            │ Age > 3m (180s)           │
                         │            ▼                           │
                         │     [3: Maybe Active (maybe_online)]   │ Pulse
                         │            │                           │ Received
                         │            │ Age > 7m (420s)           │ (≤180s)
                         │            ▼                           │
                         │     [4: Maybe Inactive (degraded)]     │
                         │            │                           │
                         │            │ Age > 10m (600s)          │
                         │            ▼                           │
                         │     [2: Inactive (offline)] ───────────┘
                         │
                         └─────── [0: On Queue (in_queue)]
                                  (Ping dispatched, age ≤ 60s)
```

### Comprehensive Status Code Reference

| Status Code | Enum Label | Internal State | Description & Criteria |
|---|---|---|---|
| `0` | **On Queue** | `in_queue` | Telemetry query/ping recently dispatched to the device and awaiting acknowledgment (age ≤ 60 seconds). |
| `1` | **Active** | `online` | Device is operational, connected, and has transmitted a valid heartbeat within the last **3 minutes** (≤ 180 seconds). |
| `2` | **Inactive** | `offline` | Device is completely offline. No valid heartbeat received for over **10 minutes** (> 600 seconds). |
| `3` | **Maybe Active** | `maybe_online` | Heartbeat pulse is slightly delayed (last seen between **3 to 7 minutes** ago / 180s–420s). |
| `4` | **Maybe Inactive** | `degraded` | Heartbeat interrupted. Device has not reported in **7 to 10 minutes** (420s–600s). Requires inspection. |
| `5` | **Never Active** | `never_active` | Device has an IP address assigned in the database but has never transmitted a verified telemetry payload. |
| `6` | **Not Connected** | `failed` | Application is not linked to device hardware (device IP address is absent from registry). |

---

## 3. Thresholds & Operational SLAs

| Threshold Metric | Duration Window | Evaluation Interval | Action Triggered |
|---|---|---|---|
| **Heartbeat Frequency** | 30 seconds | Continuous | Android relay pushes ping to Firestore / Core Engine. |
| **Ping Queue Window** | 60 seconds | On Status Query | Marks device as `in_queue` (`0`) while awaiting response. |
| **Healthy Online Window** | 0 to 180 seconds | Real-Time | State evaluates as `online` (`1`). Active payment routing enabled. |
| **Slight Delay Warning** | 180 to 420 seconds | Real-Time | State evaluates as `maybe_online` (`3`). Warning alert logged. |
| **Critical Delay Warning** | 420 to 600 seconds | Real-Time | State evaluates as `degraded` (`4`). Administrator alert pushed. |
| **Offline Failover** | > 600 seconds | Real-Time | State evaluates as `offline` (`2`). Transactions routed to backup SIM device. |

---

## 4. Downtime Calculation Algorithm

SkyPay computes elapsed hardware downtime and human-readable `last_active` strings using timestamp subtraction:

$$\Delta t = t_{	ext{current}} - t_{	ext{last\_heartbeat}}$$

### Formatting Rules:
* `now`: $\Delta t < 60	ext{ seconds}$
* `Xm`: $60	ext{s} \le \Delta t < 3,600	ext{s}$ (e.g., `5m`)
* `Xh`: $1	ext{h} \le \Delta t < 24	ext{h}$ (e.g., `2h`)
* `Xd`: $1	ext{d} \le \Delta t < 7	ext{d}$ (e.g., `3d`)
* `Xw`: $1	ext{w} \le \Delta t < 30	ext{d}$ (e.g., `1w`)
* `Xmo`: $30	ext{d} \le \Delta t < 365	ext{d}$ (e.g., `2mo`)
* `Xy`: $\Delta t \ge 365	ext{d}$ (e.g., `1y`)
* `never`: No pulse recorded in history

---

## 5. Telemetry Payload & Vitals Metadata



### cURL


```bash
curl -X GET "https://core.skypaybd.top/api/v2/device/status" \
  -H "DEVICE-KEYS: dev_key_pos_01,dev_key_pos_02,dev_key_backup"
```




### Node.js


```javascript
const axios = require('axios');

async function getDeviceStatus(deviceKeys) {
  const url = 'https://core.skypaybd.top/api/v2/device/status';
  const response = await axios.get(url, {
    headers: { 'DEVICE-KEYS': deviceKeys.join(',') }
  });
  return response.data;
}
```




### Python


```python
import requests

def query_devices(device_keys_list):
    url = "https://core.skypaybd.top/api/v2/device/status"
    headers = {"DEVICE-KEYS": ",".join(device_keys_list)}
    response = requests.get(url, headers=headers)
    return response.json()
```




### Success Response (HTTP 200 OK)
```json
{
  "status": true,
  "results": [
    {
      "name": "POS Terminal 1",
      "last_active": "now",
      "status": 1
    },
    {
      "name": "POS Terminal 2",
      "last_active": "4m",
      "status": 3
    },
    {
      "name": "Backup SIM Device",
      "last_active": "15m",
      "status": 2
    }
  ]
}
```

---

---

# Webhooks & Instant Payment Notifications (IPN)

Webhooks allow merchant applications to react to payment state transitions in real time without relying on customer browser redirection or repetitive backend polling. Whenever a payment is completed or changes status, SkyPay dispatches an automated HTTP POST notification containing the transaction payload directly to your configured `webhook_url`.

---

## 1. Webhook Delivery Lifecycle

```text
┌──────────────────────┐        ┌──────────────────────┐        ┌──────────────────────┐
│ Customer Payment     │───────>│ SkyPay Core Engine   │───────>│ Merchant Server      │
│ (MFS / QR / Crypto)  │        │ (Settlement Confirmed│        │ (Webhook Listener)   │
└──────────────────────┘        └──────────┬───────────┘        └──────────┬───────────┘
                                           │                               │
                                           │  HTTP POST (IPN Payload)      │
                                           ├──────────────────────────────>│
                                           │  X-SkyPay-Signature: HMAC     │
                                           │                               │
                                           │  HTTP 200 OK (within 5s)      │
                                           │<──────────────────────────────┤
                                           │                               │
                                           │ [If Timeout / Error: Retry]   │
                                           ├──────────────────────────────>│
```

---

## 2. Incoming Payload Schema

The webhook payload is delivered as raw JSON in the request body:

| Property | Type | Nullable | Description |
|---|---|---|---|
| `paymentMethod` | String | No | Payment channel used: `bkash`, `nagad`, `rocket`, `upay`, `banglaqr`, `binance`, `bep20`. |
| `transactionId` | String | No | Unique verified transaction identifier (MFS TrxID, EVM TxHash, Binance Order ID). |
| `paymentAmount` | String / Numeric | No | Formatted transaction settlement amount (e.g., `"1500.00"`). |
| `paymentFee` | String / Numeric | No | Processing fee deducted by SkyPay (e.g., `"0.00"`). |
| `status` | String | No | Transaction state: `completed`, `pending`, `failed`. |
| `meta_data` | Object / Null | Yes | Custom metadata dictionary provided during session initialization. |
| `timestamp` | String (ISO 8601) | No | UTC timestamp of event generation (e.g., `"2026-10-21T04:25:00Z"`). |

### Example Webhook JSON Body
```json
{
  "paymentMethod": "bkash",
  "transactionId": "BLM37A9B1X",
  "paymentAmount": "1500.00",
  "paymentFee": "0.00",
  "status": "completed",
  "meta_data": {
    "order_id": "ORD-99120",
    "user_id": 4821
  },
  "timestamp": "2026-10-21T04:25:00Z"
}
```

---

## 3. Webhook Signature Verification

To guarantee that the notification originates from SkyPay and has not been intercepted, tampered with, or forged, SkyPay includes an HMAC-SHA256 signature in the HTTP request headers:

* **Header**: `X-SkyPay-Signature`

### Verification Implementations



### Node.js


```javascript
const express = require('express');
const crypto = require('crypto');

const app = express();

app.use(express.json({
  verify: (req, res, buf) => {
    req.rawBody = buf.toString('utf8');
  }
}));

app.post('/api/payment-webhook', (req, res) => {
  const signature = req.headers['x-skypay-signature'];
  const brandKey = process.env.SKYPAY_BRAND_KEY;

  if (!signature) {
    return res.status(401).json({ error: 'Missing X-SkyPay-Signature header' });
  }

  const expectedSignature = crypto
    .createHmac('sha256', brandKey)
    .update(req.rawBody)
    .digest('hex');

  const isValid = crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expectedSignature, 'hex')
  );

  if (!isValid) {
    return res.status(403).json({ error: 'Invalid webhook signature' });
  }

  const event = req.body;
  if (event.status === 'completed') {
    // Process order idempotently
    console.log(`Payment confirmed: ${event.transactionId} for ${event.paymentAmount} BDT`);
  }

  return res.status(200).json({ status: true });
});
```




### PHP


```php
<?php

$signature = $_SERVER['HTTP_X_SKYPAY_SIGNATURE'] ?? '';
$brandKey = getenv('SKYPAY_BRAND_KEY') ?: 'your_active_brand_key';

$rawPayload = file_get_contents('php://input');

if (empty($signature) || empty($rawPayload)) {
    http_response_code(400);
    exit(json_encode(['error' => 'Missing signature or payload']));
}

$expectedSignature = hash_hmac('sha256', $rawPayload, $brandKey);

if (!hash_equals($expectedSignature, $signature)) {
    http_response_code(403);
    exit(json_encode(['error' => 'Invalid cryptographic signature']));
}

$data = json_decode($rawPayload, true);

if (($data['status'] ?? '') === 'completed') {
    $trxId = $data['transactionId'];
    $amount = $data['paymentAmount'];
    // Fulfill order idempotently
}

http_response_code(200);
echo json_encode(['status' => true]);
```




### Python


```python
import os
import hmac
import hashlib
from flask import Flask, request, jsonify

app = Flask(__name__)
BRAND_KEY = os.getenv("SKYPAY_BRAND_KEY", "your_active_brand_key").encode('utf-8')

@app.route('/api/payment-webhook', methods=['POST'])
def handle_webhook():
    incoming_sig = request.headers.get('X-SkyPay-Signature')
    if not incoming_sig:
        return jsonify({"error": "Missing signature header"}), 401

    raw_body = request.get_data()
    expected_sig = hmac.new(BRAND_KEY, raw_body, hashlib.sha256).hexdigest()

    if not hmac.compare_digest(incoming_sig, expected_sig):
        return jsonify({"error": "Invalid signature"}), 403

    payload = request.get_json()
    if payload.get("status") == "completed":
        trx_id = payload.get("transactionId")
        amount = payload.get("paymentAmount")
        # Fulfill order idempotently

    return jsonify({"status": True}), 200
```




---

## 4. Retry Schedule & Acknowledgement

### Expected Acknowledgement
* **Status Code**: `HTTP 200 OK`
* **Response Body**: `{"status": true}` or `OK`
* **Timeout Limit**: 5,000 milliseconds (5 seconds).

### Exponential Backoff Retry Matrix (8 Attempts over 24h)

| Attempt # | Delay From Prior Attempt | Elapsed Time Since Event |
|---|---|---|
| **1 (Initial)** | Immediate | 0 seconds |
| **2** | 5 minutes | 5 minutes |
| **3** | 15 minutes | 20 minutes |
| **4** | 30 minutes | 50 minutes |
| **5** | 1 hour | 1 hour 50 minutes |
| **6** | 2 hours | 3 hours 50 minutes |
| **7** | 6 hours | 9 hours 50 minutes |
| **8 (Final)** | 12 hours | 21 hours 50 minutes |

---

## 5. Idempotency & Concurrency Handling

Because network anomalies can cause notifications to be redelivered, your listener must be strictly **idempotent**:
1. **Unique Database Constraint**: Add a unique index to your payments table on `transaction_id`.
2. **Atomic Row Lock**: Lock the order record before processing:
   ```sql
   SELECT id, status FROM orders WHERE transaction_id = 'BLM37A9B1X' FOR UPDATE;
   ```
3. **Status Check**: If already `completed`, immediately return `HTTP 200 OK` without duplicating fulfillment.

---

---

# Error Code Reference & Troubleshooting

This document provides a comprehensive reference of all HTTP status codes, machine-readable error string constants, operational failure causes, and actionable troubleshooting steps across the SkyPay API platform.

---

## 1. Global HTTP Status Codes

| HTTP Status | Category | Meaning & System Implication |
|---|---|---|
| **`200 OK`** | Success | Request succeeded. Response body contains valid transaction or telemetry payload. |
| **`400 Bad Request`** | Client Error | Malformed JSON, missing required headers/fields, duplicate TrxID, or amount mismatch. |
| **`401 Unauthorized`** | Authentication | Missing, malformed, inactive, or invalid `BRAND-KEY`, `GATEWAY-KEY`, or token. |
| **`403 Forbidden`** | Authorization | Merchant account suspended by compliance, or IP address not whitelisted. |
| **`404 Not Found`** | Resource Missing | Payment session expired, transaction ID not found, or device keys unmatched. |
| **`405 Method Not Allowed`** | Protocol | Incorrect HTTP verb used (e.g., executing a `GET` request on a `POST`-only route). |
| **`422 Unprocessable Entity`** | Validation | Semantic validation failure (e.g., amount out of range `1` to `1,000,000`, invalid URL syntax). |
| **`429 Too Many Requests`** | Rate Limiting | Request threshold exceeded. Backoff required. |
| **`500 Internal Server Error`** | Server Error | Database commit error, transaction deadlock, or unhandled internal exception. |
| **`502 Bad Gateway`** | Upstream Error | Upstream provider timeout (e.g., BSC decentralized RPC node, Binance Pay API, or FCM relay). |
| **`503 Service Unavailable`** | Maintenance | SkyPay Core Engine undergoing scheduled maintenance or temporary rate-limiting. |

---

## 2. Comprehensive System Error Reference Table

| HTTP Status | Error Constant | Applicable Endpoints | Root Cause | Recommended Troubleshooting Steps |
|---|---|---|---|---|
| **400** | `MISSING_PARAMETERS` | All Endpoints | Required request body fields are absent. | Ensure `amount`, `cus_name`, or `transaction_id` are supplied in valid JSON format. |
| **400** | `DUPLICATE_TRANSACTION_ID` | `/api/v2/payment/verify` | The customer-submitted TrxID has already been credited. | Check your internal orders database. Ensure customer is not reusing an old SMS proof. |
| **400** | `AMOUNT_MISMATCH` | `/api/v2/payment/verify` | Transferred amount does not match session invoice amount. | Prompt customer to send the exact bill amount or issue an adjustment invoice. |
| **400** | `DEVICE_OFFLINE` | `/api/v2/gateway/send` | Android SMS relay device has no assigned IP or is disconnected. | Inspect the physical Android phone. Verify Wi-Fi/cellular connection and open `SkyPay.apk`. |
| **400** | `MISSING_DEVICE_KEYS` | `/api/v2/device/status` | `DEVICE-KEYS` header was omitted from the request. | Provide a comma-separated list of valid device identifiers in the `DEVICE-KEYS` header. |
| **401** | `UNAUTHORIZED` | All Endpoints | No authentication header detected. | Pass `BRAND-KEY: <key>` or `Authorization: Bearer <key>` in your HTTP request headers. |
| **401** | `INVALID_BRAND_KEY` | Payment Endpoints | The provided Brand Key does not match any active merchant. | Verify your credentials in the Merchant Portal under **API Settings**. |
| **401** | `INVALID_GATEWAY_KEY` | `/api/v2/gateway/send` | 18-character SMS gateway key is incorrect or deactivated. | Verify your hardware key in the **SMS Gateway > Devices** tab of the portal. |
| **403** | `ACCOUNT_SUSPENDED` | All Endpoints | Merchant account suspended due to compliance, chargeback, or policy issues. | Contact SkyPay Merchant Support at `support@skypaybd.top` to resolve account flags. |
| **403** | `IP_FORBIDDEN` | All Endpoints | Request originated from an IP not listed in merchant whitelist. | Update your static server IP list in Merchant Portal **Security > IP Whitelist**. |
| **404** | `SESSION_NOT_FOUND` | `/api/v2/payment/verify` | Session `id` is invalid or does not exist. | Verify that session token returned by `/api/v2/payment/create` was stored correctly. |
| **404** | `SESSION_EXPIRED` | `/api/v2/payment/verify` | Payment session exceeded the 15-minute validity window. | Prompt user to re-initialize a new payment session. |
| **404** | `TRANSACTION_NOT_FOUND` | `/api/payment/verify`, `/api/v2/payment/verify` | TrxID was not found in device SMS logs or blockchain. | Wait 10 seconds and retry. If still absent, verify customer sent money to correct wallet. |
| **404** | `NO_MATCHING_DEVICES` | `/api/v2/device/status` | None of supplied `DEVICE-KEYS` match registered hardware. | Check device key spelling and ensure device was not deleted from merchant dashboard. |
| **405** | `METHOD_NOT_ALLOWED` | All Endpoints | Incorrect HTTP verb used (e.g. `GET` on `/api/v2/payment/create`). | Verify API documentation and switch to `POST`. |
| **422** | `OUT_OF_BOUNDS_AMOUNT` | `/api/payment/create` | Bill amount is `< 1.00` or `> 1,000,000.00` BDT. | Restrict payment amounts to the allowed range (1 BDT to 1,000,000 BDT). |
| **422** | `INVALID_URL_SYNTAX` | `/api/payment/create` | `success_url` or `cancel_url` is not an absolute HTTPS URL. | Ensure all return URLs include the `https://` protocol and a valid domain name. |
| **429** | `RATE_LIMIT_EXCEEDED` | All Endpoints | Merchant exceeded rate limit (e.g., > 120 requests/minute). | Implement exponential backoff and jitter in client requests. Cache non-critical queries. |
| **500** | `DATABASE_COMMIT_FAILED`| All Endpoints | Database deadlocked during concurrent transaction commitment. | Retry request after 2 seconds. Transaction state remains uncommitted. |
| **502** | `BSC_RPC_TIMEOUT` | `/api/v2/payment/verify` | Binance Smart Chain RPC node failed to return block receipt in 10s. | Retry verification after 10 seconds. SkyPay automatically fails over to backup RPC nodes. |
| **502** | `BINANCE_API_TIMEOUT` | `/api/v2/payment/verify` | Binance Pay Open API connection timed out. | Retry verification after 5 seconds. |
| **502** | `FCM_DISPATCH_FAILURE`| `/api/v2/gateway/send` | Firebase Cloud Messaging failed to wake up target Android phone. | Verify relay phone has an active internet connection and Google Play Services enabled. |

---

## 3. Diagnostic & Troubleshooting Workflow

```text
               ┌──────────────────────────────────────┐
               │         API Request Failed?          │
               └──────────────────┬───────────────────┘
                                  │
                 ┌────────────────┴────────────────┐
                 ▼                                 ▼
         [HTTP 4xx Error]                  [HTTP 5xx Error]
                 │                                 │
         ┌───────┴───────┐                 ┌───────┴───────┐
         ▼               ▼                 ▼               ▼
     [401 / 403]     [400 / 422]         [502]           [500]
     Credential     Payload / Schema   Upstream Node    Database
     Inspection     Validation         Timeout/Retry    Deadlock
         │               │                 │               │
         ▼               ▼                 ▼               ▼
    Verify .env     Check Data Types   Retry in 10s    Retry with
    & IP Rules      & Constraints      Auto-Failover   Backoff
```

---

---

# Changelog & Versioning History

SkyPay follows [Semantic Versioning (SemVer 2.0.0)](https://semver.org/). API releases are numbered in the format `vMAJOR.MINOR.PATCH`.

---

## Release History

### v2.2.0 — October 2026 (Current Stable)
**Added**:
* **Multi-Device Telemetry Endpoint (`/api/v2/device/status`)**: Query operational states, battery percentages, and downtime formatting across multiple Android relay devices using comma-separated keys.
* **HMAC-SHA256 Webhook Signatures**: Outgoing instant payment notifications now include the `X-SkyPay-Signature` header for cryptographic origin verification and tamper prevention.
* **BSC Node Multi-Provider Failover**: Decentralized BEP20 USDT transaction verification now queries secondary RPC nodes (Ankr, QuickNode) if the primary public BSC node times out.

---

### v2.1.0 — July 2026
**Added**:
* **Programmatic SMS Gateway API (`/api/v2/gateway/send`)**: Dispatch real outbound SMS messages via connected merchant Android relay hardware.
* **Unicode & Bangla SMS Encoding**: Full native support for Bangla character sets and multi-part SMS messages.
* **FCM HTTP v1 Migration**: Upgraded internal push dispatching to OAuth2 JWT Google Service Account credentials for sub-second wakeups.

---

### v2.0.0 — March 2026 (Major Release)
**Added**:
* **Headless In-App API (`/api/v2/payment/create` & `/api/v2/payment/verify`)**: 100% white-label API architecture for Flutter, React Native, iOS, Android, and Telegram bots.
* **Bangla QR Interoperability**: Automatic reconciliation of merchant QR push notifications via sender MSISDN matching.
* **BEP20 USDT Smart Contract Engine**: Automatic on-chain reconciliation of Binance Smart Chain USDT transfers using EVM transaction hashes.
* **Binance Pay Direct Integration**: Authenticated server-to-server Binance Open API settlement.

---

### v1.2.0 — November 2025
**Added**:
* **Custom Metadata Support**: Merchants can attach arbitrary JSON objects (`meta_data`) during payment session initialization.
* **Customer Profile Auto-Fill**: Hosted checkout supports `cus_name` and `cus_email` parameters to pre-fill invoice contact fields.
* **Automated Webhook Callbacks**: Automatic HTTP POST delivery on hosted payment execution.

---

### v1.0.0 — January 2025 (Initial Release)
**Added**:
* Launch of SkyPay Core Hosted Checkout Gateway (`/api/payment/create`, `/api/execute/{id}`, `/api/payment/verify`).
* Direct automated verification for bKash and Nagad personal accounts via Android SMS relay bridge.
* WooCommerce and WHMCS payment gateway plugins.

---

## Migration Guide: Migrating from v1 Hosted to v2 Headless

If your application currently uses Hosted Checkout (v1) and you want to build custom, in-app checkout screens without browser redirection, follow these migration steps:

### 1. Update Session Creation Endpoint
* **Old Endpoint**: `POST https://core.skypaybd.top/api/payment/create`
* **New Endpoint**: `POST https://core.skypaybd.top/api/v2/payment/create`

### 2. Update Request Parameters
Remove `success_url` and `cancel_url` from your payload (these are not needed in headless mode). Supply `cus_name`, `amount`, and optional `meta_data`.

### 3. Handle Method Arrays
Instead of receiving a `payment_url`, your client application will receive an array of active `methods`:
```json
{
  "id": "ids_98fbc102e4aa",
  "methods": [
    { "name": "bkash", "personal": "017XXXXXXXX" },
    { "name": "binance", "amount_usdt": 19.2 }
  ]
}
```

### 4. Direct Client Verification
When the customer submits their transaction proof, submit it directly to `/api/v2/payment/verify`:
```json
{
  "id": "ids_98fbc102e4aa",
  "method": "bkash",
  "transaction_id": "CKL48A9D01"
}
```

---
