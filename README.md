# FinTrack

**Income & Expense Tracking System** — an installable PWA for logging daily income and expenses across multiple branches, with a Google Sheets backend and production-grade native Web Push Notifications delivered even when the app is completely closed or swiped from mobile RAM.

![Frontend](https://img.shields.io/badge/frontend-HTML%20%2F%20CSS%20%2F%20JS-2EE8B4)
![Backend](https://img.shields.io/badge/backend-Google%20Apps%20Script-4285F4)
![Push](https://img.shields.io/badge/push-Web%20Push%20%2F%20VAPID-FF6B6B)
![Build](https://img.shields.io/badge/build-none%20required-success)

---

This README is provided in two languages. Pick the section for your preferred language — both cover the same setup steps.

- [🇺🇸 English Guide](#-english-guide)
- [🇹🇭 คู่มือภาษาไทย](#-คู่มือภาษาไทย)

---

## 🇺🇸 English Guide

## ✨ Features

- **🌐 Multi-language UI** — English, Thai, Lao, Vietnamese, and Burmese, switchable on the fly. All translation strings live in [lang.js](lang.js).
- **🔐 Secure PIN login** — per-staff PIN authentication with lockout after failed attempts
- **🏪 Dynamic branch management** — add, rename, or remove shop branches from the UI
- **💱 Multi-currency support** — LAK, THB, and USD with live currency switching
- **📎 Smart receipt uploads** — drag-and-drop slip photos, auto-compressed before upload to Google Drive
- **📊 Live dashboard** — daily totals, metric cards, and filterable entry lists
- **🔔 Native cross-device Web Push** — any device that submits a transaction broadcasts a notification to every registered device, even when the PWA is closed or swiped from RAM
- **⚙️ In-app Settings panel** — Apps Script Web App URL and VAPID Public Key are entered directly in the UI and stored in `localStorage` — no source code editing required
- **🔊 Sound & vibration fallback** — audible chime on desktop; vibration on mobile when autoplay is blocked
- **📱 Installable PWA** — full home-screen install on iOS and Android via `manifest.json`

---

## 🧱 Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | Vanilla HTML / CSS / JS — [index.html](index.html) + [lang.js](lang.js), no frameworks, no build step |
| Backend | [Google Apps Script](https://www.google.com/script/start/) Web App (`Code.gs`) |
| Database | Google Sheets |
| File storage | Google Drive |
| Push delivery | Browser Web Push API + VAPID (RFC 8292) |
| Offline cache | [service-worker.js](service-worker.js) — caches `index.html` and `lang.js` for offline use |

---

## 🔔 Web Push Architecture

FinTrack uses the **standard browser Web Push API** — no Firebase, no third-party push service, no monthly fees.

```text
[Any device submits a transaction]
        │
        ▼
  Google Sheets (new row appended)
        │
        ▼  onNewTransaction() trigger fires automatically
  Code.gs (Google Apps Script)
        │  generates VAPID JWT (ES256 / ECDSA P-256)
        │  broadcastPushNotification() loops every saved subscription
        │  and POSTs the payload via UrlFetchApp
        ▼
  Browser Push Service
  (FCM for Chrome  /  Mozilla Push for Firefox  /  APNs for Safari)
        │
        ▼  delivered even when the app is fully closed / swiped from RAM
  Service Worker (service-worker.js)
        │  push event fires → showNotification()
        ▼
  Every registered device's lock screen / notification tray
```

The notification reaches every subscribed device through the browser's own push infrastructure — the originating device, teammates' phones, and any other browser that has subscribed all receive it. The PWA does not need to be open or in memory; the Service Worker wakes briefly, shows the notification, then sleeps again.

---

## ⚙️ Setup Guide

### Prerequisites

- A Google account (for Sheets, Drive, and Apps Script)
- The `openssl` command-line tool (to generate your VAPID key pair)

---

### Step 1 — Create the Google Sheet and Drive Folder

1. Create a new Google Sheet. This will be your transaction database.
2. Add a tab named exactly **`Users`** with these columns in row 1:

   | Column A | Column B | Column C |
   | --- | --- | --- |
   | Staff Name | PIN | Status |
   | Alice | 1234 | Active |
   | Bob | 5678 | Active |

   `Status` must be `Active` or `Locked`. Daily entry tabs are created automatically by the script.

3. Create a folder in Google Drive — uploaded receipt photos are stored here.
4. Note the **Sheet ID** (the long string in the Sheet URL) and the **Drive Folder ID** (same location in the folder URL). You will paste these into the script in Step 2.

---

### Step 2 — Deploy the Google Apps Script Backend

> ## 🚨 CRITICAL WARNING — Container-Bound Script Required
>
> Do **not** create your project from [script.google.com](https://script.google.com/) using **New project**. That creates a **Standalone Script**, which is **not bound to your spreadsheet**.
>
> If you use a Standalone Script, the **`onNewTransaction` On-Change trigger in Step 4 will be impossible to configure** — the **"From spreadsheet" (จากสเปรดชีต)** event source and the **"On change" (เมื่อมีการเปลี่ยนแปลง)** event type will not appear as options in the Trigger dialog at all, because a standalone script has no spreadsheet to bind to. Cross-device push notifications will silently never fire.
>
> **You must create a Container-Bound Script instead:**
>
> 1. Open the Google Sheet you created in Step 1 directly in your browser.
> 2. In the top menu bar, click **Extensions (ส่วนขยาย)**.
> 3. Click **Apps Script**.
> 4. This opens a script editor that is permanently bound to this specific spreadsheet. Only this type of project will show "From spreadsheet" and "On change" as trigger options in Step 4.

1. With the container-bound editor open (from Extensions → Apps Script above), delete the empty `Code.gs` content, then paste in the full contents of [Code.gs](Code.gs) from this repository.

2. **Enable V8 Runtime — this is required.**
   - Click the gear icon **⚙️ Project Settings** in the left sidebar.
   - Under **Runtime version**, select **V8**.
   - Click **Save**.

   > The ECDSA P-256 signing engine uses `BigInt()`, which requires V8. The default Rhino runtime does not support BigInt and will fail.

3. Fill in your spreadsheet credentials **directly inside the Apps Script editor** — never commit real values to this repository:

   ```js
   const SPREADSHEET_ID  = 'YOUR_GOOGLE_SHEET_ID_HERE';
   const DRIVE_FOLDER_ID = 'YOUR_DRIVE_FOLDER_ID_HERE';
   ```

4. Fill in your VAPID credentials (generated in [Step 3](#step-3--generate-and-configure-vapid-credentials) below):

   ```js
   const PUSH_VAPID_PUBLIC  = 'YOUR_VAPID_PUBLIC_KEY_HERE';
   const PUSH_VAPID_PRIVATE = 'YOUR_VAPID_PRIVATE_KEY_HERE';
   const PUSH_VAPID_SUBJECT = 'mailto:your-email@example.com';
   ```

5. Click **Deploy → New deployment**:
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Click **Deploy** and authorize all Google permission requests.

6. Copy the deployment URL. It follows this format:

   ```text
   https://script.google.com/macros/s/[YOUR_DEPLOYMENT_ID]/exec
   ```

   You will paste this into the FinTrack **Settings** panel in Step 5 — no source file needs to be edited.

---

### Step 3 — Generate and Configure VAPID Credentials

VAPID (Voluntary Application Server Identification) is the cryptographic proof that only your server can push notifications to your subscribers. Each deployment needs its own unique key pair.

#### Generate your key pair

Run these commands once on your local machine (`openssl` required). First, generate a P-256 private key file:

```bash
openssl ecparam -name prime256v1 -genkey -noout -out vapid_private.pem
```

Next, export the public key. The output is a base64url string roughly 87 characters long — this value is safe to use in your Settings panel and in `Code.gs`:

```bash
openssl ec -in vapid_private.pem -pubout -outform DER 2>/dev/null | tail -c 65 | base64 | tr '+/' '-_' | tr -d '=\n'
```

Then export the private key. The output is a base64url string roughly 43 characters long — **never commit this value anywhere**:

```bash
openssl ec -in vapid_private.pem -outform DER 2>/dev/null | dd bs=1 skip=7 count=32 2>/dev/null | base64 | tr '+/' '-_' | tr -d '=\n'
```

Finally, delete the PEM file — you only need the two base64url strings produced above:

```bash
rm vapid_private.pem
```

#### Where each key belongs

**Public key** (safe to share — it is public by cryptographic design):

Open the FinTrack app, tap **⋮ More → ⚙️ Settings**, paste the public key into the **VAPID Public Key** field, and tap **Save VAPID Key**. The value is stored in your browser's `localStorage` — no source file needs editing.

Also update `PUSH_VAPID_PUBLIC` in your Apps Script project with the same value:

```js
const PUSH_VAPID_PUBLIC = 'YOUR_VAPID_PUBLIC_KEY_HERE';
```

**Private key** (never commit, never paste into the frontend):

Open your Apps Script project at [script.google.com](https://script.google.com/) and locate this line:

```js
const PUSH_VAPID_PRIVATE = 'YOUR_VAPID_PRIVATE_KEY_HERE';
```

Replace `YOUR_VAPID_PRIVATE_KEY_HERE` with your generated private key **directly in the Apps Script editor**. Do not paste it into any file you would `git add`, and do not paste it into the FinTrack Settings panel — the frontend only ever needs the public key.

> **Security rule:** The private key must only ever exist inside the Apps Script editor (or a local secrets manager). If you accidentally paste it into a file and `git add` it, treat the key as permanently compromised — generate a brand new key pair immediately and rotate both values before pushing anything.

---

### Step 4 — Install the On-Change Trigger

This trigger fires `onNewTransaction()` automatically every time a new row is appended to your spreadsheet — regardless of which device submitted it.

1. In the Apps Script editor, click the **⏱ Triggers** icon in the left sidebar.
2. Click **+ Add Trigger** (bottom right).
3. Configure as follows:

   | Setting | Value |
   | --- | --- |
   | Function to run | `onNewTransaction` |
   | Deployment to run | Head |
   | Event source | From spreadsheet |
   | Event type | **On change** |

4. Click **Save**. Google will ask you to re-authorize — accept all permissions.

> If **"From spreadsheet"** or **"On change"** does not appear in the dropdown options, your script project is a Standalone Script, not a Container-Bound Script. Go back to [Step 2](#step-2--deploy-the-google-apps-script-backend) and re-create the project via **Extensions → Apps Script** from inside the Google Sheet itself.
>
> `onNewTransaction` only acts when `e.changeType === 'INSERT_ROW'`. Edits, deletions, and other changes are ignored. When a new row is detected, it reads the Staff Name, Item Name, Currency, Price, Type, and Shop from that row and calls `broadcastPushNotification(title, body)`, which loops through every subscription stored in the `PushSubscriptions` sheet and dispatches a Web Push to each one.

---

### Step 5 — Connect the Frontend (Settings Panel)

All configuration now happens inside the app — no source file needs to be edited.

1. Open [index.html](index.html) in a browser, or visit your deployed GitHub Pages URL.
2. Tap the **⋮ More** button (top right of the app).
3. Under **⚙️ Settings**:
   - Paste your Apps Script deployment URL (from Step 2.6) into the **App Token or Web App URL** field and tap **Save Script**.
   - Paste your VAPID public key (from Step 3) into the **VAPID Public Key** field and tap **Save VAPID Key**.
4. Both values are saved in `localStorage` and used immediately for all API requests and push subscriptions — the app is now fully connected to your backend.

---

### Step 6 — Subscribe Each Device to Web Push

Every device that should receive background notifications must subscribe individually. Each subscription is unique to that browser and device.

**On each device (mobile or desktop):**

1. Open the FinTrack PWA. On mobile, install it to the home screen first:
   - **Android (Chrome):** tap the browser menu → **Add to Home Screen**
   - **iOS (Safari):** tap the Share button → **Add to Home Screen**

2. Complete [Step 5](#step-5--connect-the-frontend-settings-panel) on this device if you haven't already — the VAPID public key must be saved before subscribing.

3. Tap **⋮ More** → **🔑 Request Notification Permission** — grant permission when prompted by the browser.

4. Tap **⋮ More** → **📲 Subscribe to Push Notifications**.

5. The subscription is saved to your `PushSubscriptions` sheet automatically.

> Subscriptions are per-browser, per-device, and per-VAPID-key pair. If you regenerate your VAPID keys, all devices must re-subscribe.

---

### Step 7 — Test End-to-End

1. In the Apps Script editor, open **Execution log** (View → Logs).
2. Copy any subscription JSON string from column B of the `PushSubscriptions` sheet.
3. Run the following manually in the script editor console, replacing the placeholder with your copied subscription JSON:

   ```js
   sendWebPushNotification('FinTrack Test', 'New entry received', '{ ...paste your subscription JSON here... }');
   ```

4. Check the execution log — a `200 OK` response means the push service accepted the notification. The notification should appear on the target device within a few seconds.

5. For a full cross-device test, submit a transaction from one device and confirm that **every** subscribed device — including others — receives the notification via `broadcastPushNotification`.

---

## ⚠️ Apps Script Syntax Constraint — BigInt Literals

Apps Script's V8 parser **rejects** the `n`-suffix BigInt literal syntax at save time. Code such as the example below throws `ParseError: Unexpected token ILLEGAL` and cannot be saved:

```js
if (candidate >= 1n && candidate < _P256_N) return candidate;
while (exp > 0n) { }
```

Always use the `BigInt()` constructor instead — Apps Script V8 parses the constructor form correctly:

```js
if (candidate >= BigInt(1) && candidate < _P256_N) return candidate;
while (exp > BigInt(0)) { }
```

[Code.gs](Code.gs) already uses `_BI0`, `_BI1`, `_BI2`, `_BI3` constants throughout the ECDSA engine to avoid this pitfall. Do not rewrite those lines back to `n`-suffix syntax.

---

## 🚀 Running Locally

FinTrack is a set of static files — [index.html](index.html), [lang.js](lang.js), [service-worker.js](service-worker.js) — no `npm install`, no bundler, no server required.

If you use VS Code, the **Live Server** extension is the simplest option: right-click [index.html](index.html) and choose **Open with Live Server**.

Alternatively, serve the project directory with Python's built-in server:

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080` in your browser.

> Service Workers and Web Push require HTTPS or `localhost`. The GitHub Pages deployment is always HTTPS and works out of the box.

---

## 🧹 Resetting Repository History (Advanced / Optional)

If you ever need to permanently wipe your fork's commit history — for example, after accidentally committing a secret — you can reinitialize the repository locally and force-push a single clean commit.

First, remove the existing Git history and start a fresh repository:

```bash
rm -rf .git
git init
git branch -M main
```

Next, stage and commit the current working tree as a single new initial commit:

```bash
git add .
git commit -m "initial: clean history"
```

Finally, re-link your remote and overwrite the remote history with a force push:

```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main --force
```

> **Warning:** `git push --force` permanently overwrites the remote branch history. Anyone with an existing clone will need to re-clone the repository. Any secret that was ever committed should still be treated as compromised and rotated — force-pushing removes it from the branch, but GitHub may retain unreferenced objects for a short period before garbage collection.

### 🖥️ Platform Compatibility

The commands above use Unix-style syntax (`rm -rf`, forward-slash paths, etc.):

- **macOS and Linux:** these commands work natively in the default Terminal — no changes needed.
- **Windows:** run these commands in **Git Bash** (installed with Git for Windows) or **PowerShell**. In PowerShell, `rm -rf .git` works as an alias for `Remove-Item -Recurse -Force .git`; all other commands (`git init`, `git add`, `git commit`, `git remote`, `git push`) are identical across platforms since they are Git commands, not shell commands.

---

## 🧠 Why Serverless?

FinTrack has no server to rent or maintain. The entire backend runs on Google's infrastructure:

- **Google Sheets** — transaction database
- **Google Drive** — receipt photo storage
- **Google Apps Script Web App** — REST API + cross-device push notification dispatcher

The frontend stores your Apps Script URL, VAPID public key, session, language preference, and branch list in `localStorage` — no backend session, no cookies, no user database. Zero hosting cost. Your data stays in your own Google account.

---

## 🔒 Security Rules

| What | Rule |
| --- | --- |
| `SPREADSHEET_ID` | Paste in Apps Script editor only — never commit to this repo |
| `DRIVE_FOLDER_ID` | Same |
| `PUSH_VAPID_PRIVATE` | Generated locally, pasted in Apps Script editor only — treat like a password |
| `PUSH_VAPID_PUBLIC` | Safe to share — it is a public key by cryptographic design |
| VAPID Public Key (frontend) | Entered via the in-app Settings panel, stored in `localStorage` — never hardcoded in `index.html` or `lang.js` |
| Deployment URL | Entered via the in-app Settings panel, stored in `localStorage` — keep private, it is your live API endpoint |

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

---

## 🇹🇭 คู่มือภาษาไทย

## ✨ ฟีเจอร์

- **🌐 รองรับหลายภาษา** — อังกฤษ, ไทย, ลาว, เวียดนาม และพม่า สามารถสลับได้ทันที คำแปลทั้งหมดอยู่ในไฟล์ [lang.js](lang.js)
- **🔐 เข้าสู่ระบบด้วย PIN ที่ปลอดภัย** — ยืนยันตัวตนด้วย PIN รายบุคคล พร้อมล็อกบัญชีหากกรอกผิดหลายครั้ง
- **🏪 จัดการสาขาแบบไดนามิก** — เพิ่ม เปลี่ยนชื่อ หรือลบสาขาได้จากหน้าแอปโดยตรง
- **💱 รองรับหลายสกุลเงิน** — LAK, THB และ USD พร้อมสลับสกุลเงินได้แบบเรียลไทม์
- **📎 แนบสลิปอัจฉริยะ** — ลากวางรูปสลิป ระบบบีบอัดอัตโนมัติก่อนอัปโหลดไป Google Drive
- **📊 แดชบอร์ดแบบเรียลไทม์** — ยอดรวมรายวัน การ์ดสรุปข้อมูล และรายการที่กรองได้
- **🔔 Web Push ข้ามอุปกรณ์** — เมื่อมีการบันทึกรายการจากอุปกรณ์ใด ระบบจะส่งการแจ้งเตือนไปยังอุปกรณ์ที่ลงทะเบียนทุกเครื่อง แม้แอปถูกปิดหรือถูกปัดออกจาก RAM
- **⚙️ แผงตั้งค่าในแอป** — กรอก Apps Script Web App URL และ VAPID Public Key ได้จากหน้าแอปโดยตรง บันทึกใน `localStorage` ไม่ต้องแก้ไขซอร์สโค้ด
- **🔊 เสียงและการสั่นสำรอง** — มีเสียงเตือนบนเดสก์ท็อป และสั่นบนมือถือเมื่อเล่นเสียงอัตโนมัติถูกบล็อก
- **📱 ติดตั้งเป็น PWA ได้** — ติดตั้งบนหน้าจอหลักของ iOS และ Android ผ่าน `manifest.json`

---

## 🧱 เทคโนโลยีที่ใช้

| ส่วน | เทคโนโลยี |
| --- | --- |
| Frontend | HTML / CSS / JS ธรรมดา — [index.html](index.html) + [lang.js](lang.js) ไม่ใช้เฟรมเวิร์ก ไม่ต้องมีขั้นตอน build |
| Backend | [Google Apps Script](https://www.google.com/script/start/) Web App (`Code.gs`) |
| ฐานข้อมูล | Google Sheets |
| ที่จัดเก็บไฟล์ | Google Drive |
| การส่ง Push | Browser Web Push API + VAPID (RFC 8292) |
| แคชออฟไลน์ | [service-worker.js](service-worker.js) — แคช `index.html` และ `lang.js` เพื่อใช้งานออฟไลน์ |

---

## 🔔 สถาปัตยกรรม Web Push

FinTrack ใช้ **Web Push API มาตรฐานของเบราว์เซอร์** — ไม่ใช้ Firebase ไม่ใช้บริการ push จากบุคคลที่สาม ไม่มีค่าใช้จ่ายรายเดือน

```text
[อุปกรณ์ใดบันทึกรายการ]
        │
        ▼
  Google Sheets (เพิ่มแถวใหม่)
        │
        ▼  ทริกเกอร์ onNewTransaction() ทำงานอัตโนมัติ
  Code.gs (Google Apps Script)
        │  สร้าง VAPID JWT (ES256 / ECDSA P-256)
        │  broadcastPushNotification() วนซ้ำทุกการสมัครรับที่บันทึกไว้
        │  และส่ง payload ผ่าน UrlFetchApp
        ▼
  Browser Push Service
  (FCM สำหรับ Chrome  /  Mozilla Push สำหรับ Firefox  /  APNs สำหรับ Safari)
        │
        ▼  ส่งถึงแม้แอปถูกปิดหรือถูกปัดออกจาก RAM
  Service Worker (service-worker.js)
        │  เมื่อมี push event → showNotification()
        ▼
  หน้าจอล็อก / ถาดแจ้งเตือนของทุกอุปกรณ์ที่ลงทะเบียน
```

การแจ้งเตือนจะถูกส่งไปยังทุกอุปกรณ์ที่สมัครรับผ่านโครงสร้างพื้นฐาน push ของเบราว์เซอร์เอง — ทั้งอุปกรณ์ต้นทาง อุปกรณ์ของเพื่อนร่วมงาน และเบราว์เซอร์อื่นที่สมัครรับไว้จะได้รับการแจ้งเตือนทั้งหมด ไม่จำเป็นต้องเปิดแอปหรือให้แอปอยู่ในหน่วยความจำ Service Worker จะตื่นขึ้นมาสั้น ๆ แสดงการแจ้งเตือน แล้วกลับไปพักอีกครั้ง

---

## ⚙️ คู่มือการติดตั้ง

### สิ่งที่ต้องมีก่อน

- บัญชี Google (สำหรับ Sheets, Drive และ Apps Script)
- เครื่องมือ `openssl` ในเทอร์มินัล (สำหรับสร้างคู่กุญแจ VAPID)

---

### ขั้นที่ 1 — สร้าง Google Sheet และโฟลเดอร์ Drive

1. สร้าง Google Sheet ใหม่ ใช้เป็นฐานข้อมูลรายการธุรกรรม
2. เพิ่มแท็บชื่อ **`Users`** (ต้องตรงตามนี้) พร้อมหัวคอลัมน์แถวที่ 1:

   | คอลัมน์ A | คอลัมน์ B | คอลัมน์ C |
   | --- | --- | --- |
   | Staff Name | PIN | Status |
   | Alice | 1234 | Active |
   | Bob | 5678 | Active |

   `Status` ต้องเป็น `Active` หรือ `Locked` แท็บรายการรายวันจะถูกสร้างขึ้นโดยสคริปต์โดยอัตโนมัติ

3. สร้างโฟลเดอร์ใน Google Drive — ใช้เก็บรูปสลิปที่อัปโหลด
4. จด **Sheet ID** (สตริงยาวในลิงก์ของ Sheet) และ **Drive Folder ID** (ตำแหน่งเดียวกันในลิงก์โฟลเดอร์) เพื่อนำไปใส่ในสคริปต์ในขั้นที่ 2

---

### ขั้นที่ 2 — ติดตั้ง Backend ด้วย Google Apps Script

> ## 🚨 คำเตือนสำคัญ — ต้องใช้ Container-Bound Script เท่านั้น
>
> **อย่า** สร้างโปรเจกต์จาก [script.google.com](https://script.google.com/) ด้วยปุ่ม **New project** เพราะวิธีนี้จะสร้าง **Standalone Script** ซึ่ง **ไม่ได้ผูกกับสเปรดชีตของคุณ**
>
> ถ้าใช้ Standalone Script **ทริกเกอร์ On-Change ของ `onNewTransaction` ในขั้นที่ 4 จะไม่สามารถตั้งค่าได้** — ตัวเลือก event source **"From spreadsheet" (จากสเปรดชีต)** และ event type **"On change" (เมื่อมีการเปลี่ยนแปลง)** จะไม่ปรากฏให้เลือกในหน้าต่าง Trigger เลย เพราะ standalone script ไม่มีสเปรดชีตให้ผูก ผลคือการแจ้งเตือนข้ามอุปกรณ์จะไม่ทำงานเลยโดยไม่มีข้อผิดพลาดให้เห็น
>
> **คุณต้องสร้าง Container-Bound Script แทน ด้วยวิธีนี้:**
>
> 1. เปิด Google Sheet ที่สร้างในขั้นที่ 1 ในเบราว์เซอร์
> 2. ที่เมนูด้านบน คลิก **Extensions (ส่วนขยาย)**
> 3. คลิก **Apps Script**
> 4. ระบบจะเปิดตัวแก้ไขสคริปต์ที่ผูกกับสเปรดชีตนี้อย่างถาวร เฉพาะโปรเจกต์ประเภทนี้เท่านั้นที่จะแสดง "From spreadsheet" และ "On change" เป็นตัวเลือกในขั้นที่ 4

1. เมื่อเปิดตัวแก้ไขแบบ container-bound แล้ว (จาก Extensions → Apps Script ด้านบน) ให้ลบเนื้อหาเปล่าใน `Code.gs` แล้ววางเนื้อหาทั้งหมดจาก [Code.gs](Code.gs) ในรีโพนี้

2. **เปิดใช้ V8 Runtime — จำเป็นต้องทำ**
   - คลิกไอคอนรูปเฟือง **⚙️ Project Settings** ที่แถบด้านซ้าย
   - ที่ **Runtime version** เลือก **V8**
   - คลิก **Save**

   > เอนจินเซ็น ECDSA P-256 ใช้ `BigInt()` ซึ่งต้องใช้ V8 รันไทม์ Rhino แบบเดิมไม่รองรับ BigInt และจะทำงานล้มเหลว

3. กรอกข้อมูลของสเปรดชีต **ในตัวแก้ไข Apps Script เท่านั้น** — ห้าม commit ค่าจริงเข้ารีโพนี้:

   ```js
   const SPREADSHEET_ID  = 'YOUR_GOOGLE_SHEET_ID_HERE';
   const DRIVE_FOLDER_ID = 'YOUR_DRIVE_FOLDER_ID_HERE';
   ```

4. กรอกข้อมูล VAPID (สร้างได้ใน [ขั้นที่ 3](#ขั้นที่-3--สร้างและตั้งค่า-vapid-credentials) ด้านล่าง):

   ```js
   const PUSH_VAPID_PUBLIC  = 'YOUR_VAPID_PUBLIC_KEY_HERE';
   const PUSH_VAPID_PRIVATE = 'YOUR_VAPID_PRIVATE_KEY_HERE';
   const PUSH_VAPID_SUBJECT = 'mailto:your-email@example.com';
   ```

5. คลิก **Deploy → New deployment**:
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - คลิก **Deploy** และอนุญาตสิทธิ์ที่ Google ขอทั้งหมด

6. คัดลอก URL การ deploy ซึ่งจะมีรูปแบบดังนี้:

   ```text
   https://script.google.com/macros/s/[YOUR_DEPLOYMENT_ID]/exec
   ```

   คุณจะนำ URL นี้ไปวางในแผง **Settings** ของ FinTrack ในขั้นที่ 5 — ไม่ต้องแก้ไฟล์ใด ๆ

---

### ขั้นที่ 3 — สร้างและตั้งค่า VAPID Credentials

VAPID (Voluntary Application Server Identification) คือการพิสูจน์เชิงรหัสว่ามีเพียงเซิร์ฟเวอร์ของคุณเท่านั้นที่ส่ง push ไปยังผู้สมัครรับได้ แต่ละการ deploy ต้องมีคู่กุญแจของตัวเอง

#### สร้างคู่กุญแจ

รันคำสั่งเหล่านี้ครั้งเดียวบนเครื่องของคุณ (ต้องมี `openssl`) ขั้นแรกสร้างไฟล์ private key แบบ P-256:

```bash
openssl ecparam -name prime256v1 -genkey -noout -out vapid_private.pem
```

จากนั้น export public key ผลลัพธ์จะเป็นสตริง base64url ยาวประมาณ 87 ตัวอักษร — ใช้ในแผง Settings และใน `Code.gs` ได้อย่างปลอดภัย:

```bash
openssl ec -in vapid_private.pem -pubout -outform DER 2>/dev/null | tail -c 65 | base64 | tr '+/' '-_' | tr -d '=\n'
```

จากนั้น export private key ผลลัพธ์จะเป็นสตริง base64url ยาวประมาณ 43 ตัวอักษร — **ห้าม commit ค่านี้ที่ใดเด็ดขาด**:

```bash
openssl ec -in vapid_private.pem -outform DER 2>/dev/null | dd bs=1 skip=7 count=32 2>/dev/null | base64 | tr '+/' '-_' | tr -d '=\n'
```

สุดท้าย ลบไฟล์ PEM — คุณต้องการเพียงสตริง base64url ทั้งสองที่ได้ด้านบน:

```bash
rm vapid_private.pem
```

#### กุญแจแต่ละตัวใช้ที่ไหน

**Public key** (ปลอดภัยที่จะเผยแพร่ — เป็นคีย์สาธารณะโดยการออกแบบเชิงรหัส):

เปิดแอป FinTrack แล้วแตะ **⋮ More → ⚙️ ตั้งค่า / Settings** วางค่า public key ลงในช่อง **VAPID Public Key** แล้วแตะ **บันทึก VAPID Key** ค่านี้จะถูกบันทึกใน `localStorage` ของเบราว์เซอร์ ไม่ต้องแก้ไฟล์ใด ๆ

อัปเดต `PUSH_VAPID_PUBLIC` ในโปรเจกต์ Apps Script ด้วยค่าเดียวกัน:

```js
const PUSH_VAPID_PUBLIC = 'YOUR_VAPID_PUBLIC_KEY_HERE';
```

**Private key** (ห้าม commit และห้ามวางใน frontend):

เปิดโปรเจกต์ Apps Script ที่ [script.google.com](https://script.google.com/) แล้วหาบรรทัดนี้:

```js
const PUSH_VAPID_PRIVATE = 'YOUR_VAPID_PRIVATE_KEY_HERE';
```

แทนที่ `YOUR_VAPID_PRIVATE_KEY_HERE` ด้วย private key ที่สร้างไว้ **ในตัวแก้ไข Apps Script เท่านั้น** ห้ามวางลงในไฟล์ที่จะ `git add` และห้ามวางในแผง Settings ของ FinTrack — ฝั่ง frontend ต้องการเพียง public key เท่านั้น

> **กฎความปลอดภัย:** private key ต้องอยู่ในตัวแก้ไข Apps Script เท่านั้น (หรือตัวจัดการความลับในเครื่อง) หากวางลงในไฟล์โดยไม่ตั้งใจและ `git add` ไปแล้ว ให้ถือว่าคีย์นั้นรั่วไหลแล้วทันที สร้างคู่กุญแจใหม่และเปลี่ยนค่าทั้งสองก่อน push ใด ๆ

---

### ขั้นที่ 4 — ติดตั้งทริกเกอร์ On-Change

ทริกเกอร์นี้จะเรียก `onNewTransaction()` โดยอัตโนมัติทุกครั้งที่มีแถวใหม่ถูกเพิ่มในสเปรดชีต ไม่ว่าจะมาจากอุปกรณ์ใดก็ตาม

1. ในตัวแก้ไข Apps Script คลิกไอคอน **⏱ Triggers** ที่แถบด้านซ้าย
2. คลิก **+ Add Trigger** (มุมขวาล่าง)
3. ตั้งค่าดังนี้:

   | การตั้งค่า | ค่า |
   | --- | --- |
   | Function to run | `onNewTransaction` |
   | Deployment to run | Head |
   | Event source | From spreadsheet |
   | Event type | **On change** |

4. คลิก **Save** Google จะขอให้คุณอนุญาตสิทธิ์อีกครั้ง — ยอมรับทั้งหมด

> ถ้า **"From spreadsheet"** หรือ **"On change"** ไม่ปรากฏในตัวเลือก แสดงว่าโปรเจกต์สคริปต์ของคุณเป็น Standalone Script ไม่ใช่ Container-Bound Script ให้กลับไปที่ [ขั้นที่ 2](#ขั้นที่-2--ติดตั้ง-backend-ด้วย-google-apps-script) และสร้างโปรเจกต์ใหม่ผ่าน **Extensions → Apps Script** จากภายใน Google Sheet เท่านั้น
>
> `onNewTransaction` จะทำงานเฉพาะเมื่อ `e.changeType === 'INSERT_ROW'` การแก้ไขหรือลบแถวจะถูกข้าม เมื่อพบแถวใหม่ ระบบจะอ่านชื่อพนักงาน ชื่อรายการ สกุลเงิน ราคา ประเภท และสาขาจากแถวนั้น แล้วเรียก `broadcastPushNotification(title, body)` ซึ่งจะวนซ้ำทุกการสมัครรับที่บันทึกไว้ในชีต `PushSubscriptions` และส่ง Web Push ไปยังแต่ละรายการ

---

### ขั้นที่ 5 — เชื่อมต่อ Frontend (แผง Settings)

การตั้งค่าทั้งหมดทำได้จากในแอป ไม่ต้องแก้ไฟล์ใด ๆ

1. เปิด [index.html](index.html) ในเบราว์เซอร์ หรือเข้า URL ที่ deploy บน GitHub Pages
2. แตะปุ่ม **⋮ More** (มุมขวาบนของแอป)
3. ภายใต้ **⚙️ ตั้งค่า / Settings**:
   - วาง URL การ deploy ของ Apps Script (จากขั้นที่ 2.6) ในช่อง **App Token or Web App URL** แล้วแตะ **บันทึกสคริปต์**
   - วาง VAPID public key (จากขั้นที่ 3) ในช่อง **VAPID Public Key** แล้วแตะ **บันทึก VAPID Key**
4. ค่าทั้งสองจะถูกบันทึกใน `localStorage` และใช้งานทันทีสำหรับทุก API request และการสมัครรับ push — แอปพร้อมเชื่อมต่อกับ backend ของคุณแล้ว

---

### ขั้นที่ 6 — สมัครรับ Web Push สำหรับแต่ละอุปกรณ์

ทุกอุปกรณ์ที่ต้องการรับการแจ้งเตือนต้องสมัครรับแยกกัน การสมัครรับจะผูกกับเบราว์เซอร์และอุปกรณ์นั้น ๆ เท่านั้น

**สำหรับแต่ละอุปกรณ์ (มือถือหรือเดสก์ท็อป):**

1. เปิดแอป FinTrack สำหรับมือถือ ให้ติดตั้งลงหน้าจอหลักก่อน:
   - **Android (Chrome):** แตะเมนูเบราว์เซอร์ → **Add to Home Screen**
   - **iOS (Safari):** แตะปุ่ม Share → **Add to Home Screen**

2. ทำตาม [ขั้นที่ 5](#ขั้นที่-5--เชื่อมต่อ-frontend-แผง-settings) ในอุปกรณ์นี้ก่อนหากยังไม่ทำ — ต้องบันทึก VAPID public key ก่อนสมัครรับ

3. แตะ **⋮ More** → **🔑 Request Notification Permission** — อนุญาตเมื่อเบราว์เซอร์ถาม

4. แตะ **⋮ More** → **📲 Subscribe to Push Notifications**

5. ระบบจะบันทึกการสมัครรับลงในชีต `PushSubscriptions` โดยอัตโนมัติ

> การสมัครรับจะผูกกับคู่ของเบราว์เซอร์ อุปกรณ์ และ VAPID key หากสร้างคู่กุญแจ VAPID ใหม่ ทุกอุปกรณ์ต้องสมัครรับใหม่

---

### ขั้นที่ 7 — ทดสอบแบบ End-to-End

1. ในตัวแก้ไข Apps Script เปิด **Execution log** (View → Logs)
2. คัดลอกสตริง JSON ของการสมัครรับจากคอลัมน์ B ของชีต `PushSubscriptions`
3. รันคำสั่งนี้ใน console ของตัวแก้ไขสคริปต์ โดยแทนที่ placeholder ด้วย JSON ที่คัดลอกมา:

   ```js
   sendWebPushNotification('FinTrack Test', 'มีรายการใหม่เข้ามา', '{ ...วาง subscription JSON ของคุณที่นี่... }');
   ```

4. ดู execution log — ผลลัพธ์ `200 OK` แปลว่า push service รับการแจ้งเตือนแล้ว การแจ้งเตือนควรปรากฏบนอุปกรณ์ปลายทางภายในไม่กี่วินาที

5. สำหรับการทดสอบแบบข้ามอุปกรณ์เต็มรูปแบบ ให้บันทึกรายการจากอุปกรณ์หนึ่ง แล้วตรวจสอบว่า **ทุก** อุปกรณ์ที่สมัครรับไว้ — รวมถึงอุปกรณ์อื่น — ได้รับการแจ้งเตือนผ่าน `broadcastPushNotification`

---

## ⚠️ ข้อจำกัดด้านไวยากรณ์ของ Apps Script — BigInt Literals

ตัวแยกวิเคราะห์ V8 ของ Apps Script **ปฏิเสธ** ไวยากรณ์ BigInt literal แบบมีคำต่อท้าย `n` ตอนบันทึก โค้ดตัวอย่างด้านล่างจะเกิด `ParseError: Unexpected token ILLEGAL` และไม่สามารถบันทึกได้:

```js
if (candidate >= 1n && candidate < _P256_N) return candidate;
while (exp > 0n) { }
```

ให้ใช้ตัวสร้าง `BigInt()` แทนเสมอ — Apps Script V8 รองรับรูปแบบนี้:

```js
if (candidate >= BigInt(1) && candidate < _P256_N) return candidate;
while (exp > BigInt(0)) { }
```

[Code.gs](Code.gs) ใช้ค่าคงที่ `_BI0`, `_BI1`, `_BI2`, `_BI3` ตลอดทั้งเอนจิน ECDSA เพื่อหลีกเลี่ยงปัญหานี้อยู่แล้ว ห้ามเปลี่ยนกลับเป็นไวยากรณ์แบบ `n`

---

## 🚀 การรันในเครื่อง

FinTrack คือไฟล์สแตติกชุดเดียว — [index.html](index.html), [lang.js](lang.js), [service-worker.js](service-worker.js) — ไม่ต้อง `npm install` ไม่ต้องใช้ bundler ไม่ต้องมีเซิร์ฟเวอร์

หากใช้ VS Code ส่วนขยาย **Live Server** เป็นวิธีที่ง่ายที่สุด: คลิกขวาที่ [index.html](index.html) แล้วเลือก **Open with Live Server**

หรือเปิดเซิร์ฟเวอร์ในตัวของ Python จากไดเรกทอรีของโปรเจกต์:

```bash
python3 -m http.server 8080
```

จากนั้นเปิด `http://localhost:8080` ในเบราว์เซอร์

> Service Worker และ Web Push ต้องใช้ HTTPS หรือ `localhost` การ deploy บน GitHub Pages เป็น HTTPS อยู่แล้วจึงใช้งานได้ทันที

---

## 🧹 การล้างประวัติ Git ของรีโพ (ขั้นสูง / ทางเลือก)

หากต้องการล้างประวัติ commit ของฟอร์กอย่างถาวร — เช่น หลังจากที่มีการ commit ความลับเข้าไปโดยไม่ตั้งใจ — สามารถสร้างรีโพใหม่ในเครื่องและ force-push commit เดียวที่สะอาดได้

ขั้นแรก ลบประวัติ Git เดิมและเริ่มรีโพใหม่:

```bash
rm -rf .git
git init
git branch -M main
```

จากนั้น stage และ commit ไฟล์ปัจจุบันทั้งหมดเป็น commit เริ่มต้นใหม่:

```bash
git add .
git commit -m "initial: clean history"
```

สุดท้าย เชื่อม remote ใหม่และเขียนทับประวัติ remote ด้วย force push:

```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main --force
```

> **คำเตือน:** `git push --force` จะเขียนทับประวัติของ remote branch อย่างถาวร ผู้ที่มี clone อยู่แล้วต้อง clone ใหม่ ความลับใดที่เคย commit ไปแล้วยังต้องถือว่ารั่วไหลและต้องเปลี่ยนค่าใหม่ — การ force-push เพียงลบออกจาก branch แต่ GitHub อาจยังเก็บ object ที่ไม่ถูกอ้างอิงไว้ชั่วระยะหนึ่งก่อนล้างจริง

### 🖥️ ความเข้ากันได้ของแพลตฟอร์ม

คำสั่งด้านบนใช้ไวยากรณ์แบบ Unix (`rm -rf`, เครื่องหมาย `/` ในพาธ ฯลฯ):

- **macOS และ Linux:** คำสั่งเหล่านี้ใช้งานได้ทันทีใน Terminal เริ่มต้น ไม่ต้องแก้ไขอะไร
- **Windows:** ให้รันคำสั่งเหล่านี้ใน **Git Bash** (มาพร้อม Git for Windows) หรือ **PowerShell** ใน PowerShell คำสั่ง `rm -rf .git` เป็น alias ของ `Remove-Item -Recurse -Force .git` ส่วนคำสั่งอื่น (`git init`, `git add`, `git commit`, `git remote`, `git push`) เหมือนกันทุกแพลตฟอร์มเพราะเป็นคำสั่งของ Git ไม่ใช่ของ shell

---

## 🧠 ทำไมไม่ต้องมีเซิร์ฟเวอร์?

FinTrack ไม่มีเซิร์ฟเวอร์ให้เช่าหรือดูแล ระบบ backend ทั้งหมดทำงานบนโครงสร้างพื้นฐานของ Google:

- **Google Sheets** — ฐานข้อมูลรายการธุรกรรม
- **Google Drive** — ที่เก็บรูปสลิป
- **Google Apps Script Web App** — REST API และตัวกระจายการแจ้งเตือนข้ามอุปกรณ์

ฝั่ง frontend จะบันทึก Apps Script URL, VAPID public key, session, ภาษาที่เลือก และรายชื่อสาขาไว้ใน `localStorage` — ไม่มี backend session ไม่มี cookie ไม่มีฐานข้อมูลผู้ใช้ ไม่มีค่าใช้จ่ายในการโฮสต์ ข้อมูลของคุณอยู่ในบัญชี Google ของคุณเองเท่านั้น

---

## 🔒 กฎความปลอดภัย

| รายการ | กฎ |
| --- | --- |
| `SPREADSHEET_ID` | วางในตัวแก้ไข Apps Script เท่านั้น — ห้าม commit เข้ารีโพนี้ |
| `DRIVE_FOLDER_ID` | เช่นเดียวกัน |
| `PUSH_VAPID_PRIVATE` | สร้างในเครื่อง วางในตัวแก้ไข Apps Script เท่านั้น — ต้องดูแลเหมือนรหัสผ่าน |
| `PUSH_VAPID_PUBLIC` | ปลอดภัยที่จะเผยแพร่ — เป็น public key โดยการออกแบบเชิงรหัส |
| VAPID Public Key (frontend) | กรอกผ่านแผง Settings ในแอป บันทึกใน `localStorage` — ห้าม hardcode ใน `index.html` หรือ `lang.js` |
| Deployment URL | กรอกผ่านแผง Settings ในแอป บันทึกใน `localStorage` — เก็บเป็นความลับ เพราะเป็น endpoint API จริงของคุณ |

---

## 📄 สัญญาอนุญาต

โปรเจกต์นี้อยู่ภายใต้ [MIT License](LICENSE)
