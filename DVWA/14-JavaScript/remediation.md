# Remediation — JavaScript Attacks Lab

## Primary Fix: Never rely on client-side-only logic for security decisions
The core principle: any check that matters for security must be enforced server-side, where the application has full control and the client cannot inspect or tamper with the logic. Client-side JavaScript can improve user experience (instant feedback, reduced round-trips) but must never be the sole gatekeeper for anything security-relevant.

**Vulnerable pattern:**
```javascript
// Client generates and self-validates its own token — server just checks the format matches
function generate_token() {
    document.getElementById("token").value = md5(rot13(phrase));
}
```

**Fixed pattern — server-issued, server-validated tokens:**
```php
// Server generates a cryptographically random token, tied to the session, on page load
session_start();
$_SESSION['form_token'] = bin2hex(random_bytes(32));
// Token embedded in a hidden field, but the SERVER remembers the expected value

// On submission, server checks the submitted token against its own stored value
if (!isset($_POST['token']) || !hash_equals($_SESSION['form_token'], $_POST['token'])) {
    die('Invalid or missing token.');
}
// Invalidate after use to prevent replay
unset($_SESSION['form_token']);
```

This closes the vulnerability because the token's validity is determined by a value only the server knows (stored server-side in the session), not by a formula an attacker can independently compute.

## Defense in Depth

1. **Treat all client-side code as fully readable and reimplementable by any user** — Assume any JavaScript shipped to the browser will be read, understood, and bypassed by a sufficiently motivated party. Design security controls accordingly: the browser is an untrusted execution environment from the server's perspective.
2. **Use this pattern correctly for legitimate anti-automation (CAPTCHA, proof-of-work, rate limiting)** — If the actual goal is to slow down or deter bot traffic, use purpose-built server-validated mechanisms (real CAPTCHA services with server-side verification callbacks — see Lab 06's remediation — or server-side rate limiting by IP/session), not a client-computed hash that provides no cryptographic guarantee of human/browser involvement.
3. **Minification/obfuscation is not a security control** — While this lab's JavaScript wasn't heavily obfuscated, it's worth noting explicitly: minifying or obfuscating client-side code raises the effort to read it, but does not prevent it (as trivially demonstrated by browser DevTools' automatic code formatting, or dedicated de-obfuscation tooling). Never treat obfuscation as equivalent to actual access control.
4. **Server-side validation should be the default assumption, with client-side checks as a UX enhancement layered on top** — A common, safer pattern: implement the real check server-side first, then optionally add a client-side pre-check purely to give users faster feedback (e.g., "this phrase looks wrong" before they even submit) — but always re-verify identically on the server regardless of what the client-side check concluded.
5. **Audit for this pattern elsewhere in the codebase** — Since this is a design pattern rather than a single bug, search for other instances where the client computes or self-issues values (tokens, prices, permissions, validation flags) that the server trusts without independent verification.

## Verification
After remediation, re-run the exploitation attempt documented in `commands.md`, but with a token guessed, forged, or computed via the old client-side formula. Confirm the server now rejects it (since it no longer matches a real server-issued value), and confirm the legitimate flow (loading the page, letting the server issue a real token, then submitting that exact token) still succeeds.
