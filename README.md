# RDCCPS Consultancy Form — Netlify Functions + Google Sheets API

The existing form design is retained in `index.html`. The submission handler now sends each registration to `/.netlify/functions/submit-lead`; the function writes it to the first worksheet in the configured Google Spreadsheet. This does not use Google Apps Script.

## Before deploying

1. In Google Cloud Console, create/select a project and enable **Google Sheets API**.
2. Create a **service account** and create a JSON key. Keep the JSON private.
3. Open the target Google Sheet and share it with the service account's `client_email` as **Editor**. Do not make the sheet public.
4. Set `GOOGLE_SHEET_ID` to the target spreadsheet ID in Netlify.
5. In Netlify, open the site settings and add these environment variables under **Project configuration → Environment variables**:
   - `GOOGLE_SHEET_ID` = spreadsheet ID
   - `GOOGLE_SERVICE_ACCOUNT_JSON` = the entire service-account JSON as a single-line JSON value
   - `ALLOWED_ORIGIN` = the production origin of your Netlify site
   - `ADMIN_ACCESS_KEY` = a strong organizer-only key used to sign in to the Admin Responses page
   Ensure the variables are available to Functions. Never put the service-account JSON in HTML, JavaScript, Git, or a public repository.
6. Redeploy the site after setting the variables.

## Deploy from VS Code

```bash
npm install
npm run dev
```

Netlify CLI starts the site locally, usually at `http://localhost:8888`. For local testing, set `ALLOWED_ORIGIN=http://localhost:8888` in a local `.env` file (never commit `.env`). Netlify Dev reads local `.env` values.

To deploy with the Netlify CLI after linking the existing site:

```bash
npx netlify login
npx netlify link
npx netlify deploy --build --prod
```

Alternatively, push the project to the Git repository connected to the existing Netlify site and let Netlify deploy it. Check that the publish directory is the project root because `index.html` is there.

## Response storage and admin access

- Public form submissions are appended by `submit-lead` to the configured Google Sheet. The form shows success only after the Sheets API confirms the write.
- `admin-auth` checks the organizer key on the server and returns a short-lived session token. The key is never embedded in the public page.
- `admin-leads` uses that token to read, refresh, and delete responses in the same Google Sheet. The admin page refreshes from the sheet every 15 seconds, so submissions from other devices appear without using local browser storage.
- The spreadsheet must stay private and be shared with the service account email as Editor.
- Never use the admin key as the college phone number or another publicly known value. Do not paste service-account JSON or admin keys into chat, Git, or logs.

## Notes

- The function creates column headers if the first worksheet is completely empty.
- Add a CAPTCHA or rate-limiting layer before large-scale public use to reduce spam. Do not claim a registration was saved unless the server returns success.
