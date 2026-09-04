# Remediation — OWASP A02: Cryptographic Failures

## Finding 1: Default Admin Credentials

**Fix:**
- Force credential rotation on first login for all seeded/default accounts, or eliminate default accounts entirely in favor of an explicit setup/invite flow.
- Enforce a strong password policy (minimum length, complexity, blocklist of common passwords) at account creation.
- Add rate limiting and account lockout / exponential backoff on the login endpoint to slow down credential-stuffing attempts.

## Finding 2: Sensitive Data in JWT Payload

**Fix:**
- Keep JWT payloads minimal — only include what's strictly needed for authorization (e.g., user ID, role), never password hashes or other sensitive fields.
- If richer user data is needed server-side, look it up from the database using the ID in the token rather than embedding it client-side.
- Consider short-lived tokens plus refresh tokens to limit the exposure window if a token is intercepted.

## Finding 3: Unsalted MD5 Password Hashing

**Fix:**
- Replace MD5 with a modern, purpose-built password hashing algorithm: bcrypt, scrypt, or argon2id. These are deliberately slow and resistant to GPU/rainbow-table attacks.
- Always use a unique, randomly generated salt per user, stored alongside the hash (most modern libraries handle this automatically).
- Migrate existing users transparently: re-hash with the new algorithm on their next successful login.

**General principle:** Cryptographic controls fail silently — a broken hash or an over-exposed token doesn't produce an error, it just quietly gives attackers what they need. Treat password storage and token design as security-critical code requiring dedicated review, not incidental implementation details.

**Recommended controls:**
- Automated dependency/config scanning to flag deprecated crypto primitives (MD5, SHA1, DES) in code review or CI
- Periodic credential audits for default/seeded accounts in any environment
- JWT payload review as part of API design sign-off, not an afterthought
