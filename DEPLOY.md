# Deploying the invitation and saving feedback

## 1. Font (Bradley Hand)

Download Bradley Hand from [fonts-online.ru](https://fonts-online.ru/fonts/bradley-hand) (personal use).  
Place in the project root:

- `BradleyHand.woff2` (preferred) and/or  
- `BradleyHand.woff`, `BradleyHand.ttf`

The page already references these in `@font-face`. If the files are in a `fonts/` folder, update paths in `index.html` to e.g. `url('./fonts/BradleyHand.woff2')`.

## 2. Images

Ensure these files are in the same folder as `index.html` (or update paths in the HTML):

- `bg.png` — background
- `side ornaments light.png` — side decorations
- `hero-sun.png` — rotating sun in hero
- `divider.png` — section dividers

## 3. Deploying the site

- **Netlify / Vercel:** Drag the project folder to [Netlify Drop](https://app.netlify.com/drop) or connect the repo to [Vercel](https://vercel.com). No build step needed; serve the folder as static files.
- **GitHub Pages:** Push the repo, enable Pages in repo Settings → Pages, choose branch and root (or `/docs` if you use that folder).
- **Any static host:** Upload the folder (with `index.html`, images, and font files) to your host’s web root.

## 4. Saving feedback form data

The form sends form-encoded fields: `name` and `attend` (`will_be` or `will_not_be`).

### Option A: Formspree (no backend)

1. Sign up at [formspree.io](https://formspree.io).
2. Create a form; you’ll get a URL like `https://formspree.io/f/xxxxx`.
3. In `index.html`, set:
   ```js
   const FORM_ENDPOINT = 'https://formspree.io/f/xxxxx';
   ```
4. Formspree expects form-encoded data by default. Either:
   - Change the submit code to send as `application/x-www-form-urlencoded` and use Formspree’s field names, or
   - Use Formspree’s “AJAX” endpoint and keep sending JSON if your plan supports it (or use their recommended format).

Easiest is to post as form data. In the script, replace the fetch with:

```js
const form = new FormData();
form.append('name', name);
form.append('attend', attendValue);
fetch(FORM_ENDPOINT, { method: 'POST', body: form })
  .then(() => showThankYou())
  .catch(() => { console.log('Form data:', payload); showThankYou(); });
```

Responses will appear in the Formspree dashboard and can be emailed to you.

### Option B: Netlify Forms (if you deploy on Netlify)

1. Add `netlify` attribute to the form:
   ```html
   <form class="rsvp-form" id="rsvpForm" netlify>
   ```
2. Add hidden inputs so Netlify can capture them:
   ```html
   <input type="hidden" name="form-name" value="rsvpForm">
   <!-- and use name="name", name="attend" -->
   ```
3. Submit as normal (or use Netlify’s form redirect). Submissions appear under Netlify dashboard → Forms.

### Option D: Google Sheets + email (free, no submission limit)

Use **Google Apps Script** to append each submission to a Google Sheet and optionally email yourself. No third-party signup and no submission cap.

**1. Create a Google Sheet**

- Go to [sheets.new](https://sheets.new).
- In row 1, add headers: `Name` | `Attend` | `Date` (or similar).
- Note the Sheet ID from the URL: `https://docs.google.com/spreadsheets/d/ **SHEET_ID** /edit`.

**2. Add the script**

- In the sheet: **Extensions → Apps Script**.
- Delete any sample code and paste this (replace `YOUR_SHEET_ID` and, if you want email, `your@email.com`):

```js
const SHEET_ID = 'YOUR_SHEET_ID';
const NOTIFY_EMAIL = 'your@email.com';  // set to '' to disable email

function doPost(e) {
  try {
    var params = e.parameter || {};
    var name = params.name || '';
    var attend = params.attend || '';
    var sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();
    sheet.appendRow([name, attend, new Date()]);

    if (NOTIFY_EMAIL) {
      MailApp.sendEmail(NOTIFY_EMAIL, 'New RSVP: ' + name, 'Name: ' + name + '\nAttend: ' + attend);
    }

    return ContentService.createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeader('Access-Control-Allow-Origin', '*');
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeader('Access-Control-Allow-Origin', '*');
  }
}
```

**3. Deploy as web app**

- **Deploy → New deployment** → type: **Web app**.
- Description: e.g. `RSVP endpoint`.
- **Execute as:** Me; **Who has access:** Anyone.
- Click **Deploy**, copy the **Web app URL** (looks like `https://script.google.com/macros/s/xxxxx/exec`).

**4. Use the URL in your site**

In `index.html`, set:

```js
const FORM_ENDPOINT = 'https://script.google.com/macros/s/xxxxx/exec';
```

Your form already sends `name` and `attend` as form data, so no code changes are needed. New rows appear in the sheet; if you set `NOTIFY_EMAIL`, you’ll get an email per submission.

### Option E: Your own backend (API)

Set `FORM_ENDPOINT` to your API URL. The script sends a POST with form fields `name` and `attend`. Implement a handler that saves to a database or spreadsheet and returns 2xx on success.

---

**Quick test without backend:** Leave `FORM_ENDPOINT` as `''`. Submissions will only be logged in the browser console and the thank-you message will still show.
