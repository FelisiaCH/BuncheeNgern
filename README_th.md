# FinTrack

[Read the English guide here](README.md)

**FinTrack** คือ Progressive Web App (PWA) ประสิทธิภาพสูง ออกแบบมาสำหรับการบันทึกรายรับ-รายจ่ายแบบรองรับหลายภาษาในภูมิภาค ใช้งานผ่านมือถือได้ทันที เข้าสู่ระบบด้วยบัญชี Google บันทึกรายการได้ทุกที่ และแจ้งเตือนทีมงานผ่าน Telegram แบบเรียลไทม์

![Version](https://img.shields.io/badge/version-1.1.0-2EE8B4)
![Platform](https://img.shields.io/badge/platform-PWA-4285F4)
![Auth](https://img.shields.io/badge/sign--in-Google-EA4335)
![Notifications](https://img.shields.io/badge/notifications-Telegram-26A5E4)
![Languages](https://img.shields.io/badge/languages-5%20supported-success)

---

## ส่วนที่ 1: ภาพรวมและฟีเจอร์เด่นของ FinTrack (v1.1.0)

- **🔐 ระบบล็อกอิน Google ไร้รหัสผ่าน** — ยืนยันตัวตนได้อย่างรวดเร็วและปลอดภัยด้วยบัญชี Google ของคุณ ไม่ต้องตั้งรหัสผ่านหรือจดจำบัญชีแยกใดๆ
- **📲 แจ้งเตือนเข้า Telegram ทันที** — ทุกครั้งที่บันทึกรายการ ระบบจะส่งสรุปข้อมูลแบบเรียลไทม์ไปยังช่อง Telegram ของฝ่ายบริหารโดยอัตโนมัติ
- **⚙️ หน้าตั้งค่าแยกส่วน** — หน้าตั้งค่าแบบเต็มหน้าจอ แยกออกจากหน้าบันทึกรายการหลักอย่างชัดเจน
- **🌗 สลับโหมดมืด/สว่าง** — สลับธีมการแสดงผลได้ทันที และบันทึกสเตทไว้ใน `localStorage` เพื่อจดจำการตั้งค่าในครั้งถัดไป
- **🌐 รองรับ 5 ภาษาในภูมิภาค** — รองรับภาษาอังกฤษ, ไทย, ลาว, เวียดนาม และพม่า แบบครบถ้วน
- **📱 ติดตั้งเป็นแอปได้** — เพิ่ม FinTrack ลงหน้าโฮมสกรีนเพื่อใช้งานเหมือนแอปจริง พร้อมรองรับการใช้งานออฟไลน์

---

## ส่วนที่ 2: คู่มือการใช้งานสำหรับพนักงานและผู้ใช้ทั่วไป

1. **เข้าสู่ระบบ** — เปิดลิงก์ FinTrack แล้วแตะ **"Sign in with Google"** เพื่อยืนยันตัวตนด้วยบัญชี Google ของคุณ
2. **บันทึกรายการ** — กรอกรายละเอียด: จำนวนเงิน, ชื่อรายการ, สาขา/สถานที่, ชื่อผู้บันทึก และสกุลเงิน
3. **บันทึกข้อมูล** — แตะ **"Save Entry"** เพื่อบันทึกรายการ ระบบจะส่งสรุปข้อมูลไปยังช่อง Telegram ของฝ่ายบริหารโดยอัตโนมัติ
4. **ปรับแต่งการใช้งาน** — เข้าหน้า **Settings** ได้ทุกเมื่อ เพื่อสลับโหมดสว่าง/มืด หรือเปลี่ยนภาษาที่แสดง

---

## ส่วนที่ 3: ขั้นตอนการตั้งค่าระบบหลังบ้าน

FinTrack ทำงานบน Google Sheets ร่วมกับ Apps Script หากต้องการติดตั้งระบบของคุณเอง:

1. เปิด Google Sheet ของคุณ แล้วไปที่ **ส่วนขยาย ▸ Apps Script** (Extensions ▸ Apps Script)
2. นำเนื้อหาจากไฟล์ `Code.gs` ไปวางแทนที่
3. ตั้งค่าโทเค็น Telegram บอทที่ด้านบนของไฟล์ `Code.gs`:

```
TELEGRAM_BOT_TOKEN = 'your-telegram-bot-token'
TELEGRAM_CHAT_ID   = 'your-telegram-chat-id'
```

4. คลิก **Deploy ▸ New deployment** เลือกประเภท **Web app** และตั้งค่า:
   - **Execute as: Me**
   - **Who has access: Anyone**
5. คัดลอก Web App URL ที่ได้ แล้วนำไปกรอกในช่องเชื่อมต่อของแอป FinTrack

> **หมายเหตุ:** การตั้งค่า "Execute as: Me" และ "Who has access: Anyone" จำเป็นต่อการใช้งาน เพื่อให้แอปเชื่อมต่อกับ backend ได้โดยไม่เจอปัญหาการ redirect ยืนยันตัวตนหรือ CORS

---

## ภาพรวมโครงสร้างระบบ

| ส่วนประกอบ | เทคโนโลยี |
| --- | --- |
| Frontend | HTML5, CSS Variables, Progressive Web App (PWA) |
| Backend | Google Apps Script — Workspace Cloud Core |
| ฐานข้อมูล | Google Sheets |
| การแจ้งเตือน | Telegram Bot API |
