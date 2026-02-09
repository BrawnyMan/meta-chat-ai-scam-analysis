# Meta Chat AI Scam – Technical Analysis

> ⚠️ **Disclaimer**  
> This repository documents a **fake “Meta Chat AI” website and backend** for awareness, research, and abuse reporting.  
> It does **not** facilitate fraud or misuse.

---

## 1. Overview

This operation is a **Meta / Facebook impersonation and data-harvesting scam**.

It uses:
- A **fake Meta-branded website**
- A **Next.js frontend**
- A **Netlify serverless backend**
- A **Google Sheets–based data store**

Its purpose is to silently collect user data and prevent victims from accessing or removing it.

---

## 2. Initial Contact (Social Engineering)

The scam begins with a **direct message from another user** on a social platform.  
The message creates mild urgency and contains a link to the fake site.

![Initial scam message from another user](images/message.png)

No money is requested at this stage, which lowers suspicion.

---

## 3. Fake Website Flow

### 3.1 Fake Verification Page

After clicking the link, the user sees a **“Confirm you are not a robot”** page.  
This imitates real security checks but uses no real CAPTCHA.

![Fake robot check page](images/not_a_robot_check.png)

---

### 3.2 Timestamp Redirect

The site redirects to a URL containing a numeric value resembling a Unix timestamp:

/contact/1770471578746


This makes the flow appear session-based and legitimate.

![Timestamp redirect in URL](images/time_url.png)

---

### 3.3 Meta-Like Interface

The main page imitates **Meta / Facebook settings or support pages** and prompts the user to submit personal details.

![Fake Meta Chat AI page](images/main_page.png)

A visual comparison shows copied design cues but no real Meta functionality.

Fake:
![Fake site vs real Meta settings](images/fake.png)

Real:
![Fake site vs real Meta settings](images/real.png)

---

## 4. Frontend Architecture

The frontend is built with **Next.js**, confirmed by:
- `/_next/static/*` assets
- React Server Components
- Client-side routing

![Next.js assets loaded by the site](images/page_code.png)

Only **one element is functional**:
- the **form submit button**

All other UI elements are decorative.

---

## 5. Automatic Data Collection

Before any form interaction, the site **automatically collects data on page load**:
- Client IP address
- Approximate location (Geo-IP)
- Region and language settings

![Automatic data collection on page load](images/geo.png)

The site adapts language based on region, using **Google Translate**, increasing perceived legitimacy.

![Page translated based on region](images/transalte.png)

---

## 6. Data Collection Form

The final step on the website is a form that collects user-provided data.

| Field | Description |
|------|------------|
| IP address | Client IP (already known) |
| Name | Arbitrary input |
| Email | Victim email |
| Phone | Victim phone |
| Password | User-entered credential / sensitive input |

The user is not informed where this data is stored or how it will be used.

![Data collection form](images/form.png)

### 6.1 Two-Factor Authentication Prompt (Unverified)

After submitting initial credentials, the interface presents a **2FA-related prompt**, suggesting an additional verification step.

![2FA prompt shown on fake site](images/2fa.png)

Important notes:
- !!!This step was **not fully tested**!!!

Based on common phishing techniques, this step is **likely intended** to:
- Capture one-time passwords (OTP)
- Enable real-time credential replay or account takeover
- Increase the perceived legitimacy of the process

However, this remains an **inference**, not a confirmed implementation detail.

---

## 7. Form Submission → Backend Request

When the form is submitted, the browser sends a **POST request** to the backend API.

**Endpoint:**

POST https://zehtevano-preverjanjee.netlify.app/api/sheets


This request is made by submitting the form.

![Network request sent on form submission](images/req.png)
![Network request](images/sheet.png)

---

## 8. Backend API (Netlify Functions)

The `/api/sheets` endpoint is implemented as a **Netlify Serverless Function**.

Its behavior:
- Accepts incoming form data
- Appends it to a Google Sheet
- Does not expose any read, edit, or delete functionality

The API is effectively **write-only**.

---

## 9. Backend Response

After data is stored, the backend returns a minimal success response.

Example response:
```json
{
    "spreadsheetId":"1IiBC-peOD7oUaU2FxUXd2li-0D4lSR8sKu5j00NkGpw",
    "updates": {
        "updatedColumns":6,
        "updatedCells":6,
        "spreadsheetId":"1IiBC-peOD7oUaU2FxUXd2li-0D4lSR8sKu5j00NkGpw",
        "updatedRange":"Sheet1!A3563:F3563",
        "updatedRows":1
    },
    "tableRange":"Sheet1!A1:N3562"
}
```

