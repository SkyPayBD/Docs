# SkyPay — Mobile App Integration Guide
<div align="center">
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png" width="100" alt="SkyPay Logo" />
<br/>
<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789098593_de7d9d238ad2e0178762.png" width="420" alt="SkyPay Banner" />
<br/><br/>
**Headless Payment Integration for Native & Cross-Platform Mobile Apps**
*Accept automated payments via **bKash**, **Nagad**, **Rocket**, and **Upay** — directly inside your Flutter, Kotlin, Java, or Swift app — without any browser redirect or WebView.*
<br/>
[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/API%20Version-v2.0%20Headless-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
[![Platforms](https://img.shields.io/badge/Platforms-Flutter%20%7C%20Kotlin%20%7C%20Java%20%7C%20Swift-f97316?style=for-the-badge&logo=android&logoColor=white)](#)
</div>

---

> 📖 **This document covers mobile app integration only** — Flutter, Kotlin (Android), Java (Android), and Swift (iOS). For the complete API reference and web/bot integrations, see the main documentation at [skypaybd.top/docs](https://skypaybd.top/docs) or the [SkyPayBD/Docs](https://github.com/SkyPayBD/Docs) repository.

---

## 📋 Table of Contents
- [Why Headless API v2 for Mobile](#-why-headless-api-v2-for-mobile)
- [API Base URL](#-api-base-url)
- [Authentication](#-authentication)
- [Endpoint Directory](#-endpoint-directory)
- [Security Warning — BRAND-KEY Handling](#-security-warning--brand-key-handling)
- [Platform Config Files](#-platform-config-files)
- [3-Step Headless Payment Flow](#-3-step-headless-payment-flow)
- [Architecture Per Platform](#-architecture-per-platform)
  - [Flutter](#flutter-architecture)
  - [Kotlin (Android)](#kotlin-android-architecture)
  - [Java (Android)](#java-android-architecture)
  - [Swift (iOS)](#swift-ios-architecture)
- [Retry Logic (All Platforms)](#-retry-logic-all-platforms)
- [Idempotency & Fraud Prevention](#-idempotency--fraud-prevention)
- [Method Name Validation](#-method-name-validation)
- [Complete Flow Summary](#-complete-flow-summary)
- [Error Reference](#-error-reference)
- [Integration Checklist](#-integration-checklist)
- [Quick Links](#-quick-links)
- [Contact & Support](#-contact--support)

---

## 🌐 Why Headless API v2 for Mobile

Mobile apps cannot use the **Hosted Gateway (v1)** because it requires browser redirects that break the native app experience. Instead, all mobile app integrations must use the **Headless API v2**, which keeps the entire payment flow inside the app.

The Hosted Gateway redirects the user to an external SkyPay webpage. In a mobile app, this would require a WebView, which:
- Breaks the native UX
- Makes it impossible to cleanly intercept the callback
- Requires handling deep links or custom URL schemes

The Headless API v2 eliminates all of this. Your app calls the API, gets active wallet numbers, shows them to the user natively, collects the TrxID through a native input, and verifies it — all without leaving the app.

**This guide applies equally to:** Flutter · Kotlin (Android) · Java (Android) · Swift (iOS) — the API contract is identical; only the client-side code differs.

---

## 🔗 API Base URL
All Headless API v2 calls use:
```
https://core.skypaybd.top
```

---

## 🔐 Authentication
Every request requires exactly **one header** — there is no `SECRET-KEY`:
```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
Accept: application/json
```

---

## 📡 Endpoint Directory

| Action | Method | Full Endpoint URL |
| :--- | :--- | :--- |
| Initiate Payment Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| Verify Transaction | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

---

## 🔒 Security Warning — BRAND-KEY Handling

For production apps, the `BRAND-KEY` must **NOT** be shipped inside the mobile app binary. Anyone can decompile an APK or IPA and extract hardcoded strings. The recommended production architecture is:

1. Your mobile app talks to **your own backend server**
2. Your backend server holds the `BRAND-KEY` and calls SkyPay on behalf of the app
3. The app only ever receives the wallet numbers and session ID — relayed through your backend

For development or internal tools, the `BRAND-KEY` can temporarily live in a config file, but it must be removed before any public release.

---

## ⚙️ Platform Config Files

### Flutter — `lib/config/skypay_config.dart`
Define constants:
- `kSkyPayV2CreateUrl` → `https://core.skypaybd.top/api/v2/payment/create`
- `kSkyPayV2VerifyUrl` → `https://core.skypaybd.top/api/v2/payment/verify`
- `kSkyPayBrandKey` → your brand key (from environment or secure storage in production)

### Kotlin (Android) — `config/SkyPayConfig.kt`
Create an `object SkyPayConfig`:
- `const val V2_CREATE_URL`
- `const val V2_VERIFY_URL`
- `const val BRAND_KEY`

### Java (Android) — `config/SkyPayConfig.java`
Create a `final class SkyPayConfig` with `public static final String` constants for the same values.

### Swift (iOS) — `Config/SkyPayConfig.swift`
Create an `enum SkyPayConfig` with `static let` constants for both URLs and the brand key.

---

## 🔄 3-Step Headless Payment Flow

### Step 1 — Initiate Payment Session

When the user triggers a payment (taps "Buy", "Deposit", "Subscribe", etc.), call:

```
POST https://core.skypaybd.top/api/v2/payment/create
```

**Request body:**
```json
{
  "cus_name": "User's name or username",
  "amount": 500,
  "meta_data": {
    "user_id": "internal_user_id",
    "product_id": "PLAN_PRO",
    "platform": "android"
  }
}
```

**Response parsing — critical fields:**
- `id` (String): The SkyPay session ID. **Save it immediately** in your ViewModel / state object — you will need it in Step 3. If lost, the session cannot be verified and a new one must be created.
- `methods` (Array): Each element represents a payment channel (`bkash`, `nagad`, `rocket`, `upay`) with:
  - `name`: channel identifier
  - `active_payments.personal`: boolean — is the personal (Send Money) number active?
  - `active_payments.agent`: boolean — is the agent (Cash In) number active?
  - `active_payments.payment`: boolean — is merchant payment active? (bKash only)
  - `personal`: wallet number for Send Money
  - `agent`: wallet number for Cash In
  - `payment`: wallet number for Merchant Pay

**Filtering rule:** Only display a wallet option if the corresponding `active_payments` flag is `true` **and** the number string is not empty. Never show the user a channel with an inactive flag or empty number.

### Step 2 — Display Payment Instructions

Build a native payment screen inside the app that shows:
- The exact BDT amount the user must send
- A list of available wallet options (filtered from Step 1)
- For each wallet: channel name, type (Personal/Agent), and the number to send to
- Clear instruction: "Send the exact amount, then enter your TrxID"
- A text input field for the TrxID
- If multiple channels are available, a selector for which one the user paid with (radio buttons or dropdown)
- A submit/verify button and a cancel option

### Step 3 — Verify Transaction

When the user submits their TrxID, call:

```
POST https://core.skypaybd.top/api/v2/payment/verify
```

**Request body:**
```json
{
  "id": "<session_id_from_step_1>",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

The `method` field must always be **strict lowercase**: `bkash`, `nagad`, `rocket`, `upay`. Apply the platform's lowercase function before sending.

**Response handling:**
- `status: true` → Payment verified. Proceed with in-app fulfillment (unlock content, add balance, activate plan).
- `status: false` → Check the `message` field:
  - TrxID not found / SMS not yet received → show a **Retry** button; ask the user to wait 10 seconds. Allow retries for up to 2–3 minutes.
  - TrxID already used → show a permanent error: "This transaction ID has already been used."
  - Session expired → tell the user to start the payment process again.

---

## 🏗️ Architecture Per Platform

### Flutter Architecture

**Service layer** — `lib/services/skypay_service.dart`
- Use the `http` package (or `dio` for more features)
- `Future<Map<String, dynamic>> createSession(...)` → calls the v2 create endpoint
- `Future<Map<String, dynamic>> verifySession(String sessionId, String method, String txnId)` → calls the v2 verify endpoint
- Handle HTTP exceptions and return structured error maps

**State management** — `lib/providers/payment_provider.dart` (or `PaymentCubit`/`PaymentBloc` if using BLoC)
- State includes: `sessionId`, `amount`, `availableMethods`, `selectedMethod`, `status` (idle/loading/awaitingTxnId/verifying/success/error), `errorMessage`
- `initiatePayment(String cusName, double amount, Map metadata)` → calls service, updates state with session ID and methods
- `verifyPayment(String txnId)` → calls service, updates state to success or error

**UI layer**
- `PaymentScreen` (StatelessWidget or ConsumerWidget) observing provider state
- Loading indicator during API calls
- Wallet list once methods are available
- TrxID input after the user selects a channel
- Retry button on pending verification failure
- Success/error message on final outcome

**HTTP headers for Flutter (`http` package):**
```dart
headers: {
  'BRAND-KEY': SkyPayConfig.kSkyPayBrandKey,
  'Content-Type': 'application/json',
  'Accept': 'application/json',
}
```

### Kotlin (Android) Architecture

**Network layer** — Retrofit + OkHttp
- `SkyPayApiService` interface with two `suspend` functions: `createPayment(...)` and `verifyPayment(...)`
- `SkyPayRetrofitClient` object that builds the Retrofit instance with:
  - `baseUrl("https://core.skypaybd.top")`
  - An OkHttp interceptor adding the `BRAND-KEY` and `Content-Type` headers to every request
  - `GsonConverterFactory` for JSON serialization

**Data models** — Kotlin data classes:
- `SkyPayCreateRequest(cusName, amount, metaData)`
- `SkyPayCreateResponse(status, id, brand, methods)`
- `SkyPayMethod(name, activePayments, personal, agent, payment)`
- `SkyPayVerifyRequest(id, method, transactionId)`
- `SkyPayVerifyResponse(status, amount, cusName, id)`

**ViewModel** — `PaymentViewModel` (extends `ViewModel`) with:
- `MutableStateFlow<PaymentUiState>` or `LiveData` for UI state
- `fun initiatePayment(cusName: String, amount: Double, metaData: Map<String, Any>)` — calls repository in `viewModelScope.launch`
- `fun verifyPayment(txnId: String)` — calls repository in a coroutine
- Stores `sessionId` and `selectedMethod` between steps

**Repository** — `SkyPayRepository` sits between ViewModel and Retrofit, handles error wrapping and returns `Result<T>`.

**UI** — A single `PaymentFragment` or `PaymentActivity` with multiple view states managed by the ViewModel, using `lifecycleScope.collect` or `observe` on ViewModel LiveData.

### Java (Android) Architecture

Same architecture as Kotlin but using:
- `Call<T>` with Retrofit callbacks instead of coroutines
- `MutableLiveData<T>` for the ViewModel
- `SkyPayRetrofitClient` built with `OkHttpClient.Builder().addInterceptor(...)`
- POJOs instead of data classes for request/response models

### Swift (iOS) Architecture

**Network layer** — `URLSession` or Alamofire
- `SkyPayService.swift` as a class or actor
- `func createPayment(...) async throws -> SkyPayCreateResponse`
- `func verifyPayment(...) async throws -> SkyPayVerifyResponse`
- Add the BRAND-KEY header: `request.setValue(SkyPayConfig.brandKey, forHTTPHeaderField: "BRAND-KEY")`

**Data models** — `Codable` structs for all request/response types. Use `CodingKeys` to map snake_case JSON keys to camelCase Swift properties (e.g. `cus_name` → `cusName`).

**ViewModel** — `PaymentViewModel` as `ObservableObject` with `@Published` properties:
- `@Published var sessionId: String?`
- `@Published var availableMethods: [SkyPayMethod]`
- `@Published var uiState: PaymentUiState`
- `func initiatePayment(...)` — calls service with `async/await`, updates state on `@MainActor`
- `func verifyPayment(txnId: String)` — calls service, updates state

**UI** — Build the payment flow as a SwiftUI `View` observing the ViewModel with `@StateObject` or `@ObservedObject`.

---

## ⏱️ Retry Logic (All Platforms)

SkyPay's Android SMS bridge takes **5–20 seconds** to receive and relay the customer's payment SMS. If the user submits their TrxID immediately after paying, verification may return `false` because the SMS has not yet arrived.

Implement the following retry logic:
1. After a failed verify call, check whether the error message indicates "not found" or "not yet received"
2. If yes: show the user "Payment matching in progress, please wait..."
3. Show a **Retry** button that becomes active after 10 seconds
4. On retry: re-call the verify endpoint with the same `sessionId`, `method`, and `txnId`
5. Allow up to 5–6 retry attempts within a 3-minute window
6. If all retries fail: ask the user to contact support with their TrxID

Do not retry indefinitely — once the retry window expires, show a final failure message.

---

## 🔁 Idempotency & Fraud Prevention

- Store the fulfilled `sessionId` and `transactionId` in your app's backend database after successful verification
- Before calling verify, check your own backend to confirm this session has not already been completed
- SkyPay blocks TrxID reuse on its side, but your app backend must also prevent duplicate order fulfillment

---

## 🔡 Method Name Validation

Always normalize `method` to lowercase before sending it to the verify endpoint:
- Flutter/Dart: `selectedMethod.toLowerCase()`
- Kotlin: `selectedMethod.lowercase()`
- Java: `selectedMethod.toLowerCase(Locale.ROOT)`
- Swift: `selectedMethod.lowercased()`

Accepted values: `bkash`, `nagad`, `rocket`, `upay` — all lowercase, no spaces.

---

## 📊 Complete Flow Summary

```
User taps "Pay 500 BDT"
    → POST /api/v2/payment/create
    → Save sessionId, parse and filter methods

App shows wallet numbers to user
    → User opens bKash/Nagad/etc app
    → User sends money to displayed number
    → User copies TrxID from SMS notification

User enters TrxID in app and taps "Verify"
    → POST /api/v2/payment/verify { id, method, transaction_id }
    → status: true  → fulfill order inside app
    → status: false (pending)   → show retry button, wait 10s
    → status: false (permanent) → show error
```

---

## ❌ Error Reference

| HTTP Code | Message | Cause | Fix |
| :--- | :--- | :--- | :--- |
| `400` | `Invalid transaction ID or transaction already used.` | TrxID not found in incoming SMS logs, or already claimed by another order. | Ask the user to double-check the TrxID; for a fresh payment, wait 10–20s and retry. |
| `400` | `This payment session has already been completed.` | The session was already verified earlier. | Do not re-verify — check your own DB for order status. |
| `400` | `Unsupported payment method supplied.` | `method` is not `bkash`, `nagad`, `rocket`, or `upay`, or is not lowercase. | Normalize with the platform's lowercase function before sending. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | The key is wrong, or the brand is deactivated. | Verify the key on your backend / Brand Management dashboard. |
| `403` | `No active SMS sync device found for this account.` | The merchant Android phone is offline or the SkyPay APK service is stopped. | Restart the SkyPay APK and confirm the phone is online. |
| `404` | `Payment session not found or expired.` | The session `id` is invalid or expired. | Call `/create` again for a fresh session. |

---

## ✅ Integration Checklist

- [ ] `BRAND-KEY` lives only on your backend server — never bundled in the app binary
- [ ] The app talks to your backend, and your backend talks to SkyPay
- [ ] Session `id` is saved immediately after `/create` and passed through to `/verify`
- [ ] Only channels with `active_payments = true` and a non-empty number are shown
- [ ] `method` is always normalized to strict lowercase before verification
- [ ] A 10–20 second retry window is implemented for SMS sync latency
- [ ] Order fulfillment only happens after receiving `status: true` from `/verify`
- [ ] Your backend database prevents double-fulfillment on duplicate verify calls
- [ ] Native UI (not a WebView) is used for the entire payment flow

---

## 🔗 Quick Links

| Resource | URL |
| :--- | :--- |
| 🔑 Login | [skypaybd.top/sign-in](https://skypaybd.top/sign-in) |
| 📝 Register | [skypaybd.top/sign-up](https://skypaybd.top/sign-up) |
| 🏷️ Brand Management | [skypaybd.top/user/brands](https://skypaybd.top/user/brands) |
| 📱 Device Management | [skypaybd.top/user/devices](https://skypaybd.top/user/devices) |
| 📦 Subscription Plans | [skypaybd.top/user/plans](https://skypaybd.top/user/plans) |
| 📖 Full API Documentation | [skypaybd.top/docs](https://skypaybd.top/docs) |
| 🐙 Documentation Repository | [github.com/SkyPayBD/Docs](https://github.com/SkyPayBD/Docs) |
| 📄 Privacy Policy | [skypaybd.top/legal#privacy-policy](https://skypaybd.top/legal#privacy-policy) |
| 📋 Terms of Service | [skypaybd.top/legal#terms](https://skypaybd.top/legal#terms) |

---

## 📞 Contact & Support
<div align="center">

| Channel | Link |
| :--- | :--- |
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
<div align="center">
*© 2024–2026 SkyPay BD. All rights reserved.*
*SkyPay Mobile Integration Guide — Flutter · Kotlin · Java · Swift*
</div>
