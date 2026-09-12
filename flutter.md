# SkyPay Payment Gateway — Mobile App Integration Guide for AI Agents
## Covers: Flutter · Kotlin (Android) · Java (Android) · Swift (iOS)

## Overview

This document is a complete instruction set for integrating SkyPay BD into **native or cross-platform mobile applications**. Mobile apps cannot use the Hosted Gateway (v1) because it requires browser redirects that break the native app experience. Instead, all mobile app integrations must use the **Headless API v2**, which keeps the entire payment flow inside the app.

Always read the main SkyPay `README.md` API documentation first before writing any integration code.

---

## Why Headless API v2 for Mobile

The Hosted Gateway redirects the user to an external SkyPay webpage. In a mobile app, this would require a WebView, which:
- Breaks the native UX
- Makes it impossible to intercept the callback
- Requires handling deep links or custom URL schemes

The Headless API v2 eliminates all of this. Your app calls the API, gets active wallet numbers, shows them to the user natively, collects the TrxID through a native input, and verifies it — all without leaving the app.

---

## Endpoints

Both endpoints use POST with JSON body and `BRAND-KEY` header:

- **Initiate session:** `POST https://core.skypaybd.top/api/v2/payment/create`
- **Verify transaction:** `POST https://core.skypaybd.top/api/v2/payment/verify`

---

## Authentication

All requests require:
```
BRAND-KEY: <your_brand_key>
Content-Type: application/json
Accept: application/json
```

**Security Warning:** For production apps, the BRAND-KEY must NOT be shipped inside the mobile app binary. Anyone can decompile an APK or IPA and extract hardcoded strings. The recommended production architecture is:

1. Your mobile app talks to **your own backend server**
2. Your backend server holds the BRAND-KEY and calls SkyPay on behalf of the app
3. The app only receives the wallet numbers and session ID from your backend

For development or internal tools, the BRAND-KEY can temporarily be placed in a config file, but it must be removed before public release.

---

## Config File (per platform)

### Flutter: `lib/config/skypay_config.dart`

Define constants:
- `kSkyPayV2CreateUrl` → `https://core.skypaybd.top/api/v2/payment/create`
- `kSkyPayV2VerifyUrl` → `https://core.skypaybd.top/api/v2/payment/verify`
- `kSkyPayBrandKey` → your brand key (from environment or secure storage in production)

### Kotlin (Android): `config/SkyPayConfig.kt`

Create an `object SkyPayConfig`:
- `const val V2_CREATE_URL`
- `const val V2_VERIFY_URL`
- `const val BRAND_KEY`

### Java (Android): `config/SkyPayConfig.java`

Create a `final class SkyPayConfig` with `public static final String` constants for the same values.

### Swift (iOS): `Config/SkyPayConfig.swift`

Create an `enum SkyPayConfig` with `static let` constants for both URLs and the brand key.

---

## 3-Step Headless Payment Flow

### Step 1 — Initiate Payment Session

When the user triggers a payment (taps "Buy", "Deposit", "Subscribe", etc.):

Call `POST /api/v2/payment/create` with this JSON body:
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
- `id` (string): This is the SkyPay session ID. Save it immediately in your ViewModel / state — you will need it in Step 3. If you lose this ID, the session cannot be verified.
- `methods` (array): Each element represents a payment channel (bkash, nagad, rocket, upay) with:
  - `name`: channel identifier
  - `active_payments.personal`: boolean — is the personal (Send Money) number active?
  - `active_payments.agent`: boolean — is the agent (Cash In) number active?
  - `active_payments.payment`: boolean — is merchant payment active? (bKash only currently)
  - `personal`: the wallet number for Send Money
  - `agent`: the wallet number for Cash In
  - `payment`: the wallet number for Merchant Pay

**Filtering rule:** Only display a wallet option if the corresponding `active_payments` flag is `true` AND the number string is not empty. Never show the user a channel with an inactive flag or empty number.

### Step 2 — Display Payment Instructions

Build a native payment screen inside the app. This screen should show:
- The exact BDT amount the user must send
- A list of available wallet options (filtered from Step 1)
- For each wallet: the channel name, type (Personal/Agent), and the number to send money to
- Clear instruction: "Send the exact amount, then enter your TrxID"
- A text input field for the TrxID
- If multiple channels are available, a way for the user to select which one they used (e.g., radio buttons or a dropdown)
- A submit/verify button
- A cancel option

### Step 3 — Verify Transaction

When the user submits their TrxID:

Call `POST /api/v2/payment/verify` with:
```json
{
  "id": "<session_id_from_step_1>",
  "method": "bkash",
  "transaction_id": "BLA38KDK2M"
}
```

The `method` must always be **strict lowercase**: `bkash`, `nagad`, `rocket`, `upay`. Apply `.toLowerCase()` / `.lowercased()` before sending.

**Response handling:**
- `status: true` → Payment verified. Proceed with in-app fulfillment (unlock content, add balance, activate plan).
- `status: false` → Check the `message` field:
  - If message indicates TrxID not found or SMS not yet received → Show a "Retry" button. Tell the user to wait 10 seconds and try again. Allow retries for up to 2–3 minutes.
  - If message indicates already used TrxID → Show permanent error: "This transaction ID has already been used."
  - If message indicates session expired → Tell the user to start the payment process again.

---

## Architecture Per Platform

### Flutter Architecture

**Service layer:** Create `lib/services/skypay_service.dart`
- Use the `http` package (or `dio` for more features)
- `Future<Map<String, dynamic>> createSession(...)` → calls v2 create endpoint
- `Future<Map<String, dynamic>> verifySession(String sessionId, String method, String txnId)` → calls v2 verify endpoint
- Handle HTTP exceptions and return structured error maps

