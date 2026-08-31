# Commands – DVWA CSRF

## 1. Overview

This file documents the commands and browser-based actions used during the DVWA CSRF assessment.

The testing was performed against the local DVWA installation:

```text
http://localhost:8080/DVWA/
```

The primary testing tool was **Firefox Developer Tools**, because Burp Suite was not available in the lab environment.

---

# 2. Navigate to the CSRF Directory

From the portfolio repository:

```bash
cd ~/Alrazzaq_Labs/DVWA/03-CSRF
```

Verify the directory contents:

```bash
ls -la
```

Expected structure:

```text
README.md
methodology.md
commands.md
findings.md
remediation.md
screenshots/
```

---

# 3. Verify Screenshot Evidence

Navigate to the screenshot directory:

```bash
cd screenshots
```

List the evidence files:

```bash
ls -l
```

Expected files:

```text
01-csrf-request.png
02-csrf-poc.png
```

Return to the CSRF directory:

```bash
cd ..
```

---

# 4. Open DVWA CSRF Module

Open the following URL in Firefox:

```text
http://localhost:8080/DVWA/vulnerabilities/csrf/
```

Verify that the page displays:

```text
Vulnerability: Cross Site Request Forgery (CSRF)
```

---

# 5. Verify DVWA Security Level

The lab was performed at:

```text
Security Level: low
```

The security level can be checked from the DVWA interface.

---

# 6. Submit a Controlled Password Change

Enter a controlled test value into:

```text
New password:
test1234!
```

and:

```text
Confirm password:
test1234!
```

Then select:

```text
Change
```

This generates the password-change request.

---

# 7. Open Firefox Developer Tools

Use:

```text
F12
```

or:

```text
Ctrl + Shift + I
```

Open:

```text
Network
```

Then submit the password-change form again.

---

# 8. Inspect the CSRF Request

Select the request associated with:

```text
/DVWA/vulnerabilities/csrf/
```

Inspect the request method and parameters.

The observed request used:

```text
GET
```

The relevant parameters were:

```text
password_new=test1234!
password_conf=test1234!
Change=Change
```

---

# 9. Inspect Request Headers

The request headers can be viewed from the Firefox Network panel.

Important headers observed include:

```text
Host: localhost:8080
Cookie: PHPSESSID=<authenticated-session>; security=low
Referer: http://localhost:8080/DVWA/vulnerabilities/csrf/
```

The session cookie demonstrates that the request was made within the authenticated DVWA session.

---

# 10. Inspect Response

The captured request returned:

```text
HTTP/1.1 200 OK
```

Important response information included:

```text
Server: Apache/2.4.52 (Ubuntu)
Content-Type: text/html;charset=utf-8
```

---

# 11. Check for CSRF Token

Inspect the request parameters for a dedicated CSRF token.

Example of a token that would normally be expected in a protected implementation:

```text
csrf_token=<random-value>
```

No dedicated CSRF token was observed in the tested password-change request.

---

# 12. Optional curl Request Inspection

The local endpoint can also be requested using `curl`:

```bash
curl -i "http://localhost:8080/DVWA/vulnerabilities/csrf/"
```

This can be used to inspect the HTTP response headers.

---

# 13. Authenticated Request Testing

For local lab testing, an authenticated session can be supplied using the session cookie.

Example format:

```bash
curl -i \
  -H 'Cookie: PHPSESSID=<YOUR_SESSION_ID>; security=low' \
  "http://localhost:8080/DVWA/vulnerabilities/csrf/"
```

Replace:

```text
<YOUR_SESSION_ID>
```

with the current session ID from the browser.

**Do not commit real session cookies to GitHub.**

---

# 14. Evidence Verification

Verify the evidence files:

```bash
ls -lh screenshots/
```

Expected:

```text
01-csrf-request.png
02-csrf-poc.png
```

The screenshots are referenced in the README using:

```markdown
![CSRF Request](screenshots/01-csrf-request.png)
```

and:

```markdown
![CSRF Proof of Concept](screenshots/02-csrf-poc.png)
```

---

# 15. Check Git Status

From the `03-CSRF` directory:

```bash
git status
```

This shows the files added or modified for the lab.

---

# 16. Review Files Before Commit

Verify all required files:

```bash
find . -maxdepth 2 -type f | sort
```

Expected structure:

```text
./README.md
./commands.md
./findings.md
./methodology.md
./remediation.md
./screenshots/01-csrf-request.png
./screenshots/02-csrf-poc.png
```

---

# 17. Security Note

Never include active authentication credentials or session identifiers in portfolio files.

Do not commit values such as:

```text
PHPSESSID=<real-session>
```

Instead, document them generically:

```text
PHPSESSID=<authenticated-session>
```

This prevents accidental exposure of authentication information.

---

# 18. Summary

The main commands used for this lab were:

```bash
cd ~/Alrazzaq_Labs/DVWA/03-CSRF
ls -la
cd screenshots
ls -l
cd ..
curl -i "http://localhost:8080/DVWA/vulnerabilities/csrf/"
git status
find . -maxdepth 2 -type f | sort
```

Browser-based testing was performed using:

```text
Firefox
   ↓
Developer Tools
   ↓
Network
   ↓
CSRF Request
   ↓
Headers / Parameters / Response
```

---

## 19. Result

The testing successfully captured and documented the password-change request and provided evidence for the CSRF assessment.

The technical findings are documented in:

```text
findings.md
```

Recommended security fixes are documented in:

```text
remediation.md
```
