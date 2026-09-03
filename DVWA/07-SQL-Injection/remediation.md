# Remediation — SQL Injection Lab

## Primary Fix: Parameterized Queries (Prepared Statements)
Never concatenate user input into a SQL string. Use bound parameters so the database driver treats input strictly as data, not executable query syntax.

**Vulnerable pattern (conceptual):**
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '" . $_GET['id'] . "'";
$result = mysqli_query($conn, $query);
```

**Fixed pattern (PHP + mysqli prepared statements):**
```php
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $_GET['id']);
$stmt->execute();
$result = $stmt->get_result();
```

Equivalent constructs exist in every major language/framework (PDO in PHP, parameterized queries in Python's `psycopg2`/`mysql-connector`, JPA/Hibernate named parameters in Java, `sqlx`/parameterized queries in Node, etc.). This is the single most effective fix and should be treated as non-negotiable for any code path that builds a query from user input.

## Defense in Depth

1. **Input validation** — Since `user_id` should be numeric, enforce that server-side (`is_numeric()` / cast to int / regex allow-list) before it ever reaches the query layer. This doesn't replace parameterization but reduces attack surface.
2. **Least privilege database accounts** — The web application's DB user should not have privileges to read `information_schema`, access unrelated schemas, or perform DDL/DML outside what the app strictly requires. This limits blast radius even if an injection is later found elsewhere.
3. **Password storage** — Replace unsalted MD5 with a modern adaptive hashing algorithm: `bcrypt`, `argon2id`, or `PBKDF2` with a high work factor and per-user salt. MD5 (salted or not) should never be used for password storage.
4. **Web Application Firewall (WAF)** — Not a substitute for fixing the code, but a WAF (e.g., ModSecurity with the OWASP Core Rule Set) can catch and block common injection payloads as a compensating control while remediation is rolled out.
5. **Error handling** — Ensure database errors are never returned to the client; verbose SQL error messages assist attackers in refining injection payloads (error-based SQLi).
6. **Automated testing** — Add SAST (e.g., static analysis for string-concatenated queries) and DAST (e.g., automated SQLi scanning with tools like `sqlmap` in a controlled pipeline) to CI to catch regressions.

## Verification
After remediation, re-run the payloads documented in `commands.md` against the patched endpoint. All of them should either return no results, a generic error, or be rejected outright — the injected SQL should never be interpreted as part of the query structure.
