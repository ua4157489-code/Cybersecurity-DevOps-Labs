# Findings — OWASP A07: Identification and Authentication Failures

## Finding 1: No Brute-Force Protection on Login

**Severity:** High
**Endpoint:** `POST /rest/user/login`
**Status:** Vulnerable

### Description
The login endpoint imposes no meaningful barrier against repeated failed authentication attempts: no account lockout, no progressive delay, no CAPTCHA challenge, and no HTTP-level rate limiting. This is notable given the application's own dependency manifest (confirmed in the A06 lab) includes `express-rate-limit` — the capability to rate-limit exists in the codebase but is not effectively applied to this endpoint.

### Reproduction
30 failed login attempts against a known account (`admin@juice-sh.op`) were submitted in a tight loop:
```
time (for i in {1..30}; do curl ... ; done)
```
**Result:** 30/30 attempts returned `401 Unauthorized`. Total wall-clock time: **1.25 seconds**. Zero `429 Too Many Requests` responses. No `X-RateLimit-*` or `Retry-After` headers observed at any point.

### Impact
An attacker can attempt credential-stuffing or password-guessing attacks against any known account at essentially unlimited speed (roughly 24 attempts/second sustained in this test, likely faster with parallelized requests), with no server-side friction whatsoever.

### Evidence
- `screenshots/01-no-bruteforce-protection.png`

---

## Finding 2: No Password Complexity Enforcement

**Severity:** Medium
**Endpoint:** `POST /api/Users`
**Status:** Vulnerable

### Description
The registration endpoint accepts a single-character password (`"a"`) with no minimum length, character-class, or complexity requirement enforced.

### Reproduction
```
curl -s -X POST http://localhost:3000/api/Users \
  -d '{"email":"weakpass@test.com","password":"a","passwordRepeat":"a"}'
```
**Result:** `201 Created` — account successfully registered with the weak password.

### Impact
Combined with Finding 1 (no brute-force protection), weak password policy substantially lowers the effort required for account takeover: accounts are free to use trivially guessable passwords, and there is nothing stopping an attacker from testing them at speed.

### Evidence
- `screenshots/02-weak-password-accepted.png`

---

## Finding 3: No Token Expiration and No Server-Side Logout — Indefinite Session Validity

**Severity:** High
**Endpoint:** Authentication token issuance / session lifecycle
**Status:** Vulnerable

### Description
Two related gaps combine into a single serious finding:

1. **No server-side logout exists.** `GET /rest/user/logout` (and equivalent paths) do not resolve to a real API route — logout is handled entirely client-side (deleting the token from browser storage), with no server-side session/token invalidation mechanism at all.
2. **Issued JWTs contain no `exp` (expiration) claim.** The decoded token payload contains only `iat` (issued-at) with no expiry.

### Reproduction
```
curl -s -X POST http://localhost:3000/rest/user/login -d '{"email":"...","password":"..."}'
# decode payload:
python3 -c "... base64.urlsafe_b64decode(payload) ..."
```
**Result:** Decoded payload contains `iat` only — no `exp`. Token confirmed functional against `GET /rest/basket/26` (`200 OK`).

### Impact
A token captured by any means (network interception, XSS, shared/public device, etc.) remains valid **indefinitely** — there is no time-based expiry to eventually neutralize it, and no user-initiated action ("logging out") has any real server-side effect on it. The only way to invalidate a compromised token would be a password change that happens to also change server-side validation state (not confirmed in this session) or manual server-side key rotation.

### Evidence
- `screenshots/03-token-no-expiration-claim.png`

---

## Finding 4: Password Hash Embedded Directly in JWT Payload

**Severity:** Medium
**Endpoint:** Authentication token issuance (all logins)
**Status:** Vulnerable

### Description
The JWT payload issued on every login includes the user's own password hash as a plain field:
```json
"password": "b8f58c3067916bbfb50766aa8bddd42c"
```
JWTs are base64-encoded, not encrypted — this hash is trivially readable by anyone who can see the token (the browser itself, browser extensions, any logging/proxy infrastructure that captures request headers, or an attacker who intercepts a single request). This directly compounds the weak, unsalted MD5-style hashing already identified in the A02 (Cryptographic Failures) lab: the hash doesn't need to be stolen from a database breach when it's handed out inside every authenticated request's own credentials.

### Impact
Unnecessary exposure of sensitive authentication material through a channel (the token itself) that has no legitimate need to carry it. Increases the attack surface for offline password-cracking attempts, especially given the hash format's known weakness.

### Evidence
- `screenshots/04-password-hash-in-token-payload.png`

## Not Pursued This Session
- Security-question-based password reset flow (guessable answers, no rate limiting)
