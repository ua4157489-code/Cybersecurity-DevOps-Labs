# Security Findings — DVWA Brute Force

## Finding: Insufficient Brute-Force Protection

### Finding ID

```text
DVWA-BF-001
```

### Severity

**High**

### Vulnerability Type

**Authentication / Brute Force**

### CWE

**CWE-307 — Improper Restriction of Excessive Authentication Attempts**

---

## 1. Description

The DVWA Brute Force module permits repeated authentication attempts without effective protection against password guessing.

During testing, multiple password candidates were submitted against the `admin` account. The application continued processing authentication attempts and ultimately accepted the valid password.

The testing was performed against a deliberately vulnerable DVWA instance running locally.

---

## 2. Affected Endpoint

```text
GET /DVWA/vulnerabilities/brute/
```

### Parameters

```text
username
password
Login
```

Example:

```text
GET /DVWA/vulnerabilities/brute/?username=admin&password=<PASSWORD>&Login=Login
```

---

## 3. Preconditions

The following conditions were present in the lab:

* DVWA was running locally.
* DVWA security level was set to **Low**.
* A valid DVWA session was available.
* The target account was `admin`.
* The tester had authorization to perform the assessment.

---

## 4. Evidence

### Failed Authentication

An invalid password produced:

```text
HTTP/1.1 200 OK
```

with the response:

```text
Username and/or password incorrect.
```

---

### Successful Authentication

A valid password produced:

```text
HTTP/1.1 200 OK
```

with:

```text
Welcome to the password protected area admin
```

---

### Automated Testing Evidence

The automated test produced:

```text
[-] Failed: 123456
[+] Password found: password
```

This confirms that the password-testing process successfully identified a valid credential in the intentionally vulnerable lab.

---

## 5. Technical Analysis

The application accepts authentication attempts through a GET request.

The password is supplied as a URL parameter:

```text
password=<PASSWORD>
```

The application provides distinguishable responses for failed and successful authentication.

An attacker can therefore:

1. Identify the authentication endpoint.
2. Identify the required parameters.
3. Submit multiple password candidates.
4. Compare the application responses.
5. Identify a successful authentication attempt.

The lack of effective brute-force protections increases the feasibility of automated password guessing.

---

## 6. Root Cause

The primary issue is insufficient authentication-attempt controls.

The application does not adequately enforce mechanisms such as:

* Rate limiting
* Progressive delays
* Account protection
* CAPTCHA/risk-based challenges
* MFA
* Brute-force detection

Additionally, the lab uses GET parameters for authentication, which exposes credentials in the URL.

---

## 7. Security Impact

Successful brute-force attacks can result in:

### Account Compromise

An attacker may gain access to a user's account if a weak password is discovered.

### Unauthorized Access

A compromised account can provide access to resources and functionality available to that user.

### Privilege Escalation Risk

If an administrative account is compromised, the potential impact can be significantly greater.

### Credential Reuse

A discovered password may also be useful against other services if the user reuses credentials.

### Data Exposure

Compromised accounts may expose sensitive application data.

---

## 8. Risk Assessment

| Factor                  | Assessment                   |
| ----------------------- | ---------------------------- |
| Attack Complexity       | Low                          |
| Authentication Required | Session required in this lab |
| User Interaction        | None                         |
| Exploitability          | High in lab configuration    |
| Confidentiality Impact  | Potentially High             |
| Integrity Impact        | Potentially High             |
| Availability Impact     | Potentially Low              |
| Overall Risk            | High                         |

> Risk assessment is contextual. DVWA is intentionally designed to be vulnerable and should not be treated as a production system.

---

## 9. Recommended Remediation

### Priority 1 — Implement Rate Limiting

Limit authentication attempts by account, IP address, device, or a combination of signals.

Example:

```text
Repeated failed attempts
        ↓
Rate limit
        ↓
Progressive delay
        ↓
Additional verification
```

---

### Priority 2 — Implement MFA

Require a second authentication factor for sensitive accounts.

Examples:

* TOTP
* Hardware security keys
* Push authentication

---

### Priority 3 — Detect Excessive Failed Logins

Generate security events when an unusual number of failed authentication attempts occur.

Example detection rule:

```text
10+ failed logins
from the same source
within a short period
        ↓
Generate security alert
```

---

### Priority 4 — Use Strong Password Controls

Prevent commonly used and compromised passwords.

Examples of weak passwords that should be rejected:

```text
password
123456
admin
qwerty
```

---

### Priority 5 — Secure Password Storage

Passwords should be stored using modern password-hashing algorithms such as:

```text
Argon2id
bcrypt
scrypt
```

Passwords should never be stored in plaintext.

---

### Priority 6 — Use POST for Authentication

The lab submits credentials through:

```text
GET /...?username=admin&password=...
```

Production applications should generally submit credentials using **POST** and enforce HTTPS.

This prevents passwords from unnecessarily appearing in URLs, browser history, proxy logs, and other URL-based logging.

---

## 10. SOC Detection Recommendations

A SOC should monitor authentication telemetry for brute-force behavior.

### Detection Scenario

```text
Source IP
    │
    ├── Failed login
    ├── Failed login
    ├── Failed login
    ├── Failed login
    ├── Failed login
    │
    ▼
Successful login
    │
    ▼
SOC Alert
```

### Useful Detection Signals

* Multiple failed logins from one source.
* Multiple usernames targeted by one source.
* Rapid authentication attempts.
* Successful login following many failures.
* Repeated attempts against privileged accounts.
* Authentication activity from unusual sources.

---

## 11. Suggested SIEM Alert

### Alert Name

```text
Possible Brute Force Authentication Attack
```

### Example Logic

```text
IF
failed_login_count >= 5
WITHIN
5 minutes
FROM
same source IP
THEN
generate security alert
```

A production detection rule should be tuned to the organization's normal authentication behavior to reduce false positives.

---

## 12. Validation After Remediation

After implementing protections, repeat the same authorized test.

Expected behavior:

```text
Attempt 1 → Failed
Attempt 2 → Failed
Attempt 3 → Failed
Attempt 4 → Failed
Attempt 5 → Failed
Attempt 6 → Rate Limited / Delayed
```

Additional validation should confirm:

* Rate limiting is triggered.
* Security events are logged.
* SOC/SIEM alerts are generated.
* MFA is enforced where required.
* Successful authentication remains possible for legitimate users.
* Passwords are not exposed in URLs.

---

# 13. Final Assessment

**Status:** Confirmed

The DVWA Brute Force module was successfully tested in the local lab environment. Automated password testing identified a valid credential, demonstrating the impact of insufficient brute-force protection.

The primary remediation is to implement **rate limiting, progressive authentication delays, MFA, strong password controls, secure password handling, and monitoring/detection**.

---

## Evidence Checklist

* [x] Target identified
* [x] Authentication endpoint identified
* [x] HTTP request analyzed
* [x] Failed authentication confirmed
* [x] Successful authentication confirmed
* [x] Automated testing performed
* [x] Vulnerability confirmed
* [x] Impact assessed
* [x] Remediation documented
* [x] SOC detection recommendations documented
