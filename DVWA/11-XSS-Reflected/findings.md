# Findings — XSS (Reflected) Lab

## Vulnerability
**Type:** Reflected Cross-Site Scripting (XSS)
**Location:** `/vulnerabilities/xss_r/?name=` (GET parameter, server-side output)
**CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')
**Severity:** High
**CVSS 3.1 (estimated):** 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N)

## Description
The server takes the `name` GET parameter and embeds it directly into the HTML response (inside a `<pre>` tag, prefixed with "Hello") without any HTML encoding or sanitization. Any HTML or `<script>` content included in this parameter is rendered and executed exactly as if it were part of the original page — this is a textbook reflected XSS vulnerability, requiring only that a victim be convinced to click (or otherwise load) a maliciously crafted link.

## Evidence

### 1. Server-side reflection confirmed directly via curl
**Payload:** `name=<script>alert('XSS')</script>`

**Raw server response:**
```html
<pre>Hello <script>alert('XSS')</script></pre>
```

The payload is present, unescaped, in the HTTP response body — no browser or JavaScript execution needed to demonstrate this half of the vulnerability.

### 2. Execution confirmed in browser
**URL:**
```
http://localhost:4280/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>#
```
**Result:** JavaScript alert box fired on page load, displaying `XSS` — see `screenshots/01-reflected-xss-popup.png`.

## Impact
- **Confidentiality:** High — an attacker-controlled script executing in a victim's authenticated session can read `document.cookie` (if not `HttpOnly`), access any data rendered on the page, or make authenticated requests on the victim's behalf via `fetch()`/XHR.
- **Integrity:** High — the injected script can modify the DOM arbitrarily, including injecting fake forms to harvest credentials, or silently performing state-changing actions using the victim's session.
- **Availability:** Low direct impact.
- **Delivery vector:** Because the entire payload is contained in a URL, it's easily delivered via phishing email, a shortened/obfuscated link, a malicious redirect, or embedded in a third-party site — the victim need only be logged into DVWA when the link is clicked.
- **Distinguishing factor from Lab 10 (DOM XSS):** This vulnerability is directly detectable from server responses alone (via `curl`, automated scanners, or proxy tools like Burp Suite intercepting raw HTTP traffic), making it generally easier to find via automated scanning than DOM-based variants, which require a JavaScript-executing browser context to detect reliably.

## Root Cause
The server-side code responsible for rendering this page outputs `$_GET['name']` directly into the HTML response without passing it through an output-encoding function (e.g., PHP's `htmlspecialchars()`). Any characters with special meaning in HTML (`<`, `>`, `"`, `'`, `&`) are passed through verbatim instead of being converted to their safe HTML entity equivalents.
