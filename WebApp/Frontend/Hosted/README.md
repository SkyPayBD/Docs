# SkyPay Hosted Checkout Gateway — HTML / CSS / Vanilla JavaScript Integration Guide

> **For AI Agents, Developers & Frontend Integrators**
> This document is the complete, step-by-step reference for integrating SkyPay's **Hosted Checkout Gateway (API v1)** into any static or dynamic website using **pure HTML, CSS, and Vanilla JavaScript** — no framework required.
> All existing logic in your HTML file must be preserved when integrating this gateway. Only add the new payment section as instructed.

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Documentation](https://img.shields.io/badge/Read%20Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Gateway](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/API%20Version-v1.0%20Hosted-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
[![Last Updated](https://img.shields.io/badge/Updated-September%202026-10b981?style=for-the-badge&logo=clock&logoColor=white)](https://skypaybd.top/docs)

---

> 📖 **Online Documentation:** https://skypaybd.top/docs
> 🌐 **Official Website:** https://skypaybd.top
> ⚡ **API Core Endpoint:** `https://core.skypaybd.top`
> 🔐 **Authentication:** Only `BRAND-KEY` header is required — no SECRET-KEY needed.
> 📂 **Also See:** [Main README](https://github.com/SkyPayBD/Docs/blob/main/README.md) · [Hosted API Reference](https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md) · [Headless API Reference](https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md)

---

## 📋 Table of Contents

- [1. Overview — What This Guide Covers](#1-overview--what-this-guide-covers)
- [2. How the Hosted Flow Works in a Browser](#2-how-the-hosted-flow-works-in-a-browser)
- [3. Architecture: Frontend vs Backend Responsibilities](#3-architecture-frontend-vs-backend-responsibilities)
- [4. Prerequisites](#4-prerequisites)
- [5. Step 1 — Collecting User Input (HTML Form)](#5-step-1--collecting-user-input-html-form)
- [6. Step 2 — Calling the Create Endpoint via JavaScript (Fetch API)](#6-step-2--calling-the-create-endpoint-via-javascript-fetch-api)
- [7. Step 3 — Redirecting the User to the SkyPay Hosted Page](#7-step-3--redirecting-the-user-to-the-skypay-hosted-page)
- [8. Step 4 — Handling the Callback on Return (Reading URL Parameters)](#8-step-4--handling-the-callback-on-return-reading-url-parameters)
- [9. Step 5 — Verifying the Payment via JavaScript](#9-step-5--verifying-the-payment-via-javascript)
- [10. Step 6 — Fulfilling the Order or Showing Result](#10-step-6--fulfilling-the-order-or-showing-result)
- [11. Full Working Example — Complete HTML Page](#11-full-working-example--complete-html-page)
- [12. Integrating Into an Existing HTML File](#12-integrating-into-an-existing-html-file)
- [13. CSS Styling Reference](#13-css-styling-reference)
- [14. API Reference Summary](#14-api-reference-summary)
- [15. Callback URL Parameters Reference](#15-callback-url-parameters-reference)
- [16. Error Handling & Edge Cases](#16-error-handling--edge-cases)
- [17. Security Rules for Frontend Integrations](#17-security-rules-for-frontend-integrations)
- [18. Integration Checklist](#18-integration-checklist)
- [19. Official Resources](#19-official-resources)

---

## 1. Overview — What This Guide Covers

This guide explains how to integrate **SkyPay Hosted Checkout (API v1)** using only:

- **HTML** — input forms and UI structure
- **CSS** — styling the payment section
- **Vanilla JavaScript** — calling the API, handling redirects, reading callback parameters, verifying payment

**What "Hosted" means:** When the user clicks "Pay Now", they are redirected to SkyPay's secure hosted checkout page at `core.skypaybd.top`. On that page, SkyPay handles the entire payment experience — wallet selection, number display, TrxID input, and SMS verification. After the payment is complete (or cancelled), SkyPay redirects the user back to your website URL with result parameters.

**You do not need to build any payment UI.** SkyPay builds it for you.

### When to Use This Guide

Use this when:
- You are building a **static website**, **landing page**, or **simple web app** in HTML/JS
- You want to accept bKash / Nagad / Rocket / Upay payments **without any PHP, Node.js, or framework backend**
- You want a fast integration — this can be done in **under 30 minutes**

> ⚠️ **Important Note on Security:** The `/api/payment/create` call sends your `BRAND-KEY`. In a pure frontend (browser) setup, this key will be visible in the browser's network tab. For production use with real money, it is strongly recommended to proxy this call through a lightweight backend (even a simple Cloudflare Worker, Vercel serverless function, or PHP endpoint) so your `BRAND-KEY` is never exposed in public JavaScript. This guide covers both the **direct frontend approach** (for demos and development) and the **backend-proxy approach** (for production).

---

## 2. How the Hosted Flow Works in a Browser

```
[User on your HTML page]
         │
         │  1. Fills in name, email, amount
         │  2. Clicks "Pay Now"
         ▼
[JavaScript calls your backend proxy OR directly calls SkyPay API]
POST https://core.skypaybd.top/api/payment/create
Headers: { BRAND-KEY, Content-Type }
Body:    { cus_name, cus_email, amount, success_url, cancel_url, metadata }
         │
         ▼
[SkyPay returns: { status: true, payment_url: "https://core.skypaybd.top/checkout/..." }]
         │
         ▼
[JavaScript does: window.location.href = payment_url]
         │
         ▼
[User is now on SkyPay's hosted payment page]
User selects bKash / Nagad / Rocket / Upay
User sends money from their MFS app
User enters their SMS Transaction ID (TrxID) on SkyPay's page
SkyPay verifies via Android SMS bridge (5–20 seconds)
         │
         ▼
[SkyPay redirects user back to your success_url with query parameters]
https://yoursite.com/payment.html?transactionId=BLA38KDK2M&paymentMethod=bkash&paymentAmount=500.00&status=completed
         │
         ▼
[Your JavaScript reads window.location.search on page load]
Detects the transactionId and status parameters
         │
         ▼
[If status === "completed" → calls /api/payment/verify to confirm]
POST https://core.skypaybd.top/api/payment/verify
Body: { transaction_id: "BLA38KDK2M" }
         │
         ▼
[API returns: { status: "COMPLETED", amount: "500.00", ... }]
         │
         ▼
[JavaScript shows success message and runs fulfillment logic]
(add balance to user account, unlock feature, etc.)
```

---

## 3. Architecture: Frontend vs Backend Responsibilities

| Responsibility | Frontend (HTML/JS) | Backend (Server/Proxy) |
|---|---|---|
| Collect user name, email, amount | ✅ Yes | — |
| Display payment button | ✅ Yes | — |
| Call `/api/payment/create` (demo) | ✅ Yes (key exposed) | — |
| Call `/api/payment/create` (production) | ❌ No | ✅ Yes (key hidden) |
| Redirect to `payment_url` | ✅ Yes | — |
| Read callback URL parameters on return | ✅ Yes | — |
| Call `/api/payment/verify` | ⚠️ OK for demo | ✅ Preferred for production |
| Add balance / fulfill order in database | ❌ Never | ✅ Always |

> **Rule:** The BRAND-KEY must never be exposed in production JavaScript. Use a backend endpoint or serverless proxy that receives the user's data from the frontend and makes the actual SkyPay API call server-side.

---

## 4. Prerequisites

### 4.1 BRAND-KEY

- Log into your **SkyPay Merchant Dashboard** at https://skypaybd.top
- Navigate to **Brands** and create a new Brand for your website
- Copy the generated `BRAND-KEY`
- For production: store this in your server environment (`.env`, Cloudflare Worker secret, etc.)
- For demo/testing: you may place it in your JavaScript temporarily, but **remove before going public**

### 4.2 Merchant Android Device

- Install **SkyPay Merchant Sync APK** (download from https://skypaybd.top/public/assets/downloads/SkyPay.apk)
- Install on a dedicated Android phone (Android 7.0+) with your merchant SIM cards inserted
- Grant **SMS Listener Permission** and **Notification Access**
- Disable **Battery Optimization** for the SkyPay app
- Enter your `BRAND-KEY` inside the app and tap **Connect Device**
- Keep the phone **powered on and online 24/7**

> Without this phone connected, all `/create` calls will return `403 Forbidden`.

### 4.3 Your Website URLs

You need two publicly accessible URLs on your website for the callback:
- `success_url` — where SkyPay sends the user after successful payment (e.g. `https://yoursite.com/payment.html`)
- `cancel_url` — where SkyPay sends the user if they cancel (e.g. `https://yoursite.com/payment.html?cancelled=true`)

Both of these can be the **same HTML page** — your JavaScript will detect the callback parameters on load and show the appropriate UI.

---

## 5. Step 1 — Collecting User Input (HTML Form)

Place this HTML where you want your payment section to appear. If the user is already logged in on your website, pre-fill the `cus_name` and `cus_email` fields from your existing user session data. If no user data is available (e.g. demo mode), show the input fields empty so the user can fill them manually.

```html
<!-- SkyPay Payment Section — paste inside your <body> where needed -->
<div id="skypay-payment-section" class="skypay-container">

  <!-- Step 1: User Details & Amount (shown by default) -->
  <div id="skypay-step-input" class="skypay-card">
    <div class="skypay-header">
      <h2>💳 Make a Payment</h2>
      <p>Powered by <a href="https://skypaybd.top" target="_blank">SkyPay BD</a></p>
    </div>

    <div class="skypay-form">

      <!-- Customer Name -->
      <!-- If user is logged in: set this value from your session -->
      <!-- If demo mode: leave empty, user fills it in -->
      <div class="skypay-field">
        <label for="skypay-name">Full Name</label>
        <input
          type="text"
          id="skypay-name"
          placeholder="Enter your full name"
          autocomplete="name"
        />
        <!-- To pre-fill from your existing user system:
             document.getElementById('skypay-name').value = currentUser.name;
             You can also hide this field if user is logged in. -->
      </div>

      <!-- Customer Email -->
      <div class="skypay-field">
        <label for="skypay-email">Email Address</label>
        <input
          type="email"
          id="skypay-email"
          placeholder="Enter your email"
          autocomplete="email"
        />
        <!-- To pre-fill: document.getElementById('skypay-email').value = currentUser.email; -->
      </div>

      <!-- Payment Amount -->
      <div class="skypay-field">
        <label for="skypay-amount">Amount (BDT)</label>
        <input
          type="number"
          id="skypay-amount"
          placeholder="e.g. 500"
          min="1"
          step="1"
        />
        <!-- To set a fixed amount (e.g. product price):
             document.getElementById('skypay-amount').value = productPrice;
             document.getElementById('skypay-amount').readOnly = true; -->
      </div>

      <!-- Error Message Display -->
      <div id="skypay-input-error" class="skypay-error" style="display:none;"></div>

      <!-- Pay Now Button -->
      <button id="skypay-pay-btn" class="skypay-btn-primary" onclick="skyPayInitiateHosted()">
        💳 Pay Now
      </button>

    </div>
  </div>

  <!-- Step 2: Loading State (shown while waiting for API response) -->
  <div id="skypay-step-loading" class="skypay-card" style="display:none;">
    <div class="skypay-loading">
      <div class="skypay-spinner"></div>
      <p>Creating your payment session...</p>
      <small>Please wait, do not close this page.</small>
    </div>
  </div>

  <!-- Step 3: Success Result (shown after user returns from payment page) -->
  <div id="skypay-step-success" class="skypay-card" style="display:none;">
    <div class="skypay-result skypay-result-success">
      <div class="skypay-result-icon">✅</div>
      <h2>Payment Verified!</h2>
      <p id="skypay-success-message">Your payment has been confirmed successfully.</p>
      <div id="skypay-success-details" class="skypay-details-box"></div>
      <button class="skypay-btn-secondary" onclick="skyPayReset()">Make Another Payment</button>
    </div>
  </div>

  <!-- Step 4: Failed/Cancelled Result -->
  <div id="skypay-step-failed" class="skypay-card" style="display:none;">
    <div class="skypay-result skypay-result-failed">
      <div class="skypay-result-icon">❌</div>
      <h2>Payment Not Confirmed</h2>
      <p id="skypay-failed-message">Your payment could not be verified at this time.</p>
      <button class="skypay-btn-primary" onclick="skyPayReset()">Try Again</button>
    </div>
  </div>

</div>
<!-- End SkyPay Payment Section -->
```

---

## 6. Step 2 — Calling the Create Endpoint via JavaScript (Fetch API)

Place this `<script>` block before your closing `</body>` tag, or in your existing JS file.

```html
<script>
// ============================================================
// SKYPAY HOSTED GATEWAY — VANILLA JAVASCRIPT INTEGRATION
// Version: 1.0 | API: v1 (Hosted)
// Docs: https://skypaybd.top/docs
// ============================================================

// ─────────────────────────────────────────────
// CONFIGURATION — Edit these values
// ─────────────────────────────────────────────

const SKYPAY_CONFIG = {

  // ⚠️ SECURITY WARNING:
  // For DEMO / LOCAL TESTING: you may put your BRAND-KEY here directly.
  // For PRODUCTION: remove the BRAND_KEY from here.
  // Instead, set PROXY_URL to your backend endpoint (PHP, Node.js, Cloudflare Worker, etc.)
  // that holds the BRAND-KEY securely and makes the API call server-side.
  BRAND_KEY: 'YOUR_BRAND_KEY_HERE',

  // Your backend proxy URL (set this for production, leave empty for demo/direct mode)
  // If set, the JS will POST user data here and your backend will call SkyPay.
  // If empty, the JS will call SkyPay directly (BRAND-KEY will be visible in network tab).
  PROXY_URL: '',

  // The URL SkyPay will redirect the user to after successful payment.
  // This must be a publicly accessible URL — localhost will NOT work in production.
  // Both success and cancel can point to the same page; JS will detect the parameters.
  SUCCESS_URL: window.location.origin + window.location.pathname,

  // The URL SkyPay will redirect the user to if they cancel payment.
  CANCEL_URL: window.location.origin + window.location.pathname + '?skypay_cancelled=true',

  // SkyPay API base URL — do not change this
  API_BASE: 'https://core.skypaybd.top',
};

// ─────────────────────────────────────────────
// OPTIONAL: Pre-fill from your user session
// ─────────────────────────────────────────────
// If your website has a logged-in user system, uncomment and adapt this block.
// Replace `window.currentUser` with however your site stores user info.
//
// window.addEventListener('DOMContentLoaded', function () {
//   const user = window.currentUser || null; // your user object
//   if (user) {
//     const nameField = document.getElementById('skypay-name');
//     const emailField = document.getElementById('skypay-email');
//     if (nameField && user.name) {
//       nameField.value = user.name;
//       nameField.readOnly = true; // optional: prevent editing
//     }
//     if (emailField && user.email) {
//       emailField.value = user.email;
//       emailField.readOnly = true;
//     }
//   }
// });

// ─────────────────────────────────────────────
// MAIN FUNCTION: Initiate Hosted Payment
// Called when user clicks "Pay Now"
// ─────────────────────────────────────────────
async function skyPayInitiateHosted() {

  // 1. Read input values
  const name   = document.getElementById('skypay-name')?.value?.trim();
  const email  = document.getElementById('skypay-email')?.value?.trim();
  const amount = document.getElementById('skypay-amount')?.value?.trim();

  // 2. Validate inputs
  const errorDiv = document.getElementById('skypay-input-error');

  if (!name) {
    skyPayShowError(errorDiv, 'Please enter your full name.');
    return;
  }
  if (!email || !isValidEmail(email)) {
    skyPayShowError(errorDiv, 'Please enter a valid email address.');
    return;
  }
  if (!amount || isNaN(amount) || parseFloat(amount) <= 0) {
    skyPayShowError(errorDiv, 'Please enter a valid payment amount (must be greater than 0).');
    return;
  }

  skyPayHideError(errorDiv);

  // 3. Switch UI to loading state
  skyPayShowStep('loading');

  // 4. Build the request payload
  const payload = {
    cus_name:    name,
    cus_email:   email,
    amount:      parseFloat(amount),
    success_url: SKYPAY_CONFIG.SUCCESS_URL,
    cancel_url:  SKYPAY_CONFIG.CANCEL_URL,
    // Metadata: attach any relevant data from your site
    // This will be returned to you in the /verify response so you can identify the order.
    metadata: {
      // Add your internal order/user reference here:
      // order_id:  'ORD-' + Date.now(),
      // user_id:   window.currentUser?.id || null,
      // plan:      'BASIC',
      initiated_at: new Date().toISOString(),
      source: 'html_integration',
    },
  };

  // 5. Call the API (direct or via proxy)
  try {
    let data;

    if (SKYPAY_CONFIG.PROXY_URL) {
      // ─── Production Mode: call your backend proxy ───
      // Your backend should accept this data, add the BRAND-KEY, and forward to SkyPay.
      const response = await fetch(SKYPAY_CONFIG.PROXY_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      });
      data = await response.json();

    } else {
      // ─── Demo / Direct Mode: call SkyPay API directly ───
      // ⚠️ BRAND-KEY is visible in network tab — only for testing!
      const response = await fetch(SKYPAY_CONFIG.API_BASE + '/api/payment/create', {
        method: 'POST',
        headers: {
          'BRAND-KEY':    SKYPAY_CONFIG.BRAND_KEY,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(payload),
      });
      data = await response.json();
    }

    // 6. Handle the API response
    if (data && data.status === true && data.payment_url) {
      // ✅ Session created — redirect user to SkyPay's hosted payment page
      window.location.href = data.payment_url;

    } else {
      // ❌ API returned an error
      skyPayShowStep('input');
      skyPayShowError(errorDiv, data?.message || 'Failed to create payment session. Please try again.');
    }

  } catch (err) {
    // Network error or unexpected failure
    skyPayShowStep('input');
    skyPayShowError(errorDiv, 'Network error. Please check your connection and try again.');
    console.error('[SkyPay] Create error:', err);
  }
}
</script>
```

---

## 7. Step 3 — Redirecting the User to the SkyPay Hosted Page

This is handled automatically by the line:

```javascript
window.location.href = data.payment_url;
```

The user's browser navigates to SkyPay's secure checkout page (e.g. `https://core.skypaybd.top/checkout/order/f7b3a9c2d1e04856`). On that page:

1. The user selects their preferred MFS channel (bKash / Nagad / Rocket / Upay)
2. SkyPay displays the active merchant wallet number
3. The user opens their MFS app, sends the exact amount
4. The user receives an SMS TrxID from their telecom
5. The user enters the TrxID on SkyPay's page and clicks "Verify"
6. SkyPay matches the TrxID against the Android SMS bridge in real-time (5–20 seconds)
7. SkyPay redirects the user back to your `success_url` with result parameters

You do not need to write any code for steps 1–7. SkyPay handles all of it.

---

## 8. Step 4 — Handling the Callback on Return (Reading URL Parameters)

When SkyPay redirects the user back to your page, it appends these parameters to your `success_url`:

```
https://yoursite.com/payment.html
  ?transactionId=BLA38KDK2M
  &paymentMethod=bkash
  &paymentAmount=500.00
  &paymentFee=0.00
  &status=completed
```

Your JavaScript must read these on page load. Add this to your existing script block:

```html
<script>
// ─────────────────────────────────────────────
// CALLBACK HANDLER: Runs on every page load
// Detects if user has returned from SkyPay payment page
// ─────────────────────────────────────────────
window.addEventListener('DOMContentLoaded', function () {

  const params = new URLSearchParams(window.location.search);

  const transactionId  = params.get('transactionId');
  const paymentMethod  = params.get('paymentMethod');
  const paymentAmount  = params.get('paymentAmount');
  const status         = params.get('status');
  const cancelled      = params.get('skypay_cancelled');

  // Case 1: User cancelled the payment
  if (cancelled === 'true') {
    skyPayShowStep('failed');
    document.getElementById('skypay-failed-message').textContent =
      'You cancelled the payment. Please try again when you are ready.';
    skyPayCleanURL();
    return;
  }

  // Case 2: User returned from a completed payment
  if (transactionId && status) {

    if (status === 'completed') {
      // ✅ Payment appears completed — MUST verify via API before doing anything
      skyPayVerifyPayment(transactionId, paymentMethod, paymentAmount);

    } else if (status === 'pending') {
      // ⏳ Payment is pending — show a waiting message
      skyPayShowStep('failed');
      document.getElementById('skypay-failed-message').textContent =
        'Your payment is pending verification. Please wait a moment and refresh the page, or contact support.';
      skyPayCleanURL();

    } else {
      // ❌ Payment failed
      skyPayShowStep('failed');
      document.getElementById('skypay-failed-message').textContent =
        'Payment verification failed. If you sent money, please contact support with your Transaction ID: ' + transactionId;
      skyPayCleanURL();
    }
  }

  // Case 3: No callback parameters — normal page load, show the input form
  // (default state, nothing to do)
});
</script>
```

---

## 9. Step 5 — Verifying the Payment via JavaScript

> ⚠️ **CRITICAL SECURITY RULE:** Never fulfill an order or add balance based only on the callback URL parameters. Always call `/api/payment/verify` to confirm the transaction is genuinely recorded in SkyPay's database.

```html
<script>
// ─────────────────────────────────────────────
// VERIFY PAYMENT: Called after reading callback params
// ─────────────────────────────────────────────
async function skyPayVerifyPayment(transactionId, paymentMethod, paymentAmount) {

  // Show loading while we verify
  skyPayShowStep('loading');
  const loadingText = document.querySelector('#skypay-step-loading p');
  if (loadingText) loadingText.textContent = 'Verifying your payment...';

  try {
    let data;

    if (SKYPAY_CONFIG.PROXY_URL) {
      // ─── Production: call your backend verify proxy ───
      // Your backend adds BRAND-KEY and calls SkyPay /verify
      const response = await fetch(SKYPAY_CONFIG.PROXY_URL + '/verify', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ transaction_id: transactionId }),
      });
      data = await response.json();

    } else {
      // ─── Demo / Direct: call SkyPay verify API directly ───
      const response = await fetch(SKYPAY_CONFIG.API_BASE + '/api/payment/verify', {
        method: 'POST',
        headers: {
          'BRAND-KEY':    SKYPAY_CONFIG.BRAND_KEY,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ transaction_id: transactionId }),
      });
      data = await response.json();
    }

    // Handle verify response
    if (data && data.status === 'COMPLETED') {
      // ✅ Payment is genuinely verified
      skyPayOnVerified(data, paymentAmount, paymentMethod);

    } else {
      // ❌ Payment not confirmed
      skyPayShowStep('failed');
      document.getElementById('skypay-failed-message').textContent =
        'Payment verification returned: ' + (data?.status || 'unknown status') +
        '. If you believe this is an error, please contact support with Transaction ID: ' + transactionId;
    }

  } catch (err) {
    skyPayShowStep('failed');
    document.getElementById('skypay-failed-message').textContent =
      'Network error during verification. Please contact support with your Transaction ID: ' + transactionId;
    console.error('[SkyPay] Verify error:', err);
  }

  skyPayCleanURL();
}

// ─────────────────────────────────────────────
// PAYMENT VERIFIED: Called on successful verification
// ─────────────────────────────────────────────
function skyPayOnVerified(verifyData, displayAmount, displayMethod) {

  skyPayShowStep('success');

  // Update success message
  document.getElementById('skypay-success-message').textContent =
    'Your payment of BDT ' + (verifyData.amount || displayAmount) + ' via ' +
    (verifyData.payment_method || displayMethod || 'MFS') + ' has been confirmed.';

  // Show payment details
  const details = document.getElementById('skypay-success-details');
  if (details) {
    details.innerHTML = `
      <table class="skypay-details-table">
        <tr><td>Customer</td><td><strong>${verifyData.cus_name || '—'}</strong></td></tr>
        <tr><td>Amount</td><td><strong>BDT ${verifyData.amount || displayAmount || '—'}</strong></td></tr>
        <tr><td>Method</td><td><strong>${capitalizeFirst(verifyData.payment_method || displayMethod || '—')}</strong></td></tr>
        <tr><td>Transaction ID</td><td><strong>${verifyData.transaction_id || '—'}</strong></td></tr>
        <tr><td>Status</td><td><strong class="skypay-status-ok">✅ COMPLETED</strong></td></tr>
      </table>
    `;
  }

  // ─────────────────────────────────────────────
  // YOUR FULFILLMENT LOGIC GOES HERE
  // ─────────────────────────────────────────────
  // This is where you integrate with your website's own systems.
  // Examples:
  //
  // (A) If this is a DEMO page — nothing to do, just show success.
  //
  // (B) If you have a balance system:
  //     Call your backend API to add the verified amount to the user's account.
  //     Example:
  //     fetch('/api/user/add-balance', {
  //       method: 'POST',
  //       headers: { 'Content-Type': 'application/json' },
  //       body: JSON.stringify({
  //         user_id: window.currentUser?.id,
  //         amount: verifyData.amount,
  //         transaction_id: verifyData.transaction_id,
  //         payment_method: verifyData.payment_method,
  //         metadata: verifyData.metadata,
  //       }),
  //     });
  //
  // (C) If this is an order:
  //     Mark the order as paid using your existing order system:
  //     const orderId = verifyData.metadata?.order_id;
  //     fetch('/api/orders/' + orderId + '/mark-paid', { method: 'POST', ... });
  //
  // (D) If this is a subscription:
  //     Activate or extend the user's plan:
  //     fetch('/api/subscriptions/activate', { method: 'POST', ... });
  //
  // NOTE: Any action that involves your database MUST go through your backend server.
  // Never update a database directly from client-side JavaScript.

  console.log('[SkyPay] Payment verified. Full response:', verifyData);
}
</script>
```

---

## 10. Step 6 — Fulfilling the Order or Showing Result

After `skyPayOnVerified()` runs:

- The success card is shown with the payment details table
- Your custom fulfillment logic (inside the comments in `skyPayOnVerified`) is executed
- For a **demo page**: just the success UI is enough — no backend call needed
- For a **real website**: call your own backend API (e.g. `/api/add-balance`) with the verified payment data to update your database

---

## 11. Full Working Example — Complete HTML Page

This is a standalone, copy-pasteable HTML file that demonstrates the complete hosted payment flow. Save it as `payment.html` on your web server (must be publicly accessible — localhost will not work with SkyPay redirects in production).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>SkyPay Hosted Payment Demo</title>

  <style>
    /* ── SkyPay Base Styles ── */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
      background: #f0f4f8;
      color: #1e293b;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .skypay-container {
      width: 100%;
      max-width: 480px;
    }

    .skypay-card {
      background: #ffffff;
      border-radius: 16px;
      box-shadow: 0 4px 24px rgba(0,0,0,0.10);
      padding: 32px 28px;
    }

    .skypay-header {
      text-align: center;
      margin-bottom: 28px;
    }

    .skypay-header h2 {
      font-size: 22px;
      font-weight: 700;
      color: #1e293b;
      margin-bottom: 6px;
    }

    .skypay-header p {
      font-size: 13px;
      color: #64748b;
    }

    .skypay-header a {
      color: #2563eb;
      text-decoration: none;
    }

    .skypay-form { display: flex; flex-direction: column; gap: 18px; }

    .skypay-field { display: flex; flex-direction: column; gap: 6px; }

    .skypay-field label {
      font-size: 14px;
      font-weight: 600;
      color: #374151;
    }

    .skypay-field input {
      width: 100%;
      padding: 12px 14px;
      border: 1.5px solid #d1d5db;
      border-radius: 8px;
      font-size: 15px;
      color: #1e293b;
      background: #f8fafc;
      transition: border-color 0.2s;
      outline: none;
    }

    .skypay-field input:focus { border-color: #2563eb; background: #fff; }

    .skypay-field input[readonly] { color: #6b7280; cursor: default; }

    .skypay-btn-primary {
      width: 100%;
      padding: 14px;
      background: linear-gradient(135deg, #2563eb, #1d4ed8);
      color: #fff;
      border: none;
      border-radius: 10px;
      font-size: 16px;
      font-weight: 700;
      cursor: pointer;
      transition: opacity 0.2s, transform 0.1s;
      margin-top: 4px;
    }

    .skypay-btn-primary:hover { opacity: 0.92; transform: translateY(-1px); }
    .skypay-btn-primary:active { transform: translateY(0); }

    .skypay-btn-secondary {
      width: 100%;
      padding: 12px;
      background: #f1f5f9;
      color: #374151;
      border: 1.5px solid #d1d5db;
      border-radius: 10px;
      font-size: 15px;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.2s;
      margin-top: 12px;
    }

    .skypay-btn-secondary:hover { background: #e2e8f0; }

    .skypay-error {
      background: #fef2f2;
      border: 1px solid #fecaca;
      color: #dc2626;
      padding: 10px 14px;
      border-radius: 8px;
      font-size: 14px;
    }

    .skypay-loading {
      text-align: center;
      padding: 20px 0;
    }

    .skypay-spinner {
      width: 44px;
      height: 44px;
      border: 4px solid #e2e8f0;
      border-top-color: #2563eb;
      border-radius: 50%;
      animation: skyPaySpin 0.8s linear infinite;
      margin: 0 auto 16px;
    }

    @keyframes skyPaySpin { to { transform: rotate(360deg); } }

    .skypay-loading p { font-size: 16px; font-weight: 600; color: #1e293b; margin-bottom: 6px; }
    .skypay-loading small { font-size: 13px; color: #64748b; }

    .skypay-result { text-align: center; padding: 8px 0; }
    .skypay-result-icon { font-size: 52px; margin-bottom: 12px; }
    .skypay-result h2 { font-size: 22px; font-weight: 700; margin-bottom: 10px; }
    .skypay-result p { font-size: 15px; color: #475569; margin-bottom: 20px; line-height: 1.6; }

    .skypay-result-success h2 { color: #16a34a; }
    .skypay-result-failed h2 { color: #dc2626; }

    .skypay-details-box {
      background: #f8fafc;
      border: 1px solid #e2e8f0;
      border-radius: 10px;
      padding: 16px;
      margin: 0 0 20px;
      text-align: left;
    }

    .skypay-details-table { width: 100%; border-collapse: collapse; font-size: 14px; }
    .skypay-details-table td { padding: 7px 4px; color: #475569; }
    .skypay-details-table td:first-child { color: #64748b; width: 45%; }
    .skypay-details-table td strong { color: #1e293b; }
    .skypay-status-ok { color: #16a34a !important; }

    .skypay-powered {
      text-align: center;
      margin-top: 18px;
      font-size: 12px;
      color: #94a3b8;
    }

    .skypay-powered a { color: #2563eb; text-decoration: none; }
  </style>
</head>
<body>

<!-- ══════════════════════════════════════════ -->
<!-- SkyPay Hosted Payment Integration Section  -->
<!-- ══════════════════════════════════════════ -->
<div id="skypay-payment-section" class="skypay-container">

  <!-- Input Form -->
  <div id="skypay-step-input" class="skypay-card">
    <div class="skypay-header">
      <h2>💳 Make a Payment</h2>
      <p>Powered by <a href="https://skypaybd.top" target="_blank">SkyPay BD</a></p>
    </div>
    <div class="skypay-form">
      <div class="skypay-field">
        <label for="skypay-name">Full Name</label>
        <input type="text" id="skypay-name" placeholder="Enter your full name" autocomplete="name" />
      </div>
      <div class="skypay-field">
        <label for="skypay-email">Email Address</label>
        <input type="email" id="skypay-email" placeholder="Enter your email" autocomplete="email" />
      </div>
      <div class="skypay-field">
        <label for="skypay-amount">Amount (BDT)</label>
        <input type="number" id="skypay-amount" placeholder="e.g. 500" min="1" step="1" />
      </div>
      <div id="skypay-input-error" class="skypay-error" style="display:none;"></div>
      <button id="skypay-pay-btn" class="skypay-btn-primary" onclick="skyPayInitiateHosted()">
        💳 Pay Now
      </button>
    </div>
  </div>

  <!-- Loading State -->
  <div id="skypay-step-loading" class="skypay-card" style="display:none;">
    <div class="skypay-loading">
      <div class="skypay-spinner"></div>
      <p>Creating your payment session...</p>
      <small>Please wait, do not close this page.</small>
    </div>
  </div>

  <!-- Success State -->
  <div id="skypay-step-success" class="skypay-card" style="display:none;">
    <div class="skypay-result skypay-result-success">
      <div class="skypay-result-icon">✅</div>
      <h2>Payment Verified!</h2>
      <p id="skypay-success-message">Your payment has been confirmed successfully.</p>
      <div id="skypay-success-details" class="skypay-details-box"></div>
      <button class="skypay-btn-secondary" onclick="skyPayReset()">Make Another Payment</button>
    </div>
  </div>

  <!-- Failed State -->
  <div id="skypay-step-failed" class="skypay-card" style="display:none;">
    <div class="skypay-result skypay-result-failed">
      <div class="skypay-result-icon">❌</div>
      <h2>Payment Not Confirmed</h2>
      <p id="skypay-failed-message">Your payment could not be verified at this time.</p>
      <button class="skypay-btn-primary" onclick="skyPayReset()">Try Again</button>
    </div>
  </div>

  <div class="skypay-powered">
    Secured by <a href="https://skypaybd.top" target="_blank">SkyPay BD</a> &nbsp;·&nbsp;
    <a href="https://skypaybd.top/docs" target="_blank">Documentation</a>
  </div>

</div>
<!-- ══════════════════════════════════════════ -->

<script>
// ============================================================
// SKYPAY HOSTED GATEWAY — COMPLETE VANILLA JS IMPLEMENTATION
// ============================================================

const SKYPAY_CONFIG = {
  BRAND_KEY:   'YOUR_BRAND_KEY_HERE', // ⚠️ Use proxy in production
  PROXY_URL:   '',                    // Set to your backend endpoint for production
  SUCCESS_URL: window.location.origin + window.location.pathname,
  CANCEL_URL:  window.location.origin + window.location.pathname + '?skypay_cancelled=true',
  API_BASE:    'https://core.skypaybd.top',
};

// ── On page load: check if returning from payment ──
window.addEventListener('DOMContentLoaded', function () {

  // Optional: Pre-fill user data from your session
  // if (window.currentUser) {
  //   document.getElementById('skypay-name').value  = window.currentUser.name  || '';
  //   document.getElementById('skypay-email').value = window.currentUser.email || '';
  // }

  const params       = new URLSearchParams(window.location.search);
  const transactionId = params.get('transactionId');
  const status        = params.get('status');
  const cancelled     = params.get('skypay_cancelled');
  const paymentMethod = params.get('paymentMethod');
  const paymentAmount = params.get('paymentAmount');

  if (cancelled === 'true') {
    skyPayShowStep('failed');
    document.getElementById('skypay-failed-message').textContent =
      'You cancelled the payment. You can try again anytime.';
    skyPayCleanURL();
    return;
  }

  if (transactionId && status === 'completed') {
    skyPayVerifyPayment(transactionId, paymentMethod, paymentAmount);
  } else if (transactionId && status) {
    skyPayShowStep('failed');
    document.getElementById('skypay-failed-message').textContent =
      'Payment status: ' + status + '. If you sent money, contact support with TrxID: ' + transactionId;
    skyPayCleanURL();
  }
});

// ── Initiate: called when user clicks Pay Now ──
async function skyPayInitiateHosted() {
  const name    = document.getElementById('skypay-name')?.value?.trim();
  const email   = document.getElementById('skypay-email')?.value?.trim();
  const amount  = document.getElementById('skypay-amount')?.value?.trim();
  const errDiv  = document.getElementById('skypay-input-error');

  if (!name)  { skyPayShowError(errDiv, 'Please enter your full name.'); return; }
  if (!email || !isValidEmail(email)) { skyPayShowError(errDiv, 'Please enter a valid email address.'); return; }
  if (!amount || isNaN(amount) || parseFloat(amount) <= 0) { skyPayShowError(errDiv, 'Please enter a valid amount greater than 0.'); return; }
  skyPayHideError(errDiv);

  skyPayShowStep('loading');

  const payload = {
    cus_name:    name,
    cus_email:   email,
    amount:      parseFloat(amount),
    success_url: SKYPAY_CONFIG.SUCCESS_URL,
    cancel_url:  SKYPAY_CONFIG.CANCEL_URL,
    metadata:    { initiated_at: new Date().toISOString(), source: 'html_integration' },
  };

  try {
    let data;
    const endpoint = SKYPAY_CONFIG.PROXY_URL || (SKYPAY_CONFIG.API_BASE + '/api/payment/create');
    const headers  = SKYPAY_CONFIG.PROXY_URL
      ? { 'Content-Type': 'application/json' }
      : { 'Content-Type': 'application/json', 'BRAND-KEY': SKYPAY_CONFIG.BRAND_KEY };

    const res = await fetch(endpoint, { method: 'POST', headers, body: JSON.stringify(payload) });
    data = await res.json();

    if (data?.status === true && data?.payment_url) {
      window.location.href = data.payment_url;
    } else {
      skyPayShowStep('input');
      skyPayShowError(errDiv, data?.message || 'Failed to create payment session. Please try again.');
    }
  } catch (err) {
    skyPayShowStep('input');
    skyPayShowError(errDiv, 'Network error. Please check your connection and try again.');
    console.error('[SkyPay] Create error:', err);
  }
}

// ── Verify: called after returning from hosted page ──
async function skyPayVerifyPayment(transactionId, paymentMethod, paymentAmount) {
  skyPayShowStep('loading');
  const loadingText = document.querySelector('#skypay-step-loading p');
  if (loadingText) loadingText.textContent = 'Verifying your payment...';

  try {
    const endpoint = SKYPAY_CONFIG.PROXY_URL
      ? SKYPAY_CONFIG.PROXY_URL + '/verify'
      : SKYPAY_CONFIG.API_BASE + '/api/payment/verify';
    const headers  = SKYPAY_CONFIG.PROXY_URL
      ? { 'Content-Type': 'application/json' }
      : { 'Content-Type': 'application/json', 'BRAND-KEY': SKYPAY_CONFIG.BRAND_KEY };

    const res  = await fetch(endpoint, { method: 'POST', headers, body: JSON.stringify({ transaction_id: transactionId }) });
    const data = await res.json();

    if (data?.status === 'COMPLETED') {
      skyPayShowStep('success');
      document.getElementById('skypay-success-message').textContent =
        'BDT ' + (data.amount || paymentAmount) + ' via ' + capitalizeFirst(data.payment_method || paymentMethod || 'MFS') + ' — confirmed.';

      const details = document.getElementById('skypay-success-details');
      if (details) {
        details.innerHTML = `
          <table class="skypay-details-table">
            <tr><td>Customer</td><td><strong>${data.cus_name || '—'}</strong></td></tr>
            <tr><td>Amount</td><td><strong>BDT ${data.amount || paymentAmount || '—'}</strong></td></tr>
            <tr><td>Method</td><td><strong>${capitalizeFirst(data.payment_method || paymentMethod || '—')}</strong></td></tr>
            <tr><td>Transaction ID</td><td><strong>${data.transaction_id || transactionId || '—'}</strong></td></tr>
            <tr><td>Status</td><td><strong class="skypay-status-ok">✅ COMPLETED</strong></td></tr>
          </table>
        `;
      }

      // ── YOUR FULFILLMENT LOGIC HERE ──
      // Example: add balance to user account via your backend
      // fetch('/api/user/add-balance', {
      //   method: 'POST',
      //   headers: { 'Content-Type': 'application/json' },
      //   body: JSON.stringify({ amount: data.amount, txn_id: data.transaction_id }),
      // });

      console.log('[SkyPay] Verified OK:', data);

    } else {
      skyPayShowStep('failed');
      document.getElementById('skypay-failed-message').textContent =
        'Verification status: ' + (data?.status || 'unknown') +
        '. Contact support with TrxID: ' + transactionId;
    }
  } catch (err) {
    skyPayShowStep('failed');
    document.getElementById('skypay-failed-message').textContent =
      'Network error during verification. Contact support with TrxID: ' + transactionId;
    console.error('[SkyPay] Verify error:', err);
  }

  skyPayCleanURL();
}

// ── Utility Functions ──
function skyPayShowStep(step) {
  ['input', 'loading', 'success', 'failed'].forEach(s => {
    const el = document.getElementById('skypay-step-' + s);
    if (el) el.style.display = (s === step) ? 'block' : 'none';
  });
}

function skyPayShowError(el, msg) {
  if (!el) return;
  el.textContent = msg;
  el.style.display = 'block';
}

function skyPayHideError(el) {
  if (!el) return;
  el.textContent = '';
  el.style.display = 'none';
}

function skyPayReset() {
  skyPayShowStep('input');
  skyPayCleanURL();
  const loadingText = document.querySelector('#skypay-step-loading p');
  if (loadingText) loadingText.textContent = 'Creating your payment session...';
}

function skyPayCleanURL() {
  if (window.history && window.history.replaceState) {
    window.history.replaceState({}, document.title, window.location.pathname);
  }
}

function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function capitalizeFirst(str) {
  if (!str) return str;
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}
</script>

</body>
</html>
```

---

## 12. Integrating Into an Existing HTML File

If you already have an HTML file with existing code (navigation, user login, product listings, etc.), follow these steps to add SkyPay without breaking anything.

### Step A — Add CSS

Copy all styles inside the `<style>` block from Section 11 and paste them into your existing `<style>` tag or your CSS file. All class names are prefixed with `skypay-` to avoid conflicts with your existing styles.

### Step B — Add HTML Structure

Copy only the `<div id="skypay-payment-section">` block from Section 11 and paste it wherever you want the payment form to appear on your page (e.g. inside a modal, a sidebar, or a dedicated section).

### Step C — Add JavaScript

Copy the entire `<script>` block from Section 11 and paste it before your closing `</body>` tag, after your existing scripts. The `SKYPAY_CONFIG` object at the top of the script is the only thing you need to configure.

### Step D — Pre-fill User Data (If Applicable)

If your site has a login system, add this after the user loads:

```javascript
// Run after your user auth logic completes
function onUserLoggedIn(user) {
  // Pre-fill SkyPay fields
  const nameField  = document.getElementById('skypay-name');
  const emailField = document.getElementById('skypay-email');
  if (nameField  && user.name)  { nameField.value  = user.name;  nameField.readOnly  = true; }
  if (emailField && user.email) { emailField.value = user.email; emailField.readOnly = true; }
  // Optionally hide the fields entirely and pass values directly to payload
}
```

### Step E — Set a Fixed Amount (If Applicable)

If the payment is for a specific product with a fixed price:

```javascript
// After page loads, set the amount and lock it
const amountField = document.getElementById('skypay-amount');
if (amountField) {
  amountField.value    = 299; // your product price
  amountField.readOnly = true;
}
```

---

## 13. CSS Styling Reference

All SkyPay classes are prefixed with `skypay-` to avoid conflicts. You can override any of them in your own CSS file. Key classes:

| Class | Purpose |
|---|---|
| `.skypay-container` | Outer wrapper, max-width 480px |
| `.skypay-card` | White card panel with shadow |
| `.skypay-header` | Title and subtitle area |
| `.skypay-form` | Flex column form container |
| `.skypay-field` | Label + input wrapper |
| `.skypay-btn-primary` | Blue "Pay Now" button |
| `.skypay-btn-secondary` | Grey secondary button |
| `.skypay-error` | Red error message box |
| `.skypay-loading` | Loading spinner container |
| `.skypay-spinner` | CSS animated spinner |
| `.skypay-result` | Success/failed result wrapper |
| `.skypay-details-box` | Payment details table container |
| `.skypay-details-table` | Details key-value table |

---

## 14. API Reference Summary

### Create Hosted Payment URL

```
POST https://core.skypaybd.top/api/payment/create

Headers:
  BRAND-KEY:    <your_brand_key>
  Content-Type: application/json

Body:
{
  "cus_name":    "Full Name",
  "cus_email":   "email@example.com",
  "amount":      500,
  "success_url": "https://yoursite.com/payment.html",
  "cancel_url":  "https://yoursite.com/payment.html?skypay_cancelled=true",
  "metadata":    { "order_id": "ORD-001" }     ← optional, returned in verify
}

Success Response:
{
  "status":      true,
  "message":     "Payment URL generated successfully.",
  "payment_url": "https://core.skypaybd.top/checkout/order/abc123"
}
→ Redirect user to payment_url immediately.
```

### Verify Payment

```
POST https://core.skypaybd.top/api/payment/verify

Headers:
  BRAND-KEY:    <your_brand_key>
  Content-Type: application/json

Body:
{
  "transaction_id": "BLA38KDK2M"
}

Success Response:
{
  "status":         "COMPLETED",
  "cus_name":       "Full Name",
  "cus_email":      "email@example.com",
  "amount":         "500.00",
  "transaction_id": "BLA38KDK2M",
  "payment_method": "bkash",
  "metadata":       { "order_id": "ORD-001" }
}
→ Only fulfill order when status === "COMPLETED"
```

---

## 15. Callback URL Parameters Reference

When SkyPay redirects the user to your `success_url` after payment, these parameters are appended:

| Parameter | Type | Example | Description |
|---|---|---|---|
| `transactionId` | String | `BLA38KDK2M` | The verified SMS Transaction ID |
| `paymentMethod` | String | `bkash` | Channel used (`bkash`, `nagad`, `rocket`, `upay`) |
| `paymentAmount` | Numeric | `500.00` | Net amount paid in BDT |
| `paymentFee` | Numeric | `0.00` | Gateway fee (if any) |
| `status` | String | `completed` | Outcome: `completed`, `pending`, or `failed` |

> ⚠️ These parameters are for display and routing only. Always call `/api/payment/verify` before any fulfillment action.

---

## 16. Error Handling & Edge Cases

| Scenario | What Happens | How JS Handles It |
|---|---|---|
| User fills invalid email | `/create` not called | Input validation shows error |
| User fills amount = 0 | `/create` not called | Input validation shows error |
| API returns `status: false` | No redirect | Error shown in form |
| Android phone offline | API returns `403` | `fetch` rejects, network error shown |
| User cancels on SkyPay page | Redirected to `cancel_url` | JS detects `skypay_cancelled=true` |
| Callback status is `pending` | Verify not called | Info message shown |
| Verify returns non-COMPLETED | No fulfillment | Failed state shown |
| Network error during verify | No fulfillment | Error shown with TrxID for support |
| User manually types fake params | Verify will return false | No fulfillment happens |

---

## 17. Security Rules for Frontend Integrations

| Rule | Details |
|---|---|
| **Never expose BRAND-KEY in production** | Use a backend proxy or serverless function (Cloudflare Worker, Vercel Function, AWS Lambda) to hold and use the BRAND-KEY server-side |
| **Never fulfill on callback params alone** | Always call `/api/payment/verify` and check `status === "COMPLETED"` |
| **Never modify database from JS** | Database updates (adding balance, activating subscription) must go through your backend API |
| **Always clean the URL after reading params** | Use `history.replaceState()` to remove payment params from the address bar |
| **Idempotency** | If the user refreshes the success page, the URL params may still be there — your backend must check that the transaction ID hasn't already been fulfilled |

---

## 18. Integration Checklist

Before going live:

- [ ] `BRAND_KEY` is stored in a backend `.env` or serverless secret — not in public JS
- [ ] A backend proxy is set up for `/create` and `/verify` calls (production)
- [ ] `SUCCESS_URL` and `CANCEL_URL` are publicly accessible HTTPS URLs
- [ ] The page correctly detects callback parameters on load via `URLSearchParams`
- [ ] `status === "completed"` triggers a `/verify` API call before any fulfillment
- [ ] Fulfillment logic is only executed after `verify` returns `"status": "COMPLETED"`
- [ ] URL is cleaned after reading callback parameters (`history.replaceState`)
- [ ] Input validation prevents empty or invalid name/email/amount
- [ ] Merchant Android phone is on, connected, and SkyPay APK is running
- [ ] Your backend prevents double-fulfillment if the same `transactionId` is submitted twice

---

## 19. Official Resources

| Resource | Link |
|---|---|
| Official Website | https://skypaybd.top |
| Interactive Documentation | https://skypaybd.top/docs |
| API Core Domain | https://core.skypaybd.top |
| GitHub Documentation Repo | https://github.com/SkyPayBD/Docs |
| Main README | https://github.com/SkyPayBD/Docs/blob/main/README.md |
| Hosted API Reference | https://github.com/SkyPayBD/Docs/blob/main/Version/Hosted/README.md |
| Headless API Reference | https://github.com/SkyPayBD/Docs/blob/main/Version/Headless/README.md |
| Merchant Sync Android APK | https://skypaybd.top/public/assets/downloads/SkyPay.apk |
| WordPress WooCommerce Plugin | https://skypaybd.top/public/assets/downloads/WP.zip |
| WHMCS Gateway Module | https://skypaybd.top/public/assets/downloads/WHMCS.zip |
| WhatsApp Support | https://wa.me/+8801761844968 |
| Telegram Support | https://t.me/BD_Prime_Minister |

---

> **Looking for a zero-redirect, fully in-page payment experience?**
> If you want the user to select payment method, enter TrxID, and verify — all without ever leaving your page — see the [SkyPay Headless API v2 HTML Integration Guide](../Headless/README.md).

---

*© SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
*Website: https://skypaybd.top | Docs: https://skypaybd.top/docs | Support: https://wa.me/+8801761844968*

