# 📝 Lab 09 — Weak Session IDs Notes

## Session ID

A session ID is a value used by a web application to identify and maintain a user's session.

---

## Secure Session ID

A secure session ID should be:

- Random
- Unpredictable
- High entropy
- Protected from disclosure

---

## Weak Session ID

A weak session ID may be:

- Sequential
- Predictable
- Short
- Based on known information

---

## Observed Pattern

```text
6 → 7 → 8 → 9 → 10
```

The observed sequence demonstrates predictable session ID generation.

---

## Security Risks

Predictable session identifiers can increase the risk of:

- Session enumeration
- Session hijacking
- Unauthorized access

---

## Important Cookie Attributes

### Secure

Sends the cookie only over HTTPS.

### HttpOnly

Prevents client-side JavaScript from directly accessing the cookie.

### SameSite

Helps reduce certain cross-site request risks.

---

## Key Takeaway

Session identifiers must be generated using secure and unpredictable mechanisms.

---

## Lab Result

Successfully identified a predictable `dvwaSession` sequence in DVWA at Low security.
