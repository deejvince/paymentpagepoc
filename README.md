# 💳 Mock PSP Payment Flow PoC

This project is a **Proof of Concept (PoC)** for simulating a **third-party payment experience** similar to real-world PSP (Payment Service Provider) integrations, including fallback strategies for mobile and in-app WebView environments.

It includes:
- A **main payment page** (`index.html`)
- A **mock PSP confirmation page** (`psp.html`)
- A **return handler** (`return.html`) that finalizes the user flow

---

## 🔍 Purpose

This PoC explores how to:
- Simulate redirect-based PSP flows (e.g., PayPal, Paysafe)
- Handle **popup blocking** and **fallbacks** (especially on iOS and WKWebView)
- Collect and submit **additional fields** required by specific payment methods
- Return to the original page with **payment status** (success or failure)
- Ensure a smooth experience across:
  - ✅ Desktop browsers
  - ✅ Mobile Safari
  - ✅ iOS WKWebView

---

## 🚀 Live Demo Setup

### 1. Deploy to Netlify (or similar static host)

You can deploy each file from this repo directly:

| File        | Purpose                                | Example Domain                    |
|-------------|----------------------------------------|-----------------------------------|
| `index.html` | Main payment entry page                | `https://your-main-site.netlify.app` |
| `psp.html`   | Mock PSP confirmation page             | `https://your-psp-site.netlify.app` |
| `return.html`| Redirect and postMessage handler       | Same domain as `index.html`        |

Make sure:
- The `psp.html` and `index.html` live on **separate origins/domains**
- You set the correct `returnUrl` in PSP redirects

---

### 2. Configure Beeceptor (or your own mock API)

Use [https://beeceptor.com](https://beeceptor.com) to simulate backend behavior:

Create an endpoint, e.g.  
`https://paypagemockapi.free.beeceptor.com`

#### Mock rules:
| Condition | Response |
|----------|----------|
| `amount=10` | `{ "redirectUrl": "https://psp.../psp.html?returnUrl=..." }` |
| `amount=20` | `{ "fields": [{ "name": "email", "label": "Email" }, ...] }` |
| `amount=50` | Same as 10, but for high value |

For `/api/payment/submit-fields`, return:
```json
{ "redirectUrl": "https://psp-site.netlify.app/psp.html?returnUrl=https://main-site.netlify.app/return.html" }
```

### 🧪 Test Scenarios
Scenario	Input	Behavior
Quick Payment	£10	Opens popup → PSP → return
Fields Required	£20	Shows field overlay → complete → PSP
Direct Redirect	£50	Redirects immediately to PSP

All statuses are returned to the main page with feedback (✔ / ✖) and styling.

# ✅ Testing & Platform Notes

This document describes how to test the Mock PSP Payment Flow PoC across platforms and explains key architectural decisions to ensure compatibility, especially with WebKit-based WebViews.

---

## 🧪 Test Scenarios

Use the input **amount** to trigger different backend responses and flows.

| Amount | Scenario Description                                      | Flow Behavior                                                                 |
|--------|-----------------------------------------------------------|--------------------------------------------------------------------------------|
| £10    | Direct payment flow (no extra fields)                     | Opens popup → PSP page → return                                                |
| £20    | Requires additional fields (e.g. email, country)          | Shows overlay → submit fields → opens PSP → return                             |
| £50    | High-value transaction with immediate redirect to PSP     | Opens popup → PSP page → return                                                |

All scenarios are handled from `index.html`, and result status (✔ or ✖) is reflected back on the payment page.

---

## 📱 Platform Compatibility

| Platform                 | Behavior                                                             | Status  |
|--------------------------|----------------------------------------------------------------------|---------|
| Chrome (Desktop)         | Full popup support                                                   | ✅ OK    |
| Safari (Desktop)         | Full popup support                                                   | ✅ OK    |
| Safari (iOS)             | Popup must be opened **on click** to avoid being blocked            | ✅ OK    |
| WebKit WKWebView (iOS)   | Popup scripting restrictions; fallback logic must avoid injection    | ✅ OK    |
| Android WebView (Chrome) | Similar restrictions; fallback handled                               | ✅ OK    |

> ❗ Important: In **WKWebView**, script injection into a popup using `document.write` is blocked. This PoC avoids that entirely.

---

## 🛠 Key Design Considerations

- **Popup Window Strategy**  
  Popup is opened immediately on user click to comply with browser gesture requirements (especially Safari).

- **Fallback Handling**  
  If the popup is blocked (`popup == null`), the script uses a full-page redirect and restores context with `localStorage.paymentReturn`.

- **Field Collection Flow**  
  If extra fields are needed (`amount = 20`), the form appears as an overlay. Upon submission, the response is used to redirect to the PSP page.

- **Return Messaging**  
  `return.html` uses `postMessage` when possible, or falls back to URL redirect with `status` in query params.

---

## 📁 File Overview

```text
index.html         # Main UI and business logic
psp.html           # Simulated PSP (Pay/Cancel buttons)
return.html        # Handles PSP result and communicates back

