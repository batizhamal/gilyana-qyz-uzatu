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

The form sends a JSON body: `{ "name": "...", "attend": "will_be" | "will_not_be" }`.

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

### Option C: Your own backend

Set `FORM_ENDPOINT` to your API URL (e.g. `https://your-api.com/rsvp`). The script sends a POST with JSON:

```json
{ "name": "Guest Name", "attend": "will_be" }
```

Implement a handler that saves to a database or spreadsheet and returns 2xx on success.

### Option D: Google Sheets

Use a serverless function (e.g. Netlify Function, Vercel Serverless) or a service like [SheetDB](https://sheetdb.io/) / [Google Apps Script Web App](https://developers.google.com/apps-script/guides/web) that accepts POST and appends a row to a Google Sheet. Point `FORM_ENDPOINT` to that URL.

---

**Quick test without backend:** Leave `FORM_ENDPOINT` as `''`. Submissions will only be logged in the browser console and the thank-you message will still show.
