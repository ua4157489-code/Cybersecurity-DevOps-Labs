# Lab 07: SQL Injection

## Overview
This lab demonstrates a classic in-band (UNION-based) SQL Injection vulnerability in DVWA's "SQL Injection" module, exploited at **Low** security level. The application concatenates unsanitized user input directly into a SQL query, allowing an attacker to bypass intended query logic, fingerprint the backend database, enumerate schema, and exfiltrate user credentials.

## Environment
- **Target:** DVWA (Damn Vulnerable Web Application) v1.10 *Development*
- **Deployment:** Docker (`vulnerables/web-dvwa`), mapped to `http://localhost:4280`
- **Backend DB:** MariaDB `10.1.26-MariaDB-0+deb9u1`
- **Security Level:** Low
- **PHPIDS:** Disabled

## Objective
Demonstrate authentication bypass, database enumeration, and credential extraction via SQL Injection in the "User ID" lookup field.

## Summary of Impact
An unauthenticated injection point in a single input field allowed full compromise of the `users` table, exposing every account's username and password hash. Combined with the use of unsalted MD5 hashing, this represents a **Critical** severity finding — full credential compromise with negligible attacker effort.

## Files
| File | Description |
|---|---|
| `commands.md` | Exact commands run during testing |
| `methodology.md` | Step-by-step approach and reasoning |
| `findings.md` | Extracted data and vulnerability details |
| `remediation.md` | Recommended fixes |
| `screenshots/` | Visual evidence |

## References
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [PortSwigger: SQL Injection](https://portswigger.net/web-security/sql-injection)
- [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
