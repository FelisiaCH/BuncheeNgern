# FinTrack

**Income & Expense Tracking System** — an installable PWA for logging daily income and expenses across multiple branches, backed by a Google Sheets database via Apps Script, with Google Sign-In for user identity and instant Telegram notifications for every new transaction.

![Frontend](https://img.shields.io/badge/frontend-HTML%20%2F%20CSS%20%2F%20JS-2EE8B4)
![Backend](https://img.shields.io/badge/backend-Google%20Apps%20Script-4285F4)
![Auth](https://img.shields.io/badge/auth-Google%20Identity%20Services-EA4335)
![Notifications](https://img.shields.io/badge/notifications-Telegram%20Bot%20API-26A5E4)
![Build](https://img.shields.io/badge/build-none%20required-success)

---

This README is provided in two languages. Pick the section for your preferred language — both cover the same setup steps.

- [🇺🇸 English Setup Guide](#-english-setup-guide)
- [🇹🇭 คู่มือการตั้งค่าภาษาไทย](#-คู่มือการตั้งค่าภาษาไทย)

---

## 🇺🇸 English Setup Guide

### ✨ Features

- **🔐 Google Sign-In** — users authenticate with their Google account via Google Identity Services. No passwords, PINs, or credential database.
- **🌐 Multi-language UI** — English, Thai, Lao, Vietnamese, and Burmese, switchable on the fly. All translation strings live in [lang.js](lang.js).
- **🏪 Dynamic branch management** — add, rename, or remove shop branches from the UI
- **💱 Multi-currency support** — LAK, THB, and USD with live currency switching
- **📎 Smart receipt uploads** — slip photos are auto-compressed before upload to Google Drive
- **📊 Live dashboard** — daily totals, metric cards, and filterable entry lists
- **📲 Instant Telegram notifications** — every new transaction sends a formatted message to your Telegram chat via the Telegram Bot API
- **⚙️ Standalone Settings page** — a dedicated bottom-nav tab for language, theme, connection, and branch management
- **🌗 Light / Dark mode** — instant theme switching via CSS variables, persisted to `localStorage`
- **📱 Installable PWA** — full home-screen install on iOS and Android via `manifest.json`

---

### 🧱 Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | Vanilla HTML / CSS / JS — [index.html](index.html) + [lang.js](lang.js), no frameworks, no build step |
| Authentication | [Google Identity Services](https://developers.google.com/identity/gsi/web) (Sign in with Google) |
| Backend | [Google Apps Script](https://www.google.com/script/start/) Web App (`Code.gs`) |
| Database | Google Sheets |
| File storage | Google Drive |
| Notification delivery | Telegram Bot API (`UrlFetchApp.fetch`) |
| Offline cache | [service-worker.js](service-worker.js) — caches `index.html` and `lang.js` for offline use |

---

### 🏗️ Architecture

```
Browser (PWA)
  └─ Google Identity Services → verified { name, email } JWT
  └─ index.html  ──fetch──▶  Apps Script Web App (Code.gs)
                                  ├─ Google Sheets   (entry storage)
                                  ├─ Google Drive     (slip uploads)
                                  └─ Telegram Bot API (notifications)
```

The Apps Script Web App is the single backend endpoint. It must be deployed with **"Execute as: Me"** and **"Who has access: Anyone"** so that the public PWA can call it without CORS or login redirects. The user's identity is established client-side by Google Sign-In and sent along with every write request as `userEmail`.

---

### 🚀 Setup Steps

#### 1. Create the Google Sheet

Create a new Google Sheet. This will act as your database — daily entries are written to per-date tabs automatically.

#### 2. Deploy the Apps Script backend

1. In your Google Sheet, open **Extensions ▸ Apps Script**.
2. Replace the default content with the contents of [Code.gs](Code.gs).
3. Fill in the configuration constants at the top of the file:

```
SPREADSHEET_ID
DRIVE_FOLDER_ID
TELEGRAM_BOT_TOKEN
TELEGRAM_CHAT_ID
```

- `SPREADSHEET_ID` — the ID from your Sheet's URL (`/d/<SPREADSHEET_ID>/edit`)
- `DRIVE_FOLDER_ID` — a Google Drive folder ID where slip images will be uploaded
- `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` — see step 4 below

4. Click **Deploy ▸ New deployment**.
5. Choose type **Web app**.
6. Set:
   - **Execute as: Me**
   - **Who has access: Anyone**
7. Click **Deploy** and copy the resulting Web App URL.

> ⚠️ **"Execute as: Me" + "Who has access: Anyone" is mandatory.** This lets the public PWA call the API directly without triggering CORS errors or Google login redirects, while the script itself still runs with your permissions to access the Sheet and Drive folder.

#### 3. Set up a Telegram bot for notifications

1. Message [@BotFather](https://t.me/BotFather) on Telegram and create a new bot with `/newbot`. Copy the bot token into `TELEGRAM_BOT_TOKEN`.
2. Start a chat with your new bot (or add it to a group) and send any message.
3. Visit `https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/getUpdates` in your browser to find your `chat.id`. Copy that value into `TELEGRAM_CHAT_ID`.

#### 4. Set up Google Sign-In

1. Go to the [Google Cloud Console](https://console.cloud.google.com/apis/credentials) and create (or select) a project.
2. Create an **OAuth 2.0 Client ID** of type **Web application**.
3. Add the URL where you will host the PWA (e.g. `https://yourusername.github.io`) under **Authorized JavaScript origins**.
4. Copy the generated Client ID into `index.html`:

```
const GOOGLE_CLIENT_ID = 'YOUR_GOOGLE_CLIENT_ID_HERE';
```

#### 5. Configure and host the PWA

1. Host [index.html](index.html), [lang.js](lang.js), [service-worker.js](service-worker.js), and [manifest.json](manifest.json) on any static host (GitHub Pages, Netlify, etc.) — HTTPS is required for both the PWA service worker and Google Sign-In.
2. Open the hosted app, sign in with Google, then enter your Apps Script Web App URL (from step 2.7) in the connection field. It is saved to `localStorage` — no source code editing needed for this part.

---

### 📂 Project Structure

| File | Purpose |
| --- | --- |
| [index.html](index.html) | Main application — UI, styles, and client-side logic |
| [lang.js](lang.js) | Translation strings for all supported languages |
| [Code.gs](Code.gs) | Apps Script backend — Sheets, Drive, and Telegram integration |
| [service-worker.js](service-worker.js) | PWA offline cache |
| [manifest.json](manifest.json) | PWA install manifest |

---

## 🇹🇭 คู่มือการตั้งค่าภาษาไทย

### ✨ ฟีเจอร์เด่น

- **🔐 เข้าสู่ระบบด้วย Google** — ผู้ใช้ยืนยันตัวตนด้วยบัญชี Google ผ่าน Google Identity Services ไม่มีรหัสผ่านหรือฐานข้อมูลบัญชีผู้ใช้
- **🌐 รองรับหลายภาษา** — อังกฤษ, ไทย, ลาว, เวียดนาม และพม่า สลับได้ทันที ข้อความแปลทั้งหมดอยู่ใน [lang.js](lang.js)
- **🏪 จัดการสาขาแบบไดนามิก** — เพิ่ม แก้ไข หรือลบสาขาได้จากหน้าแอป
- **💱 รองรับหลายสกุลเงิน** — LAK, THB และ USD สลับสกุลเงินได้ทันที
- **📎 อัปโหลดสลิปอัจฉริยะ** — รูปสลิปจะถูกบีบอัดอัตโนมัติก่อนอัปโหลดไปยัง Google Drive
- **📊 แดชบอร์ดสด** — สรุปยอดรายวัน การ์ดข้อมูล และรายการที่กรองได้
- **📲 แจ้งเตือนผ่าน Telegram ทันที** — ทุกรายการใหม่จะส่งข้อความไปยังแชท Telegram ของคุณผ่าน Telegram Bot API
- **⚙️ หน้าตั้งค่าแยกเฉพาะ** — แท็บด้านล่างสำหรับภาษา ธีม การเชื่อมต่อ และการจัดการสาขา
- **🌗 โหมดสว่าง / มืด** — สลับธีมได้ทันทีผ่าน CSS variables และจดจำการตั้งค่าใน `localStorage`
- **📱 ติดตั้งเป็น PWA ได้** — ติดตั้งบนหน้าโฮมสกรีนได้ทั้ง iOS และ Android ผ่าน `manifest.json`

---

### 🧱 เทคโนโลยีที่ใช้

| ส่วน | เทคโนโลยี |
| --- | --- |
| Frontend | HTML / CSS / JS ธรรมดา — [index.html](index.html) + [lang.js](lang.js) ไม่ใช้เฟรมเวิร์ก ไม่ต้อง build |
| การยืนยันตัวตน | [Google Identity Services](https://developers.google.com/identity/gsi/web) (Sign in with Google) |
| Backend | [Google Apps Script](https://www.google.com/script/start/) Web App (`Code.gs`) |
| ฐานข้อมูล | Google Sheets |
| จัดเก็บไฟล์ | Google Drive |
| ส่งการแจ้งเตือน | Telegram Bot API (`UrlFetchApp.fetch`) |
| แคชออฟไลน์ | [service-worker.js](service-worker.js) — แคช `index.html` และ `lang.js` สำหรับใช้งานออฟไลน์ |

---

### 🏗️ โครงสร้างระบบ

```
เบราว์เซอร์ (PWA)
  └─ Google Identity Services → JWT ที่ยืนยันแล้ว { name, email }
  └─ index.html  ──fetch──▶  Apps Script Web App (Code.gs)
                                  ├─ Google Sheets   (จัดเก็บรายการ)
                                  ├─ Google Drive     (อัปโหลดสลิป)
                                  └─ Telegram Bot API (แจ้งเตือน)
```

Apps Script Web App คือ backend หลักเพียงตัวเดียว ต้อง deploy ด้วยค่า **"Execute as: Me"** และ **"Who has access: Anyone"** เพื่อให้ PWA สาธารณะเรียก API ได้โดยไม่เจอปัญหา CORS หรือถูก redirect ไปหน้าล็อกอิน Google ตัวตนผู้ใช้จะถูกยืนยันที่ฝั่งเบราว์เซอร์ด้วย Google Sign-In และส่งไปพร้อมทุกคำขอบันทึกข้อมูลในชื่อ `userEmail`

---

### 🚀 ขั้นตอนการตั้งค่า

#### 1. สร้าง Google Sheet

สร้าง Google Sheet ใหม่ ใช้เป็นฐานข้อมูล — รายการรายวันจะถูกเขียนลงแท็บแยกตามวันที่โดยอัตโนมัติ

#### 2. Deploy ส่วน Backend (Apps Script)

1. ในชีตของคุณ เปิด **Extensions ▸ Apps Script**
2. แทนที่เนื้อหาเริ่มต้นด้วยเนื้อหาจาก [Code.gs](Code.gs)
3. กรอกค่าคอนฟิกที่ด้านบนของไฟล์:

```
SPREADSHEET_ID
DRIVE_FOLDER_ID
TELEGRAM_BOT_TOKEN
TELEGRAM_CHAT_ID
```

- `SPREADSHEET_ID` — รหัสจาก URL ของชีต (`/d/<SPREADSHEET_ID>/edit`)
- `DRIVE_FOLDER_ID` — รหัสโฟลเดอร์ Google Drive สำหรับเก็บรูปสลิป
- `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` — ดูขั้นตอนที่ 4

4. คลิก **Deploy ▸ New deployment**
5. เลือกประเภท **Web app**
6. ตั้งค่า:
   - **Execute as: Me**
   - **Who has access: Anyone**
7. คลิก **Deploy** แล้วคัดลอก Web App URL ที่ได้

> ⚠️ **ต้องตั้งค่า "Execute as: Me" และ "Who has access: Anyone" เท่านั้น** เพื่อให้ PWA สาธารณะเรียก API ได้ตรงโดยไม่เจอ CORS หรือถูก redirect ไปหน้าล็อกอิน Google ขณะที่สคริปต์ยังทำงานด้วยสิทธิ์ของคุณในการเข้าถึงชีตและโฟลเดอร์ Drive

#### 3. ตั้งค่า Telegram Bot สำหรับการแจ้งเตือน

1. แชทกับ [@BotFather](https://t.me/BotFather) บน Telegram และสร้างบอทใหม่ด้วยคำสั่ง `/newbot` จากนั้นคัดลอก token ไปใส่ใน `TELEGRAM_BOT_TOKEN`
2. เริ่มแชทกับบอทของคุณ (หรือเพิ่มเข้ากลุ่ม) แล้วส่งข้อความใดก็ได้
3. เปิด `https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/getUpdates` ในเบราว์เซอร์เพื่อหาค่า `chat.id` แล้วคัดลอกไปใส่ใน `TELEGRAM_CHAT_ID`

#### 4. ตั้งค่า Google Sign-In

1. ไปที่ [Google Cloud Console](https://console.cloud.google.com/apis/credentials) และสร้าง (หรือเลือก) โปรเจกต์
2. สร้าง **OAuth 2.0 Client ID** ประเภท **Web application**
3. เพิ่ม URL ที่จะใช้ host PWA (เช่น `https://yourusername.github.io`) ในช่อง **Authorized JavaScript origins**
4. คัดลอก Client ID ที่ได้ไปใส่ใน `index.html`:

```
const GOOGLE_CLIENT_ID = 'YOUR_GOOGLE_CLIENT_ID_HERE';
```

#### 5. ตั้งค่าและโฮสต์ PWA

1. โฮสต์ [index.html](index.html), [lang.js](lang.js), [service-worker.js](service-worker.js) และ [manifest.json](manifest.json) บน static host ใดก็ได้ (GitHub Pages, Netlify ฯลฯ) — ต้องใช้ HTTPS ทั้งสำหรับ service worker ของ PWA และ Google Sign-In
2. เปิดแอปที่โฮสต์ไว้ เข้าสู่ระบบด้วย Google จากนั้นกรอก Apps Script Web App URL (จากขั้นตอนที่ 2.7) ในช่องเชื่อมต่อ ค่านี้จะถูกบันทึกใน `localStorage` โดยไม่ต้องแก้ไขซอร์สโค้ด

---

### 📂 โครงสร้างโปรเจกต์

| ไฟล์ | หน้าที่ |
| --- | --- |
| [index.html](index.html) | แอปหลัก — UI, สไตล์ และ logic ฝั่งไคลเอนต์ |
| [lang.js](lang.js) | ข้อความแปลภาษาทั้งหมด |
| [Code.gs](Code.gs) | Backend ของ Apps Script — เชื่อมต่อ Sheets, Drive และ Telegram |
| [service-worker.js](service-worker.js) | แคชออฟไลน์ของ PWA |
| [manifest.json](manifest.json) | ไฟล์ manifest สำหรับติดตั้ง PWA |
