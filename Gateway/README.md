# SkyPay SMS Gateway API — Complete Guide

![SkyPay Logo](https://skypaybd.top/public/uploads/admin/356a192b7913b04c54574d18c28d46e6395428ab/1789095642_d2193dfe3264f3a5ec9c.png)

**Send SMS programmatically through a connected Android device and its SIM using the SkyPay SMS Gateway API.**

[![API Core](https://img.shields.io/badge/API%20Core-core.skypaybd.top-0f172a?style=for-the-badge&logo=serverfault&logoColor=white)](https://core.skypaybd.top)
[![Online Docs](https://img.shields.io/badge/Online%20Docs-skypaybd.top%2Fdocs-7c3aed?style=for-the-badge&logo=gitbook&logoColor=white)](https://skypaybd.top/docs)

---

## 📋 Table of Contents

- [What the SMS Gateway Does](#-what-the-sms-gateway-does)
- [Requirements](#-requirements)
- [Setup Workflow](#-setup-workflow)
  - [1. Activate a Plan](#1-activate-a-plan)
  - [2. Create a Device](#2-create-a-device)
  - [3. Install and Configure the Android App](#3-install-and-configure-the-android-app)
  - [4. Create an SMS Gateway](#4-create-an-sms-gateway)
  - [5. Use the Gateway Key](#5-use-the-gateway-key)
- [Gateway Management](#-gateway-management)
- [How the SMS Gateway Works](#-how-the-sms-gateway-works)
- [API Reference](#-api-reference)
  - [Base URL](#-base-url)
  - [Authentication](#-authentication)
  - [Send SMS](#-send-sms)
  - [Request Parameters](#-request-parameters)
  - [Success Response](#-success-response)
  - [Error Responses](#-error-responses)
- [Implementation Examples](#-implementation-examples)
- [Message Length & Multipart SMS](#-message-length--multipart-sms)
- [Internet Requirement](#-internet-requirement)
- [Complete Flow](#-complete-flow)
- [Troubleshooting](#-troubleshooting)
- [Security Notes](#-security-notes)

---

## 📨 What the SMS Gateway Does

The SkyPay SMS Gateway lets your application send an SMS through an Android phone that you have connected and configured as a gateway device.

Your application does **not** need to communicate directly with the phone's SIM. Instead:

```text
Your Application
      │
      │ POST /api/sms/send
      │ Gateway Key + Number + Message
      ▼
SkyPay API Server
      │
      │ Firebase / FCM
      ▼
Configured Android Device
      │
      │ Android SMS service
      ▼
Selected SIM
      │
      ▼
Recipient
```

The Android device is the actual SMS sender. SkyPay's API and Firebase/FCM bridge deliver the command to that device.

---

## ✅ Requirements

Before using the SMS Gateway API, you need:

- An active/eligible SkyPay plan.
- At least one device created from the SkyPay website.
- The SkyPay Android application installed on the device.
- The Android app logged in with the account email and the Device Key.
- Required Android permissions granted.
- The Gateway configured in the Android app.
- An SMS Gateway created and linked to the intended device.
- The generated 18-character alphanumeric Gateway Key.
- Internet access on the Android device while it needs to receive FCM commands.

> **Important:** Internet is required for the Android device to receive the FCM command from SkyPay. Once the command has already reached the phone, a temporary loss of internet does not by itself stop the phone from sending the SMS through its cellular network.

---

# ⚙️ Setup Workflow

## 1. Activate a Plan

First, activate an eligible plan from the SkyPay website.

The device and gateway management features are available after the required plan is active.

---

## 2. Create a Device

Before creating an SMS Gateway, you must create a device.

From the SkyPay website:

1. Open your account/dashboard.
2. Go to the device management area.
3. Create a new device.
4. Give the device a recognizable name.
5. Save the device.
6. Copy the generated **Device Key**.

The Device Key is used by the Android application to associate that physical phone with your SkyPay account.

### Device Key vs Gateway Key

These are different credentials:

| Credential | Purpose |
|---|---|
| **Device Key** | Used by the Android app to log in and identify/configure the physical device. |
| **Gateway Key** | Used by your application/backend to call the SMS Gateway API. |

Do not confuse the two.

---

## 3. Install and Configure the Android App

Download the SkyPay Android application from the official website.

**Download:**  
https://skypaybd.top/public/assets/downloads/SkyPay.apk

Install it on the same Android device that you created in Device Management.

### Log in

Open the app and use:

- **Email:** Your SkyPay account email.
- **Device Key:** The Device Key copied from Device Management.

After login, grant the permissions requested by the application.

The permissions are required for the gateway to receive and process SMS-related operations.

### Configure the Gateway

The application has a **Gateway** section in the bottom navigation.

Open:

```text
Gateway
   ↓
Gateway Setup
```

Configure the gateway from there.

---

## 4. Create an SMS Gateway

After the Android device has been created and configured, create the SMS Gateway from the SkyPay website.

Open the website's settings area and locate the **Gateway** option from the settings/navigation area. Select **SMS Gateway** to open the gateway management screen.

### Create Gateway

1. Click **SMS Gateway**.
2. Choose **Create Gateway**.
3. Enter a name for the gateway.
4. Select the Android device that should send the SMS.
5. Save the gateway.

The device dropdown contains the devices already created under your account.

> **If there are no devices in the dropdown:** create a Device first, then return to SMS Gateway and create the gateway.

### Gateway Key Generation

When you save the SMS Gateway, the backend automatically generates a random **18-character alphanumeric Gateway Key**.

The key contains English letters and numbers, for example:

```text
A7kP92xL4mQ8zT1cR5
```

The exact generated value is unique and should be treated as a secret credential.

---

## 5. Use the Gateway Key

After the gateway is created, use its Gateway Key in your server-side application.

Example:

```http
GATEWAY-KEY: YOUR_18_CHARACTER_GATEWAY_KEY
```

Then call:

```http
POST https://core.skypaybd.top/api/sms/send
```

with:

```json
{
  "number": "01712345678",
  "message": "Your verification code is 482910."
}
```

Your backend sends the request to SkyPay, and SkyPay routes it to the Android device linked to that gateway.

---

# 🧩 Gateway Management

The SMS Gateway management screen shows the gateways created under your account.

Each gateway is associated with:

- Gateway name
- Linked Android device
- Generated Gateway Key
- Gateway status
- Available management actions

### Mobile View

On smaller screens, the Gateway Key may not be displayed directly to preserve space and keep the interface usable.

The gateway card can provide actions such as:

- **Copy**
- **Edit**
- **Regenerate**
- **Delete**

Use **Copy** to copy the Gateway Key when it is available.

### Desktop View

On desktop-sized screens, there is enough space to display the Gateway Key more directly along with the gateway/device information and management actions.

### Regenerate Key

If you regenerate the Gateway Key:

- The old key should no longer be used.
- Update your application's environment/configuration with the new key.
- Never commit the new key to a public repository.

### Delete Gateway

Deleting the gateway removes that gateway configuration. Applications using its old key should stop using it.

---

# 🔄 How the SMS Gateway Works

Once everything is configured, the actual message flow is simple:

```text
1. Your Application
       │
       │ Gateway Key
       │ Number
       │ Message
       ▼
2. SkyPay SMS API
       │
       │ Validate Gateway
       ▼
3. Linked Android Device
       │
       │ FCM message
       ▼
4. Android Gateway App
       │
       │ Number + Message
       ▼
5. Selected SIM
       │
       │ SMS
       ▼
6. Recipient
```

The backend resolves the Gateway Key to the linked device and obtains the device's FCM registration token from the datastore. Firebase Cloud Messaging is then used to deliver the message command to the Android device.

The Android application receives the command and sends the supplied text to the supplied number using the selected SIM.

---

# 📡 API Reference

## 🌐 Base URL

All SMS Gateway API requests use:

```text
https://core.skypaybd.top
```

### Endpoint

```text
POST https://core.skypaybd.top/api/sms/send
```

**Content-Type:**

```http
Content-Type: application/json
```

---

## 🔐 Authentication

Authenticate using the **18-character SMS Gateway Key** in an HTTP header.

### Recommended Header

```http
GATEWAY-KEY: YOUR_18_CHARACTER_GATEWAY_KEY
```

### Supported Alternatives

```http
gateway-key: YOUR_18_CHARACTER_GATEWAY_KEY
```

or:

```http
key: YOUR_18_CHARACTER_GATEWAY_KEY
```

The server checks the headers in this order:

```text
GATEWAY-KEY
     ↓
gateway-key
     ↓
key
```

Use `GATEWAY-KEY` in production integrations for clarity and consistency.

---

# 📤 Send SMS

### Request

```http
POST https://core.skypaybd.top/api/sms/send
```

### Headers

```http
GATEWAY-KEY: YOUR_18_CHARACTER_GATEWAY_KEY
Content-Type: application/json
```

### JSON Body

```json
{
  "number": "01712345678",
  "message": "Your verification code is 482910."
}
```

---

## 🧾 Request Parameters

| Field | Type | Required | Alias | Description |
|---|---|:---:|---|---|
| `number` | String | **Yes** | `phone` | Destination mobile number, e.g. `01712345678`. |
| `message` | String | **Yes** | `text` | Text message to deliver. |

### Equivalent Request

The aliases can be used instead:

```json
{
  "phone": "01712345678",
  "text": "Your verification code is 482910."
}
```

The API accepts either pair:

```text
number + message
```

or:

```text
phone + text
```

---

# 📥 Success Response

### `200 OK`

Returned when the message has been successfully queued/dispatched through FCM to the linked device.

```json
{
  "status": true,
  "message": "SMS queued successfully for delivery.",
  "data": {
    "number": "01712345678",
    "text": "Your verification code is 482910.",
    "device_name": "My Android Phone",
    "device_ip": "192.168.1.100",
    "message_id": "0:1234567890abcdef",
    "timestamp": "2025-01-15 10:30:45"
  }
}
```

### Response Fields

| Field | Description |
|---|---|
| `status` | Boolean success state. |
| `message` | Human-readable result message. |
| `data.number` | Destination number. |
| `data.text` | Message content. |
| `data.device_name` | Name of the linked Android device. |
| `data.device_ip` | Device IP information returned by the gateway. |
| `data.message_id` | FCM/message identifier. |
| `data.timestamp` | Server-side timestamp associated with the request. |

> A successful API response means the message was accepted/queued for delivery through the FCM bridge. The final SMS is sent by the Android device's selected SIM.

---

# ❌ Error Responses

All errors use a JSON response with:

```json
{
  "status": false,
  "message": "..."
}
```

## `405 Method Not Allowed`

Only `POST` is accepted.

```json
{
  "status": false,
  "message": "Method not allowed. Only POST requests are accepted."
}
```

---

## `401 Unauthorized` — Missing Gateway Key

```json
{
  "status": false,
  "message": "GATEWAY-KEY header is required."
}
```

---

## `401 Unauthorized` — Invalid or Inactive Gateway Key

```json
{
  "status": false,
  "message": "Invalid or inactive GATEWAY-KEY provided."
}
```

---

## `404 Not Found` — Linked Device Missing

```json
{
  "status": false,
  "message": "Linked device not found."
}
```

---

## `400 Bad Request` — Device Offline / Missing IP

```json
{
  "status": false,
  "message": "Target device is offline or inactive."
}
```

---

## `400 Bad Request` — Device Owner Cannot Be Resolved

```json
{
  "status": false,
  "message": "Unable to resolve device owner email."
}
```

---

## `400 Bad Request` — Device Key Missing

```json
{
  "status": false,
  "message": "Device key is missing."
}
```

---

## `400 Bad Request` — Required Fields Missing

```json
{
  "status": false,
  "message": "Both number and text fields are required."
}
```

---

## `500 Internal Server Error` — Firebase Configuration

Example:

```json
{
  "status": false,
  "message": "Server configuration missing: google-services.json not found."
}
```

Other configuration/authentication failures can include:

```json
{
  "status": false,
  "message": "Firebase Project ID missing in configuration."
}
```

```json
{
  "status": false,
  "message": "Server configuration missing: service-account.json not found."
}
```

```json
{
  "status": false,
  "message": "Failed to generate Firebase authentication token."
}
```

---

## `502 Bad Gateway` — Firestore Device Record

```json
{
  "status": false,
  "message": "Device record not found on datastore."
}
```

---

## `404 Not Found` — FCM Token Missing

```json
{
  "status": false,
  "message": "FCM registration token not found for this device."
}
```

---

## `4xx / 502` — FCM Delivery Failure

If Firebase rejects or fails to process the message, the API returns:

```json
{
  "status": false,
  "message": "<FCM error message here>"
}
```

The `message` field contains the relevant FCM error detail.

---

# 💻 Implementation Examples

## cURL

```bash
curl -X POST "https://core.skypaybd.top/api/sms/send" \
     -H "GATEWAY-KEY: YOUR_18_CHARACTER_GATEWAY_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "number": "01712345678",
       "message": "Your verification code is 482910."
     }'
```

---

## PHP — cURL

```php
<?php

$curl = curl_init();

$payload = [
    "number"  => "01712345678",
    "message" => "Your verification code is 482910."
];

curl_setopt_array($curl, [
    CURLOPT_URL            => "https://core.skypaybd.top/api/sms/send",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST           => true,
    CURLOPT_POSTFIELDS     => json_encode($payload),
    CURLOPT_HTTPHEADER     => [
        "GATEWAY-KEY: YOUR_18_CHARACTER_GATEWAY_KEY",
        "Content-Type: application/json"
    ],
]);

$response = curl_exec($curl);
curl_close($curl);

echo $response;
```

---

## Node.js — Fetch

```js
const sendSms = async () => {
  const response = await fetch("https://core.skypaybd.top/api/sms/send", {
    method: "POST",
    headers: {
      "GATEWAY-KEY": "YOUR_18_CHARACTER_GATEWAY_KEY",
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      number: "01712345678",
      message: "Your verification code is 482910."
    })
  });

  const result = await response.json();
  console.log(result);
};

sendSms();
```

---

## Python — Requests

```python
import requests

url = "https://core.skypaybd.top/api/sms/send"

headers = {
    "GATEWAY-KEY": "YOUR_18_CHARACTER_GATEWAY_KEY",
    "Content-Type": "application/json"
}

payload = {
    "number": "01712345678",
    "message": "Your verification code is 482910."
}

response = requests.post(url, json=payload, headers=headers)

print(response.json())
```

---

# 📱 Android Gateway Configuration

The Android application is the physical bridge between the SkyPay API and the SIM card.

Inside the app:

```text
Dashboard
   ↓
Gateway
   ↓
Gateway Setup
```

The gateway setup is responsible for configuring the phone that will send outgoing SMS.

### SMS Permission

The application must have the required SMS permissions before it can send messages through the device.

### SIM Selection

The gateway provides a SIM selection option.

- If the phone has **one SIM**, select that SIM.
- If the phone has **two SIMs**, select which SIM should be used for outgoing SMS.
- The selected SIM is used when the API sends a message command to the device.

### Outgoing SMS Notifications

The gateway also provides an option controlling notifications for outgoing SMS messages.

You can:

```text
ON  → receive notifications for outgoing SMS
OFF → disable outgoing SMS notifications
```

This setting controls the notification behavior; it does not change the API request format.

---

# ✉️ Message Length & Multipart SMS

The SMS Gateway accepts normal text messages as well as longer messages.

If a message is longer than the normal single-SMS size, the Android device handles it as a **multipart SMS** through the Android SMS service.

Your API request remains the same:

```json
{
  "number": "01712345678",
  "message": "A longer message..."
}
```

You do not need to manually split the message into multiple API requests.

---

# 🌐 Internet Requirement

The Android gateway device needs an active internet connection when it must receive an SMS command from SkyPay through Firebase Cloud Messaging.

The basic relationship is:

```text
Internet
   ↓
Firebase / FCM
   ↓
Android Gateway App
   ↓
Cellular Network / SIM
   ↓
SMS Recipient
```

Once the FCM command has already reached the device, the phone can use its cellular network to send the SMS. Therefore, the API bridge's internet requirement is primarily for receiving the remote command.

For reliable operation, keep the gateway device connected to stable Wi-Fi or mobile data.

---

# 🔁 Complete Flow

```text
┌──────────────────────────────┐
│ 1. Activate SkyPay Plan      │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 2. Create Android Device     │
│    → Device Key generated    │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 3. Install Android App       │
│    → Login with Email + Key  │
│    → Grant permissions       │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 4. Configure Gateway         │
│    → Gateway section         │
│    → SMS permission          │
│    → Select SIM              │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 5. Create SMS Gateway        │
│    → Select Device           │
│    → Save                    │
│    → 18-char Gateway Key     │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 6. Your Backend              │
│    POST /api/sms/send        │
│    Gateway Key + Number      │
│    + Message                 │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 7. SkyPay API                │
│    Validate Gateway          │
│    Resolve linked device     │
│    Resolve FCM token         │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 8. Firebase Cloud Messaging  │
│    Delivers command          │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 9. Android Gateway App       │
│    Receives number + text    │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 10. Selected SIM             │
│     Sends SMS                 │
└──────────────┬───────────────┘
               ↓
          📱 Recipient
```

---

# 🛠️ Troubleshooting

### `GATEWAY-KEY header is required.`

Make sure your request contains:

```http
GATEWAY-KEY: YOUR_18_CHARACTER_GATEWAY_KEY
```

and that the header is not accidentally placed inside the JSON body.

### `Invalid or inactive GATEWAY-KEY provided.`

Check that:

- The key was copied correctly.
- The gateway still exists.
- The gateway is active.
- You did not regenerate the key without updating your application.

### `Linked device not found.`

Open Gateway Management and verify that the gateway is linked to an existing device.

### `Target device is offline or inactive.`

Check:

- Android phone is powered on.
- SkyPay app is running/configured.
- Internet connection is available for FCM.
- Device/gateway configuration is active.

### `FCM registration token not found for this device.`

Open the SkyPay Android app and make sure the device has successfully logged in and registered with the backend.

### SMS is not being sent

Verify:

1. SMS permission is granted.
2. The correct SIM is selected.
3. The SIM has cellular service.
4. The Android device can receive FCM commands.
5. The Gateway Key belongs to the intended gateway/device.
6. The destination number is valid.

---

# 🔒 Security Notes

- Treat the **Gateway Key as a secret credential**.
- Do not publish Gateway Keys in GitHub repositories.
- Do not put a Gateway Key directly into public frontend JavaScript.
- Store the Gateway Key in your backend environment/configuration.
- Use HTTPS for all API communication.
- If a key is exposed, regenerate the gateway key and update your backend.
- Keep Device Keys separate from Gateway Keys.
- Only create gateways for devices you control and have permission to use for SMS sending.
- Do not use the gateway to send unwanted, deceptive, or abusive messages.

---

## 📚 Related Documentation

- Main SkyPay documentation: [`../README.md`](../README.md)
- API Core: https://core.skypaybd.top
- Online Documentation: https://skypaybd.top/docs
- SkyPay Website: https://skypaybd.top

---

<div align="center">

**SkyPay SMS Gateway API**

*Programmatic SMS delivery through your configured Android device.*

</div>
