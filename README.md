# FinTrack

[อ่านคู่มือภาษาไทยที่นี่](README_th.md)

**FinTrack** is a high-efficiency, mobile-first Progressive Web App (PWA) built for fast, localized financial transaction tracking. Sign in with your Google account, log income and expenses on the go, and keep your team instantly in sync via Telegram — all from a clean, installable app on your phone.

![Version](https://img.shields.io/badge/version-1.1.0-2EE8B4)
![Platform](https://img.shields.io/badge/platform-PWA-4285F4)
![Auth](https://img.shields.io/badge/sign--in-Google-EA4335)
![Notifications](https://img.shields.io/badge/notifications-Telegram-26A5E4)
![Languages](https://img.shields.io/badge/languages-5%20supported-success)

---

## 1. Product Overview & Features (v1.1.0)

- **🔐 Native Google Sign-In** — Quick, secure identity verification using your existing Google account. Completely passwordless — no usernames, PINs, or accounts to remember.
- **📲 Instant Telegram Alerts** — Every transaction you save sends a real-time summary notification straight to your management's Telegram channel.
- **⚙️ Standalone Settings Page** — A dedicated, full-screen settings view for your preferences, separated from the main entry screen.
- **🌗 Visual Customization** — Switch instantly between Light and Dark mode. Your choice is remembered automatically for next time.
- **🌐 5-Language AEC Support** — Fully localized in English, Thai, Lao, Vietnamese, and Burmese.
- **📱 Installable App** — Add FinTrack to your home screen for a native app experience, with offline support built in.

---

## 2. End-User Operational Guide

1. **Sign in** — Open the FinTrack link and tap **"Sign in with Google"** to verify your identity with your Google account.
2. **Log a transaction** — Fill in the details: amount, item name, branch/location, your name, and currency.
3. **Save it** — Tap **"Save Entry"** to record the transaction. A summary notification is sent automatically to the management Telegram channel.
4. **Customize your experience** — Open the **Settings** tab anytime to switch between Light/Dark mode or change your display language.

---

## 3. Technical Deployment Setup

FinTrack runs on a Google Sheets + Apps Script backend. To deploy your own instance:

1. Open your Google Sheet, then go to **Extensions ▸ Apps Script**.
2. Paste in the contents of `Code.gs`.
3. Set your Telegram bot credentials at the top of `Code.gs`:

```
TELEGRAM_BOT_TOKEN = 'your-telegram-bot-token'
TELEGRAM_CHAT_ID   = 'your-telegram-chat-id'
```

4. Click **Deploy ▸ New deployment**, choose **Web app**, and set:
   - **Execute as: Me**
   - **Who has access: Anyone**
5. Copy the generated Web App URL and enter it into the FinTrack app's connection field.

> **Note:** "Execute as: Me" with "Who has access: Anyone" is required so the app can reach the backend without authentication redirects or CORS errors.

---

## Architecture Overview

| Layer | Technology |
| --- | --- |
| Frontend | HTML5, CSS Variables, Progressive Web App (PWA) |
| Backend | Google Apps Script — Workspace Cloud Core |
| Database | Google Sheets |
| Notifications | Telegram Bot API |
