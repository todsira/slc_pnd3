# PND3 WHT Input (ภ.ง.ด.3 — บุคคลธรรมดา)

Web app for entering withholding-tax (ภ.ง.ด.3) records into the **Input** sheet of the
`SLC PND3 WHT` Google Sheet, and adding individuals to the **Payee 3** sheet. It is the twin of the
ภ.ง.ด.53 app: same layout and behaviour, teal instead of navy, and the payee form is for **individuals**.

```
GitHub Pages (index.html + app.js)  ──fetch──▶  Apps Script Web App (Code.gs)  ──▶  SLC PND3 WHT sheet
```

> Use a **separate repository and a separate Apps Script** from the ภ.ง.ด.53 app. The two sites keep their
> API keys under different browser storage names, so both can be open in the same browser.

## Tax entry tab (Input sheet)

Same as ภ.ง.ด.53: Ref = B.E. year + 3-digit number; Date dd/mm/yyyy B.E.; searchable Payee ID (by ID, name,
surname or ID number); Type ค่าบริการ / ค่าขนส่ง / ค่าประกัน + any type already in the sheet + custom;
Rate 1% / 3% / 5% / custom; WHT = Amount × Rate; Thai confirmation; current + previous month list sorted by Ref
(newest first) with edit. A blank row is added first if the last entry is on the sheet's final row.

* Payees whose **In Use** is unticked/FALSE are hidden from the Payee search (old entries that use them can still be edited).
* Rate and Amount copy the formatting of the row above, so the sheet's existing `1%` style is kept.

## Payee tab (Payee 3 sheet, columns A–P)

| Column | Behaviour |
|---|---|
| ลำดับ | Next number after the highest existing one. |
| Payee ID | Typed; must not already exist. |
| คำนำหน้า | นาย / นาง / นางสาว, any prefix already in the sheet, or custom. |
| ชื่อ / นามสกุล | Separate fields, both required. |
| เลขประจำตัวผู้เสียภาษี | 13-digit ID number, formatted `3-6698-00054-80-3`, stored as text. Check digit verified (warning + explicit confirm if it fails). The same number can't be added twice. |
| ที่อยู่ / ตำบล / อำเภอ / จังหวัด / รหัสไปรษณีย์ | Province suggestions; 5-digit postal code. |
| ชื่อสำหรับกรอกข้อมูล | Built automatically: `นายทรงเกียรติ ปิลันธนโยธิน`. Editable. |
| ที่อยู่สำหรับกรอกข้อมูล | Built automatically (`ต./อ./จ.`, or `แขวง/เขต/กรุงเทพมหานคร`). Editable. |
| ธนาคาร / เลขบัญชี | Optional. Bank has suggestions; account number digits only, stored as text. |
| In Use | Set to TRUE. The checkbox (data validation) of the row above is copied. |

Formulas in the row above are copied down instead of typed values, and a blank row is added first if needed.
The payee list shows bank/account and marks payees not in use with **ไม่ใช้งาน**.

## Setup

### 1. Apps Script (backend)

1. Open the Google Sheet → **Extensions → Apps Script**.
2. Replace the contents of `Code.gs` with `apps-script/Code.gs` from this repo. Save.
3. In the function drop-down choose **setupApiKey** → **Run**. Approve the permissions.
   Open **Execution log** and copy the API key it prints. (Run it again any time to rotate the key.)
4. **Deploy → New deployment** → type **Web app**
   * Execute as: **Me**
   * Who has access: **Anyone**
5. Copy the Web app URL (ends with `/exec`).

> After changing `Code.gs` later: **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**.
> This keeps the same URL.

### 2. GitHub Pages (front end)

1. Create a repository (e.g. `pnd3-wht`) and upload `index.html`, `app.js`, `config.js`, `README.md`
   (and `apps-script/Code.gs` if you want it versioned).
2. Edit `config.js` and paste the Web app URL into `API_URL`. Commit.
3. **Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)`** → Save.
4. Open `https://<your-username>.github.io/<repo>/`, click **ตั้งค่ารหัส** and paste the API key.
   The key is stored only in that browser.

## Security notes

* Never commit the API key. The Web app URL in `config.js` is visible to anyone who can see the site;
  the key is what stops strangers from writing to the sheet.
* GitHub Pages on a free account requires a **public** repository. The repo only contains code — no
  sheet data — but if you'd rather keep it private, use a paid plan or host the same files anywhere static.
* Every request is validated server-side (date, Payee ID must exist in Payee 3, rate 0–100, amount > 0),
  and the Name is always taken from the sheet, not from the browser.

