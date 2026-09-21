# SkyPay Headless Payment API v2 — HTML / CSS / Vanilla JavaScript Integration Guide

> **For AI Agents, Developers & Frontend Integrators**
> This document is the complete, step-by-step reference for integrating SkyPay's **Headless Payment API (v2)** into any website using **pure HTML, CSS, and Vanilla JavaScript** — no framework required.
> The user never leaves your page. No external redirect happens. Everything — wallet selection, TrxID entry, and verification — happens directly inside your HTML page.
> All existing logic in your HTML file must be preserved when integrating this gateway. Only add the new payment section as instructed.

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Documentation](https://img.shields.io/badge/Read%20Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Gateway](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/API%20Version-v2.0%20Headless-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
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
- [2. How the Headless Flow Works in a Browser](#2-how-the-headless-flow-works-in-a-browser)
- [3. Architecture: Frontend vs Backend Responsibilities](#3-architecture-frontend-vs-backend-responsibilities)
- [4. Prerequisites](#4-prerequisites)
- [5. The 3-Step Payment Flow](#5-the-3-step-payment-flow)
- [6. Step 1 — Collecting User Input & Creating a Session](#6-step-1--collecting-user-input--creating-a-session)
- [7. Step 2 — Displaying Active Payment Methods](#7-step-2--displaying-active-payment-methods)
- [8. Step 3 — TrxID Input & Verification](#8-step-3--trxid-input--verification)
- [9. Step 4 — Fulfilling the Order or Showing Result](#9-step-4--fulfilling-the-order-or-showing-result)
- [10. Retry Logic for SMS Sync Latency](#10-retry-logic-for-sms-sync-latency)
- [11. Full Working Example — Complete HTML Page](#11-full-working-example--complete-html-page)
- [12. Integrating Into an Existing HTML File](#12-integrating-into-an-existing-html-file)
- [13. CSS Styling Reference](#13-css-styling-reference)
- [14. API Reference Summary](#14-api-reference-summary)
- [15. API Response Fields — methods[] Array](#15-api-response-fields--methods-array)
- [16. Error Handling & Edge Cases](#16-error-handling--edge-cases)
- [17. Security Rules for Frontend Integrations](#17-security-rules-for-frontend-integrations)
- [18. Integration Checklist](#18-integration-checklist)
- [19. Official Resources](#19-official-resources)

---

## 1. Overview — What This Guide Covers

This guide explains how to integrate **SkyPay Headless API v2** using only:

- **HTML** — multi-step UI structure
- **CSS** — styling the payment section
- **Vanilla JavaScript** — calling the API, rendering wallet options, handling TrxID input, verifying payment, and retry logic

**What "Headless" means:** The user never leaves your page. After you call the `/create` endpoint, your JavaScript receives the live merchant wallet numbers directly from the API. You render these wallet numbers inside your own page UI. The user selects a payment method, sends money from their MFS app, then enters their TrxID back into your page. Your JavaScript then calls `/verify` to confirm the payment in real time. Everything happens on your page.

**You build the UI. SkyPay handles the payment matching.**

### When to Use This Guide

Use this when:
- You want a **seamless, no-redirect payment experience** on your website
- You are building a **single-page web app**, **dashboard**, or **web-based panel**
- You want **full control over the payment UI** (design, colors, layout)
- You want the payment flow to feel native to your website, not outsourced to another domain

> ⚠️ **Important Note on Security:** In a pure frontend (browser) JavaScript setup, your `BRAND-KEY` will be visible in the browser's network inspector. For production use, route the `/create` and `/verify` calls through a lightweight backend (PHP endpoint, Cloudflare Worker, Vercel serverless function, etc.) that holds the `BRAND-KEY` server-side. This guide covers both the **direct frontend approach** (for demos and development) and the **backend-proxy approach** (for production).

---

## 2. How the Headless Flow Works in a Browser

```
[User on your HTML page]
         │
         │  1. User fills in name (if not already known) and amount
         │  2. User clicks "Continue to Payment"
         ▼
[JavaScript calls SkyPay API — Step 1: Create Session]
POST https://core.skypaybd.top/api/v2/payment/create
Headers: { BRAND-KEY, Content-Type }
Body:    { cus_name, amount, meta_data }
         │
         ▼
[SkyPay returns: session id + active merchant wallet numbers for each MFS channel]
{
  "status": true,
  "id": "a1b2c3d4e5f6g7h8",        ← SAVE THIS — required for verify
  "methods": [
    { "name": "bkash",  "active_payments": { "personal": true, ... }, "personal": "01XXXXXXXX" },
    { "name": "nagad",  "active_payments": { "personal": true, ... }, "personal": "01XXXXXXXX" },
    { "name": "rocket", "active_payments": { "personal": true, ... }, "personal": "01XXXXXXXX" },
    { "name": "upay",   "active_payments": { "personal": true, ... }, "personal": "01XXXXXXXXX" }
  ]
}
         │
         ▼
[JavaScript renders the wallet number cards — Step 2: Show Payment Options]
Only channels where active_payments.personal (or .agent or .payment) is true are shown.
Each card displays:
 - MFS logo/name
 - Wallet number (with copy button)
 - "Send Money" type label
User clicks on the channel they want to use (bKash / Nagad / Rocket / Upay)
         │
         ▼
[User opens their MFS app, sends exact amount to the displayed number]
[User receives an SMS TrxID from the telecom network]
         │
         ▼
[Step 3: User enters TrxID into your page's input field]
[JavaScript calls SkyPay verify endpoint]
POST https://core.skypaybd.top/api/v2/payment/verify
Headers: { BRAND-KEY, Content-Type }
Body:    { id: "<session_id>", method: "bkash", transaction_id: "BLA38KDK2M" }
         │
         ▼
[SkyPay matches TrxID against incoming SMS on merchant Android phone — 5 to 20 seconds]
         │
     ┌───┴────────────┐
  SUCCESS           NOT YET (SMS still in transit)
     │                 │
     ▼                 ▼
  { status: true }   { status: false }
  Amount confirmed   Show retry button
  Fulfill order      Allow up to 3 retries over 2–3 min
```

---

## 3. Architecture: Frontend vs Backend Responsibilities

| Responsibility | Frontend (HTML/JS) | Backend (Server/Proxy) |
|---|---|---|
| Collect user name and amount | ✅ Yes | — |
| Display payment method cards | ✅ Yes | — |
| Copy wallet number to clipboard | ✅ Yes | — |
| Call `/api/v2/payment/create` (demo) | ✅ Yes (key exposed) | — |
| Call `/api/v2/payment/create` (production) | ❌ No | ✅ Yes (key hidden) |
| Save session `id` | ✅ Yes (in JS variable) | ✅ Better in session/DB |
| Call `/api/v2/payment/verify` (demo) | ✅ Yes (key exposed) | — |
| Call `/api/v2/payment/verify` (production) | ❌ No | ✅ Yes (key hidden) |
| Add balance / fulfill order in database | ❌ Never | ✅ Always |

---

## 4. Prerequisites

### 4.1 BRAND-KEY

- Log into your **SkyPay Merchant Dashboard** at https://skypaybd.top
- Create a **Brand** under your merchant account
- Copy the generated `BRAND-KEY`
- For production: store in server `.env`, Cloudflare Worker secrets, or similar
- For demo/testing: you may put it in JS temporarily — remove before going public

### 4.2 Merchant Android Device

- Install **SkyPay Merchant Sync APK** from https://skypaybd.top/public/assets/downloads/SkyPay.apk
- Install on an Android phone (7.0+) containing your merchant SIM cards (bKash, Nagad, Rocket, Upay)
- Grant **SMS Listener Permission** and **Notification Access**
- Disable **Battery Optimization** for the SkyPay app
- Enter your `BRAND-KEY` inside the app and tap **Connect Device**
- Keep the phone **powered on and connected to the internet 24/7**

> Without this phone, all `/create` calls will return `403 Forbidden`. The Android phone is the real-time SMS bridge — it reads incoming MFS payment SMS and forwards them to SkyPay's cloud for matching.

### 4.3 Wallet Numbers Shown = What You Have Configured

The `methods[]` array returned by `/create` only includes the MFS channels you have set up in your SkyPay Brand dashboard. If bKash is not in the response, it means it is not configured. **Always show only what the API returns.**

---

## 5. The 3-Step Payment Flow

The headless integration follows exactly 3 UI steps, rendered on the same page:

```
STEP 1 — INPUT
  └─ User provides: name (if not pre-filled), amount
  └─ Click "Continue" → JS calls /api/v2/payment/create
  └─ Receives: session id + active wallet numbers

STEP 2 — SELECT METHOD & PAY
  └─ JS renders cards for each active payment channel (bKash, Nagad, Rocket, Upay)
  └─ Each card shows: channel name, wallet number, copy button
  └─ User clicks a card to select it (highlighted)
  └─ User opens MFS app, sends exact amount to the wallet number
  └─ User clicks "I Have Paid" → proceeds to TrxID entry

STEP 3 — ENTER TrxID & VERIFY
  └─ Text input: "Enter your SMS Transaction ID"
  └─ Button: "Verify Payment"
  └─ JS calls /api/v2/payment/verify with { id, method, transaction_id }
  └─ If verified → show success → run fulfillment logic
  └─ If not yet → show retry button (up to 3 retries, 10s gap)
  └─ If permanently failed → show error message
```

---

## 6. Step 1 — Collecting User Input & Creating a Session

### HTML Structure for Step 1

```html
<!-- Step 1: Input Card -->
<div id="sp-step-input" class="sp-card">
  <div class="sp-header">
    <h2>💳 Add Balance</h2>
    <p>Pay directly with bKash, Nagad, Rocket, or Upay</p>
  </div>
  <div class="sp-form">

    <!-- Customer Name -->
    <!-- If user is logged in on your site: set value from session and hide or make readonly -->
    <!-- If demo mode: show empty input -->
    <div class="sp-field">
      <label for="sp-name">Full Name</label>
      <input type="text" id="sp-name" placeholder="Enter your full name" autocomplete="name" />
      <!-- Pre-fill example: document.getElementById('sp-name').value = window.currentUser.name; -->
    </div>

    <!-- Payment Amount -->
    <div class="sp-field">
      <label for="sp-amount">Amount (BDT)</label>
      <input type="number" id="sp-amount" placeholder="e.g. 500" min="1" step="1" />
      <!-- Fixed amount example: document.getElementById('sp-amount').value = 299; -->
    </div>

    <div id="sp-input-error" class="sp-error" style="display:none;"></div>

    <button class="sp-btn-primary" onclick="spCreateSession()">
      Continue to Payment →
    </button>
  </div>
</div>
```

### JavaScript: Create Session (Step 1)

```javascript
// State: holds session data across steps
const spState = {
  sessionId:     null,   // id from /create response
  selectedMethod: null,  // "bkash" | "nagad" | "rocket" | "upay"
  amount:         null,  // amount from input
  retryCount:     0,     // for SMS sync retry logic
  maxRetries:     3,     // maximum verify attempts
};

async function spCreateSession() {
  const name   = document.getElementById('sp-name')?.value?.trim();
  const amount = document.getElementById('sp-amount')?.value?.trim();
  const errDiv = document.getElementById('sp-input-error');

  // Validate
  if (!name)  { spShowError(errDiv, 'Please enter your full name.'); return; }
  if (!amount || isNaN(amount) || parseFloat(amount) <= 0) {
    spShowError(errDiv, 'Please enter a valid amount greater than 0 BDT.');
    return;
  }
  spHideError(errDiv);
  spState.amount = parseFloat(amount);

  spShowStep('loading');
  spSetLoadingText('Setting up your payment session...');

  const payload = {
    cus_name: name,
    amount:   spState.amount,
    // meta_data: attach any relevant internal references
    meta_data: {
      // user_id:   window.currentUser?.id || null,
      // order_ref: 'ORDER-' + Date.now(),
      source:    'html_headless_integration',
      initiated: new Date().toISOString(),
    },
  };

  try {
    const data = await spCallAPI('/api/v2/payment/create', payload);

    if (data?.status === true && data?.id && Array.isArray(data?.methods)) {
      spState.sessionId = data.id;
      spRenderMethods(data.methods, data.brand);
      spShowStep('methods');
    } else {
      spShowStep('input');
      spShowError(errDiv, data?.message || 'Failed to create payment session. Please try again.');
    }
  } catch (err) {
    spShowStep('input');
    spShowError(errDiv, 'Network error. Please check your connection and try again.');
    console.error('[SkyPay Headless] Create error:', err);
  }
}
```

---

## 7. Step 2 — Displaying Active Payment Methods

This is the most critical UI step. The API returns a `methods[]` array. **Only render methods where the relevant `active_payments` flag is `true`**. If a wallet number is empty `""` or its flag is `false`, do not show it.

### JavaScript: Render Payment Method Cards

```javascript
// Called after /create succeeds. Builds method cards from API response.
function spRenderMethods(methods, brand) {
  const container = document.getElementById('sp-methods-list');
  if (!container) return;
  container.innerHTML = '';

  const icons = {
    bkash:  '📱',
    nagad:  '📲',
    rocket: '🚀',
    upay:   '💳',
  };

  const labels = {
    bkash:  'bKash',
    nagad:  'Nagad',
    rocket: 'Rocket',
    upay:   'Upay',
  };

  let hasAnyMethod = false;

  methods.forEach(function (method) {
    const name    = method.name;          // "bkash" | "nagad" | "rocket" | "upay"
    const active  = method.active_payments || {};
    const numbers = [];

    // ── Only add numbers where the corresponding flag is true ──
    if (active.personal && method.personal) {
      numbers.push({ type: 'Send Money', number: method.personal });
    }
    if (active.agent && method.agent) {
      numbers.push({ type: 'Cash In (Agent)', number: method.agent });
    }
    if (active.payment && method.payment) {
      numbers.push({ type: 'Merchant Pay', number: method.payment });
    }

    // If this method has no active numbers at all, skip it
    if (numbers.length === 0) return;

    hasAnyMethod = true;

    // Use the first active number as the primary number to display
    // (typically "Send Money" / personal is the most common)
    const primary = numbers[0];

    const card = document.createElement('div');
    card.className   = 'sp-method-card';
    card.dataset.name   = name;
    card.dataset.number = primary.number;

    card.innerHTML = `
      <div class="sp-method-icon">${icons[name] || '💳'}</div>
      <div class="sp-method-info">
        <div class="sp-method-name">${labels[name] || name}</div>
        <div class="sp-method-type">${primary.type}</div>
        <div class="sp-method-number">${primary.number}</div>
      </div>
      <button class="sp-copy-btn" onclick="spCopyNumber(event, '${primary.number}')">Copy</button>
    `;

    card.addEventListener('click', function (e) {
      if (e.target.classList.contains('sp-copy-btn')) return;
      spSelectMethod(name, primary.number, card);
    });

    container.appendChild(card);
  });

  if (!hasAnyMethod) {
    container.innerHTML = '<p class="sp-no-methods">No payment methods are currently active. Please try again later or contact support.</p>';
  }

  // Update amount display
  const amountDisplay = document.getElementById('sp-amount-display');
  if (amountDisplay) amountDisplay.textContent = 'BDT ' + spState.amount.toFixed(2);

  // Display brand support info
  if (brand) {
    const brandEl = document.getElementById('sp-brand-support');
    if (brandEl) {
      brandEl.innerHTML = `Support: <a href="https://wa.me/${brand.mobile}" target="_blank">${brand.name}</a>`;
    }
  }
}

// Called when user clicks a method card
function spSelectMethod(methodName, walletNumber, cardEl) {
  // Remove selection from all cards
  document.querySelectorAll('.sp-method-card').forEach(c => c.classList.remove('sp-method-selected'));

  // Select this card
  if (cardEl) cardEl.classList.add('sp-method-selected');

  spState.selectedMethod = methodName;

  // Show the "I Have Paid" button
  const confirmBtn = document.getElementById('sp-confirm-paid-btn');
  if (confirmBtn) {
    confirmBtn.style.display = 'block';
    confirmBtn.textContent   = `✅ I Have Paid via ${methodName.charAt(0).toUpperCase() + methodName.slice(1)}`;
  }
}

// Called when user clicks "I Have Paid" — moves to TrxID entry
function spConfirmPaid() {
  if (!spState.selectedMethod) {
    alert('Please select a payment method first.');
    return;
  }
  spShowStep('verify');
}

// Copy wallet number to clipboard
async function spCopyNumber(event, number) {
  event.stopPropagation();
  try {
    await navigator.clipboard.writeText(number);
    const btn = event.target;
    btn.textContent = '✓ Copied!';
    btn.style.background = '#16a34a';
    btn.style.color = '#fff';
    setTimeout(() => {
      btn.textContent = 'Copy';
      btn.style.background = '';
      btn.style.color = '';
    }, 2000);
  } catch (err) {
    // Fallback for older browsers
    const input = document.createElement('input');
    input.value = number;
    document.body.appendChild(input);
    input.select();
    document.execCommand('copy');
    document.body.removeChild(input);
  }
}
```

### HTML Structure for Step 2

```html
<!-- Step 2: Payment Methods Card -->
<div id="sp-step-methods" class="sp-card" style="display:none;">
  <div class="sp-header">
    <h2>Select Payment Method</h2>
    <p>Send exactly <strong id="sp-amount-display">BDT —</strong> to the number below</p>
  </div>

  <div class="sp-instruction-box">
    <ol class="sp-instruction-list">
      <li>Click a payment method below to select it</li>
      <li>Copy the wallet number shown</li>
      <li>Open your MFS app and send the <strong>exact amount</strong></li>
      <li>After sending, click "I Have Paid"</li>
    </ol>
  </div>

  <!-- Method cards rendered here by spRenderMethods() -->
  <div id="sp-methods-list" class="sp-methods-list"></div>

  <!-- Shown after user selects a method -->
  <button id="sp-confirm-paid-btn" class="sp-btn-success" onclick="spConfirmPaid()" style="display:none;">
    ✅ I Have Paid
  </button>

  <p id="sp-brand-support" class="sp-support-text"></p>

  <button class="sp-btn-back" onclick="spShowStep('input')">← Back</button>
</div>
```

---

## 8. Step 3 — TrxID Input & Verification

### HTML Structure for Step 3

```html
<!-- Step 3: TrxID Entry & Verify Card -->
<div id="sp-step-verify" class="sp-card" style="display:none;">
  <div class="sp-header">
    <h2>Enter Transaction ID</h2>
    <p>Enter the TrxID from your payment SMS to confirm your payment</p>
  </div>

  <div class="sp-form">
    <div class="sp-field">
      <label for="sp-trxid">SMS Transaction ID (TrxID)</label>
      <input
        type="text"
        id="sp-trxid"
        placeholder="e.g. BLA38KDK2M"
        autocomplete="off"
        autocorrect="off"
        autocapitalize="characters"
      />
    </div>

    <div class="sp-selected-method-info" id="sp-selected-method-display"></div>

    <div id="sp-verify-error" class="sp-error" style="display:none;"></div>

    <button id="sp-verify-btn" class="sp-btn-primary" onclick="spVerifyPayment()">
      🔍 Verify Payment
    </button>

    <!-- Retry button (shown after first failed attempt) -->
    <button id="sp-retry-btn" class="sp-btn-retry" onclick="spVerifyPayment()" style="display:none;">
      🔄 Retry Verification
    </button>

    <p class="sp-sms-note">
      ⏱️ If you just paid, the verification may take 5–20 seconds. Click Retry if needed.
    </p>

    <button class="sp-btn-back" onclick="spShowStep('methods')">← Change Payment Method</button>
  </div>
</div>
```

### JavaScript: Verify Payment (Step 3) with Retry Logic

```javascript
// Called when user clicks "Verify Payment" or "Retry"
async function spVerifyPayment() {
  const trxid  = document.getElementById('sp-trxid')?.value?.trim().toUpperCase();
  const errDiv = document.getElementById('sp-verify-error');

  if (!trxid) {
    spShowError(errDiv, 'Please enter your Transaction ID from the payment SMS.');
    return;
  }

  if (!spState.sessionId) {
    spShowError(errDiv, 'Session expired. Please go back and start a new payment.');
    return;
  }

  if (!spState.selectedMethod) {
    spShowError(errDiv, 'No payment method selected. Please go back and select a method.');
    return;
  }

  spHideError(errDiv);

  // Disable verify button during attempt
  const verifyBtn = document.getElementById('sp-verify-btn');
  const retryBtn  = document.getElementById('sp-retry-btn');
  if (verifyBtn) { verifyBtn.disabled = true; verifyBtn.textContent = '🔍 Verifying...'; }
  if (retryBtn)  { retryBtn.disabled  = true; retryBtn.textContent  = '🔄 Retrying...';  }

  try {
    const payload = {
      id:             spState.sessionId,
      method:         spState.selectedMethod,  // MUST be lowercase: "bkash", "nagad", etc.
      transaction_id: trxid,
    };

    // ── CRITICAL: method must be strictly lowercase ──
    // "bkash" ✅  |  "Bkash" ❌  |  "BKASH" ❌  |  "bKash" ❌
    // This is enforced above via spState.selectedMethod which is always lowercase.

    const data = await spCallAPI('/api/v2/payment/verify', payload);

    if (data?.status === true) {
      // ✅ Payment verified successfully
      spState.retryCount = 0;
      spOnVerified(data);

    } else {
      // ❌ Not verified — may be SMS still in transit
      spState.retryCount++;

      if (verifyBtn) { verifyBtn.disabled = false; verifyBtn.textContent = '🔍 Verify Payment'; }

      if (spState.retryCount < spState.maxRetries) {
        // Show retry option
        const remaining = spState.maxRetries - spState.retryCount;
        spShowError(
          errDiv,
          `Payment not confirmed yet. The SMS may still be syncing (5–20 seconds). ` +
          `Please wait a moment and click Retry. (${remaining} attempt${remaining !== 1 ? 's' : ''} remaining)`
        );
        if (retryBtn) {
          retryBtn.style.display = 'block';
          retryBtn.disabled = false;
          retryBtn.textContent = '🔄 Retry Verification';
        }
      } else {
        // Max retries reached
        spShowError(
          errDiv,
          `Verification failed after ${spState.maxRetries} attempts. ` +
          `Please double-check your Transaction ID and ensure you paid the exact amount (BDT ${spState.amount}). ` +
          `If the problem persists, contact support.`
        );
        if (retryBtn) { retryBtn.style.display = 'none'; }
        // Show contact support
        const supportMsg = document.getElementById('sp-contact-support');
        if (supportMsg) supportMsg.style.display = 'block';
      }
    }
  } catch (err) {
    if (verifyBtn) { verifyBtn.disabled = false; verifyBtn.textContent = '🔍 Verify Payment'; }
    if (retryBtn)  { retryBtn.disabled  = false; retryBtn.textContent  = '🔄 Retry';          }
    spShowError(errDiv, 'Network error during verification. Please check your connection and try again.');
    console.error('[SkyPay Headless] Verify error:', err);
  }
}
```

---

## 9. Step 4 — Fulfilling the Order or Showing Result

```javascript
// Called when /verify returns status: true
function spOnVerified(verifyData) {
  spShowStep('success');

  // Update success card content
  const msgEl     = document.getElementById('sp-success-message');
  const detailsEl = document.getElementById('sp-success-details');

  if (msgEl) {
    msgEl.textContent =
      'Your payment of BDT ' + (verifyData.amount || spState.amount) +
      ' via ' + spCapitalize(spState.selectedMethod) + ' has been confirmed!';
  }

  if (detailsEl) {
    detailsEl.innerHTML = `
      <table class="sp-details-table">
        <tr><td>Customer</td>       <td><strong>${verifyData.cus_name || '—'}</strong></td></tr>
        <tr><td>Amount</td>         <td><strong>BDT ${verifyData.amount || spState.amount}</strong></td></tr>
        <tr><td>Payment Method</td> <td><strong>${spCapitalize(spState.selectedMethod)}</strong></td></tr>
        <tr><td>Session ID</td>     <td><strong>${verifyData.id || spState.sessionId}</strong></td></tr>
        <tr><td>Status</td>         <td><strong class="sp-status-ok">✅ VERIFIED</strong></td></tr>
      </table>
    `;
  }

  // ─────────────────────────────────────────────────────
  // YOUR FULFILLMENT LOGIC GOES HERE
  // ─────────────────────────────────────────────────────
  //
  // (A) DEMO MODE — nothing extra needed; success screen is the result.
  //
  // (B) ADD BALANCE to user account via your backend:
  //     fetch('/api/user/add-balance', {
  //       method: 'POST',
  //       headers: { 'Content-Type': 'application/json' },
  //       body: JSON.stringify({
  //         amount:         verifyData.amount,
  //         method:         spState.selectedMethod,
  //         session_id:     verifyData.id,
  //         // user_id:     window.currentUser?.id,
  //       }),
  //     }).then(r => r.json()).then(result => {
  //       if (result.success) {
  //         console.log('Balance added:', result.new_balance);
  //       }
  //     });
  //
  // (C) ACTIVATE SUBSCRIPTION:
  //     fetch('/api/subscriptions/activate', {
  //       method: 'POST',
  //       headers: { 'Content-Type': 'application/json' },
  //       body: JSON.stringify({ plan: 'PRO', session_id: verifyData.id }),
  //     });
  //
  // (D) MARK ORDER AS PAID:
  //     const orderId = verifyData.meta_data?.order_ref || null;
  //     fetch('/api/orders/' + orderId + '/pay', { method: 'POST', ... });
  //
  // ⚠️ RULE: Any database change must go through your backend server.
  //          Never update your database directly from client-side JavaScript.

  console.log('[SkyPay Headless] Payment verified OK:', verifyData);
}
```

### HTML Structure for Success and Failed States

```html
<!-- Success Card -->
<div id="sp-step-success" class="sp-card" style="display:none;">
  <div class="sp-result sp-result-success">
    <div class="sp-result-icon">✅</div>
    <h2>Payment Verified!</h2>
    <p id="sp-success-message">Your payment has been confirmed.</p>
    <div id="sp-success-details" class="sp-details-box"></div>
    <button class="sp-btn-secondary" onclick="spReset()">Make Another Payment</button>
  </div>
</div>
```

---

## 10. Retry Logic for SMS Sync Latency

When a customer sends money via bKash or Nagad, the telecom network delivers an SMS to your merchant Android phone. The SkyPay Sync App reads this SMS and pushes it to the cloud. This takes **5 to 20 seconds**.

If the user submits their TrxID immediately after paying, the SMS may still be in transit. The first `/verify` call may return `false` even though the payment is real. This is normal.

**The built-in retry logic in this guide handles this automatically:**

```
User submits TrxID
       │
       ▼
Call /api/v2/payment/verify
       │
  ┌────┴────────────────┐
status: true          status: false
  │                       │
  ▼                       ▼
Fulfill              retryCount++
  order             Is retryCount < maxRetries (3)?
                         │
                    ┌────┴────┐
                   Yes        No
                    │         │
                    ▼         ▼
              Show        Show permanent
              "Retry"     error + support
              button      contact info
                    │
             User clicks Retry
             (after 10–20 seconds)
                    │
                    ▼
             Call /verify again
```

Key variables that control retry behavior:

```javascript
spState.retryCount = 0;    // increments on each failed verify
spState.maxRetries = 3;    // change this to allow more/fewer retries
```

You can also add an automatic countdown before retry becomes available:

```javascript
// After a failed verify, auto-enable retry after 10 seconds:
const retryBtn = document.getElementById('sp-retry-btn');
retryBtn.disabled = true;
retryBtn.textContent = 'Retry available in 10s...';
let count = 10;
const interval = setInterval(function () {
  count--;
  retryBtn.textContent = 'Retry available in ' + count + 's...';
  if (count <= 0) {
    clearInterval(interval);
    retryBtn.disabled = false;
    retryBtn.textContent = '🔄 Retry Verification';
  }
}, 1000);
```

---

## 11. Full Working Example — Complete HTML Page

This is a standalone, copy-pasteable HTML file demonstrating the complete headless payment flow. Save it as `payment.html` on your web server.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>SkyPay Headless Payment Demo</title>

  <style>
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

    .sp-container { width: 100%; max-width: 500px; }

    .sp-card {
      background: #fff;
      border-radius: 16px;
      box-shadow: 0 4px 24px rgba(0,0,0,0.10);
      padding: 28px 24px;
      margin-bottom: 0;
    }

    .sp-header { text-align: center; margin-bottom: 24px; }
    .sp-header h2 { font-size: 21px; font-weight: 700; color: #1e293b; margin-bottom: 6px; }
    .sp-header p { font-size: 14px; color: #64748b; }
    .sp-header strong { color: #2563eb; }

    .sp-form { display: flex; flex-direction: column; gap: 16px; }
    .sp-field { display: flex; flex-direction: column; gap: 5px; }
    .sp-field label { font-size: 13px; font-weight: 600; color: #374151; }
    .sp-field input {
      padding: 11px 13px;
      border: 1.5px solid #d1d5db;
      border-radius: 8px;
      font-size: 15px;
      color: #1e293b;
      background: #f8fafc;
      outline: none;
      transition: border-color 0.2s;
    }
    .sp-field input:focus { border-color: #2563eb; background: #fff; }

    .sp-btn-primary {
      width: 100%; padding: 13px;
      background: linear-gradient(135deg, #2563eb, #1d4ed8);
      color: #fff; border: none; border-radius: 10px;
      font-size: 15px; font-weight: 700; cursor: pointer;
      transition: opacity 0.2s;
    }
    .sp-btn-primary:hover { opacity: 0.90; }
    .sp-btn-primary:disabled { opacity: 0.60; cursor: not-allowed; }

    .sp-btn-success {
      width: 100%; padding: 13px;
      background: linear-gradient(135deg, #16a34a, #15803d);
      color: #fff; border: none; border-radius: 10px;
      font-size: 15px; font-weight: 700; cursor: pointer;
      margin-top: 8px; transition: opacity 0.2s;
    }
    .sp-btn-success:hover { opacity: 0.90; }

    .sp-btn-retry {
      width: 100%; padding: 11px;
      background: #f59e0b; color: #fff; border: none; border-radius: 10px;
      font-size: 14px; font-weight: 600; cursor: pointer; margin-top: 4px;
    }
    .sp-btn-retry:disabled { opacity: 0.6; cursor: not-allowed; }

    .sp-btn-secondary {
      width: 100%; padding: 11px;
      background: #f1f5f9; color: #374151;
      border: 1.5px solid #d1d5db; border-radius: 10px;
      font-size: 14px; font-weight: 600; cursor: pointer; margin-top: 10px;
    }
    .sp-btn-secondary:hover { background: #e2e8f0; }

    .sp-btn-back {
      background: none; border: none; color: #6b7280;
      font-size: 13px; cursor: pointer; margin-top: 8px;
      text-decoration: underline;
    }

    .sp-error {
      background: #fef2f2; border: 1px solid #fecaca;
      color: #dc2626; padding: 10px 13px;
      border-radius: 8px; font-size: 13px; line-height: 1.5;
    }

    .sp-loading { text-align: center; padding: 24px 0; }
    .sp-spinner {
      width: 42px; height: 42px;
      border: 4px solid #e2e8f0;
      border-top-color: #2563eb;
      border-radius: 50%;
      animation: spSpin 0.8s linear infinite;
      margin: 0 auto 14px;
    }
    @keyframes spSpin { to { transform: rotate(360deg); } }
    .sp-loading p { font-size: 15px; font-weight: 600; color: #1e293b; }
    .sp-loading small { font-size: 12px; color: #64748b; }

    .sp-instruction-box {
      background: #eff6ff; border: 1px solid #bfdbfe;
      border-radius: 10px; padding: 14px 16px; margin-bottom: 16px;
    }
    .sp-instruction-list { padding-left: 18px; }
    .sp-instruction-list li { font-size: 13px; color: #1d4ed8; margin-bottom: 5px; line-height: 1.5; }
    .sp-instruction-list li strong { color: #1e3a8a; }

    .sp-methods-list { display: flex; flex-direction: column; gap: 10px; margin-bottom: 12px; }

    .sp-method-card {
      display: flex; align-items: center; gap: 12px;
      border: 2px solid #e2e8f0; border-radius: 12px;
      padding: 14px 16px; cursor: pointer;
      transition: border-color 0.2s, background 0.2s;
      background: #fff;
    }
    .sp-method-card:hover { border-color: #93c5fd; background: #f0f9ff; }
    .sp-method-selected { border-color: #2563eb !important; background: #eff6ff !important; }

    .sp-method-icon { font-size: 28px; flex-shrink: 0; }
    .sp-method-info { flex: 1; min-width: 0; }
    .sp-method-name { font-size: 15px; font-weight: 700; color: #1e293b; }
    .sp-method-type { font-size: 11px; color: #64748b; margin: 1px 0 3px; }
    .sp-method-number { font-size: 14px; font-weight: 600; color: #2563eb; letter-spacing: 0.5px; }

    .sp-copy-btn {
      flex-shrink: 0; padding: 6px 12px;
      background: #f1f5f9; border: 1px solid #d1d5db;
      border-radius: 6px; font-size: 12px; font-weight: 600;
      color: #374151; cursor: pointer; transition: background 0.2s, color 0.2s;
    }
    .sp-copy-btn:hover { background: #e2e8f0; }

    .sp-no-methods { color: #dc2626; font-size: 14px; text-align: center; padding: 20px 0; }

    .sp-support-text { font-size: 12px; color: #94a3b8; text-align: center; margin-top: 8px; }
    .sp-support-text a { color: #2563eb; text-decoration: none; }

    .sp-sms-note { font-size: 12px; color: #64748b; text-align: center; margin-top: 4px; }

    .sp-selected-method-info {
      background: #f0fdf4; border: 1px solid #bbf7d0;
      border-radius: 8px; padding: 10px 14px; font-size: 13px; color: #166534;
    }

    .sp-result { text-align: center; padding: 6px 0; }
    .sp-result-icon { font-size: 50px; margin-bottom: 12px; }
    .sp-result h2 { font-size: 21px; font-weight: 700; margin-bottom: 10px; }
    .sp-result p { font-size: 14px; color: #475569; margin-bottom: 16px; line-height: 1.6; }
    .sp-result-success h2 { color: #16a34a; }

    .sp-details-box {
      background: #f8fafc; border: 1px solid #e2e8f0;
      border-radius: 10px; padding: 14px 16px; margin-bottom: 16px; text-align: left;
    }
    .sp-details-table { width: 100%; border-collapse: collapse; font-size: 13px; }
    .sp-details-table td { padding: 6px 4px; color: #475569; vertical-align: top; }
    .sp-details-table td:first-child { color: #94a3b8; width: 45%; }
    .sp-details-table td strong { color: #1e293b; }
    .sp-status-ok { color: #16a34a !important; }

    .sp-powered { text-align: center; margin-top: 14px; font-size: 12px; color: #94a3b8; }
    .sp-powered a { color: #2563eb; text-decoration: none; }

    #sp-contact-support {
      display: none; background: #fff7ed; border: 1px solid #fed7aa;
      border-radius: 8px; padding: 10px 14px; font-size: 13px;
      color: #9a3412; text-align: center; margin-top: 6px;
    }
    #sp-contact-support a { color: #c2410c; font-weight: 600; }
  </style>
</head>
<body>

<div class="sp-container">

  <!-- ─── STEP 1: Input ─── -->
  <div id="sp-step-input" class="sp-card">
    <div class="sp-header">
      <h2>💳 Add Balance</h2>
      <p>Pay directly using bKash, Nagad, Rocket, or Upay</p>
    </div>
    <div class="sp-form">
      <div class="sp-field">
        <label for="sp-name">Full Name</label>
        <input type="text" id="sp-name" placeholder="Enter your full name" autocomplete="name" />
      </div>
      <div class="sp-field">
        <label for="sp-amount">Amount (BDT)</label>
        <input type="number" id="sp-amount" placeholder="e.g. 500" min="1" step="1" />
      </div>
      <div id="sp-input-error" class="sp-error" style="display:none;"></div>
      <button class="sp-btn-primary" onclick="spCreateSession()">Continue to Payment →</button>
    </div>
  </div>

  <!-- ─── LOADING ─── -->
  <div id="sp-step-loading" class="sp-card" style="display:none;">
    <div class="sp-loading">
      <div class="sp-spinner"></div>
      <p id="sp-loading-text">Setting up your payment session...</p>
      <small>Please wait, do not close this page.</small>
    </div>
  </div>

  <!-- ─── STEP 2: Methods ─── -->
  <div id="sp-step-methods" class="sp-card" style="display:none;">
    <div class="sp-header">
      <h2>Select Payment Method</h2>
      <p>Send exactly <strong id="sp-amount-display">—</strong> to a number below</p>
    </div>
    <div class="sp-instruction-box">
      <ol class="sp-instruction-list">
        <li>Click a channel to select it</li>
        <li>Copy the wallet number and send the <strong>exact amount</strong> via your MFS app</li>
        <li>After paying, click "I Have Paid"</li>
      </ol>
    </div>
    <div id="sp-methods-list" class="sp-methods-list"></div>
    <button id="sp-confirm-paid-btn" class="sp-btn-success" onclick="spConfirmPaid()" style="display:none;">
      ✅ I Have Paid
    </button>
    <p id="sp-brand-support" class="sp-support-text"></p>
    <button class="sp-btn-back" onclick="spShowStep('input')">← Back</button>
  </div>

  <!-- ─── STEP 3: Verify ─── -->
  <div id="sp-step-verify" class="sp-card" style="display:none;">
    <div class="sp-header">
      <h2>Enter Transaction ID</h2>
      <p>Enter the TrxID from your payment SMS</p>
    </div>
    <div class="sp-form">
      <div id="sp-selected-method-display" class="sp-selected-method-info" style="display:none;"></div>
      <div class="sp-field">
        <label for="sp-trxid">SMS Transaction ID (TrxID)</label>
        <input type="text" id="sp-trxid" placeholder="e.g. BLA38KDK2M"
               autocomplete="off" autocorrect="off" autocapitalize="characters" />
      </div>
      <div id="sp-verify-error" class="sp-error" style="display:none;"></div>
      <button id="sp-verify-btn" class="sp-btn-primary" onclick="spVerifyPayment()">
        🔍 Verify Payment
      </button>
      <button id="sp-retry-btn" class="sp-btn-retry" onclick="spVerifyPayment()" style="display:none;">
        🔄 Retry Verification
      </button>
      <p class="sp-sms-note">⏱️ SMS sync may take 5–20 seconds. Use Retry if needed.</p>
      <div id="sp-contact-support">
        Need help? Contact us on
        <a href="https://wa.me/+8801761844968" target="_blank">WhatsApp</a> or
        <a href="https://t.me/BD_Prime_Minister" target="_blank">Telegram</a>
      </div>
      <button class="sp-btn-back" onclick="spShowStep('methods')">← Change Method</button>
    </div>
  </div>

  <!-- ─── SUCCESS ─── -->
  <div id="sp-step-success" class="sp-card" style="display:none;">
    <div class="sp-result sp-result-success">
      <div class="sp-result-icon">✅</div>
      <h2>Payment Verified!</h2>
      <p id="sp-success-message">Your payment has been confirmed.</p>
      <div id="sp-success-details" class="sp-details-box"></div>
      <button class="sp-btn-secondary" onclick="spReset()">Make Another Payment</button>
    </div>
  </div>

  <div class="sp-powered">
    Powered by <a href="https://skypaybd.top" target="_blank">SkyPay BD</a> &nbsp;·&nbsp;
    <a href="https://skypaybd.top/docs" target="_blank">Docs</a>
  </div>

</div>

<script>
// =====================================================================
// SKYPAY HEADLESS API v2 — COMPLETE VANILLA JS IMPLEMENTATION
// =====================================================================

const SP_CONFIG = {
  BRAND_KEY: 'YOUR_BRAND_KEY_HERE',  // ⚠️ Use backend proxy for production
  PROXY_URL: '',                     // Set to your backend endpoint for production
  API_BASE:  'https://core.skypaybd.top',
};

const spState = {
  sessionId:      null,
  selectedMethod: null,
  amount:         null,
  retryCount:     0,
  maxRetries:     3,
};

// ─── Generic API caller ───
async function spCallAPI(path, payload) {
  const url     = SP_CONFIG.PROXY_URL ? (SP_CONFIG.PROXY_URL + path) : (SP_CONFIG.API_BASE + path);
  const headers = SP_CONFIG.PROXY_URL
    ? { 'Content-Type': 'application/json' }
    : { 'Content-Type': 'application/json', 'BRAND-KEY': SP_CONFIG.BRAND_KEY };
  const res = await fetch(url, { method: 'POST', headers, body: JSON.stringify(payload) });
  return await res.json();
}

// ─── STEP 1: Create Session ───
async function spCreateSession() {
  const name   = document.getElementById('sp-name')?.value?.trim();
  const amount = document.getElementById('sp-amount')?.value?.trim();
  const errDiv = document.getElementById('sp-input-error');

  if (!name) { spShowError(errDiv, 'Please enter your full name.'); return; }
  if (!amount || isNaN(amount) || parseFloat(amount) <= 0) {
    spShowError(errDiv, 'Please enter a valid amount greater than 0 BDT.'); return;
  }
  spHideError(errDiv);
  spState.amount = parseFloat(amount);
  spState.retryCount = 0;
  spShowStep('loading');
  spSetLoadingText('Setting up your payment session...');

  try {
    const data = await spCallAPI('/api/v2/payment/create', {
      cus_name: name,
      amount:   spState.amount,
      meta_data: { source: 'html_headless', initiated: new Date().toISOString() },
    });

    if (data?.status === true && data?.id && Array.isArray(data?.methods)) {
      spState.sessionId = data.id;
      spRenderMethods(data.methods, data.brand);
      spShowStep('methods');
    } else {
      spShowStep('input');
      spShowError(errDiv, data?.message || 'Failed to create payment session.');
    }
  } catch (err) {
    spShowStep('input');
    spShowError(errDiv, 'Network error. Please check your connection.');
    console.error('[SkyPay] Create error:', err);
  }
}

// ─── STEP 2: Render methods ───
function spRenderMethods(methods, brand) {
  const container = document.getElementById('sp-methods-list');
  container.innerHTML = '';
  const icons  = { bkash: '📱', nagad: '📲', rocket: '🚀', upay: '💳' };
  const labels = { bkash: 'bKash', nagad: 'Nagad', rocket: 'Rocket', upay: 'Upay' };
  let hasAny = false;

  methods.forEach(function (m) {
    const active = m.active_payments || {};
    let number = null, type = null;
    if (active.personal && m.personal) { number = m.personal; type = 'Send Money'; }
    else if (active.agent && m.agent)  { number = m.agent;    type = 'Cash In (Agent)'; }
    else if (active.payment && m.payment) { number = m.payment; type = 'Merchant Pay'; }
    if (!number) return;
    hasAny = true;

    const card = document.createElement('div');
    card.className = 'sp-method-card';
    card.dataset.name = m.name;
    card.innerHTML = `
      <div class="sp-method-icon">${icons[m.name] || '💳'}</div>
      <div class="sp-method-info">
        <div class="sp-method-name">${labels[m.name] || m.name}</div>
        <div class="sp-method-type">${type}</div>
        <div class="sp-method-number">${number}</div>
      </div>
      <button class="sp-copy-btn" onclick="spCopyNumber(event,'${number}')">Copy</button>
    `;
    card.addEventListener('click', function (e) {
      if (e.target.classList.contains('sp-copy-btn')) return;
      document.querySelectorAll('.sp-method-card').forEach(c => c.classList.remove('sp-method-selected'));
      card.classList.add('sp-method-selected');
      spState.selectedMethod = m.name;
      const btn = document.getElementById('sp-confirm-paid-btn');
      if (btn) { btn.style.display = 'block'; btn.textContent = `✅ I Have Paid via ${labels[m.name]}`; }
    });
    container.appendChild(card);
  });

  if (!hasAny) container.innerHTML = '<p class="sp-no-methods">No active payment methods. Contact support.</p>';

  const ad = document.getElementById('sp-amount-display');
  if (ad) ad.textContent = 'BDT ' + spState.amount.toFixed(2);

  if (brand) {
    const bs = document.getElementById('sp-brand-support');
    if (bs) bs.innerHTML = `Support: <a href="https://wa.me/${brand.mobile}" target="_blank">${brand.name}</a>`;
  }
}

async function spCopyNumber(event, number) {
  event.stopPropagation();
  try { await navigator.clipboard.writeText(number); } catch (e) { }
  const btn = event.target;
  const orig = btn.textContent;
  btn.textContent = '✓ Copied!'; btn.style.background = '#16a34a'; btn.style.color = '#fff';
  setTimeout(() => { btn.textContent = orig; btn.style.background = ''; btn.style.color = ''; }, 2000);
}

function spConfirmPaid() {
  if (!spState.selectedMethod) { alert('Please select a payment method first.'); return; }
  const d = document.getElementById('sp-selected-method-display');
  if (d) {
    d.style.display = 'block';
    d.textContent = `You selected: ${spState.selectedMethod.charAt(0).toUpperCase() + spState.selectedMethod.slice(1)} — please enter your TrxID below.`;
  }
  spShowStep('verify');
}

// ─── STEP 3: Verify Payment ───
async function spVerifyPayment() {
  const trxid  = document.getElementById('sp-trxid')?.value?.trim().toUpperCase();
  const errDiv = document.getElementById('sp-verify-error');

  if (!trxid) { spShowError(errDiv, 'Please enter your Transaction ID from the payment SMS.'); return; }
  if (!spState.sessionId) { spShowError(errDiv, 'Session expired. Please go back and start over.'); return; }
  if (!spState.selectedMethod) { spShowError(errDiv, 'No payment method selected. Please go back.'); return; }
  spHideError(errDiv);

  const vBtn = document.getElementById('sp-verify-btn');
  const rBtn = document.getElementById('sp-retry-btn');
  if (vBtn) { vBtn.disabled = true; vBtn.textContent = '🔍 Verifying...'; }
  if (rBtn) { rBtn.disabled = true; rBtn.textContent = '🔄 Retrying...'; }

  try {
    const data = await spCallAPI('/api/v2/payment/verify', {
      id:             spState.sessionId,
      method:         spState.selectedMethod,   // always lowercase from spState
      transaction_id: trxid,
    });

    if (data?.status === true) {
      spState.retryCount = 0;
      spOnVerified(data);
    } else {
      spState.retryCount++;
      if (vBtn) { vBtn.disabled = false; vBtn.textContent = '🔍 Verify Payment'; }

      const rem = spState.maxRetries - spState.retryCount;
      if (rem > 0) {
        spShowError(errDiv,
          `Payment not confirmed yet. SMS may still be syncing (5–20 sec). ` +
          `Wait a moment and click Retry. (${rem} attempt${rem !== 1 ? 's' : ''} left)`
        );
        if (rBtn) { rBtn.style.display = 'block'; rBtn.disabled = false; rBtn.textContent = '🔄 Retry Verification'; }
      } else {
        spShowError(errDiv,
          `Verification failed after ${spState.maxRetries} attempts. ` +
          `Please check your TrxID and ensure you paid exactly BDT ${spState.amount}.`
        );
        if (rBtn) rBtn.style.display = 'none';
        const cs = document.getElementById('sp-contact-support');
        if (cs) cs.style.display = 'block';
      }
    }
  } catch (err) {
    if (vBtn) { vBtn.disabled = false; vBtn.textContent = '🔍 Verify Payment'; }
    if (rBtn) { rBtn.disabled = false; rBtn.textContent = '🔄 Retry'; }
    spShowError(errDiv, 'Network error. Please check your connection.');
    console.error('[SkyPay] Verify error:', err);
  }
}

function spOnVerified(verifyData) {
  spShowStep('success');
  const msg = document.getElementById('sp-success-message');
  if (msg) msg.textContent = `BDT ${verifyData.amount || spState.amount} via ${spCapitalize(spState.selectedMethod)} — confirmed!`;

  const det = document.getElementById('sp-success-details');
  if (det) {
    det.innerHTML = `
      <table class="sp-details-table">
        <tr><td>Customer</td>  <td><strong>${verifyData.cus_name || '—'}</strong></td></tr>
        <tr><td>Amount</td>    <td><strong>BDT ${verifyData.amount || spState.amount}</strong></td></tr>
        <tr><td>Method</td>    <td><strong>${spCapitalize(spState.selectedMethod)}</strong></td></tr>
        <tr><td>Session ID</td><td><strong>${verifyData.id || spState.sessionId}</strong></td></tr>
        <tr><td>Status</td>    <td><strong class="sp-status-ok">✅ VERIFIED</strong></td></tr>
      </table>
    `;
  }

  // ── YOUR FULFILLMENT LOGIC HERE ──
  // fetch('/api/user/add-balance', {
  //   method: 'POST',
  //   headers: { 'Content-Type': 'application/json' },
  //   body: JSON.stringify({ amount: verifyData.amount, session_id: verifyData.id }),
  // });

  console.log('[SkyPay Headless] Verified OK:', verifyData);
}

// ─── Utility Functions ───
function spShowStep(step) {
  ['input', 'loading', 'methods', 'verify', 'success'].forEach(s => {
    const el = document.getElementById('sp-step-' + s);
    if (el) el.style.display = (s === step) ? 'block' : 'none';
  });
}

function spSetLoadingText(text) {
  const el = document.getElementById('sp-loading-text');
  if (el) el.textContent = text;
}

function spShowError(el, msg) { if (!el) return; el.textContent = msg; el.style.display = 'block'; }
function spHideError(el) { if (!el) return; el.textContent = ''; el.style.display = 'none'; }

function spReset() {
  spState.sessionId = null;
  spState.selectedMethod = null;
  spState.amount = null;
  spState.retryCount = 0;
  ['sp-trxid', 'sp-name', 'sp-amount'].forEach(id => {
    const el = document.getElementById(id);
    if (el && !el.readOnly) el.value = '';
  });
  const rBtn = document.getElementById('sp-retry-btn');
  if (rBtn) { rBtn.style.display = 'none'; rBtn.disabled = false; }
  const cs = document.getElementById('sp-contact-support');
  if (cs) cs.style.display = 'none';
  spShowStep('input');
  spSetLoadingText('Setting up your payment session...');
}

function spCapitalize(str) {
  if (!str) return str;
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

// Optional: pre-fill from your user system on page load
// window.addEventListener('DOMContentLoaded', function () {
//   if (window.currentUser) {
//     const n = document.getElementById('sp-name');
//     if (n && window.currentUser.name) { n.value = window.currentUser.name; n.readOnly = true; }
//   }
// });
</script>

</body>
</html>
```

---

## 12. Integrating Into an Existing HTML File

### Step A — Add CSS
Copy all `.sp-*` styles from Section 11 into your existing `<style>` tag or CSS file. All class names are prefixed `sp-` to avoid conflicts.

### Step B — Add HTML Structure
Copy the five `<div id="sp-step-*">` blocks and paste them where you want the payment section on your page.

### Step C — Add JavaScript
Copy the full `<script>` block from Section 11 and paste it before your closing `</body>` tag, after your existing scripts.

### Step D — Pre-fill User Data
If your website has a logged-in user:

```javascript
function onUserLoggedIn(user) {
  const nameField = document.getElementById('sp-name');
  if (nameField && user.name) {
    nameField.value = user.name;
    nameField.readOnly = true; // prevent editing
  }
}
```

### Step E — Set Fixed Amount
If payment is for a specific product:

```javascript
const amountField = document.getElementById('sp-amount');
if (amountField) {
  amountField.value = 299;
  amountField.readOnly = true;
}
```

---

## 13. CSS Styling Reference

| Class | Purpose |
|---|---|
| `.sp-container` | Outer wrapper (max-width 500px) |
| `.sp-card` | White card panel with shadow |
| `.sp-header` | Title and subtitle |
| `.sp-form` | Flex column form |
| `.sp-field` | Label + input wrapper |
| `.sp-btn-primary` | Blue action button |
| `.sp-btn-success` | Green "I Have Paid" button |
| `.sp-btn-retry` | Orange retry button |
| `.sp-btn-back` | Plain text back link |
| `.sp-error` | Red error message |
| `.sp-methods-list` | Method cards container |
| `.sp-method-card` | Individual MFS channel card |
| `.sp-method-selected` | Highlighted selected card |
| `.sp-copy-btn` | Number copy button |
| `.sp-details-table` | Verification result table |
| `.sp-status-ok` | Green "VERIFIED" text |

---

## 14. API Reference Summary

### Create Session

```
POST https://core.skypaybd.top/api/v2/payment/create

Headers:
  BRAND-KEY:    <your_brand_key>
  Content-Type: application/json

Body:
{
  "cus_name":  "Full Name",
  "amount":    500,
  "meta_data": { "user_id": "..." }    ← optional
}

Response:
{
  "status": true,
  "id":      "a1b2c3d4e5f6g7h8",       ← SAVE THIS
  "brand":   { "name": "...", "mobile": "..." },
  "methods": [ { "name": "bkash", "active_payments": {...}, "personal": "01..." }, ... ]
}
```

### Verify Payment

```
POST https://core.skypaybd.top/api/v2/payment/verify

Headers:
  BRAND-KEY:    <your_brand_key>
  Content-Type: application/json

Body:
{
  "id":             "a1b2c3d4e5f6g7h8",   ← session id from /create
  "method":         "bkash",              ← MUST be strict lowercase
  "transaction_id": "BLA38KDK2M"          ← TrxID from user's SMS
}

Response (success):
{
  "status":   true,
  "amount":   "500.00",
  "cus_name": "Full Name",
  "id":       "a1b2c3d4e5f6g7h8"
}
→ Only fulfill when status === true
```

---

## 15. API Response Fields — methods[] Array

| Field | Type | Description |
|---|---|---|
| `methods[].name` | String | Channel: `bkash`, `nagad`, `rocket`, `upay` |
| `methods[].active_payments.personal` | Boolean | If true, `personal` number is active (Send Money) |
| `methods[].active_payments.agent` | Boolean | If true, `agent` number is active (Cash In) |
| `methods[].active_payments.payment` | Boolean | If true, `payment` number is active (bKash Merchant Pay) |
| `methods[].personal` | String | Phone number for Send Money (empty `""` if inactive) |
| `methods[].agent` | String | Phone number for Agent Cash In (empty `""` if inactive) |
| `methods[].payment` | String | Merchant payment number (bKash only, empty if inactive) |

> **Rule:** Never show a number if its `active_payments` flag is `false` or if the number field is `""`.

---

## 16. Error Handling & Edge Cases

| Scenario | API Response | JS Handling |
|---|---|---|
| Invalid name or amount | Not called | Input validation prevents API call |
| BRAND-KEY invalid | `401 Unauthorized` | Error shown in form |
| Android phone offline | `403 Forbidden` | Network error caught, shown to user |
| Session ID expired | `404 Not Found` | Error shown, user told to restart |
| TrxID not yet in SMS log | `400` + false | Retry button shown (up to 3 times) |
| TrxID already used | `400` already claimed | Error shown, no fulfillment |
| Wrong method casing | `400` unsupported method | JS always stores method as lowercase |
| Max retries reached | — | Error + support contact shown |
| Network error during verify | — | Error caught, user told to retry |

---

## 17. Security Rules for Frontend Integrations

| Rule | Details |
|---|---|
| **Never expose BRAND-KEY in production** | Use a backend proxy: PHP file, Node.js route, Cloudflare Worker, Vercel Function |
| **Always use lowercase method names** | `spState.selectedMethod` is set from `method.name` which is always lowercase from API |
| **Save session ID on the client, verify server-side in production** | In a full-stack setup, save `id` on the server to prevent tampering |
| **Never update database from JS** | Fulfillment logic must call your own backend API |
| **Exact amount required** | SkyPay validates SMS amount against session amount — never modify the amount |
| **Idempotency** | Once a TrxID is verified, SkyPay marks it as claimed — handle duplicate verify calls in your backend |
| **Don't trust user-provided method** | Always use the method from `spState.selectedMethod`, which is set from API response — don't let the user freely type it |

---

## 18. Integration Checklist

Before going live:

- [ ] `BRAND_KEY` stored in backend environment — not in public JavaScript
- [ ] Backend proxy set up for `/api/v2/payment/create` and `/api/v2/payment/verify`
- [ ] Session `id` from `/create` stored immediately in `spState.sessionId`
- [ ] Only channels where `active_payments` flag is `true` are shown as cards
- [ ] `selectedMethod` is always stored as strict lowercase (`bkash`, `nagad`, etc.)
- [ ] Exact amount is passed to `/create` and displayed to user without modification
- [ ] Retry logic is implemented with a maximum of 2–3 attempts
- [ ] Fulfillment logic is called only after `status === true` from `/verify`
- [ ] Database updates go through backend API — never from client-side JS
- [ ] Backend prevents double-fulfillment if verify is called twice for same session
- [ ] Merchant Android phone is on, connected to internet, and SkyPay APK is running
- [ ] Battery optimization disabled for SkyPay APK on merchant phone

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
| WhatsApp Support | https://wa.me/+8801761844968 |
| Telegram Support | https://t.me/BD_Prime_Minister |

---

> **Prefer a redirect-based flow?**
> If you want SkyPay to handle the entire payment UI on their hosted page — with the user being redirected and returned — see the [SkyPay Hosted Gateway HTML Integration Guide](../Hosted/README.md).

---

*© SkyPay Technologies Ltd. — Automated MFS Payment Infrastructure for Bangladesh*
*Website: https://skypaybd.top | Docs: https://skypaybd.top/docs | Support: https://wa.me/+8801761844968*
