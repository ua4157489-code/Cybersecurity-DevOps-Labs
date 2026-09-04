# Findings — OWASP A03: Injection (Juice Shop)

## Finding 1: SQL Injection — Authentication Bypass

**Endpoint:** `POST /rest/user/login`
**Severity:** Critical
**CWE:** CWE-89 (SQL Injection)

### Steps to Reproduce
1. Submit a login request with the email field set to `admin@juice-sh.op'--` and any arbitrary password
2. The trailing `'--` closes the string literal in the backend SQL query and comments out the rest of the statement — including the password comparison
3. Server responds with a fully valid authentication token for the admin account, despite the real password never being supplied or known

### Evidence
- `raw-output/sqli_login_bypass.json`
- `screenshots/01-sqli-login-bypass.png`
- Decoded JWT payload confirms `role: admin` and returns the real admin password hash — proof this is a genuine authenticated admin session, not a placeholder token: `screenshots/02-sqli-jwt-decode-admin.png`

### Impact
Complete authentication bypass with zero valid credentials. An attacker who knows (or guesses) a target email address gains full admin access to the application — no password, brute force, or prior access required.

### Root Cause
The login query concatenates user-supplied input directly into a SQL statement instead of using parameterized queries / prepared statements, allowing attacker-controlled input to alter the query's logical structure.

---

## Finding 2: SQL Injection — UNION-Based Data Exfiltration via Search

**Endpoint:** `GET /rest/products/search?q=`
**Severity:** Critical
**CWE:** CWE-89 (SQL Injection)

### Steps to Reproduce
1. Baseline: search `q=apple` returns 3 legitimate product matches
2. Initial injection attempt `q=apple'))--` returns an empty result set (query broke, but not usefully)
3. Refined UNION-based payload, matching the product table's 9-column structure:
   ```
   q=xxx')) UNION SELECT id,email,password,'4','5','6','7','8','9' FROM Users--
   ```
4. Response returns 26 rows where the "product name" field contains a real user email and the "description" field contains that user's raw password hash — the entire `Users` table exfiltrated through a public, unauthenticated search box

### Evidence
- `raw-output/search_baseline.json`, `raw-output/search_sqli_union_users.json`
- `screenshots/03-sqli-union-users-dump.png`

### Impact
Total compromise of the user credential store via an endpoint that requires no authentication whatsoever. All 26 accounts on the system — including admin, CISO, support, and test accounts — had their email and password hash exposed in a single request. Combined with the weak MD5 hashing identified in the A02 lab, these hashes are trivially crackable offline.

### Root Cause
Same as Finding 1: the search endpoint builds its SQL query via string concatenation of user input rather than parameterized queries, and the resulting result set is returned to the client without validating that it matches the expected product schema/shape.
