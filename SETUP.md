# Futbol Fitness – Google Sheets Backend Setup

This site can send every registration straight into a **private Google Sheet** that only you can see.  
No paid services needed — everything runs on free Google tools.

---

## Step 1 – Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet.
2. Name it something like **Futbol Fitness Registrations**.
3. In the first row, add these column headers (exactly as shown):

   | A | B | C | D | E | F | G | H | I |
   |---|---|---|---|---|---|---|---|---|
   | Timestamp | First Name | Last Name | Email | Phone | Session ID | Session | Experience | Notes |

4. Copy the full URL of the sheet from the browser address bar.  
   You will paste this into `index.html` later as `GOOGLE_SHEET_URL`.

**Keep the sheet private** (Share → Restricted / Only people with access).  
That is your private dashboard.

---

## Step 2 – Create the Apps Script (the backend)

1. In the same Google Sheet, go to **Extensions → Apps Script**.
2. Delete any code that appears and paste the following:

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = JSON.parse(e.postData.contents);

    sheet.appendRow([
      data.timestamp || new Date().toISOString(),
      data.firstName || '',
      data.lastName || '',
      data.email || '',
      data.phone || '',
      data.sessionId || '',
      data.sessionLabel || '',
      data.experience || '',
      data.notes || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: allow GET so you can test the URL in a browser
function doGet(e) {
  return ContentService
    .createTextOutput('Futbol Fitness registration endpoint is live.')
    .setMimeType(ContentService.MimeType.TEXT);
}
```

3. Click **Save** (disk icon) and give the project a name (e.g. “Futbol Registrations”).
4. Click **Deploy → New deployment**.
5. Click the gear icon next to “Select type” and choose **Web app**.
6. Fill in:
   - **Description**: Registrations
   - **Execute as**: Me
   - **Who has access**: Anyone
7. Click **Deploy**.
8. Authorize the app when Google asks (choose your account → Advanced → Go to … → Allow).
9. Copy the **Web app URL** that appears (it looks like  
   `https://script.google.com/macros/s/AKfycb.../exec`).

---

## Step 3 – Connect the website

Open `index.html` in a text editor and find the CONFIG block near the top of the `<script>` section:

```js
const GOOGLE_SCRIPT_URL = '';          // ← paste Web App URL here
const GOOGLE_SHEET_URL  = '';          // ← paste your Sheet URL here
const OWNER_PASSWORD    = 'futbol2026'; // ← change this!
```

- Paste the **Web app URL** into `GOOGLE_SCRIPT_URL`.
- Paste the **Sheet URL** into `GOOGLE_SHEET_URL`.
- Change `OWNER_PASSWORD` to a password only you know.

Save the file.

---

## Step 4 – Test

1. Open `index.html` in a browser.
2. Fill out the registration form and submit.
3. Check your Google Sheet — a new row should appear within a few seconds.
4. Click **Owner login** in the footer of the website, enter your password, and you should see a link that opens the Sheet.

---

## Notes

- The form works even if you leave the URLs empty (demo mode) — it just shows the success message without saving data.
- Because the site uses `mode: 'no-cors'`, the browser will always treat the request as successful once it is sent. If rows are not appearing, double-check that you deployed the script as a **Web app** with access set to **Anyone**.
- To update the Apps Script later: edit the code → Deploy → Manage deployments → Edit (pencil) → Version: New version → Deploy.
- The “Owner login” is a simple client-side check. It is convenient, not military-grade security. Real privacy comes from keeping the Google Sheet sharing set to **Only you**.

That’s it — you now have a free, private backend for all registrations.
