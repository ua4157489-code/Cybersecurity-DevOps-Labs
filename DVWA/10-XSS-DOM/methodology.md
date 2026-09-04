# Methodology — XSS (DOM) Lab

## 1. Reconnaissance
Navigated to `/vulnerabilities/xss_d/`, which presents a simple language-selection dropdown. Before attempting any payload, the page's HTML source was inspected directly (via both `curl` and browser "View Source") to understand how the dropdown is populated — this is a necessary first step for DOM XSS specifically, since the vulnerability lives in client-side logic that isn't visible from server responses alone in the way server-side injection points are.

## 2. Identifying source and sink
The inline `<script>` block revealed the vulnerable pattern clearly:

- **Source** (where attacker-controlled data enters): `document.location.href` — the full current URL, fully controllable by whoever crafts the link the victim clicks
- **Sink** (where that data is used unsafely): `document.write()` — writes raw HTML/JS into the page, and critically, `decodeURI(lang)` is applied but this only decodes URL encoding, it does **not** sanitize or escape HTML/script-significant characters like `<`, `>`, or quotes

The code takes everything after `default=` in the URL and writes it directly into the DOM as HTML, without any escaping. Any HTML or `<script>` tags included in that substring will be parsed and, in the case of `<script>`, executed by the browser exactly as if it were part of the original page.

## 3. Confirming this is DOM-based, not reflected
A key methodological step: sent the exact same payload via `curl` and compared the raw HTML response with and without the payload in the URL. Both were byte-for-byte identical. This is the diagnostic signature of DOM-based XSS — if the server had reflected the input into its response (as in reflected XSS), the payload would appear literally in the curl output. Since it didn't, this confirmed the vulnerability is entirely client-side: the malicious behavior only manifests when a real browser's JavaScript engine parses `document.location.href` and executes the write.

## 4. Exploitation
Constructed a URL with a `<script>` payload appended after `default=`:
```
http://localhost:4280/vulnerabilities/xss_d/?default=<script>alert(document.domain)</script>
```

Navigated to this URL in Firefox. The page's own vulnerable JavaScript read the URL, extracted the payload, and wrote it into the DOM via `document.write()`. The browser then parsed the newly-inserted `<script>` tag as part of the live page and executed it, firing `alert(document.domain)` and displaying `localhost` — proof that arbitrary JavaScript executed in the context of the DVWA origin.

## 5. Why `document.domain` was chosen as the payload body
Using `alert(document.domain)` rather than a static string like `alert('XSS')` demonstrates something stronger than "an alert box can pop up" — it proves the injected script is executing **within the actual page's JavaScript context**, with access to page-scoped objects like `document`. This is the same context an attacker would use to steal cookies (`document.cookie`), redirect the page (`document.location`), or manipulate the DOM to phish credentials — not just display a message.
