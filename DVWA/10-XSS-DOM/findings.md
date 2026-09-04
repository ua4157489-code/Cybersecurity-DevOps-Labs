# Findings — XSS (DOM) Lab

## Vulnerability
**Type:** DOM-based Cross-Site Scripting (XSS)
**Location:** `/vulnerabilities/xss_d/?default=` (client-side, `default` URL parameter)
**CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting'), specifically the DOM-based variant
**Severity:** High
**CVSS 3.1 (estimated):** 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N) — requires victim interaction (clicking a crafted link), which is reflected in the score relative to stored XSS.

## Description
The client-side JavaScript on this page reads the current URL via `document.location.href`, extracts everything after `default=`, and writes it directly into the page using `document.write()` — with no HTML/script escaping applied. An attacker who can get a victim to click a specially-crafted link can execute arbitrary JavaScript in the victim's browser, in the security context (origin) of the DVWA application.

## Evidence

### 1. Vulnerable source and sink identified
```javascript
var lang = document.location.href.substring(document.location.href.indexOf("default=")+8);
document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
```
`decodeURI()` only reverses URL-encoding — it performs no HTML/script sanitization.

### 2. Confirmed as DOM-based (not server-reflected)
`curl` requests with and without the payload returned byte-for-byte identical server responses — proving the server never processes this parameter. The vulnerability exists entirely in client-side execution.

### 3. Exploit payload and result
**Payload:**
```
http://localhost:4280/vulnerabilities/xss_d/?default=<script>alert(document.domain)</script>
```

**Result:** Browser alert box fired, displaying `localhost` (the value of `document.domain`), proving the injected `<script>` tag executed with full access to the page's JavaScript context — see `screenshots/02-dom-xss-popup.png`.

## Impact
- **Confidentiality:** High — in a real deployment, this same technique could execute `alert(document.cookie)` or silently exfiltrate session cookies/tokens to an attacker-controlled server via `fetch()`/`XMLHttpRequest`, leading to session hijacking.
- **Integrity:** High — arbitrary DOM manipulation is possible, including injecting fake login forms (phishing within a trusted origin), defacing the page, or triggering unwanted actions on behalf of the victim (e.g., chained with CSRF-style requests using the victim's authenticated session).
- **Availability:** Low direct impact, though malicious scripts could degrade or break page functionality for the victim.
- **Delivery vector:** Because the entire payload lives in the URL, this is trivially deliverable via phishing email, malicious QR code, shortened URL, or a link posted on social media — the victim need only be logged into DVWA and click the link.

## Root Cause
Client-side JavaScript trusts `document.location.href` as safe input and writes attacker-controlled portions of it into the DOM via `document.write()` without any output encoding or sanitization. This is a client-side parallel to classic server-side injection: the vulnerable pattern is "unsanitized input reaches a dangerous sink," just entirely within the browser rather than crossing a server round-trip.
