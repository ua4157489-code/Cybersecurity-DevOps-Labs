# Remediation - OWASP A10: Server-Side Request Forgery

## Finding 1: Open Redirect via Flawed Allowlist Validation

**Fix:**
- Replace substring-containment validation (`url.includes(allowedUrl)`) with proper URL parsing and exact host/path matching. Example (pseudocode):
```js
  const parsed = new URL(toUrl)
  const allowed = ALLOWLIST.some(entry => {
    const allowedParsed = new URL(entry)
    return parsed.protocol === allowedParsed.protocol &&
           parsed.hostname === allowedParsed.hostname &&
           parsed.pathname.startsWith(allowedParsed.pathname)
  })
```
  This ensures the allowed value must match the actual scheme and hostname of the target URL, not merely appear as a substring anywhere in it.
- Never validate untrusted URLs using `includes()`, `indexOf()`, or naive `startsWith()` on the raw string - these are all bypassable by embedding the allowed string inside a larger attacker-controlled string (as demonstrated by both working exploits in this lab).
- Where possible, avoid accepting arbitrary redirect targets from user input entirely; use an indirection table (e.g. `?to=partner1` mapped server-side to a fixed URL) instead of accepting a raw URL parameter.

## General Principle (relevant beyond this one endpoint)
This exact flawed pattern - allowlist validation via substring containment rather than structured URL parsing - is also the root cause of many real-world SSRF vulnerabilities, where the same mistake is applied to server-side outbound HTTP requests (e.g., fetch-image-by-URL, webhook-URL validation, or file-import-by-URL features) rather than client-side redirects. Any codebase using this pattern should be audited wherever URL allowlisting occurs, not just at this one endpoint, since the underlying validation function (`isRedirectAllowed`) could plausibly be reused or duplicated elsewhere with the same flaw.

**Recommended controls:**
- Centralize URL/host allowlist validation into a single, well-tested utility function used everywhere the application accepts a URL from user input, and unit-test it explicitly against bypass patterns (subdomain tricks, query-parameter embedding, userinfo `@` tricks, case variation)
- For any feature where the *server itself* makes an outbound request based on user input (not just a client-side redirect), apply additional SSRF-specific defenses: block requests to private/internal IP ranges (RFC 1918, link-local, loopback), disable HTTP redirects being followed automatically by the server-side HTTP client, and restrict allowed schemes to `https`
- Include allowlist-bypass test cases in CI/CD security regression testingx
