# Findings — OWASP A02: Cryptographic Failures (Juice Shop)

## Finding 1: Default/Weak Admin Credentials Accepted

**Endpoint:** `POST /rest/user/login`
**Severity:** Critical

### Steps to Reproduce
1. Send login request with `admin@juice-sh.op` / `admin123`
2. Server returns a valid authentication token — login succeeds

### Evidence
- `raw-output/admin_login.json`
- `screenshots/01-admin-default-creds-login.png`

### Impact
A well-known default credential pair grants full admin access. Combined with weak password storage (Finding 3), this represents a critical authentication weakness — no rate limiting or credential rotation policy prevents trivial account takeover.

---

## Finding 2: Sensitive Data Exposed in JWT Payload

**Endpoint:** JWT issued at login (decoded client-side)
**Severity:** High

### Steps to Reproduce
1. Decode the base64 JWT payload from the admin token
2. Payload includes `role: admin` and the user's password field as a raw MD5 hash (`47b7bfb65fa83ac9a71dcb0f6296bb6e`)

### Evidence
- `raw-output/admin_jwt_decoded.json`
- `screenshots/02-admin-jwt-role-decode.png`

### Impact
JWTs are not encrypted, only signed/base64-encoded — anyone who intercepts a token can read its full payload. Embedding a password hash (even hashed) in a client-readable token is unnecessary exposure and violates data-minimization principles.

---

## Finding 3: Unsalted MD5 Password Hashing

**Severity:** Critical

### Steps to Reproduce
1. Take the MD5 hash extracted from the JWT payload
2. Hash the known plaintext candidate locally: `echo -n 'Passw0rd!' | md5sum`
3. Resulting hash matches the stored hash exactly, with no salt involved

### Evidence
- `screenshots/03-md5-unsalted-proof.png`

### Impact
MD5 is cryptographically broken and computationally trivial to reverse via rainbow tables or brute force, especially unsalted. Any database leak instantly exposes all user passwords in plaintext-equivalent form.

### Root Cause
The application hashes passwords with unsalted MD5 instead of a modern, slow, salted algorithm (bcrypt/argon2/scrypt), and does not enforce credential strength or rotation for privileged accounts.
