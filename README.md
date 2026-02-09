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

![Fake site vs real Meta settings](images/fake.png)
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
