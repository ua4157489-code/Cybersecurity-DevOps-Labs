# Findings - OWASP A09: Security Logging and Monitoring Failures

## Finding 1: Verbose Server Errors Expose Internal Implementation Details

**Endpoints:** `PUT /rest/basket/:id`, `GET /rest/products/:id` (and likely any route hitting the Angular catch-all handler)
**Severity:** Medium

### Steps to Reproduce
1. Send a request to a route pattern the backend doesn't explicitly handle (e.g. `PUT /rest/basket/2`, `GET /rest/products/1`)
2. Server returns HTTP 500 with a full HTML error page containing:
   - Internal file paths (`/juice-shop/build/routes/angular.js:18:18`, `/juice-shop/build/lib/insecurity.js:218:5`)
   - Node module internals (`express`, `express-jwt`, `jsonwebtoken`) with exact line numbers
   - The complete call stack, effectively exposing the server's internal routing and middleware chain

### Evidence
- `raw-output/verbose_error_basket_put.json`
- `raw-output/verbose_error_wrong_endpoint.json`
- `screenshots/01-verbose-stacktrace-leak.png`

### Impact
Detailed stack traces returned to the client leak internal architecture (file structure, dependency versions, middleware order) that materially aids an attacker in fingerprinting the stack and crafting further exploits (as demonstrated across this whole lab series, where library versions surfaced this way directly informed the A06 CVE work). From a logging/monitoring standpoint, this also indicates these errors are not being caught and logged server-side with a sanitized client response - they are effectively "logged" to the attacker instead of to the security team.

### Root Cause
No centralized error-handling middleware catches unhandled exceptions before they reach the client; the framework's default error handler is left in place in a way that surfaces full stack traces rather than a generic message plus server-side logging.

---

## Finding 2: No Brute-Force Detection or Throttling on Login

**Endpoint:** `POST /rest/user/login`
**Severity:** High

### Steps to Reproduce
1. Send 8 consecutive failed login attempts against a known account (`admin@juice-sh.op`) with different wrong passwords each time
2. All 8 attempts return `401 Unauthorized` with near-identical response times (16-24ms), no increasing delay, no lockout, no CAPTCHA challenge, and no visible change in server behavior at any point

### Evidence
- `screenshots/02-no-bruteforce-throttling.png`

### Impact
An attacker can perform unlimited password-guessing attempts against any account with no friction, rate-limiting, or detection response. Combined with the weak/default credentials already confirmed in the A02 lab, this significantly increases the practical risk of successful account compromise via brute force, and - critically for this category - there is no indication such an attack would ever be detected or alerted on.

### Root Cause
No rate-limiting middleware (e.g. `express-rate-limit` or equivalent) is applied to the authentication endpoint, and no failed-login-attempt counter, lockout mechanism, or alerting hook exists.

---

## Finding 3: No Exposed Security Event Logging or Notification Channel

**Endpoint:** `GET /rest/admin/application-configuration` (config inspection)
**Severity:** Medium

### Steps to Reproduce
1. Authenticate as admin and retrieve the full application configuration
2. Inspect the `ctf` and related config blocks for any security-notification or audit-log mechanism
3. Result: `systemWideNotifications.url` is `null`, `showFlagsInNotifications` is `false`, and no field anywhere in the configuration references a security event log, SIEM integration, or alerting endpoint

### Evidence
- `screenshots/03-no-security-notification-config.png`

### Impact
Across the entire lab series (A01-A09), no endpoint was found that exposes any record of security-relevant events (failed logins, unauthorized access attempts, admin actions, or data tampering). This is consistent with a systemic absence of security logging and monitoring, meaning that every prior finding in this lab series (IDOR access, admin credential compromise, price tampering, forged JWTs) would plausibly go completely undetected in this application as configured.

### Root Cause
No logging/monitoring/alerting subsystem is integrated into the application at the configuration level.
