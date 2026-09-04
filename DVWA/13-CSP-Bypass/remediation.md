# Remediation — Content Security Policy (CSP) Bypass Lab

## Primary Fix: Remove attacker-writable domains from the CSP whitelist
The core fix is straightforward: never whitelist a domain in `script-src` unless the application owner fully controls what content is served from it. Public paste/hosting services, URL shorteners, generic CDNs without pinned versions, and similar user-generated-content platforms should never appear in a `script-src` directive.

**Vulnerable policy:**
```
Content-Security-Policy: script-src 'self' https://pastebin.com example.com code.jquery.com https://ssl.google-analytics.com;
```

**Fixed policy — remove attacker-writable/unnecessary entries:**
```
Content-Security-Policy: script-src 'self' https://code.jquery.com https://ssl.google-analytics.com;
```
(`pastebin.com` removed entirely; `example.com` removed as it serves no legitimate purpose for this application and adds unnecessary attack surface even though it's not directly attacker-writable.)

## Defense in Depth

1. **Don't accept arbitrary URLs into `<script src>` at all** — The deeper architectural issue is that this application takes user input (the `include` parameter) and places it directly into a script tag's `src` attribute. Even with a perfectly configured CSP, this design pattern is inherently risky. If external script inclusion is genuinely needed as a feature, validate the URL against a strict allow-list server-side (not just relying on the browser-enforced CSP as the only control), and reject anything not matching exactly.
2. **Use Subresource Integrity (SRI) for any legitimate third-party scripts** — For genuinely trusted CDN sources like `code.jquery.com`, pin the exact expected script content with an SRI hash:
   ```html
   <script src="https://code.jquery.com/jquery-3.7.1.min.js"
           integrity="sha384-<hash>"
           crossorigin="anonymous"></script>
   ```
   This ensures that even if the CDN is compromised or a different file is served, the browser refuses to execute content that doesn't match the expected hash — closing the gap CSP alone doesn't cover (CSP trusts the *origin*, not the *specific content*).
3. **Prefer `'strict-dynamic'` and nonces/hashes over domain whitelisting** — Modern CSP best practice moves away from domain-based whitelisting (which is exactly what caused this vulnerability) toward nonce-based or hash-based script authorization, where each legitimate `<script>` tag on the page carries a per-request cryptographic nonce that an attacker cannot predict or forge, regardless of what domain they might otherwise get content hosted on.
4. **Enable CSP reporting** — Add a `report-uri` or `report-to` directive so that any blocked or attempted violations are logged and alertable, giving visibility into exploitation attempts (or misconfigurations) in production rather than relying on manual testing to discover them.
5. **Regularly audit CSP whitelist entries** — As applications evolve, `script-src` entries often accumulate from various integrations over time without periodic review. Treat the CSP whitelist as a piece of security-critical configuration requiring the same change-review rigor as firewall rules or IAM policies.
6. **Do not rely on incidental protections like MIME-sniffing blocks as a security boundary** — As demonstrated in this lab, the fact that Pastebin happens to serve `text/plain` with `nosniff` currently blocks this specific exploitation path, but this is not a control the application owns or can guarantee will remain true (Pastebin could change its response headers at any time, entirely outside this application's control).

## Verification
After remediation, re-run the exploitation attempt documented in `commands.md` and confirm:
1. The browser Console now shows an actual **CSP violation** message when attempting to load a script from `pastebin.com` (rather than a MIME-type block), confirming the domain is no longer whitelisted
2. Legitimate application functionality (any genuinely required third-party scripts) continues to work, ideally now with SRI hashes verified
3. Periodically re-test with a range of other public paste/hosting services to confirm no other attacker-writable domains have been inadvertently left in the whitelist
