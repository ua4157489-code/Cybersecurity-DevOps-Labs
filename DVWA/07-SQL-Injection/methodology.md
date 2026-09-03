# Methodology — SQL Injection Lab

## 1. Reconnaissance
Navigated to the "SQL Injection" module in DVWA (`/vulnerabilities/sqli/`). The page presents a single "User ID" input that returns a first/last name pair — a classic pattern for a backend query like:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '$id';
```

The single-quote-wrapped, unsanitized concatenation is the hypothesis to test.

## 2. Confirming the injection
Submitted a boolean-based payload designed to always evaluate true:

```
1' OR '1'='1
```

If the input were properly parameterized, this would either error out or return nothing. Instead, the application returned **all five rows** in the `users` table — confirming the query is vulnerable to injection and that the `OR '1'='1'` clause overrides the intended `WHERE user_id = '1'` filter.

## 3. Escalating to database enumeration
Since the query returns two columns, a UNION-based approach was viable without needing to determine column count via trial and error (already known from the visible output: 2 columns).

- Used `UNION SELECT database(), version()` to identify the active schema and DBMS version, both of which are useful for tailoring further payloads (MySQL/MariaDB-specific functions, known CVEs for that version, etc.).
- Queried `information_schema.tables` to enumerate table names within the current database without needing prior knowledge of the schema — a standard step in a black-box SQLi engagement.

## 4. Data exfiltration
With `users` identified as a table of interest (its name alone is a strong signal), pulled the `user` and `password` columns directly via UNION injection. No further authentication or privilege was required beyond the initial injection point.

## 5. Impact validation
The extracted `password` values were 32-character hex strings consistent with unsalted MD5. Cross-checked known DVWA default credentials and generated MD5 hashes locally to confirm the match, demonstrating that even without a cracking tool, these are reversible via public rainbow tables.

## 6. Why Low security level matters
At DVWA's Low setting, the vulnerable code performs no input sanitization, no parameterization, and no output encoding — this is intentionally the "textbook" case. Medium/High levels in DVWA introduce partial mitigations (e.g., `mysqli_real_escape_string`, restricted input) that change the required payload structure but often remain bypassable, which would be a natural next step beyond this lab.
