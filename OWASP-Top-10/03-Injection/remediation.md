# Remediation — OWASP A03: Injection

## Finding 1: SQL Injection — Authentication Bypass

**Fix:** Never build SQL queries by concatenating user input into a query string. Use parameterized queries / prepared statements everywhere, so the database treats input strictly as data, never as executable query syntax:

```js
// Vulnerable
db.query(`SELECT * FROM Users WHERE email = '${email}' AND password = '${hash}'`)

// Fixed
db.query('SELECT * FROM Users WHERE email = ? AND password = ?', [email, hash])
```

If an ORM is in use, ensure raw query interpolation is avoided in favor of the ORM's built-in parameter binding.

## Finding 2: SQL Injection — UNION-Based Data Exfiltration

**Fix:**
- Apply the same parameterized-query fix to the search endpoint — this is the same root cause as Finding 1, just a different query.
- Add strict input validation on the search parameter (allow-list expected characters for a product search term; reject SQL metacharacters like unescaped quotes).
- Enforce least privilege on the database account the application uses: the account powering product search should not have read access to the `Users` table (or any table outside its functional need) at the database permission level. Defense in depth here would have prevented data exfiltration even if the injection itself succeeded.
- Return generic error messages to clients on query failure — don't let database errors leak schema details (table/column names) that assist further exploitation.

**General principle:** Injection vulnerabilities exist because user input is trusted as part of a command's structure, not just its data. The fix is always the same regardless of endpoint: separate code from data at the query-construction layer, validate input, and apply least-privilege database access as a second line of defense.

**Recommended controls:**
- Static analysis / SAST tooling in CI to flag string-concatenated SQL queries
- Web Application Firewall (WAF) rules for common SQLi payload patterns as a compensating control, not a substitute for fixing the code
- Regular database privilege audits — application service accounts should never have blanket read access across all tables
- Automated regression tests asserting that known injection payloads no longer alter query behavior
