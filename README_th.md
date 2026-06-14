# FinTrack

[Read the English guide here](README.md)

PWA สำหรับบันทึกรายรับ-รายจ่ายบนมือถือ พนักงานเข้าสู่ระบบด้วย Google บันทึกรายการ แล้วระบบจะเพิ่มข้อมูลลง Google Sheet (รูปสลิปไปที่ Drive) พร้อมส่งข้อความแจ้งเตือนเข้า Telegram ของฝ่ายบริหาร ทั้งหน้าเว็บเป็นไฟล์ `index.html` ไฟล์เดียว ส่วนหลังบ้านเป็น Google Apps Script ไฟล์เดียว (`Code.gs`)

> **เรื่องความปลอดภัยที่ต้องเข้าใจ:** การล็อกอินที่นี่เป็นแค่ *ประตูกั้นแบบหลวมๆ* ไม่ใช่ระบบยืนยันตัวตนจริง เพราะ credential จาก Google ถูกอ่านในเบราว์เซอร์แต่ไม่เคยถูกตรวจสอบที่ฝั่งเซิร์ฟเวอร์ และ backend ถูกตั้งให้ "ทุกคน" เข้าถึงได้ ดังนั้นใครก็ตามที่มี Web App URL จะเขียนข้อมูลลงชีตได้ — เหมาะสำหรับทีมภายในที่เชื่อใจกันและแค่ต้องรู้ว่า "ใครบันทึกรายการนี้" แต่อย่าใช้แทนการยืนยันตัวตนจริงสำหรับข้อมูลสำคัญ ดูหัวข้อ [การเพิ่มความปลอดภัย](#การเพิ่มความปลอดภัย-ทางเลือก)

---

## สิ่งที่ต้องเตรียมก่อนเริ่ม

- บัญชี Google (สำหรับ Sheet, โฟลเดอร์ Drive, Apps Script และ OAuth client)
- Telegram bot token และ chat ID ที่จะรับการแจ้งเตือน
- ที่สำหรับโฮสต์ `index.html` แบบ **HTTPS** (GitHub Pages, Netlify, Cloudflare Pages) — Google Sign-In **ใช้ไม่ได้** บน `file://` หรือ `http://` ธรรมดา

มีค่าทั้งหมด **4 ตัว** ที่ต้องกรอก ซึ่งตอนนี้ยังเป็นค่า placeholder อยู่ และแอปจะใช้ไม่ได้จนกว่าจะกรอกครบ:

| ไฟล์ | ค่าคงที่ | คืออะไร |
| --- | --- | --- |
| `Code.gs` | `SPREADSHEET_ID` | ID ของ Google Sheet (จากใน URL) |
| `Code.gs` | `DRIVE_FOLDER_ID` | ID ของโฟลเดอร์ Drive ที่เก็บสลิป |
| `Code.gs` | `TELEGRAM_BOT_TOKEN` | จาก @BotFather |
| `Code.gs` | `TELEGRAM_CHAT_ID` | แชทที่จะรับการแจ้งเตือน |
| `index.html` | `GOOGLE_CLIENT_ID` | OAuth Web client ID (สำหรับล็อกอิน) |
| `index.html` | `SCRIPT_URL` | URL `/exec` ของ Apps Script Web App |

---

## 1. หลังบ้าน — Google Sheets + Apps Script

1. สร้าง (หรือเปิด) Google Sheet แล้วคัดลอก ID จาก URL: `docs.google.com/spreadsheets/d/`**`ส่วนนี้`**`/edit`
2. สร้างโฟลเดอร์ Drive สำหรับเก็บรูปสลิป แล้วคัดลอก ID จาก URL
3. ในชีต ไปที่ **Extensions ▸ Apps Script** ลบโค้ดเดิมทิ้ง แล้ววางทั้งหมดจาก `Code.gs` นอกจากนี้ให้สร้างไฟล์ `appsscript.json` ในโปรเจกต์เดียวกัน (ต้องเปิดตัวเลือก "Show appsscript.json manifest file" ใน Project Settings ของ editor ก่อน) แล้ววางเนื้อหาจากไฟล์ `appsscript.json` ของโปรเจกต์นี้ — ไฟล์นี้ประกาศสิทธิ์ (scopes) ที่สคริปต์ต้องใช้สำหรับ Sheets, Drive และ external request
4. กรอกค่าทั้ง 4 ตัวด้านบนของไฟล์: `SPREADSHEET_ID`, `DRIVE_FOLDER_ID`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`
5. **อนุญาตสิทธิ์ให้สคริปต์ (ขั้นตอนนี้คือตัวที่ทำให้ Telegram และการอัปโหลดสลิปทำงาน)** ในหน้า editor เลือกฟังก์ชันใดก็ได้ (เช่น `todayTab`) จาก dropdown แล้วกด **Run** หนึ่งครั้ง Google จะขอสิทธิ์ — กดอนุญาต ขั้นตอนนี้จะให้สิทธิ์ *external request* (ทำให้เรียก Telegram ได้) และสิทธิ์ *Drive* (ทำให้อัปโหลดสลิปได้) **ถ้าข้ามขั้นตอนนี้ รายการยังบันทึกได้ แต่ Telegram จะไม่ส่งและการอัปโหลดจะล้มเหลวแบบไม่มีข้อความ**
6. กด **Deploy ▸ New deployment ▸ Web app**:
   - **Execute as:** Me
   - **Who has access:** Anyone
7. คัดลอก **Web app URL** (ลงท้ายด้วย `/exec`) เอาไปวางในไฟล์ `index.html` ที่ค่า `SCRIPT_URL`

> ทุกครั้งที่แก้ `Code.gs` ต้องสร้าง **deployment ใหม่** (หรือ "Manage deployments ▸ edit ▸ New version") การเปลี่ยนแปลงถึงจะมีผล

---

## 2. Telegram bot

1. ใน Telegram ค้นหา **@BotFather** ส่ง `/newbot` ทำตามขั้นตอน แล้วคัดลอก **bot token** — คือ `TELEGRAM_BOT_TOKEN`
2. หา **chat ID**:
   - ถ้าจะแจ้งเตือนเข้า **ตัวเอง**: ส่งข้อความถึงบอทสร้างใหม่หนึ่งครั้ง (ส่งข้อความอะไรก็ได้ — บอทจะเริ่มคุยกับเราก่อนไม่ได้) แล้วเปิด `https://api.telegram.org/bot<TOKEN_ของบอท>/getUpdates` ในเบราว์เซอร์ อ่านค่า `chat.id`
   - ถ้าเข้า **กลุ่ม**: เพิ่มบอทเข้ากลุ่ม ส่งข้อความในกลุ่ม แล้วดู `chat.id` ของกลุ่มใน `getUpdates` (จะเป็นเลขติดลบ)
3. นำค่านั้นไปใส่ใน `TELEGRAM_CHAT_ID`

> อาการ "chat not found" เกือบทั้งหมดเกิดจากยังไม่เคยส่งข้อความถึงบอทก่อน หรือใส่ chat ID ผิด

---

## 3. Google Sign-In — OAuth client

จำเป็นต่อการ ปุ่มล็อกอินจะไม่ปรากฏเลยถ้าไม่ทำ

1. ไปที่ **console.cloud.google.com ▸ APIs & Services ▸ Credentials**
2. **Create Credentials ▸ OAuth client ID ▸ Application type: Web application**
3. ในช่อง **Authorized JavaScript origins** ใส่ origin ที่ใช้โฮสต์แอปเป๊ะๆ เช่น `https://yourname.github.io` (scheme + host เท่านั้น **ห้ามมี path ห้ามมี / ท้าย**) ใส่ทุก origin ที่จะใช้ รวมถึง origin ของ dev ถ้าทดสอบบนเครื่องผ่าน HTTPS
4. คัดลอก **Client ID** — คือ `GOOGLE_CLIENT_ID`

> ถ้าปุ่มล็อกอินขึ้นแล้วแต่กดแล้วไม่มีอะไรเกิดขึ้น เปิด console ของเบราว์เซอร์ดู ถ้าเจอ `origin is not allowed for the given client ID` แปลว่า origin ในขั้นตอนที่ 3 ไม่ตรงกับที่เสิร์ฟหน้าเว็บอยู่จริง

ถ้า `GOOGLE_CLIENT_ID` ยังเป็นค่า placeholder อยู่ หรือสคริปต์ Google Identity Services โหลดไม่สำเร็จ (เช่น ถูก ad blocker บล็อก หรือไม่มีเน็ต) หน้าล็อกอินจะมีข้อความแจ้งเตือนบอกปัญหาแทนที่จะเป็นพื้นที่เปล่าๆ

---

## 4. หน้าเว็บ — `index.html`

1. ใกล้ด้านบนของบล็อก `<script>` ตั้งค่า:
   ```js
   const GOOGLE_CLIENT_ID = "…client ID ของคุณ…";
   const SCRIPT_URL       = "…URL /exec ของคุณ…";
   ```
   สองค่านี้เป็นค่าคงที่ในไฟล์ — ไม่มีช่องตั้งค่าในแอปให้กรอก
2. โฮสต์โฟลเดอร์นี้แบบ HTTPS (GitHub Pages / Netlify / Cloudflare Pages) แล้วเปิดหน้าเว็บที่ URL นั้น
3. ตอนนี้ service worker จะอัปเดต app shell ที่แคชไว้ให้อัตโนมัติ: หน้า HTML จะโหลดจากเน็ตก่อน (network-first) แล้วใช้แคชสำรองเฉพาะตอนออฟไลน์ ส่วนไฟล์สแตติก เช่น ไอคอนและ `i18n/languages.js` ยังใช้แคชก่อน (cache-first) โดยทั่วไปไม่ต้องเพิ่มเลขเวอร์ชันแคชทุกครั้งที่แก้แล้ว — แต่ถ้าเปลี่ยนชื่อไฟล์สแตติก ให้เพิ่มค่า `CACHE` ใน `service-worker.js` เพื่อล้างไฟล์เก่าออก

---

## เจอปัญหาที่นี่ต่อเลย

| อาการ | สาเหตุที่เป็นไปได้ | วิธีแก้ |
| --- | --- | --- |
| ไม่มีปุ่ม "Sign in with Google" หรือมีข้อความ "ยังไม่ได้ตั้งค่า" | `GOOGLE_CLIENT_ID` ยังเป็น placeholder | ตั้งค่า (§3–4) |
| มีข้อความ "โหลดไม่สำเร็จ" หรือปุ่มขึ้นแล้วกดไม่มีอะไรเกิดขึ้น | สคริปต์ Google Identity Services ถูกบล็อก/โหลดไม่สำเร็จ หรือ origin ไม่ได้ whitelist หรือเปิดบน `file://`/`http://` | เพิ่ม origin ให้ตรงกับ OAuth client; โฮสต์แบบ HTTPS; เช็ค ad blocker |
| บันทึกได้แต่ไม่มีข้อความ Telegram | สคริปต์ยังไม่ได้รับสิทธิ์ external request หรือ chat ID/token ผิด หรือยังไม่เคยส่งข้อความถึงบอทก่อน | กด Run ฟังก์ชันใดก็ได้หนึ่งครั้งเพื่ออนุญาตสิทธิ์ (§1.5); ตรวจ bot/chat (§2) — ตอนนี้ผลลัพธ์จากเซิร์ฟเวอร์จะมี `telegramOk`/`telegramError` บอกสาเหตุที่แท้จริง |
| อัปโหลดสลิปล้มเหลว | สิทธิ์ยังไม่ได้รับ หรือ `DRIVE_FOLDER_ID` ผิด | อนุญาตสิทธิ์ (§1.5); ตรวจ folder ID |
| แก้โค้ดแล้วแต่มือถือยังไม่เปลี่ยน | แคช service worker เก่า | service worker เป็น network-first สำหรับ HTML แล้ว แค่รีโหลดก็ควรเห็นการเปลี่ยนแปลง ถ้ายังไม่เปลี่ยน ให้เพิ่มค่า `CACHE` ใน `service-worker.js` แล้วติดตั้งใหม่ |

> **Telegram กับอักขระพิเศษ:** การแจ้งเตือนส่งแบบ HTML และทุกฟิลด์ถูก HTML-escape แล้ว ดังนั้นชื่อรายการหรือชื่อสาขาที่มี `_`, `*`, `` ` ``, หรือ `[` จะปลอดภัย ไม่ทำให้ข้อความพัง (เวอร์ชันก่อนหน้าใช้ Markdown ซึ่งอักขระเหล่านี้ทำให้ Telegram ปฏิเสธข้อความด้วย HTTP 400 แบบเงียบๆ)

---

## การเพิ่มความปลอดภัย (ทางเลือก)

ถ้าต้องการให้การล็อกอินเป็นการยืนยันตัวตนจริงไม่ใช่แค่ประตูหลวมๆ:
- ส่ง credential (JWT) ของ Google ไปยัง backend ทุกครั้งที่เขียนข้อมูล แล้วตรวจสอบที่ฝั่งเซิร์ฟเวอร์ (ตรวจ signature, `aud` = client ID ของคุณ, และอีเมลอยู่ใน allow-list) ก่อนเพิ่มแถวข้อมูล
- คงค่า "Who has access: Anyone" ไว้ (Apps Script ต้องใช้ค่านี้เพื่อเลี่ยง redirect ล็อกอิน) แต่ปฏิเสธ request ที่ token หายหรือไม่ถูกต้องภายใน `doPost`

---

## โครงสร้างระบบ

| ส่วน | เทคโนโลยี |
| --- | --- |
| Frontend | PWA แบบ HTML/CSS/JS เดียว รองรับ 5 ภาษา (`i18n/languages.js`) |
| Backend | Google Apps Script (web app `doGet`/`doPost`) ประกาศสิทธิ์ใน `appsscript.json` |
| ที่เก็บข้อมูล | Google Sheets (แท็บละวัน) + Google Drive (สลิป) |
| การแจ้งเตือน | Telegram Bot API (ส่งจากฝั่งเซิร์ฟเวอร์ HTML-escape แล้ว) |

© 2026 FelisiaCH — สัญญาอนุญาต MIT
