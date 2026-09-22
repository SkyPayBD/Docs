# SkyPay - BD

<div align="center">

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png" width="100" alt="SkyPay Logo" />

<br/>

<img src="https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789098593_de7d9d238ad2e0178762.png" width="420" alt="SkyPay Banner" />

<br/><br/>

**Zero-Redirect Automated Payment Infrastructure — Pure HTML / CSS / Vanilla JavaScript Edition**

*Accept automated payments via **bKash**, **Nagad**, **Rocket**, **Upay**, and **Binance Pay** — directly inside your own website page, with plain HTML, CSS, and JavaScript. No framework required.*

<br/>

[![Official Website](https://img.shields.io/badge/Official%20Website-skypaybd.top-2563eb?style=for-the-badge&logo=googlechrome&logoColor=white)](https://skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)
[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![API Version](https://img.shields.io/badge/Headless%20API-v2.0%20Live-16a34a?style=for-the-badge&logo=statuspage&logoColor=white)](https://core.skypaybd.top)
[![GitHub](https://img.shields.io/badge/GitHub-SkyPayBD-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/SkyPayBD)

</div>

---

> 📖 **Scope of this document:** This README covers integrating **Headless Payment API v2** into a website using **only HTML, CSS, and Vanilla JavaScript** — no build tools, no framework. It does not cover the v1 Hosted Redirect Gateway or the Telegram Bot integration — those are documented separately.

---

## 🚨 Security Notice — Please Read Before Implementing

Calling `/api/v2/payment/create` and `/api/v2/payment/verify` **directly from browser JavaScript** means your `BRAND-KEY` is visible to anyone who opens their browser's Network tab or View Source. This is **not the recommended setup for production**, and we want to be upfront about exactly why, rather than bury it in a footnote.

**What is actually at risk:**
- Anyone who copies your exposed `BRAND-KEY` can call `/create` under your brand — this can flood your dashboard with fake sessions, distort your reporting, or be used to impersonate your checkout for phishing.
- `/verify` cannot be used to steal money on its own — it only returns `"status": true` for a TrxID/Order ID that genuinely matches a real incoming SMS or a real Binance transaction. Exposure does **not** let anyone credit themselves money that wasn't actually paid.
- The real risk is **abuse of your account and resources**, not direct financial loss — but it should still be taken seriously.

**Do we allow the direct frontend approach anyway? Yes.** Plenty of demos, internal tools, and low-risk projects run this way. If you choose to, we simply ask that you use it responsibly:

- **Preferred:** Route both calls through a tiny backend proxy (a PHP endpoint, a Node route, a Cloudflare Worker, or a Vercel Edge Function) that holds `BRAND-KEY` server-side and forwards the request. This is barely more work than calling the API directly and removes the exposure entirely — Section 3 explains exactly how.
- **If you must ship the key in client-side code:** obfuscate/minify it rather than leaving it as a plain readable string, rotate your `BRAND-KEY` periodically from the Dashboard, and watch your **Brand Management** session volume for anything unusual.
- Never pair a frontend-exposed key with a brand configured for large transaction amounts.
- This guide includes **both approaches** — the direct frontend call (fastest to test with) and the backend-proxy pattern (recommended before going live) — so you can choose based on your own risk tolerance.

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

| Channel | `method` Value | Personal (Send Money) | Agent (Cash In) | Merchant Pay | Verification |
|---|---|:---:|:---:|:---:|---|
| **bKash** | `bkash` | ✅ | ✅ | ✅ | Android SMS Sync (5–20 sec) |
| **Nagad** | `nagad` | ✅ | ✅ | 🔄 Under Review | Android SMS Sync (5–20 sec) |
| **Rocket** | `rocket` | ✅ | ✅ | 🔄 Under Review | Android SMS Sync (5–20 sec) |
| **Upay** | `upay` | ✅ | ❌ | 🔄 Under Review | Android SMS Sync (5–20 sec) |
| **Binance Pay** | `binance` | ✅ (USDT) | — | — | Automatic, Server-Side (Instant) |

> **Note:** `binance` does **not** use the Android SMS bridge and does not need a merchant phone. The customer sends USDT and provides a **Binance Order ID**, and SkyPay verifies it automatically on the server, instantly.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#️-prerequisites)
- [Authentication](#-authentication)
- [API Endpoint Directory](#-api-endpoint-directory)
- [Architecture: Frontend vs Backend Responsibilities](#️-architecture-frontend-vs-backend-responsibilities)
- [How the Headless Flow Works in a Browser](#-how-the-headless-flow-works-in-a-browser)
- [The Payment Flow (4 Steps)](#-the-payment-flow-4-steps)
- [Step 1 — Collecting Input & Creating a Session](#step-1--collecting-input--creating-a-session)
- [Step 2 — Displaying Active Payment Methods](#step-2--displaying-active-payment-methods)
- [Step 3 — TrxID / Order ID Input & Verification](#step-3--trxid--order-id-input--verification)
- [Step 4 — Fulfilling the Order](#step-4--fulfilling-the-order)
- [Retry Logic — MFS vs Binance](#-retry-logic--mfs-vs-binance)
- [Full Working Example — Complete HTML Page](#-full-working-example--complete-html-page)
- [Integrating Into an Existing HTML File](#-integrating-into-an-existing-html-file)
- [CSS Styling Reference](#-css-styling-reference)
- [API Reference Summary](#-api-reference-summary)
- [API Response Fields — methods[] Array](#-api-response-fields--methods-array)
- [Error Handling & Edge Cases](#-error-handling--edge-cases)
- [Security Rules for Frontend Integrations](#-security-rules-for-frontend-integrations)
- [Integration Checklist](#-integration-checklist)
- [Android Merchant Sync App Setup](#-android-merchant-sync-app-setup)
- [Contact & Support](#-contact--support)
- [Quick Links](#-quick-links)

---

## 🌐 Overview

**Headless API v2** lets a customer pay via bKash, Nagad, Rocket, Upay, or Binance Pay **without ever leaving your page**. Your JavaScript calls `/create` to receive live wallet numbers, renders them in your own UI, collects the customer's Transaction ID (or Binance Order ID), and calls `/verify` to confirm the payment — all inline, on the same page.

**You build the UI. SkyPay handles the payment matching.**

Use this guide when you want:
- A seamless, no-redirect payment experience on your website
- Full control over the payment UI's design, colors, and layout
- A payment flow for a single-page app, dashboard, or admin panel

---

## ⚙️ Prerequisites

### 1. BRAND-KEY

- Log in to the **[SkyPay Merchant Dashboard](https://skypaybd.top/user/brands)**
- Create (or open) a Brand and copy the generated **BRAND-KEY**
- For production, hold it in a backend proxy — see the Security Notice above and Section 3 below
- For local testing only, it is acceptable to place it temporarily in JavaScript — remove it before shipping publicly

### 2. Merchant Android Device — MFS Channels Only

Required for **bKash, Nagad, Rocket, Upay**. **Not required for Binance Pay.**

- Install the **SkyPay Merchant Sync APK** on an Android 7.0+ phone carrying your merchant SIM cards
- Grant **SMS Listener** and **Notification Access** permissions
- Disable **Battery Optimization** for the app
- Keep the phone powered on and connected 24/7

> ⚠️ Without an active connected Android device, `/create` calls for MFS channels will return `403 Forbidden`. Binance is unaffected — it is configured once in the Dashboard and verified server-side.

### 3. Wallet Numbers Shown = What You Have Configured

The `methods[]` array from `/create` only returns the channels you have configured and activated on your Brand dashboard. If a channel is missing from the response, it is simply not configured — **always render only what the API actually returns.**

---

## 🔐 Authentication

Every request requires exactly one header:

```http
BRAND-KEY: your_brand_key_here
Content-Type: application/json
Accept: application/json
```

> **v2 accepts `BRAND-KEY` only** — no `API-KEY`, `SECRET-KEY`, or `?api_key=` query parameter aliases are read by v2 endpoints (those only apply to the v1 Hosted Gateway). Always send the exact header name `BRAND-KEY`.

See the **Security Notice** above for why exposing this header directly in browser JavaScript is discouraged for production use.

---

## 📡 API Endpoint Directory

| Action | Method | Full Endpoint URL |
|---|:---:|---|
| Create Payment Session | `POST` | `https://core.skypaybd.top/api/v2/payment/create` |
| Verify Transaction | `POST` | `https://core.skypaybd.top/api/v2/payment/verify` |

```
https://core.skypaybd.top
```

---

## 🏗️ Architecture: Frontend vs Backend Responsibilities

| Responsibility | Direct Frontend (Demo) | Backend Proxy (Recommended) |
|---|:---:|:---:|
| Collect name, amount | ✅ | ✅ |
| Display payment method cards | ✅ | ✅ |
| Copy wallet number / UID to clipboard | ✅ | ✅ |
| Call `/api/v2/payment/create` | ✅ (key exposed in browser) | ✅ (key hidden server-side) |
| Call `/api/v2/payment/verify` | ✅ (key exposed in browser) | ✅ (key hidden server-side) |
| Save session `id` | JS variable only (lost on refresh) | Server session / database (durable) |
| Credit balance / fulfill order in your database | ❌ Never do this from the browser | ✅ Always do this on the backend |

Even if you start with the direct-frontend approach for speed, **fulfillment (crediting balance, activating a subscription, marking an order paid) must always happen on your backend**, never purely from client-side JavaScript that any user could tamper with.

---

## 🔄 How the Headless Flow Works in a Browser

```
[User on your HTML page]
         │  1. Enters name (if unknown) and amount
         │  2. Clicks "Continue to Payment"
         ▼
[JS → POST /api/v2/payment/create]
Headers: { BRAND-KEY, Content-Type }
Body:    { cus_name, amount, meta_data }
         ▼
[SkyPay returns session id + active methods[], including binance if configured]
{
  "status": true,
  "id": "a1b2c3d4e5f6g7h8",                      ← SAVE THIS
  "methods": [
    { "name": "bkash",   "active_payments": {...}, "personal": "01XXXXXXXX" },
    { "name": "nagad",   "active_payments": {...}, "personal": "01XXXXXXXX" },
    { "name": "binance", "active_payments": {...}, "personal": "UID_XXXX",
      "currency": "USDT", "dollar_rate": "120.00", "amount_usdt": "4.1667" }
  ]
}
         ▼
[JS renders method cards — MFS cards show a phone number, the Binance card
 shows a receiving UID and the exact amount_usdt to send]
User selects a channel and pays via their MFS / Binance app
         ▼
[User enters TrxID (MFS) or Order ID (Binance) into your page]
[JS → POST /api/v2/payment/verify]
Body: { id, method, transaction_id }   (Binance: send as `order_id`)
         ▼
[SkyPay verifies — MFS: 5–20 sec SMS match | Binance: instant, server-side]
         ┌───────────────┴───────────────┐
      SUCCESS                          NOT YET / ERROR
         │                                 │
         ▼                                 ▼
{ "status": true,                   { "status": false,
  "message": "...",                   "message": "..." }
  "data": { amount, cus_name,       MFS → show retry button
    payment_method, status,         Binance → show the exact error
    transaction_id, ... } }         (not a "wait and retry" case, usually)
```

---

## 🧩 The Payment Flow (4 Steps)

```
STEP 1 — INPUT
  User provides name + amount → JS calls /api/v2/payment/create
  Receives: session id + active wallet numbers / Binance UID

STEP 2 — SELECT METHOD & PAY
  JS renders a card per active channel (bKash, Nagad, Rocket, Upay, Binance)
  User selects a card, sends the exact amount (or exact USDT for Binance)
  User clicks "I Have Paid"

STEP 3 — ENTER TrxID / ORDER ID & VERIFY
  Input label switches automatically: "TrxID" for MFS, "Binance Order ID" for Binance
  JS calls /api/v2/payment/verify with { id, method, transaction_id (or order_id) }
  Success → parse the wrapped `data` object → show confirmation
  Not yet (MFS) → retry button | Permanent error → show exact message

STEP 4 — FULFILL
  Only after status === true → call your own backend to credit balance / activate order
```

---

## Step 1 — Collecting Input & Creating a Session

### HTML

```html
<div id="sp-step-input" class="sp-card">
  <div class="sp-header">
    <h2>💳 Add Balance</h2>
    <p>Pay with bKash, Nagad, Rocket, Upay, or Binance Pay</p>
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
```

### JavaScript

```javascript
const spState = {
  sessionId:      null,   // id from /create response
  selectedMethod: null,   // "bkash" | "nagad" | "rocket" | "upay" | "binance"
  selectedNumber: null,   // wallet number or Binance UID shown to the user
  amount:         null,   // BDT amount entered by the user
  amountUsdt:     null,   // populated only when Binance is selected
  retryCount:     0,
  maxRetries:     3,
};

async function spCreateSession() {
  const name   = document.getElementById('sp-name')?.value?.trim();
  const amount = document.getElementById('sp-amount')?.value?.trim();
  const errDiv = document.getElementById('sp-input-error');

  if (!name)  { spShowError(errDiv, 'Please enter your full name.'); return; }
  if (!amount || isNaN(amount) || parseFloat(amount) <= 0) {
    spShowError(errDiv, 'Please enter a valid amount greater than 0 BDT.');
    return;
  }
  spHideError(errDiv);
  spState.amount = parseFloat(amount);
  spState.retryCount = 0;

  spShowStep('loading');
  spSetLoadingText('Setting up your payment session...');

  try {
    const data = await spCallAPI('/api/v2/payment/create', {
      cus_name:  name,
      amount:    spState.amount,
      meta_data: { source: 'html_headless_integration', initiated: new Date().toISOString() },
    });

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

**Create — Error Responses**

| HTTP Code | Message | Cause |
|---|---|---|
| `401` | `BRAND-KEY header is required.` | The header was not sent or is empty. |
| `401` | `Invalid or inactive BRAND-KEY provided.` | Key does not exist, or brand deactivated. |
| `403` | `Associated merchant account is inactive.` | Merchant account suspended. |
| `403` | `No active SMS sync device found for this account.` | No Android device connected for MFS channels. |
| `400` | `Valid cus_name and numeric amount are required.` | `cus_name` empty, or `amount` missing / non-numeric / ≤ 0. |
| `400` | `meta_data must be a valid JSON object.` | `meta_data` sent as a plain string. |
| `400` | `No active payment gateways configured for this brand.` | No channels configured in the Dashboard. |
| `405` | `Method not allowed. Only POST requests are accepted.` | A non-POST method was used. |

---

## Step 2 — Displaying Active Payment Methods

**Render only what the API returns.** For MFS channels, only show a number whose `active_payments` flag is `true`. For Binance, only render the card if `active_payments.personal` is `true` and a `personal` UID is present — and always show the exact `amount_usdt` value from the response, never a value you calculate yourself.

```javascript
function spRenderMethods(methods, brand) {
  const container = document.getElementById('sp-methods-list');
  container.innerHTML = '';

  const icons  = { bkash: '📱', nagad: '📲', rocket: '🚀', upay: '💳', binance: '🟡' };
  const labels = { bkash: 'bKash', nagad: 'Nagad', rocket: 'Rocket', upay: 'Upay', binance: 'Binance Pay' };

  let hasAny = false;

  methods.forEach(function (m) {
    const active = m.active_payments || {};

    // ── Binance is structured differently from MFS channels ──
    if (m.name === 'binance') {
      if (!active.personal || !m.personal) return; // not active, skip
      hasAny = true;

      const card = document.createElement('div');
      card.className = 'sp-method-card';
      card.dataset.name = 'binance';
      card.innerHTML = `
        <div class="sp-method-icon">${icons.binance}</div>
        <div class="sp-method-info">
          <div class="sp-method-name">${labels.binance}</div>
          <div class="sp-method-type">Send exactly ${m.amount_usdt} USDT</div>
          <div class="sp-method-number">UID: ${m.personal}</div>
        </div>
        <button class="sp-copy-btn" onclick="spCopyNumber(event,'${m.personal}')">Copy</button>
      `;
      card.addEventListener('click', function (e) {
        if (e.target.classList.contains('sp-copy-btn')) return;
        spSelectMethod('binance', m.personal, card, m.amount_usdt);
      });
      container.appendChild(card);
      return;
    }

    // ── Standard MFS channels: bKash, Nagad, Rocket, Upay ──
    let number = null, type = null;
    if (active.personal && m.personal)      { number = m.personal; type = 'Send Money'; }
    else if (active.agent && m.agent)       { number = m.agent;    type = 'Cash In (Agent)'; }
    else if (active.payment && m.payment)   { number = m.payment;  type = 'Merchant Pay'; }
    if (!number) return; // no active number for this channel, skip

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
      spSelectMethod(m.name, number, card);
    });
    container.appendChild(card);
  });

  if (!hasAny) {
    container.innerHTML = '<p class="sp-no-methods">No payment methods are currently active. Please try again later or contact support.</p>';
  }

  const amountDisplay = document.getElementById('sp-amount-display');
  if (amountDisplay) amountDisplay.textContent = 'BDT ' + spState.amount.toFixed(2);

  if (brand) {
    const brandEl = document.getElementById('sp-brand-support');
    if (brandEl) brandEl.innerHTML = `Support: <a href="https://wa.me/${brand.mobile}" target="_blank">${brand.name}</a>`;
  }
}

function spSelectMethod(methodName, number, cardEl, amountUsdt) {
  document.querySelectorAll('.sp-method-card').forEach(c => c.classList.remove('sp-method-selected'));
  if (cardEl) cardEl.classList.add('sp-method-selected');

  spState.selectedMethod = methodName;   // always lowercase, comes straight from the API
  spState.selectedNumber = number;
  spState.amountUsdt     = amountUsdt || null;

  const confirmBtn = document.getElementById('sp-confirm-paid-btn');
  if (confirmBtn) {
    confirmBtn.style.display = 'block';
    confirmBtn.textContent   = `✅ I Have Paid via ${methodName.charAt(0).toUpperCase() + methodName.slice(1)}`;
  }
}

function spConfirmPaid() {
  if (!spState.selectedMethod) { alert('Please select a payment method first.'); return; }

  // Swap the Step 3 label/placeholder depending on channel
  const label = document.getElementById('sp-trxid-label');
  const input = document.getElementById('sp-trxid');
  const note  = document.getElementById('sp-sync-note');

  if (spState.selectedMethod === 'binance') {
    if (label) label.textContent = 'Binance Order ID';
    if (input) input.placeholder = 'e.g. 443903031407804416';
    if (note)  note.textContent  = '⚡ Binance Pay verifies instantly — no waiting required.';
  } else {
    if (label) label.textContent = 'SMS Transaction ID (TrxID)';
    if (input) input.placeholder = 'e.g. BLA38KDK2M';
    if (note)  note.textContent  = '⏱️ If you just paid, verification may take 5–20 seconds. Click Retry if needed.';
  }

  spShowStep('verify');
}

async function spCopyNumber(event, text) {
  event.stopPropagation();
  try { await navigator.clipboard.writeText(text); } catch (e) { /* older browsers: ignore */ }
  const btn = event.target;
  const orig = btn.textContent;
  btn.textContent = '✓ Copied!'; btn.style.background = '#16a34a'; btn.style.color = '#fff';
  setTimeout(() => { btn.textContent = orig; btn.style.background = ''; btn.style.color = ''; }, 2000);
}
```

### HTML

```html
<div id="sp-step-methods" class="sp-card" style="display:none;">
  <div class="sp-header">
    <h2>Select Payment Method</h2>
    <p>Payable: <strong id="sp-amount-display">BDT —</strong> — send the exact amount shown on your chosen method</p>
  </div>
  <div class="sp-instruction-box">
    <ol class="sp-instruction-list">
      <li>Click a payment method below to select it</li>
      <li>Copy the number / UID and send the <strong>exact amount</strong> shown via your app</li>
      <li>After paying, click "I Have Paid"</li>
    </ol>
  </div>
  <div id="sp-methods-list" class="sp-methods-list"></div>
  <button id="sp-confirm-paid-btn" class="sp-btn-success" onclick="spConfirmPaid()" style="display:none;">✅ I Have Paid</button>
  <p id="sp-brand-support" class="sp-support-text"></p>
  <button class="sp-btn-back" onclick="spShowStep('input')">← Back</button>
</div>
```

---

## Step 3 — TrxID / Order ID Input & Verification

### HTML

```html
<div id="sp-step-verify" class="sp-card" style="display:none;">
  <div class="sp-header">
    <h2>Confirm Your Payment</h2>
    <p>Enter the ID from your payment confirmation to verify</p>
  </div>
  <div class="sp-form">
    <div class="sp-field">
      <label for="sp-trxid" id="sp-trxid-label">SMS Transaction ID (TrxID)</label>
      <input type="text" id="sp-trxid" placeholder="e.g. BLA38KDK2M"
             autocomplete="off" autocorrect="off" autocapitalize="characters" />
    </div>
    <div id="sp-verify-error" class="sp-error" style="display:none;"></div>
    <button id="sp-verify-btn" class="sp-btn-primary" onclick="spVerifyPayment()">🔍 Verify Payment</button>
    <button id="sp-retry-btn" class="sp-btn-retry" onclick="spVerifyPayment()" style="display:none;">🔄 Retry Verification</button>
    <p id="sp-sync-note" class="sp-sms-note">⏱️ If you just paid, verification may take 5–20 seconds. Click Retry if needed.</p>
    <div id="sp-contact-support">
      Need help? Contact us on
      <a href="https://wa.me/+8801761844968" target="_blank">WhatsApp</a> or
      <a href="https://t.me/BD_Prime_Minister" target="_blank">Telegram</a>
    </div>
    <button class="sp-btn-back" onclick="spShowStep('methods')">← Change Payment Method</button>
  </div>
</div>
```

### JavaScript — Verify with Correct Field Naming

> **🚨 Critical fix vs. older integrations:** the identifier field name differs by channel. For **bKash / Nagad / Rocket / Upay**, send it as `transaction_id`. For **Binance**, the recommended field is `order_id` (`transaction_id` is also accepted as an alias, but `order_id` is clearer since Binance Pay itself returns an "Order ID"). The code below picks the correct field automatically based on `spState.selectedMethod`.

```javascript
async function spVerifyPayment() {
  const value  = document.getElementById('sp-trxid')?.value?.trim();
  const errDiv = document.getElementById('sp-verify-error');

  if (!value)                    { spShowError(errDiv, 'Please enter your Transaction ID / Order ID.'); return; }
  if (!spState.sessionId)        { spShowError(errDiv, 'Session expired. Please go back and start a new payment.'); return; }
  if (!spState.selectedMethod)   { spShowError(errDiv, 'No payment method selected. Please go back and select one.'); return; }
  spHideError(errDiv);

  const vBtn = document.getElementById('sp-verify-btn');
  const rBtn = document.getElementById('sp-retry-btn');
  if (vBtn) { vBtn.disabled = true; vBtn.textContent = '🔍 Verifying...'; }
  if (rBtn) { rBtn.disabled = true; rBtn.textContent = '🔄 Retrying...'; }

  const isBinance = spState.selectedMethod === 'binance';
  const payload = {
    id:     spState.sessionId,
    method: spState.selectedMethod,        // strict lowercase — always true here, comes from the API response
  };
  if (isBinance) {
    payload.order_id = value.trim();       // recommended field name for Binance
  } else {
    payload.transaction_id = value.trim().toUpperCase(); // MFS TrxIDs are typically uppercase alphanumeric
  }

  try {
    const result = await spCallAPI('/api/v2/payment/verify', payload);

    if (result?.status === true) {
      spState.retryCount = 0;
      spOnVerified(result);
    } else {
      spState.retryCount++;
      if (vBtn) { vBtn.disabled = false; vBtn.textContent = '🔍 Verify Payment'; }

      const msg = result?.message || 'Verification failed. Please check your ID and try again.';

      if (isBinance) {
        // Binance failures are usually definitive (wrong ID, wrong UID, insufficient amount,
        // already used) rather than a "still syncing" case — show the exact message,
        // but still allow one retry in case of a transient 502 from Binance's side.
        if (spState.retryCount < 2) {
          spShowError(errDiv, msg + ' If you believe this is temporary, you may retry once.');
          if (rBtn) { rBtn.style.display = 'block'; rBtn.disabled = false; rBtn.textContent = '🔄 Retry'; }
        } else {
          spShowError(errDiv, msg);
          if (rBtn) rBtn.style.display = 'none';
          const cs = document.getElementById('sp-contact-support');
          if (cs) cs.style.display = 'block';
        }
      } else {
        const remaining = spState.maxRetries - spState.retryCount;
        if (remaining > 0) {
          spShowError(errDiv,
            `${msg} The SMS may still be syncing (5–20 seconds). Please wait and click Retry. ` +
            `(${remaining} attempt${remaining !== 1 ? 's' : ''} remaining)`
          );
          if (rBtn) { rBtn.style.display = 'block'; rBtn.disabled = false; rBtn.textContent = '🔄 Retry Verification'; }
        } else {
          spShowError(errDiv, `Verification failed after ${spState.maxRetries} attempts. ${msg} Please contact support with your TrxID.`);
          if (rBtn) rBtn.style.display = 'none';
          const cs = document.getElementById('sp-contact-support');
          if (cs) cs.style.display = 'block';
        }
      }
    }
  } catch (err) {
    if (vBtn) { vBtn.disabled = false; vBtn.textContent = '🔍 Verify Payment'; }
    if (rBtn) { rBtn.disabled = false; rBtn.textContent = '🔄 Retry'; }
    spShowError(errDiv, 'Network error during verification. Please check your connection and try again.');
    console.error('[SkyPay Headless] Verify error:', err);
  }
}
```

**Verify — Error Responses**

| HTTP Code | Message | Cause |
|---|---|---|
| `400` | `id, method and transaction_id (or order_id) fields are required.` | A required field is missing. |
| `400` | `Unsupported payment method supplied.` | `method` not one of the five valid values, or not lowercase. |
| `400` | `This payment session has already been completed.` | Session already verified previously. |
| `400` | `Transaction not found. Please check Order ID.` | ID doesn't match any incoming SMS/Binance record yet. |
| `400` | `Receiver UID does not match.` | *(Binance)* Sent to a different UID than configured. |
| `400` | `Only USDT payments are accepted.` | *(Binance)* A currency other than USDT was sent. |
| `400` | `Insufficient amount received. Expected X USDT but received Y USDT.` | *(Binance)* Sent less than `amount_usdt` (tolerance ±0.0005). |
| `400` | `This Order ID has already been used.` | *(Binance)* Duplicate Order ID submission. |
| `401` | `BRAND-KEY header is required.` / `Invalid or inactive BRAND-KEY provided.` | Auth issue — check your key. |
| `403` | `No active SMS sync device found for this account.` | *(MFS)* Android device offline. |
| `404` | `Payment session not found or expired.` | Session `id` invalid — call `/create` again. |
| `502` | `Failed to communicate with Binance. Please try again.` | *(Binance)* Temporary outage on Binance's side. |

---

## Step 4 — Fulfilling the Order

> **🚨 Fixed from older versions of this guide:** the verify success response wraps all payment details inside a `data` object and includes a top-level `message` string — it is **not** a flat object with `amount`/`cus_name`/`id` at the top level. Any older integration reading `result.amount` or `result.id` directly must be updated to read `result.data.amount`, `result.data.cus_name`, and so on, as shown below.

**Correct success response shape:**

```json
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Full Name",
    "cus_email": "headless@skypaybd.top",
    "amount": 500,
    "transaction_id": "BLA38KDK2M",
    "meta_data": { "source": "html_headless_integration" },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

> `data.cus_email` is always the fixed placeholder `"headless@skypaybd.top"` for Headless v2 sessions — this is expected, not an error. `data.status` is always `"COMPLETED"` on success.

```javascript
function spOnVerified(result) {
  spShowStep('success');
  const payload = result.data || {};

  const msgEl = document.getElementById('sp-success-message');
  if (msgEl) {
    msgEl.textContent = result.message ||
      `Your payment of BDT ${payload.amount ?? spState.amount} via ${spCapitalize(payload.payment_method || spState.selectedMethod)} has been confirmed!`;
  }

  const detailsEl = document.getElementById('sp-success-details');
  if (detailsEl) {
    detailsEl.innerHTML = `
      <table class="sp-details-table">
        <tr><td>Customer</td>              <td><strong>${payload.cus_name || '—'}</strong></td></tr>
        <tr><td>Amount</td>                <td><strong>BDT ${payload.amount ?? spState.amount}</strong></td></tr>
        <tr><td>Payment Method</td>        <td><strong>${spCapitalize(payload.payment_method || spState.selectedMethod)}</strong></td></tr>
        <tr><td>Transaction / Order ID</td><td><strong>${payload.transaction_id || '—'}</strong></td></tr>
        <tr><td>Status</td>                <td><strong class="sp-status-ok">✅ ${payload.status || 'COMPLETED'}</strong></td></tr>
      </table>
    `;
  }

  // ─────────────────────────────────────────────────────
  // YOUR FULFILLMENT LOGIC GOES HERE — always call your OWN backend.
  // Never write directly to your database from this client-side script.
  // ─────────────────────────────────────────────────────
  //
  // fetch('/api/user/add-balance', {
  //   method: 'POST',
  //   headers: { 'Content-Type': 'application/json' },
  //   body: JSON.stringify({
  //     amount:         payload.amount,
  //     method:         payload.payment_method,
  //     transaction_id: payload.transaction_id,
  //     session_id:     spState.sessionId,
  //   }),
  // });

  console.log('[SkyPay Headless] Verified OK:', result);
}
```

---

## ⏱️ Retry Logic — MFS vs Binance

**MFS (bKash / Nagad / Rocket / Upay):** the telecom SMS bridge takes **5–20 seconds**. If the customer submits their TrxID immediately, `/verify` may briefly return `false` even though the payment is real — this is normal. Allow retries.

```
User submits TrxID → /verify
       │
  ┌────┴────┐
status:true  status:false
   │              │
   ▼              ▼
Fulfill      retryCount++ (max 3)
             remaining > 0 → show Retry button (wait 10s)
             remaining = 0 → show permanent error + support contact
```

**Binance Pay:** verification is **instant** — there is no SMS bridge and normally no need to "wait and retry." A failure here (wrong Order ID, wrong UID, insufficient USDT, already-used Order ID) is usually a real, permanent error and should be shown as-is. The one exception is a `502` from Binance's own service being briefly unreachable — allow at most one retry for that case, as shown in the Step 3 code above.

Auto-countdown helper (optional, works for either channel):

```javascript
function spStartRetryCountdown(seconds) {
  const retryBtn = document.getElementById('sp-retry-btn');
  if (!retryBtn) return;
  retryBtn.disabled = true;
  let count = seconds;
  retryBtn.textContent = `Retry available in ${count}s...`;
  const interval = setInterval(function () {
    count--;
    retryBtn.textContent = `Retry available in ${count}s...`;
    if (count <= 0) {
      clearInterval(interval);
      retryBtn.disabled = false;
      retryBtn.textContent = '🔄 Retry Verification';
    }
  }, 1000);
}
```

---

## 🖥️ Full Working Example — Complete HTML Page

Save as `payment.html`. This standalone page implements the full flow above, including Binance Pay, with the corrected response parsing.

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
      background: #f0f4f8; color: #1e293b; min-height: 100vh;
      display: flex; align-items: center; justify-content: center; padding: 20px;
    }
    .sp-container { width: 100%; max-width: 500px; }
    .sp-card { background: #fff; border-radius: 16px; box-shadow: 0 4px 24px rgba(0,0,0,0.10); padding: 28px 24px; }
    .sp-header { text-align: center; margin-bottom: 24px; }
    .sp-header h2 { font-size: 21px; font-weight: 700; margin-bottom: 6px; }
    .sp-header p { font-size: 14px; color: #64748b; }
    .sp-header strong { color: #2563eb; }
    .sp-form { display: flex; flex-direction: column; gap: 16px; }
    .sp-field { display: flex; flex-direction: column; gap: 5px; }
    .sp-field label { font-size: 13px; font-weight: 600; color: #374151; }
    .sp-field input {
      padding: 11px 13px; border: 1.5px solid #d1d5db; border-radius: 8px;
      font-size: 15px; color: #1e293b; background: #f8fafc; outline: none; transition: border-color 0.2s;
    }
    .sp-field input:focus { border-color: #2563eb; background: #fff; }
    .sp-btn-primary {
      width: 100%; padding: 13px; background: linear-gradient(135deg, #2563eb, #1d4ed8);
      color: #fff; border: none; border-radius: 10px; font-size: 15px; font-weight: 700; cursor: pointer;
    }
    .sp-btn-primary:disabled { opacity: 0.6; cursor: not-allowed; }
    .sp-btn-success {
      width: 100%; padding: 13px; background: linear-gradient(135deg, #16a34a, #15803d);
      color: #fff; border: none; border-radius: 10px; font-size: 15px; font-weight: 700; cursor: pointer; margin-top: 8px;
    }
    .sp-btn-retry {
      width: 100%; padding: 11px; background: #f59e0b; color: #fff; border: none;
      border-radius: 10px; font-size: 14px; font-weight: 600; cursor: pointer; margin-top: 4px;
    }
    .sp-btn-retry:disabled { opacity: 0.6; cursor: not-allowed; }
    .sp-btn-secondary {
      width: 100%; padding: 11px; background: #f1f5f9; color: #374151;
      border: 1.5px solid #d1d5db; border-radius: 10px; font-size: 14px; font-weight: 600; cursor: pointer; margin-top: 10px;
    }
    .sp-btn-back { background: none; border: none; color: #6b7280; font-size: 13px; cursor: pointer; margin-top: 8px; text-decoration: underline; }
    .sp-error { background: #fef2f2; border: 1px solid #fecaca; color: #dc2626; padding: 10px 13px; border-radius: 8px; font-size: 13px; line-height: 1.5; }
    .sp-loading { text-align: center; padding: 24px 0; }
    .sp-spinner { width: 42px; height: 42px; border: 4px solid #e2e8f0; border-top-color: #2563eb; border-radius: 50%; animation: spSpin 0.8s linear infinite; margin: 0 auto 14px; }
    @keyframes spSpin { to { transform: rotate(360deg); } }
    .sp-loading p { font-size: 15px; font-weight: 600; }
    .sp-loading small { font-size: 12px; color: #64748b; }
    .sp-instruction-box { background: #eff6ff; border: 1px solid #bfdbfe; border-radius: 10px; padding: 14px 16px; margin-bottom: 16px; }
    .sp-instruction-list { padding-left: 18px; }
    .sp-instruction-list li { font-size: 13px; color: #1d4ed8; margin-bottom: 5px; line-height: 1.5; }
    .sp-methods-list { display: flex; flex-direction: column; gap: 10px; margin-bottom: 12px; }
    .sp-method-card { display: flex; align-items: center; gap: 12px; border: 2px solid #e2e8f0; border-radius: 12px; padding: 14px 16px; cursor: pointer; transition: border-color 0.2s, background 0.2s; background: #fff; }
    .sp-method-card:hover { border-color: #93c5fd; background: #f0f9ff; }
    .sp-method-selected { border-color: #2563eb !important; background: #eff6ff !important; }
    .sp-method-icon { font-size: 28px; flex-shrink: 0; }
    .sp-method-info { flex: 1; min-width: 0; }
    .sp-method-name { font-size: 15px; font-weight: 700; }
    .sp-method-type { font-size: 11px; color: #64748b; margin: 1px 0 3px; }
    .sp-method-number { font-size: 14px; font-weight: 600; color: #2563eb; letter-spacing: 0.5px; }
    .sp-copy-btn { flex-shrink: 0; padding: 6px 12px; background: #f1f5f9; border: 1px solid #d1d5db; border-radius: 6px; font-size: 12px; font-weight: 600; color: #374151; cursor: pointer; }
    .sp-copy-btn:hover { background: #e2e8f0; }
    .sp-no-methods { color: #dc2626; font-size: 14px; text-align: center; padding: 20px 0; }
    .sp-support-text { font-size: 12px; color: #94a3b8; text-align: center; margin-top: 8px; }
    .sp-support-text a { color: #2563eb; text-decoration: none; }
    .sp-sms-note { font-size: 12px; color: #64748b; text-align: center; margin-top: 4px; }
    .sp-result { text-align: center; padding: 6px 0; }
    .sp-result-icon { font-size: 50px; margin-bottom: 12px; }
    .sp-result h2 { font-size: 21px; font-weight: 700; margin-bottom: 10px; color: #16a34a; }
    .sp-result p { font-size: 14px; color: #475569; margin-bottom: 16px; line-height: 1.6; }
    .sp-details-box { background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 10px; padding: 14px 16px; margin-bottom: 16px; text-align: left; }
    .sp-details-table { width: 100%; border-collapse: collapse; font-size: 13px; }
    .sp-details-table td { padding: 6px 4px; color: #475569; vertical-align: top; }
    .sp-details-table td:first-child { color: #94a3b8; width: 45%; }
    .sp-details-table td strong { color: #1e293b; }
    .sp-status-ok { color: #16a34a !important; }
    .sp-powered { text-align: center; margin-top: 14px; font-size: 12px; color: #94a3b8; }
    .sp-powered a { color: #2563eb; text-decoration: none; }
    #sp-contact-support { display: none; background: #fff7ed; border: 1px solid #fed7aa; border-radius: 8px; padding: 10px 14px; font-size: 13px; color: #9a3412; text-align: center; margin-top: 6px; }
    #sp-contact-support a { color: #c2410c; font-weight: 600; }
  </style>
</head>
<body>

<div class="sp-container">

  <div id="sp-step-input" class="sp-card">
    <div class="sp-header">
      <h2>💳 Add Balance</h2>
      <p>Pay with bKash, Nagad, Rocket, Upay, or Binance Pay</p>
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

  <div id="sp-step-loading" class="sp-card" style="display:none;">
    <div class="sp-loading">
      <div class="sp-spinner"></div>
      <p id="sp-loading-text">Setting up your payment session...</p>
      <small>Please wait, do not close this page.</small>
    </div>
  </div>

  <div id="sp-step-methods" class="sp-card" style="display:none;">
    <div class="sp-header">
      <h2>Select Payment Method</h2>
      <p>Payable: <strong id="sp-amount-display">—</strong></p>
    </div>
    <div class="sp-instruction-box">
      <ol class="sp-instruction-list">
        <li>Click a channel to select it</li>
        <li>Copy the number / UID and send the exact amount shown</li>
        <li>After paying, click "I Have Paid"</li>
      </ol>
    </div>
    <div id="sp-methods-list" class="sp-methods-list"></div>
    <button id="sp-confirm-paid-btn" class="sp-btn-success" onclick="spConfirmPaid()" style="display:none;">✅ I Have Paid</button>
    <p id="sp-brand-support" class="sp-support-text"></p>
    <button class="sp-btn-back" onclick="spShowStep('input')">← Back</button>
  </div>

  <div id="sp-step-verify" class="sp-card" style="display:none;">
    <div class="sp-header">
      <h2>Confirm Your Payment</h2>
      <p>Enter the ID from your payment confirmation</p>
    </div>
    <div class="sp-form">
      <div class="sp-field">
        <label for="sp-trxid" id="sp-trxid-label">SMS Transaction ID (TrxID)</label>
        <input type="text" id="sp-trxid" placeholder="e.g. BLA38KDK2M" autocomplete="off" autocorrect="off" autocapitalize="characters" />
      </div>
      <div id="sp-verify-error" class="sp-error" style="display:none;"></div>
      <button id="sp-verify-btn" class="sp-btn-primary" onclick="spVerifyPayment()">🔍 Verify Payment</button>
      <button id="sp-retry-btn" class="sp-btn-retry" onclick="spVerifyPayment()" style="display:none;">🔄 Retry Verification</button>
      <p id="sp-sync-note" class="sp-sms-note">⏱️ If you just paid, verification may take 5–20 seconds. Click Retry if needed.</p>
      <div id="sp-contact-support">
        Need help? Contact us on
        <a href="https://wa.me/+8801761844968" target="_blank">WhatsApp</a> or
        <a href="https://t.me/BD_Prime_Minister" target="_blank">Telegram</a>
      </div>
      <button class="sp-btn-back" onclick="spShowStep('methods')">← Change Method</button>
    </div>
  </div>

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
// SKYPAY HEADLESS API v2 — VANILLA JS IMPLEMENTATION (bKash / Nagad /
// Rocket / Upay / Binance) — response parsing matches the CURRENT API.
// =====================================================================

const SP_CONFIG = {
  BRAND_KEY: 'YOUR_BRAND_KEY_HERE',  // ⚠️ See the Security Notice — use a backend proxy for production
  PROXY_URL: '',                     // Set to your backend endpoint for production, e.g. '/skypay-proxy'
  API_BASE:  'https://core.skypaybd.top',
};

const spState = {
  sessionId: null, selectedMethod: null, selectedNumber: null,
  amount: null, amountUsdt: null, retryCount: 0, maxRetries: 3,
};

async function spCallAPI(path, payload) {
  const url     = SP_CONFIG.PROXY_URL ? (SP_CONFIG.PROXY_URL + path) : (SP_CONFIG.API_BASE + path);
  const headers = SP_CONFIG.PROXY_URL
    ? { 'Content-Type': 'application/json' }
    : { 'Content-Type': 'application/json', 'BRAND-KEY': SP_CONFIG.BRAND_KEY };
  const res = await fetch(url, { method: 'POST', headers, body: JSON.stringify(payload) });
  return await res.json();
}

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
      cus_name: name, amount: spState.amount,
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

function spRenderMethods(methods, brand) {
  const container = document.getElementById('sp-methods-list');
  container.innerHTML = '';
  const icons  = { bkash: '📱', nagad: '📲', rocket: '🚀', upay: '💳', binance: '🟡' };
  const labels = { bkash: 'bKash', nagad: 'Nagad', rocket: 'Rocket', upay: 'Upay', binance: 'Binance Pay' };
  let hasAny = false;

  methods.forEach(function (m) {
    const active = m.active_payments || {};

    if (m.name === 'binance') {
      if (!active.personal || !m.personal) return;
      hasAny = true;
      const card = document.createElement('div');
      card.className = 'sp-method-card';
      card.dataset.name = 'binance';
      card.innerHTML = `
        <div class="sp-method-icon">${icons.binance}</div>
        <div class="sp-method-info">
          <div class="sp-method-name">${labels.binance}</div>
          <div class="sp-method-type">Send exactly ${m.amount_usdt} USDT</div>
          <div class="sp-method-number">UID: ${m.personal}</div>
        </div>
        <button class="sp-copy-btn" onclick="spCopyNumber(event,'${m.personal}')">Copy</button>
      `;
      card.addEventListener('click', function (e) {
        if (e.target.classList.contains('sp-copy-btn')) return;
        document.querySelectorAll('.sp-method-card').forEach(c => c.classList.remove('sp-method-selected'));
        card.classList.add('sp-method-selected');
        spState.selectedMethod = 'binance';
        spState.selectedNumber = m.personal;
        spState.amountUsdt = m.amount_usdt;
        const btn = document.getElementById('sp-confirm-paid-btn');
        if (btn) { btn.style.display = 'block'; btn.textContent = '✅ I Have Paid via Binance Pay'; }
      });
      container.appendChild(card);
      return;
    }

    let number = null, type = null;
    if (active.personal && m.personal)    { number = m.personal; type = 'Send Money'; }
    else if (active.agent && m.agent)     { number = m.agent;    type = 'Cash In (Agent)'; }
    else if (active.payment && m.payment) { number = m.payment;  type = 'Merchant Pay'; }
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
      spState.selectedNumber = number;
      spState.amountUsdt = null;
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

async function spCopyNumber(event, text) {
  event.stopPropagation();
  try { await navigator.clipboard.writeText(text); } catch (e) { }
  const btn = event.target;
  const orig = btn.textContent;
  btn.textContent = '✓ Copied!'; btn.style.background = '#16a34a'; btn.style.color = '#fff';
  setTimeout(() => { btn.textContent = orig; btn.style.background = ''; btn.style.color = ''; }, 2000);
}

function spConfirmPaid() {
  if (!spState.selectedMethod) { alert('Please select a payment method first.'); return; }
  const label = document.getElementById('sp-trxid-label');
  const input = document.getElementById('sp-trxid');
  const note  = document.getElementById('sp-sync-note');
  if (spState.selectedMethod === 'binance') {
    if (label) label.textContent = 'Binance Order ID';
    if (input) input.placeholder = 'e.g. 443903031407804416';
    if (note)  note.textContent  = '⚡ Binance Pay verifies instantly — no waiting required.';
  } else {
    if (label) label.textContent = 'SMS Transaction ID (TrxID)';
    if (input) input.placeholder = 'e.g. BLA38KDK2M';
    if (note)  note.textContent  = '⏱️ If you just paid, verification may take 5–20 seconds. Click Retry if needed.';
  }
  spShowStep('verify');
}

async function spVerifyPayment() {
  const value  = document.getElementById('sp-trxid')?.value?.trim();
  const errDiv = document.getElementById('sp-verify-error');

  if (!value) { spShowError(errDiv, 'Please enter your Transaction ID / Order ID.'); return; }
  if (!spState.sessionId) { spShowError(errDiv, 'Session expired. Please go back and start over.'); return; }
  if (!spState.selectedMethod) { spShowError(errDiv, 'No payment method selected. Please go back.'); return; }
  spHideError(errDiv);

  const vBtn = document.getElementById('sp-verify-btn');
  const rBtn = document.getElementById('sp-retry-btn');
  if (vBtn) { vBtn.disabled = true; vBtn.textContent = '🔍 Verifying...'; }
  if (rBtn) { rBtn.disabled = true; rBtn.textContent = '🔄 Retrying...'; }

  const isBinance = spState.selectedMethod === 'binance';
  const payload = { id: spState.sessionId, method: spState.selectedMethod };
  if (isBinance) { payload.order_id = value; }
  else           { payload.transaction_id = value.toUpperCase(); }

  try {
    const result = await spCallAPI('/api/v2/payment/verify', payload);

    if (result?.status === true) {
      spState.retryCount = 0;
      spOnVerified(result);
    } else {
      spState.retryCount++;
      if (vBtn) { vBtn.disabled = false; vBtn.textContent = '🔍 Verify Payment'; }
      const msg = result?.message || 'Verification failed. Please check your ID and try again.';

      if (isBinance) {
        if (spState.retryCount < 2) {
          spShowError(errDiv, msg + ' If you believe this is temporary, you may retry once.');
          if (rBtn) { rBtn.style.display = 'block'; rBtn.disabled = false; rBtn.textContent = '🔄 Retry'; }
        } else {
          spShowError(errDiv, msg);
          if (rBtn) rBtn.style.display = 'none';
          const cs = document.getElementById('sp-contact-support');
          if (cs) cs.style.display = 'block';
        }
      } else {
        const rem = spState.maxRetries - spState.retryCount;
        if (rem > 0) {
          spShowError(errDiv, `${msg} SMS may still be syncing (5–20 sec). Wait and click Retry. (${rem} attempt${rem !== 1 ? 's' : ''} left)`);
          if (rBtn) { rBtn.style.display = 'block'; rBtn.disabled = false; rBtn.textContent = '🔄 Retry Verification'; }
        } else {
          spShowError(errDiv, `Verification failed after ${spState.maxRetries} attempts. ${msg}`);
          if (rBtn) rBtn.style.display = 'none';
          const cs = document.getElementById('sp-contact-support');
          if (cs) cs.style.display = 'block';
        }
      }
    }
  } catch (err) {
    if (vBtn) { vBtn.disabled = false; vBtn.textContent = '🔍 Verify Payment'; }
    if (rBtn) { rBtn.disabled = false; rBtn.textContent = '🔄 Retry'; }
    spShowError(errDiv, 'Network error. Please check your connection.');
    console.error('[SkyPay] Verify error:', err);
  }
}

function spOnVerified(result) {
  spShowStep('success');
  const payload = result.data || {};

  const msg = document.getElementById('sp-success-message');
  if (msg) msg.textContent = result.message || `BDT ${payload.amount ?? spState.amount} via ${spCapitalize(payload.payment_method || spState.selectedMethod)} — confirmed!`;

  const det = document.getElementById('sp-success-details');
  if (det) {
    det.innerHTML = `
      <table class="sp-details-table">
        <tr><td>Customer</td>              <td><strong>${payload.cus_name || '—'}</strong></td></tr>
        <tr><td>Amount</td>                <td><strong>BDT ${payload.amount ?? spState.amount}</strong></td></tr>
        <tr><td>Method</td>                <td><strong>${spCapitalize(payload.payment_method || spState.selectedMethod)}</strong></td></tr>
        <tr><td>Transaction / Order ID</td><td><strong>${payload.transaction_id || '—'}</strong></td></tr>
        <tr><td>Status</td>                <td><strong class="sp-status-ok">✅ ${payload.status || 'COMPLETED'}</strong></td></tr>
      </table>
    `;
  }

  // ── YOUR FULFILLMENT LOGIC — call your own backend, never write to a DB from here ──
  // fetch('/api/user/add-balance', {
  //   method: 'POST',
  //   headers: { 'Content-Type': 'application/json' },
  //   body: JSON.stringify({ amount: payload.amount, session_id: spState.sessionId, transaction_id: payload.transaction_id }),
  // });

  console.log('[SkyPay Headless] Verified OK:', result);
}

function spShowStep(step) {
  ['input', 'loading', 'methods', 'verify', 'success'].forEach(s => {
    const el = document.getElementById('sp-step-' + s);
    if (el) el.style.display = (s === step) ? 'block' : 'none';
  });
}
function spSetLoadingText(text) { const el = document.getElementById('sp-loading-text'); if (el) el.textContent = text; }
function spShowError(el, msg) { if (!el) return; el.textContent = msg; el.style.display = 'block'; }
function spHideError(el) { if (!el) return; el.textContent = ''; el.style.display = 'none'; }

function spReset() {
  spState.sessionId = null; spState.selectedMethod = null; spState.selectedNumber = null;
  spState.amount = null; spState.amountUsdt = null; spState.retryCount = 0;
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

function spCapitalize(str) { if (!str) return str; return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase(); }
</script>

</body>
</html>
```

---

## 🔧 Integrating Into an Existing HTML File

1. **Add CSS** — copy the `.sp-*` styles into your existing stylesheet. Class names are prefixed `sp-` to avoid conflicts.
2. **Add HTML** — copy the five `<div id="sp-step-*">` blocks to where the payment section should appear.
3. **Add JavaScript** — copy the full `<script>` block before your closing `</body>` tag.
4. **Pre-fill a logged-in user's name**, if applicable:
   ```javascript
   function onUserLoggedIn(user) {
     const nameField = document.getElementById('sp-name');
     if (nameField && user.name) { nameField.value = user.name; nameField.readOnly = true; }
   }
   ```
5. **Set a fixed amount for a specific product**, if applicable:
   ```javascript
   const amountField = document.getElementById('sp-amount');
   if (amountField) { amountField.value = 299; amountField.readOnly = true; }
   ```

---

## 🎨 CSS Styling Reference

| Class | Purpose |
|---|---|
| `.sp-container` | Outer wrapper (max-width 500px) |
| `.sp-card` | White card panel with shadow |
| `.sp-header` | Title and subtitle |
| `.sp-form` | Flex column form |
| `.sp-field` | Label + input wrapper |
| `.sp-btn-primary` | Primary blue action button |
| `.sp-btn-success` | Green "I Have Paid" button |
| `.sp-btn-retry` | Orange retry button |
| `.sp-btn-back` | Plain text back link |
| `.sp-error` | Red error message box |
| `.sp-methods-list` | Method cards container |
| `.sp-method-card` | Individual channel card (MFS or Binance) |
| `.sp-method-selected` | Highlighted selected card |
| `.sp-copy-btn` | Copy-to-clipboard button |
| `.sp-details-table` | Verification result table |
| `.sp-status-ok` | Green "COMPLETED" status text |

---

## 📖 API Reference Summary

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
  "meta_data": { "user_id": "..." }      ← optional
}

Response:
{
  "status": true,
  "id":     "a1b2c3d4e5f6g7h8",           ← SAVE THIS
  "brand":  { "name": "...", "mobile": "...", "whatsapp": "...", "email": "..." },
  "methods": [
    { "name": "bkash",   "active_payments": {...}, "personal": "01..." },
    { "name": "binance", "active_payments": {...}, "personal": "UID...",
      "currency": "USDT", "dollar_rate": "...", "amount_usdt": "..." }
  ]
}
```

### Verify Payment — bKash / Nagad / Rocket / Upay

```
POST https://core.skypaybd.top/api/v2/payment/verify

Body:
{
  "id":             "a1b2c3d4e5f6g7h8",
  "method":         "bkash",              ← strict lowercase
  "transaction_id": "BLA38KDK2M"
}

Success Response:
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Full Name",
    "cus_email": "headless@skypaybd.top",
    "amount": 500,
    "transaction_id": "BLA38KDK2M",
    "meta_data": { ... },
    "payment_method": "bkash",
    "status": "COMPLETED"
  }
}
```

### Verify Payment — Binance Pay

```
POST https://core.skypaybd.top/api/v2/payment/verify

Body:
{
  "id":       "a1b2c3d4e5f6g7h8",
  "method":   "binance",
  "order_id": "443903031407804416"        ← recommended field for Binance
}

Success Response:
{
  "status": true,
  "message": "Payment verified successfully.",
  "data": {
    "cus_name": "Full Name",
    "cus_email": "headless@skypaybd.top",
    "amount": 500,
    "transaction_id": "443903031407804416",
    "meta_data": { ... },
    "payment_method": "binance",
    "status": "COMPLETED"
  }
}
```

> **Fulfill only when the top-level `"status": true`.** Read all payment details from the nested `data` object, never from the response root.

---

## 📦 API Response Fields — methods[] Array

| Field | Type | Description |
|---|---|---|
| `methods[].name` | String | `bkash`, `nagad`, `rocket`, `upay`, or `binance` |
| `methods[].active_payments.personal` | Boolean | `personal` number/UID is active (Send Money) |
| `methods[].active_payments.agent` | Boolean | `agent` number active (Cash In) — MFS only |
| `methods[].active_payments.payment` | Boolean | `payment` number active (Merchant Pay) — bKash only |
| `methods[].active_payments.merchant` | Boolean | *(Binance)* reserved, always `false` |
| `methods[].personal` / `agent` / `payment` | String | Wallet numbers — empty `""` if inactive |
| `methods[].personal` *(Binance)* | String | Binance receiving UID |
| `methods[].currency` *(Binance)* | String | Always `"USDT"` |
| `methods[].dollar_rate` *(Binance)* | String/Number | BDT → USDT rate configured by the merchant |
| `methods[].amount_usdt` *(Binance)* | String/Number | **Exact** USDT to display — do not recalculate |

> **Rule:** Never show a number/UID if its `active_payments` flag is `false`, or if the field is an empty string `""`.

---

## ⚠️ Error Handling & Edge Cases

| Scenario | API Response | JS Handling |
|---|---|---|
| Invalid name or amount | Not called | Input validation blocks the API call |
| `BRAND-KEY` invalid | `401 Unauthorized` | Error shown in form |
| Android phone offline *(MFS only)* | `403 Forbidden` | Error caught and shown |
| Session `id` expired | `404 Not Found` | User told to restart |
| TrxID not yet in SMS log *(MFS)* | `400` + `status:false` | Retry button, up to 3 attempts |
| Order ID not found *(Binance)* | `400` + `status:false` | Shown directly, retry limited to 1 attempt |
| TrxID / Order ID already used | `400` | Error shown, no fulfillment |
| Wrong `method` casing | `400 Unsupported payment method` | JS always stores `method` as lowercase from the API |
| Insufficient USDT *(Binance)* | `400` | Exact expected amount shown in the error |
| Binance temporarily down | `502` | Treated as transient — single retry allowed |
| Network error | — | Caught, user told to retry |

---

## 🔒 Security Rules for Frontend Integrations

| Rule | Details |
|---|---|
| **Prefer a backend proxy** | See the Security Notice at the top — avoid shipping `BRAND-KEY` in public JS for production. |
| **Never write to your database from this script** | Fulfillment must always call your own backend endpoint. |
| **Always lowercase `method`** | `spState.selectedMethod` is set directly from `method.name` in the API response — never let the user type it freely. |
| **Use the correct identifier field per channel** | `transaction_id` for MFS, `order_id` for Binance (aliases exist, but this is the clearest pairing). |
| **Never modify the amount** | SkyPay validates the SMS/Binance amount against the session amount exactly — display it as returned. |
| **For Binance, always show the exact `amount_usdt`** | Never round or recalculate it yourself. |
| **Idempotency** | Once a TrxID/Order ID is verified, SkyPay marks it claimed — your backend should also guard against duplicate fulfillment. |
| **Read the response's nested `data` object** | Do not read `amount`/`cus_name`/`payment_method` from the response root — they live inside `data`. |

---

## ✅ Integration Checklist

- [ ] `BRAND_KEY` ideally sits behind a backend proxy — not in public JavaScript (see Security Notice)
- [ ] If using a proxy, both `/api/v2/payment/create` and `/api/v2/payment/verify` are routed through it
- [ ] Session `id` from `/create` is saved immediately into `spState.sessionId`
- [ ] Only channels with an active flag and a non-empty number/UID are rendered as cards
- [ ] Binance card shows the exact `amount_usdt` and receiving UID from the response
- [ ] `selectedMethod` is always strict lowercase
- [ ] Verify payload uses `transaction_id` for MFS and `order_id` for Binance
- [ ] Response parsing reads `result.data.*` and `result.message` — not root-level fields
- [ ] Retry logic: up to 3 attempts for MFS, a single retry for a transient Binance `502`
- [ ] Fulfillment only runs after `result.status === true`, and only via your backend
- [ ] Backend enforces idempotency on `transaction_id` to block double fulfillment
- [ ] Merchant Android phone is on, connected, and the SkyPay APK is running (MFS only)
- [ ] Battery optimization disabled for the SkyPay APK on the merchant phone

---

## 📱 Android Merchant Sync App Setup

*(Required for bKash, Nagad, Rocket, Upay — not required for Binance Pay)*

<div align="center">

[![Download SkyPay APK](https://img.shields.io/badge/⬇%20Download%20SkyPay%20Merchant%20App-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://skypaybd.top/public/assets/downloads/SkyPay.apk)

</div>

### Requirements
- Android 7.0 (Nougat) or higher
- Active merchant SIM cards (bKash / Nagad / Rocket / Upay) inserted in the phone
- Phone kept plugged into power 24/7
- Battery optimization **disabled** for the SkyPay app
- Uninterrupted WiFi or mobile data

### Setup Steps

**Step 1 — Download & Install:** download `SkyPay.apk` and install it. Since it is sideloaded, enable **"Install from Unknown Sources"** if prompted.

**Step 2 — Create a Device:**
1. Log in to [Device Management](https://skypaybd.top/user/devices)
2. Ensure an active subscription ([Subscription Plans](https://skypaybd.top/user/plans))
3. Click **Create Device**, name it, and save
4. Copy the **Device Key**

**Step 3 — Log In to the App:**

| Field | What to Enter |
|---|---|
| **Email** | Your registered email on skypaybd.top |
| **Device Key** | The Device Key from Device Management |

Enable **"Remember My Device"** before logging in.

> ⚠️ **IP Lock:** Each Device Key is locked to the IP of the first device that logs in with it. If blocked, **delete the device** and create a new one.

**Step 4 — Grant SMS Permissions:** tap **Grant** on the yellow permission banner. If it doesn't take effect, long-press the app icon → **App Info** → **Permissions** → enable **SMS**.

**Step 5 — Keep the Service Running:** keep the sync toggle **ON** at all times — a paused service means MFS payments cannot be verified.

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
| ⬇️ Merchant Sync APK | [skypaybd.top/public/assets/downloads/SkyPay.apk](https://skypaybd.top/public/assets/downloads/SkyPay.apk) |
| 📄 Privacy Policy | [skypaybd.top/legal#privacy-policy](https://skypaybd.top/legal#privacy-policy) |
| 📋 Terms of Service | [skypaybd.top/legal#terms](https://skypaybd.top/legal#terms) |
| 💰 Refund Policy | [skypaybd.top/legal#refund-policy](https://skypaybd.top/legal#refund-policy) |
| 💵 Pricing | [skypaybd.top/#pricing](https://skypaybd.top/#pricing) |
| ❓ FAQ | [skypaybd.top/#faq](https://skypaybd.top/#faq) |

---

<div align="center">

*© 2024–2026 SkyPay BD. All rights reserved.*

**SkyPay Headless API v2 — HTML / CSS / Vanilla JavaScript Integration**

</div>