This response does not include any previously stored data.

---

## 10. CLI Reproduction (Proof)

The same backend behavior can be reproduced using curl:

```bash
curl -X POST "https://zehtevano-preverjanjee.netlify.app/api/sheets" \
  --data-raw "{\"action\":\"append\",\"value\":[[\"215.88.211.158\",\"Ljubljana - Slovenia (SI)\",\"ime\",\"test@example.com\",\"+38612312312\",\"geslo\"]]}"
```

Successful response confirms:

- Data is accepted without authentication
- Data is written to Google Sheets
- The user has no visibility into stored entries

---

## 11. API Capabilities (Reverse-Engineered)

Testing shows the backend API exposes a **very limited action set**.

| Action | Status |
|------|-------|
| `append` | ✅ Supported |
| `update` | ✅ Supported (restricted) |
| `read` / `get` | ❌ Not supported |
| `batchGet` | ❌ Not supported |
| `delete` | ❌ Not supported |
| `batchUpdate` | ❌ Not supported |

The API is effectively **write-only** for clients.

---

## 12. Data Ingestion (`append`)

- Submissions are appended to **Google Sheets**
- Each request creates a **new row**
- **Columns A–F** store victim-submitted data
- Existing data is never returned to the client

---

## 13. Restricted Updates (`update`)

Example request:
```bash
curl https://zehtevano-preverjanjee.netlify.app/api/sheets \
  -H "content-type: application/json" \
  --data-raw "{\"action\":\"update\",\"row\":3562,\"value\":[[\"X\",\"Y\",\"Z\",\"Q\"]]}"
```

Response:
```json
{
  "updatedRange": "Sheet1!G3562:J3562",
  "updatedRows": 1,
  "updatedCells": 4
}
```

Key findings:
- row is mandatory
- Any provided range is ignored
- Only columns G–J are writable
- Columns A–F cannot be modified
- Updates append within the row rather than overwrite data

This suggests:
- A–F → raw victim data (immutable)
- G–J → internal processing / status fields

---

## 14. Update Boundary Enforcement

When update attempts exceed allowed columns, the backend returns a Google API error:

```json
{
  "code": 400,
  "message": "Requested writing within range [Sheet1!J3562:K3562], but tried writing to column [L]"
}
```

This confirms strict column limits are enforced.

---

## 15. Blocked Actions

Unsupported actions return null:
- Append without payload
- Read / get
- Delete
- Batch or structural operations

There is no way for users to read, delete, or erase submitted data.

---

## 16. Data Storage Model

The backend uses Google Sheets as a protected data store:
|Columns|Purpose|
|-----|-----|
|A–F|	Victim-submitted personal data|
|G–J|	Internal processing / status|
|K+|	Protected / unused|

Read access is restricted; attempts result in permission denied.

---


## 17. Backend Summary
- Data ingestion via append
- Limited internal updates via update
- No read, delete, or batch access
- Client visibility intentionally restricted

This design enables maximum data collection with minimal user control.

## Scam Summary & Impact

This is a **low-effort but effective data-harvesting and account-compromise operation**.

**Key characteristics:**
- Impersonates Meta / Facebook
- Uses a professional Next.js frontend
- Uses Netlify serverless functions
- Collects personal data silently
- Write-only data ingestion
- No transparency or user control

---

## Risk & Impact

Victims cannot:
- View submitted data
- Edit or delete submissions
- Verify legitimacy

Based on reports from other affected users, submitted credentials may be used to:
- Perform **account takeover**
- Rename compromised accounts (e.g., to “Meta Chat AI”)
- Send scam messages to new victims from hijacked accounts

This behavior enables **self-propagation** of the scam through trusted accounts.

Collected data can also be:
- Aggregated
- Sold
- Used for targeted phishing or follow-up attacks

---

## Conclusion

The system appears designed to:
- Appear legitimate
- Collect sensitive information
- Enable account compromise
- Prevent user oversight
- Centralize victim data

The technical design prioritizes **data capture and control over user safety**.

---

## Responsible Disclosure

If affected:
- Do not submit personal data
- Secure your account immediately (password change, revoke sessions, enable 2FA)
- Report the site to:
  - Netlify Abuse
  - Meta / Facebook impersonation channels
  - Local cybercrime authorities if necessary
