# 🔎 Lab 09 — Findings

## Finding

### Predictable Session IDs

**Severity:** Medium

The DVWA Weak Session IDs module generates session identifiers using a predictable sequential pattern.

---

## Observed Values

```text
Request 1: dvwaSession=6
Request 2: dvwaSession=7
Request 3: dvwaSession=8
Request 4: dvwaSession=9
Request 5: dvwaSession=10
```

---

## Pattern

```text
6 → 7 → 8 → 9 → 10
```

The value increases by exactly one after every generation request.

---

## Vulnerability

The session identifier is predictable and therefore does not provide the randomness expected from a secure session token.

---

## Potential Impact

An attacker who can reliably predict valid session identifiers may increase the risk of:

- Session enumeration
- Session hijacking
- Unauthorized access
- Session-management attacks

---

## Root Cause

The application uses a predictable sequential mechanism for generating the `dvwaSession` value.

---

## Evidence

Raw evidence:

```text
raw-output/session-sequence.txt
raw-output/vulnerable-page.txt
```

Screenshots:

```text
screenshots/01-weak-session-ids-page.png
screenshots/02-sequential-session-ids.png
screenshots/03-vulnerable-page-evidence.png
screenshots/04-generated-session-evidence.png
```

---

## Recommendation

Use a cryptographically secure random session ID generator with sufficient entropy.

Session cookies should also use appropriate security attributes such as:

- Secure
- HttpOnly
- SameSite

---

## Conclusion

The predictable sequence confirms a Weak Session IDs vulnerability at the selected DVWA security level.
