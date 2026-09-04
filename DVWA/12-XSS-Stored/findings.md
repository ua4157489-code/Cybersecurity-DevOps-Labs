# Findings — XSS (Stored) Lab

## Vulnerability
**Type:** Stored (Persistent) Cross-Site Scripting
**Location:** `/vulnerabilities/xss_s/` (POST parameters `txtName`, `mtxMessage`; rendered on every GET to the same endpoint)
**CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting'), CWE-565 (persistent storage of unvalidated input)
**Severity:** Critical
**CVSS 3.1 (estimated):** 8.7 (AV:N/AC:L/PR:L/UI:N/S:C/C:L/I:L/A:N) — no victim interaction required beyond normal page visit, elevated relative to reflected/DOM variants due to persistence and self-propagating exposure to all visitors.

## Description
The guestbook's `Message` field (and `Name` field) are stored server-side without any sanitization or HTML encoding, and rendered back into every subsequent page load without encoding either. Any HTML or JavaScript submitted becomes a permanent part of the page, executing automatically for every visitor — no crafted link, phishing, or per-victim social engineering required, unlike reflected or DOM-based XSS.

## Evidence

### 1. Payload submission
```
POST /vulnerabilities/xss_s/
txtName=Tester&mtxMessage=<script>alert('Stored XSS')</script>&btnSign=Sign+Guestbook
```

### 2. Persistence confirmed within the submitting session
```html
<div id="guestbook_comments">Name: Tester<br />Message: <script>alert('Stored XSS')</script><br /></div>
```

### 3. Persistence confirmed across an entirely separate, unrelated session
A fresh login (new cookie jar, never submitted the guestbook form) received the identical payload on a normal GET request:
```html
<div id="guestbook_comments">Name: Tester<br />Message: <script>alert('Stored XSS')</script><br /></div>
```

### 4. Execution confirmed in a fresh Private Browsing session
Navigating to a clean URL (`http://localhost:4280/vulnerabilities/xss_s/`, no payload in the URL) in a brand-new private session triggered the alert automatically on page load — see `screenshots/01-stored-xss-popup.png`. Page source confirms the raw `<script>` tag is embedded directly in the HTML alongside legitimate entries — see `screenshots/02-stored-xss-source-view.png`.

## Impact
- **Confidentiality:** High — every visitor's session is compromised automatically; an attacker can harvest `document.cookie` (if not `HttpOnly`), page content, or authenticated API responses from any user who views the guestbook, at scale, with zero further attacker effort after the initial injection.
- **Integrity:** High — the injected script can silently perform any action the victim's browser is capable of, including modifying the DOM, submitting additional malicious guestbook entries (worm-like self-propagation), or performing state-changing requests using each victim's own authenticated session.
- **Availability:** Low-to-moderate — a sufficiently disruptive payload could break page functionality for all users, or a resource-intensive script (e.g., a cryptomining payload) could degrade every visitor's browser performance.
- **Blast radius (the defining risk factor):** Unlike reflected/DOM XSS, which compromise one victim per malicious link click, this single injection compromises **every user who visits the page** from the moment of submission until the entry is found and removed — including users the attacker has no direct contact with, and including the site administrators themselves if they review submitted content through the normal UI.
- **Persistence:** The payload survives page reloads, new sessions, and (in a real deployment) server restarts, since it lives in the database rather than in any transient request/response cycle.

## Root Cause
User input is written to persistent storage without sanitization, and subsequently rendered into HTML output without encoding at read time. This is a "double failure": even an application that safely encodes output on the initial submission response would remain vulnerable if it fails to encode the same data on every future read from storage — both the write path and the read path need to be safe (or, more robustly, encoding should happen consistently at output time regardless of source).
