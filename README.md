# FinTrack

[อ่านคู่มือภาษาไทยที่นี่](README_th.md)

A mobile-first PWA for logging income & expenses. Staff sign in with Google, record a transaction, and the entry is appended to a Google Sheet (slip images go to Drive) while a Telegram message is sent to your management chat. Frontend is a single `index.html`; backend is one Google Apps Script file (`Code.gs`).

> **Heads-up on security:** Sign-in here is a *soft gate*, not real access control. The Google credential is read in the browser but never verified server-side, and the backend is deployed as "Anyone", so anyone who has the Web App URL can write to the sheet. That's fine for a trusted internal team that just needs "who logged this" on each entry — but don't treat it as auth for sensitive data. See [Hardening](#hardening-optional) if you need more.

---

## What you need before starting

- A Google account (for the Sheet, Drive folder, Apps Script, and OAuth client)
- A Telegram bot token + the chat ID to notify
- Somewhere to host `index.html` over **HTTPS** (GitHub Pages, Netlify, Cloudflare Pages). Google Sign-In does **not** work from `file://` or a plain `http://` origin.

There are **four** values you must fill in. Three of them are currently placeholders and the app will not work until all are set:

| Where | Constant | What it is |
| --- | --- | --- |
| `Code.gs` | `SPREADSHEET_ID` | ID of your Google Sheet (from its URL) |
| `Code.gs` | `DRIVE_FOLDER_ID` | ID of the Drive folder where slips are stored |
| `Code.gs` | `TELEGRAM_BOT_TOKEN` | From @BotFather |
| `Code.gs` | `TELEGRAM_CHAT_ID` | The chat that receives notifications |
| `index.html` | `GOOGLE_CLIENT_ID` | OAuth Web client ID (for sign-in) |
| `index.html` | `SCRIPT_URL` | The Apps Script Web App `/exec` URL |

---

## 1. Backend — Google Sheets + Apps Script

1. Create (or open) a Google Sheet. Copy its ID from the URL: `docs.google.com/spreadsheets/d/`**`THIS_PART`**`/edit`.
2. Create a Drive folder for slip images and copy its ID from the URL.
3. In the Sheet, go to **Extensions ▸ Apps Script**. Delete the default code and paste in all of `Code.gs`. Also create `appsscript.json` in the same project (the editor's "Project Settings" must have "Show appsscript.json manifest file" enabled) and paste in the contents of this repo's `appsscript.json` — it declares the Sheets, Drive, and external-request scopes the script needs.
4. Fill in the four constants at the top: `SPREADSHEET_ID`, `DRIVE_FOLDER_ID`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`.
5. **Authorize the script (this is the step that makes Telegram and slip upload work).** In the editor, pick any function (e.g. `todayTab`) from the dropdown and click **Run** once. Google will prompt for permissions — approve them. This grants the *external request* permission (needed to call Telegram) and the *Drive* permission (needed to upload slips). **If you skip this, entries still save but no Telegram message is sent and uploads fail silently.**
6. Click **Deploy ▸ New deployment ▸ Web app**:
   - **Execute as:** Me
   - **Who has access:** Anyone
7. Copy the **Web app URL** (ends in `/exec`). You'll paste it into `index.html` as `SCRIPT_URL`.

> Every time you change `Code.gs`, you must create a **new deployment** (or "Manage deployments ▸ edit ▸ New version") for the change to go live.

---

## 2. Telegram bot

1. In Telegram, message **@BotFather**, send `/newbot`, follow the prompts, and copy the **bot token** — that's `TELEGRAM_BOT_TOKEN`.
2. Get the **chat ID**:
   - For a notification to **yourself**: message your new bot once (send it any text — a bot cannot start a conversation with you), then open `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates` in a browser and read `chat.id`.
   - For a **group**: add the bot to the group, send a message in the group, then check `getUpdates` for the group's `chat.id` (it's negative).
3. Put that value in `TELEGRAM_CHAT_ID`.

> The "chat not found" failure almost always means you never sent the bot a message first, or the chat ID is wrong.

---

## 3. Google Sign-In — OAuth client

This is required for the sign-in button to appear at all.

1. Go to **console.cloud.google.com ▸ APIs & Services ▸ Credentials**.
2. **Create Credentials ▸ OAuth client ID ▸ Application type: Web application**.
3. Under **Authorized JavaScript origins**, add the exact origin where you host the app — e.g. `https://yourname.github.io` (scheme + host, **no path, no trailing slash**). Add every origin you'll use, including a dev origin if you test locally over HTTPS.
4. Copy the **Client ID** — that's `GOOGLE_CLIENT_ID`.

> If the sign-in button shows but clicking it does nothing, open the browser console — `origin is not allowed for the given client ID` means the origin in step 3 doesn't exactly match where the page is served.

If `GOOGLE_CLIENT_ID` is still a placeholder, or the Google Identity Services script fails to load (e.g. blocked by an ad blocker or no network), the login screen now shows an inline hint explaining what's wrong instead of just leaving an empty space.

---

## 4. Frontend — `index.html`

1. Near the top of the `<script>` block, set:
   ```js
   const GOOGLE_CLIENT_ID = "…your client ID…";
   const SCRIPT_URL       = "…your /exec URL…";
   ```
   These are plain constants in the file — there is no in-app settings field for them.
2. Host the folder over HTTPS (GitHub Pages / Netlify / Cloudflare Pages). Open the page on that URL.
3. The service worker now updates the cached app shell automatically: HTML pages are fetched network-first (falling back to cache only when offline), while static assets like icons and `i18n/languages.js` stay cache-first. You generally no longer need to manually bump the cache version after every change — but if you do change static asset filenames, bump `CACHE` in `service-worker.js` so old assets are evicted.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| No "Sign in with Google" button, or an inline "not configured" hint | `GOOGLE_CLIENT_ID` still a placeholder | Set it (§3–4) |
| Inline "failed to load" hint, or button appears but clicking does nothing | Google Identity Services script blocked/failed, or origin not whitelisted, or page on `file://`/`http://` | Add exact origin in OAuth client; host over HTTPS; check for ad blockers |
| Entry saves but no Telegram message | Script never authorized for external requests, or chat ID/token wrong, or the bot was never messaged first | Run a function once to authorize (§1.5); verify bot/chat (§2). The server response now includes `telegramOk`/`telegramError` so you can see the exact failure. |
| Slip upload fails | Same authorization gap, or wrong `DRIVE_FOLDER_ID` | Authorize (§1.5); check folder ID |
| Fixed the code but app unchanged on phone | Stale service-worker cache | The service worker is now network-first for HTML, so a normal reload should pick up changes; if not, bump `CACHE` in `service-worker.js` and reinstall |

> **Telegram + special characters:** notifications are sent with HTML formatting and every interpolated field is HTML-escaped, so item or branch names containing `_`, `*`, `` ` ``, or `[` are safe and won't break the message. (Earlier versions used Markdown, where these characters caused silent HTTP 400 rejections.)

---

## Hardening (optional)

If you need sign-in to be real access control rather than a soft gate:
- Send the Google credential (JWT) to the backend on each write and verify it server-side (check signature, `aud` = your client ID, and that the email is on an allow-list) before appending the row.
- Keep "Who has access: Anyone" (Apps Script needs this to avoid login redirects), but reject requests whose token is missing/invalid inside `doPost`.

---

## Architecture

| Layer | Technology |
| --- | --- |
| Frontend | Single-file HTML/CSS/JS PWA, 5-language i18n (`i18n/languages.js`) |
| Backend | Google Apps Script (`doGet`/`doPost` web app), scopes declared in `appsscript.json` |
| Storage | Google Sheets (one tab per day) + Google Drive (slips) |
| Notifications | Telegram Bot API (sent server-side, HTML-escaped) |

© 2026 FelisiaCH — MIT License
