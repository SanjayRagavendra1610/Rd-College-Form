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

## Verify

1. Open the public site and submit a test registration using clearly labelled test data.
2. Confirm the form only shows success after the function confirms the Google Sheets write.
3. Open the spreadsheet and verify the new row.
4. If it fails, open Netlify → Functions → `submit-lead` logs and inspect the error. Do not paste private-key JSON into chat or logs.

## Notes

- The function creates column headers if the first worksheet is completely empty.
- The spreadsheet must be shared with the service account email as Editor.
- The existing admin portal still contains legacy Apps Script configuration controls for its old pull/sync dashboard. The public form submission path in this package no longer calls Apps Script. The old admin dashboard's cross-device read/delete/sync controls are not migrated by this change.
- Add a CAPTCHA or rate-limiting layer before large-scale public use to reduce spam. Do not claim a registration was saved unless the server returns success.
