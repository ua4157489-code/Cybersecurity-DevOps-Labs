# Findings - OWASP A10: Server-Side Request Forgery

## Finding 1: Open Redirect via Flawed Allowlist Validation (SSRF-adjacent)

**Endpoint:** `GET /redirect?to=<url>`
**Severity:** High
**Root cause file:** `/juice-shop/build/lib/insecurity.js`

### Steps to Reproduce
1. Initial testing showed the endpoint rejects arbitrary external URLs (`https://example.com` -> 406 "Unrecognized target URL")
2. Guessing plausible allowlist entries from earlier admin-config output (`https://owasp.slack.com`) also failed - confirming a real, specific allowlist exists
3. Extracted the actual server-side source directly from the running container:
   - `redirect.js` calls `security.isRedirectAllowed(toUrl)` before issuing `res.redirect(toUrl)`
   - `insecurity.js` implements the check as:
```js
     const isRedirectAllowed = (url) => {
         let allowed = false;
         for (const allowedUrl of exports.redirectAllowlist) {
             allowed = allowed || url.includes(allowedUrl);
         }
         return allowed;
     };
```
4. The check uses `String.prototype.includes()` rather than exact match or proper URL/host parsing - meaning an allowlisted string anywhere in the supplied URL satisfies the check
5. Confirmed working bypass 1 (allowed string as a domain-name prefix, attacker domain appended as a suffix):
   `GET /redirect?to=https://github.com/juice-shop/juice-shop.attacker.com` -> `302 Found`, `Location: https://github.com/juice-shop/juice-shop.attacker.com`
6. Confirmed working bypass 2 (allowed string buried inside a query parameter on a fully attacker-controlled domain):
   `GET /redirect?to=https://evil-attacker-site.com/?x=https://github.com/juice-shop/juice-shop` -> `302 Found`, redirecting to the attacker domain

### Evidence
- `raw-output/redirect.js`, `raw-output/insecurity.js` - extracted source showing the vulnerable check
- `raw-output/redirect_bypass_startswith.txt`
- `raw-output/redirect_bypass_includes_anywhere.txt`
- `screenshots/01-vulnerable-includes-check-source.png`
- `screenshots/02-ssrf-redirect-bypass-suffix.png`
- `screenshots/03-ssrf-redirect-bypass-anywhere.png`

### Impact
The application's own server issues an HTTP redirect to any URL an attacker chooses, as long as it contains one specific allowlisted substring anywhere at all. This enables:
- **Open redirect / phishing**: an attacker can craft a link that appears to originate from the trusted Juice Shop domain (`http://localhost:3000/redirect?to=...`) but sends victims to an attacker-controlled phishing page
- **SSRF potential**: while this specific endpoint issues a client-side redirect (browser follows it) rather than a server-side fetch, the same flawed `includes()` validation pattern is the exact class of bug that leads to true SSRF wherever this allowlist check is reused for server-side outbound requests (e.g. an image-fetch-by-URL feature), since any internal address string containing the allowed substring would also pass
- **Trust exploitation**: victims and automated systems that trust the Juice Shop domain in links are misled into visiting attacker infrastructure

### Root Cause
`isRedirectAllowed()` validates using substring containment (`url.includes(allowedUrl)`) instead of validating the URL's actual scheme, host, and path against the allowlist using proper URL parsing. This allows an allowlisted string to appear anywhere in an otherwise fully attacker-controlled URL and still pass validation.
