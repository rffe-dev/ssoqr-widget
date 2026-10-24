# PayNow QR widget for Jotform (proof of concept)

Shows a branded PayNow QR code (PayNow purple, optional logo) inside a Jotform form,
drawn from the `paynow_payload` that the Spreadsheet to Form lookup fills in.

This repository holds **code only**. Never put member data, spreadsheets or payment
texts in it. The payment text is read from the form inside the parent's browser and is
never sent anywhere by this widget.

## Files

| File | What it is |
|---|---|
| `index.html` | The widget |
| `qrcode.js` | Draws QR codes (qrcode-generator, MIT licence) |
| `jsQR.js` | Reads the drawn code back to check it (jsQR 1.4.0, Apache 2.0 licence) |
| `logo.png` | Optional: the logo for the centre of the QR code (add your own) |

## Part 1: Put it on GitHub Pages

1. Sign in to GitHub (for real use, an SSO-owned account rather than a personal one).
2. Click **+** (top right), then **New repository**.
   - Name: `paynow-qr-widget`
   - Visibility: **Public** (GitHub Pages is free for public repositories)
   - Click **Create repository**.
3. On the new repository page, click **uploading an existing file**, drag in
   `index.html`, `qrcode.js`, `jsQR.js` and `README.md` (and `logo.png` if you have one),
   then click **Commit changes**.
4. Go to **Settings**, then **Pages** (left menu).
   - Source: **Deploy from a branch**
   - Branch: **main**, folder **/ (root)**, then **Save**.
5. Wait 1 to 2 minutes and refresh. The page shows your address, for example
   `https://YOUR-NAME.github.io/paynow-qr-widget/`
6. Open that address. You should see a box saying "Standalone test". Paste one of the
   `paynow_payload` values from your test spreadsheet: the QR code should appear with the
   amount and reference. If it does, hosting works.

**Logo:** use a square-ish PNG with a transparent background, at least 200 x 200 pixels,
named `logo.png`. The widget checks every code it draws; if the logo stops the code
from reading back correctly, it automatically shows the code without the logo.

## Part 2: Register it in Jotform

1. In Jotform, open the developers' widget page (jotform.com/developers/widgets) and
   choose to create a new widget.
2. Type: **iFrame Widget**.
3. Widget URL: your GitHub Pages address from Part 1, step 5.
4. Default size: width **400**, height **560** (the widget resizes itself afterwards).
5. Add these **parameters** (parameter name exactly as shown, then a friendly label):

   | Parameter name | Label | What to enter |
   |---|---|---|
   | `field` | Payment text field | The field holding `paynow_payload`, e.g. `typeA11` |
   | `logo` | Logo | `logo.png` (or leave blank for no logo) |
   | `color` | QR colour | `7C1A78` (PayNow purple) |
   | `payeeName` | Pays to (shown under the code) | `Singapore Symphony Orchestra` |
   | `size` | Display size in pixels | `280` |
   | `debug` | Show test details | `1` while testing, blank when live |

6. Save. The widget is private to your account, so no Jotform approval is needed.

## Part 3: Test in your form

1. Add the widget to **page 2** of the test form (after the fields the lookup fills in).
2. Fill in the parameters as in the table above, with `debug` set to `1`.
3. Preview the form, enter access code `27RQSLGG`, go to page 2.
4. Look at the grey test box under the widget:
   - `library methods:` lists what Jotform lets the widget do.
   - `getFieldsValueById(11) -> value found` means it can read the payment field. **Success.**
   - `no field-reading method available` means Jotform doesn't allow it. Try route 2 below.
5. If the QR code shows, scan it: S$1.00, reference TEST-001. Then try `27TPZWKN`
   (S$1.50, TEST-002) and check it changes.

**If it can't find the field:** the number Jotform uses may differ from the one in the
field's name. In the form preview, right-click the `paynow_payload` box, choose
**Inspect**, and look for `id="input_NN"`. Put that number (`NN`) in the `field` setting.

**Route 2 (if field reading isn't allowed):** add a form condition that copies the
`paynow_payload` field into this widget. The widget also accepts the payment text as
its own value. The test box will say `payment text from widget value` if this works.

When testing is finished, clear the `debug` setting so parents don't see the test box.

## What the widget does for safety

- Checks the payment text's checksum before drawing. A damaged or edited text shows
  "can't be shown, please contact us" instead of a QR code.
- Refuses expired codes and asks the parent to contact you.
- Reads every QR code it draws back with a separate reader, and only shows it if it
  decodes to exactly the payment text.
- Records "reference / amount" as the widget's answer in the submission, so you can see
  which code a parent was shown.
