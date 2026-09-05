# Remediation — OWASP A07: Identification and Authentication Failures

## Finding 1: No Brute-Force Protection on Login

**Fix:**
- Apply `express-rate-limit` (already present in the dependency tree) specifically to the login route, with a strict per-IP and/or per-account limit (e.g. 5–10 attempts per 15 minutes), returning `429 Too Many Requests` with a `Retry-After` header once exceeded.
- Add progressive delay or temporary account lockout after repeated failures from the same account, independent of source IP, to resist distributed attempts.
- Consider a CAPTCHA challenge after a small number of failed attempts as an additional layer.
- Log and alert on repeated failed-login patterns so real attacks can be detected operationally, not just blocked technically.

## Finding 2: No Password Complexity Enforcement

**Fix:**
- Enforce a minimum password length (commonly 10–12+ characters per current NIST guidance) at registration and password-change time.
- Rather than mandating specific character classes (which often just pushes users toward predictable substitutions), check submitted passwords against a known-breached-password list (e.g. Have I Been Pwned's Pwned Passwords API/dataset) and reject matches.
- Ensure this validation happens server-side, not only in client-side form validation which can be trivially bypassed via direct API calls (as demonstrated here).

## Finding 3: No Token Expiration and No Server-Side Logout

**Fix:**
- Add an `exp` claim to every issued JWT (e.g. 15–60 minutes for access tokens), paired with a refresh-token flow for longer sessions if needed.
- Implement genuine server-side session invalidation: either move to short-lived tokens validated against a server-side revocation list/blocklist (checked on each request), or adopt an architecture where logout has real effect (e.g. token versioning tied to the user record, incremented on logout or password change, checked at verification time).
- Treat "logout" as a security-relevant action requiring a real server round-trip, not a purely client-side no-op.

## Finding 4: Password Hash Embedded in JWT Payload

**Fix:**
- Remove the password hash (and any other unnecessary sensitive fields) from the JWT payload entirely. A JWT should carry only the minimum claims needed for authorization (user id, role), not a copy of stored credential material.
- Audit every field currently embedded in the token payload with the same scrutiny — anything not strictly required for authorization decisions should not travel inside a token that's readable by any party holding it.
- Independently, address the underlying weak hashing algorithm (flagged in the A02 lab) by migrating to a modern, salted, slow hash (bcrypt, scrypt, or Argon2) — this reduces the damage even if a hash is exposed via this or another channel in the future.

## General Principle
Every finding here stems from treating authentication as a one-time gate (checked at login) rather than a continuously-enforced property of a session. Real authentication security requires guarding the entry point (rate limiting, password policy) **and** the full lifecycle of the credential afterward (expiration, revocation, minimal data exposure) — a strong login check is undermined if the resulting session can be brute-forced, never expires, and leaks credential material on every use.

**Recommended controls:**
- Rate limiting applied per-endpoint by design review, not assumed from a library's mere presence in `package.json`
- A documented password policy checked server-side, ideally validated against known-breach password lists
- Short-lived tokens with a genuine server-side revocation mechanism, tested as part of any auth-related release
- A "minimum necessary claims" review for any JWT payload change, as part of code review for authentication-related pull requests
