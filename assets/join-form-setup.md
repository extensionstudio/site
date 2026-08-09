# Wire the "Join the collective" form → Google Sheet

The form is built and on-brand. To make submissions land in a Google Sheet, do this once (~3 minutes):

## 1. Make the Sheet
- Create a new Google Sheet (name it e.g. "Extension — Applicants").

## 2. Add the script
- In that Sheet: **Extensions → Apps Script**.
- Delete whatever's there and paste this:

```javascript
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);
  try {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName('Applicants') || ss.insertSheet('Applicants');
    if (sheet.getLastRow() === 0) {
      sheet.appendRow(['Timestamp','Name','Email','Craft','Location','Portfolio','Tools','Note']);
    }
    var d = e.parameter;
    sheet.appendRow([
      new Date(), d.name || '', d.email || '', d.craft || '',
      d.location || '', d.portfolio || '', d.tools || '', d.note || ''
    ]);
    return ContentService.createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

## 3. Deploy it as a Web App
- **Deploy → New deployment → (gear) Web app**.
- Description: anything. **Execute as: Me**. **Who has access: Anyone**.
- **Deploy**, authorize when prompted, and **copy the Web app URL** (ends in `/exec`).

## 4. Paste the URL into the site
- In `index.html`, find this line (in the inline script near the bottom):
  ```
  var ENDPOINT='YOUR-GOOGLE-SCRIPT-URL';
  ```
- Replace `YOUR-GOOGLE-SCRIPT-URL` with the `/exec` URL you copied. Save and redeploy the site.

That's it. Every submission appends a row: Timestamp, Name, Email, Craft, Location, Portfolio, Tools, Note.

## Notes
- The form posts with `mode:'no-cors'`, so the browser never needs CORS headers from Apps Script — submissions just work. The site shows the success screen once the request goes through.
- There's a hidden honeypot field (`_gotcha`) that silently drops bots.
- Want email alerts on each submission? Add this inside `doPost`, before the `return`:
  ```javascript
  MailApp.sendEmail('you@extstudio.com', 'New applicant: ' + (d.name||''),
    d.name + ' — ' + d.craft + ' — ' + d.location + '\n' + d.email + '\n' + d.portfolio);
  ```
