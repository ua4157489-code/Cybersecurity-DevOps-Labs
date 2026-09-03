# 🔬 Lab 09 — Methodology

## Objective

Identify whether the DVWA application generates predictable session identifiers.

---

## Testing Methodology

### 1. Access Application

The local DVWA instance was accessed at:

```text
http://localhost:4280
```

### 2. Configure Security

DVWA was configured to:

```text
Security Level: Low
```

### 3. Access Vulnerability

The Weak Session IDs module was opened.

### 4. Inspect Application

The page source was examined to understand how the application generates the `dvwaSession` cookie.

### 5. Generate Multiple Values

The Generate action was submitted five times using cURL.

### 6. Record Results

The `dvwaSession` value was captured after every request.

### 7. Analyze Pattern

The following sequence was observed:

```text
6 → 7 → 8 → 9 → 10
```

### 8. Determine Risk

Because the values follow an obvious sequential pattern, the session identifiers are predictable.

---

## Tools Used

- DVWA
- cURL
- Linux Terminal
- awk
- grep

---

## Result

The testing successfully identified predictable session ID generation.

---

## Testing Environment

The assessment was performed against a local DVWA instance in an isolated lab environment.
