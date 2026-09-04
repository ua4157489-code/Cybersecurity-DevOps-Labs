# Methodology — XSS (Reflected) Lab

## 1. Reconnaissance
Navigated to `/vulnerabilities/xss_r/`, which presents a simple "What's your name?" form submitting a `name` parameter via GET. Inspected the baseline page source via `curl` to understand the form structure before attempting injection.

## 2. Testing for reflection
Submitted a basic marker value first conceptually (in practice, went straight to the full payload given prior experience from Lab 10, but the standard approach is to first confirm *any* reflection with a benign string like `name=test123`, checking whether `test123` appears verbatim in the response, before escalating to a script payload). Since the DOM XSS lab had just demonstrated the importance of distinguishing client-side from server-side behavior, the key diagnostic step here was checking the **raw `curl` response** rather than only checking in-browser behavior.

## 3. Confirming server-side reflection (not DOM-based)
Sent the payload `<script>alert('XSS')</script>` as the `name` parameter and inspected the raw HTTP response body with `curl`:

```html
<pre>Hello <script>alert('XSS')</script></pre>
```

This is the critical methodological signal: the injected payload appears **literally, unescaped, in the server's response** — meaning the server itself is responsible for the vulnerability, having taken `$_GET['name']` and printed it directly into the page without calling any HTML-encoding function (such as PHP's `htmlspecialchars()`). This is fundamentally different from Lab 10, where curl showed identical output regardless of payload because the vulnerability lived entirely in client-side JavaScript.

## 4. Exploitation
Navigated to the same URL in Firefox:
```
http://localhost:4280/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>#
```

Since the server had already embedded the raw `<script>` tag into the HTML document it sent to the browser, the browser's HTML parser encountered and executed the script tag as a normal part of page load — no additional client-side vulnerability or JavaScript execution chain was needed, unlike the DOM XSS case. The alert fired immediately on page render, displaying `XSS`.

## 5. Comparing to Lab 10 (DOM XSS)
This lab was deliberately sequenced right after DOM XSS to reinforce the distinction between the two XSS subtypes, which are often conflated:
- **Reflected XSS (this lab):** vulnerability is in server-side output generation; detectable via `curl`/raw HTTP inspection alone
- **DOM XSS (Lab 10):** vulnerability is in client-side JavaScript execution; requires a real browser to trigger, invisible to `curl`

Both share the same underlying flaw category (untrusted input reaching an unsafe output sink without encoding) but require different detection techniques and different fixes, which is reflected in each lab's `remediation.md`.
