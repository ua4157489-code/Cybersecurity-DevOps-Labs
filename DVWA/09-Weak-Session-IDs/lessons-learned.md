# 💡 Lab 09 — Lessons Learned

## Key Lessons

### 1. Session IDs Are Security-Sensitive

Session identifiers are used to maintain authenticated sessions and must be properly protected.

### 2. Randomness Is Important

Secure session IDs must be difficult to predict.

### 3. Sequential Values Are Dangerous

A sequence such as:

```text
6 → 7 → 8 → 9 → 10
```

is predictable and should not be used for secure session management.

### 4. Cookie Analysis Is Important

Inspecting cookies can reveal weaknesses in web application session management.

### 5. Repeated Testing Helps

Generating multiple session IDs makes predictable patterns easier to identify.

### 6. Sensitive Evidence Must Be Protected

Active cookies such as `PHPSESSID` should never be uploaded to a public GitHub repository.

---

## Skills Practiced

- HTTP request analysis
- Cookie inspection
- cURL
- Linux command-line tools
- Session security testing
- Vulnerability documentation

---

## Final Takeaway

A secure web application should generate session identifiers that are random, unpredictable, and resistant to enumeration.
