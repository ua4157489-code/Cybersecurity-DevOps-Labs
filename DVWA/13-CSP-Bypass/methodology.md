# Methodology — CSP Bypass Lab

## 1. Reconnaissance — reading the CSP header first
Before attempting any exploitation, the actual `Content-Security-Policy` response header was retrieved directly, since CSP bypass techniques are entirely dependent on the specific policy in force — there is no generic "CSP bypass payload," only techniques tailored to whatever sources a given policy happens to trust.

```
script-src 'self' https://pastebin.com example.com code.jquery.com https://ssl.google-analytics.com;
```

Each whitelisted third-party domain was evaluated for exploitability:
- **`pastebin.com`** — a public paste-hosting service where anyone can publish arbitrary text content and obtain a stable URL. This is the classic "attacker-writable" whitelist entry: trusting it as a script source effectively trusts anyone with a Pastebin account.
- **`example.com`** — IANA's reserved example domain, serving static reference content with no user-generated content feature at all. Not attacker-controllable.
- **`code.jquery.com`** — a legitimate CDN for the jQuery library. Not directly attacker-writable, though CDN-hosted-library bypasses (abusing legitimate but powerful library code, e.g., certain AngularJS versions' sandbox escapes) are a known technique in some CSP bypass scenarios; not pursued here since Pastebin offered a more direct path.
- **`ssl.google-analytics.com`** — Google Analytics' tracking endpoint. Not attacker-writable in any practical sense.

Given this analysis, `pastebin.com` was the clear intended exploitation path.

## 2. Confirming the application's handling of the `include` parameter
Before attempting the full exploit, the application's server-side handling was confirmed via `curl`: submitting a URL through the `include` field resulted in that exact URL being embedded as a `<script src="...">` tag in the response HTML, with no server-side filtering, validation, or domain restriction of its own — the application relies entirely on the browser's CSP enforcement as its only control here.

## 3. Weaponizing the attack
A public Pastebin paste was created containing a simple proof-of-concept payload (`alert('CSP Bypass via Pastebin')`), and its "raw" URL was obtained — the raw endpoint serves the paste's plain text content directly, which is necessary for it to be loadable as a script resource rather than an HTML wrapper page.

## 4. Execution attempt and observing the actual browser behavior
The Pastebin raw URL was submitted through the application's form in a real browser, with DevTools Console open to observe exactly what happened at the network/execution level, rather than relying solely on whether an alert box appeared (which would have left the actual failure mode undiagnosed).

The console output revealed the request was **not blocked by CSP** — no CSP violation message appeared, confirming the policy correctly permitted the connection to the whitelisted domain, demonstrating the misconfiguration is real. Instead, a **different** browser security mechanism intervened: Pastebin's raw content is served as `Content-Type: text/plain` alongside an `X-Content-Type-Options: nosniff` header, which instructs browsers not to guess ("sniff") a more appropriate content type — and modern browsers refuse to execute script resources that aren't served with a proper JavaScript MIME type under this header.

## 5. Interpreting the result honestly
This methodology deliberately distinguishes between "the vulnerability doesn't exist" and "the vulnerability exists but this specific exploitation attempt was independently blocked by an unrelated control." The CSP misconfiguration (trusting an attacker-writable domain) is a real, present flaw regardless of whether this particular proof-of-concept fully executed — a different attacker-writable domain that serves correctly MIME-typed JavaScript (or lacks the `nosniff` header) would very plausibly succeed where Pastebin's specific server configuration happened to block execution. The finding is reported and scored accordingly: the CSP flaw is real, with a noted mitigating factor from an unrelated, currently-effective browser control.