**State management:**
- Create `lib/providers/payment_provider.dart` (or `PaymentCubit`/`PaymentBloc` if using BLoC)
- State includes: `sessionId`, `amount`, `availableMethods`, `selectedMethod`, `status` (idle/loading/awaitingTxnId/verifying/success/error), `errorMessage`
- `initiatePayment(String cusName, double amount, Map metadata)` → calls service, updates state with session ID and methods
- `verifyPayment(String txnId)` → calls service, updates state to success or error

**UI layer:**
- `PaymentScreen` (StatelessWidget or ConsumerWidget) — observes provider state
- Shows loading indicator during API calls
- Shows wallet list when methods are available
- Shows TrxID input after user selects a channel
- Shows retry button on pending verification failure
- Shows success/error message on final outcome

**HTTP call headers for Flutter (using http package):**
```
headers: {
  'BRAND-KEY': SkyPayConfig.kSkyPayBrandKey,
  'Content-Type': 'application/json',
  'Accept': 'application/json',
}
```

### Kotlin (Android) Architecture

**Network layer:** Use Retrofit + OkHttp
- Create `SkyPayApiService` interface with two suspend functions: `createPayment(...)` and `verifyPayment(...)`
- Create a `SkyPayRetrofitClient` object that builds the Retrofit instance with:
  - `baseUrl("https://core.skypaybd.top")`
  - An OkHttp interceptor that adds `BRAND-KEY` and `Content-Type` headers to every request
  - GsonConverterFactory for JSON serialization

**Data models:** Create Kotlin data classes for:
- `SkyPayCreateRequest(cusName, amount, metaData)`
- `SkyPayCreateResponse(status, id, brand, methods)`
- `SkyPayMethod(name, activePayments, personal, agent, payment)`
- `SkyPayVerifyRequest(id, method, transactionId)`
- `SkyPayVerifyResponse(status, amount, cusName, id)`

**ViewModel:** Create `PaymentViewModel` (extends `ViewModel`) with:
- `MutableStateFlow<PaymentUiState>` or `LiveData` for UI state
- `fun initiatePayment(cusName: String, amount: Double, metaData: Map<String, Any>)` — calls repository in a coroutine (`viewModelScope.launch`)
- `fun verifyPayment(txnId: String)` — calls repository in a coroutine
- Stores `sessionId` and `selectedMethod` in the ViewModel between steps

**Repository:** `SkyPayRepository` sits between ViewModel and Retrofit, handles error wrapping and returns `Result<T>`.

**UI:** Use a single `PaymentFragment` or `PaymentActivity` with multiple view states managed by the ViewModel. Use `lifecycleScope.collect` or `observe` on ViewModel LiveData.

### Java (Android) Architecture

Same architecture as Kotlin but use:
- `Call<T>` with Retrofit callbacks instead of coroutines
- `MutableLiveData<T>` for ViewModel
- `new SkyPayRetrofitClient()` with `OkHttpClient.Builder().addInterceptor(...)`
- POJOs instead of data classes for request/response models

### Swift (iOS) Architecture

**Network layer:** Use `URLSession` or Alamofire
- Create `SkyPayService.swift` as a class or actor
- `func createPayment(...) async throws -> SkyPayCreateResponse`
- `func verifyPayment(...) async throws -> SkyPayVerifyResponse`
- Add BRAND-KEY header: `request.setValue(SkyPayConfig.brandKey, forHTTPHeaderField: "BRAND-KEY")`

**Data models:** Create `Codable` structs for all request and response types. Use `CodingKeys` to map snake_case JSON keys to camelCase Swift properties (e.g., `cus_name` → `cusName`).

**ViewModel:** Create `PaymentViewModel` as `ObservableObject` with `@Published` properties:
- `@Published var sessionId: String?`
- `@Published var availableMethods: [SkyPayMethod]`
- `@Published var uiState: PaymentUiState`
- `func initiatePayment(...)` — calls service with `async/await`, updates state on `@MainActor`
- `func verifyPayment(txnId: String)` — calls service, updates state

**UI:** Build payment flow as a SwiftUI `View` observing the ViewModel with `@StateObject` or `@ObservedObject`.

---

## Retry Logic (All Platforms)

SkyPay's Android SMS bridge takes 5–20 seconds to receive and relay the customer's payment SMS. If the user submits their TrxID immediately after paying, verification may return false because the SMS has not yet arrived.

Implement the following retry logic:
1. After a failed verify call, check if the error message indicates "not found" or "not yet received"
2. If yes: show the user a message like "Payment matching in progress, please wait..."
3. Show a "Retry" button that becomes active after 10 seconds
4. On retry: re-call the verify endpoint with the same `sessionId`, `method`, and `txnId`
5. Allow up to 5–6 retry attempts within a 3-minute window
6. If all retries fail: ask the user to contact support with their TrxID

Do not retry indefinitely. After the retry window expires, show a final failure message.

---

## Idempotency and Fraud Prevention

- Store the fulfilled `sessionId` and `transactionId` in your app's backend database after successful verification
- Before calling verify, check your own backend to confirm this session has not already been completed
- SkyPay blocks TrxID reuse on its side, but your app backend must also prevent duplicate order fulfillment

---

## Method Name Validation

Before sending the `method` field to the verify endpoint, always normalize it:
- Flutter/Dart: `selectedMethod.toLowerCase()`
- Kotlin: `selectedMethod.lowercase()`
- Java: `selectedMethod.toLowerCase(Locale.ROOT)`
- Swift: `selectedMethod.lowercased()`

Accepted values: `bkash`, `nagad`, `rocket`, `upay` — all lowercase, no spaces.

---

## Complete Flow Summary

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
    → status: true → fulfill order inside app
    → status: false (pending) → show retry button, wait 10s
    → status: false (permanent) → show error
```
