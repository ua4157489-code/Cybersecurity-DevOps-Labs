# Findings — SQL Injection (Blind) Lab

## Vulnerability
**Type:** SQL Injection (Blind, Boolean-based)
**Location:** `/vulnerabilities/sqli_blind/?id=` (User ID parameter, GET)
**CWE:** CWE-89 — Improper Neutralization of Special Elements used in an SQL Command
**Severity:** Critical
**CVSS 3.1 (estimated):** 9.1 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N)

## Description
The `id` parameter on DVWA's Blind SQL Injection page is concatenated directly into a SQL query without sanitization, identical in root cause to Lab 07's standard SQL Injection. The key difference is that this endpoint returns no query data directly — only a binary "User ID exists" / no-message signal. This does not prevent exploitation; it only changes the technique. Using the exists/does-not-exist signal as an oracle, an attacker can reconstruct arbitrary database contents one bit of information at a time via automated, scripted requests.

## Evidence

### 1. Boolean oracle confirmed
| Payload | Response |
|---|---|
| `1` (baseline) | `User ID exists` |
| `1' AND '1'='1` (TRUE) | `User ID exists` |
| `1' AND '1'='2` (FALSE) | *(no message)* |

### 2. Database name extracted via automated character-by-character inference
- **Length probe:** `1' AND LENGTH(database())=4-- -` → confirmed length of 4
- **Character probes:** `1' AND ASCII(SUBSTRING(database(),N,1))=X-- -` for each position `N`
- **Result:** `dvwa` — extracted purely from true/false responses, no data ever directly echoed

### 3. Admin password hash extracted the same way
- **Length probe:** confirmed 32 characters (consistent with unsalted MD5)
- **Character probes:** iterated ASCII values 48–102 (hex charset) per position
- **Result:** `5f4dcc3b5aa765d61d8327deb882cf99`

### 4. Cross-validation against Lab 07
The blind-extracted password hash was compared against the value obtained via direct UNION-based extraction in Lab 07 (regular SQL Injection):

| Source | Value |
|---|---|
| Lab 07 (UNION SELECT, direct read) | `5f4dcc3b5aa765d61d8327deb882cf99` |
| Lab 08 (Blind, inferred bit-by-bit) | `5f4dcc3b5aa765d61d8327deb882cf99` |

**Exact match** — independently confirming both the vulnerability and the correctness of the extraction technique.

## Impact
- **Confidentiality:** Complete — any data in the database is theoretically extractable given enough automated requests, including credentials, as demonstrated.
- **Integrity/Availability:** Not directly demonstrated in this lab (read-only boolean oracle), but the same underlying injection point could support write operations (`AND (SELECT ... UPDATE ...)` style subqueries) if DB privileges allow.
- **Practical consideration:** Blind extraction requires materially more requests than direct extraction (roughly 40–95 requests per character in this test, depending on search-space narrowing), but this is a performance difference, not a security difference — automated tooling (e.g., `sqlmap`) reduces this to a non-issue in real-world attacks. The absence of visible query output should never be treated as a mitigating factor when scoring or triaging this class of vulnerability.

## Root Cause
Identical to Lab 07: user-supplied input is concatenated directly into a SQL query string instead of being passed as a bound parameter. The application additionally reveals a distinguishable, attacker-observable signal (the "exists" message) tied directly to query truthiness, which is precisely what makes blind exploitation practical rather than merely theoretical.
