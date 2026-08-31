# DVWA — Brute Force Authentication Testing

![DVWA](https://img.shields.io/badge/Target-DVWA-red)
![Security Level-Low](https://img.shields.io/badge/Security%20Level-Low-orange)
![Category-Authentication](https://img.shields.io/badge/Category-Authentication-blue)
![Status-Successful](https://img.shields.io/badge/Result-Successful-brightgreen)

## 📌 Overview

This lab demonstrates a **Brute Force Authentication Attack** against the Brute Force module of **Damn Vulnerable Web Application (DVWA)**.

The objective was to understand how weak authentication controls can allow an attacker to repeatedly submit password guesses until the valid password is discovered.

> **Lab environment only:** This test was performed against a locally hosted DVWA instance in an isolated environment.

---

## 🎯 Objectives

* Understand how a web application's login endpoint handles repeated authentication attempts.
* Identify whether password guessing is rate-limited.
* Capture and analyze HTTP authentication requests.
* Automate password testing using a simple Bash script.
* Identify the successful authentication response.
* Document the security impact.
* Recommend defensive controls against brute-force attacks.

---

## 🧪 Lab Environment

| Component        | Details                       |
| ---------------- | ----------------------------- |
| Target           | DVWA                          |
| Target URL       | `http://localhost:8080/DVWA/` |
| Vulnerability    | Brute Force Authentication    |
| Security Level   | Low                           |
| Web Server       | Apache/2.4.52 (Ubuntu)        |
| Protocol         | HTTP                          |
| HTTP Method      | GET                           |
| Username Tested  | `admin`                       |
| Testing Platform | Ubuntu Linux                  |
| Browser          | Firefox                       |
| Automation       | Bash                          |
| Network Scope    | Localhost / isolated lab      |

---

# 🔎 Attack Surface

The vulnerable endpoint was:

```text
GET /DVWA/vulnerabilities/brute/
```

The login parameters were submitted through the URL:

```text
username=admin
password=<password>
Login=Login
```

Example request:

```text
GET /DVWA/vulnerabilities/brute/?username=admin&password=password&Login=Login
```

Because the application uses the **GET** method, the supplied username and password appear directly in the URL/query string.

---

# 🛠️ Methodology

The test followed these steps:

1. Opened the DVWA Brute Force module.
2. Set the DVWA security level to **Low**.
3. Inspected the authentication request using browser Developer Tools.
4. Identified the following parameters:

```text
username
password
Login
```

5. Tested individual passwords using `curl`.
6. Confirmed that the authenticated session required the appropriate DVWA session cookie.
7. Created a Bash script to automate password testing.
8. Compared the application's response for incorrect and correct credentials.
9. Identified the valid password based on the application's response.
10. Documented the vulnerability, impact, and remediation.

---

# 💻 Manual HTTP Testing

A request was first tested manually:

```bash
curl -i \
  -H 'Cookie: PHPSESSID=YOUR_SESSION_ID; security=low' \
  "http://localhost:8080/DVWA/vulnerabilities/brute/?username=admin&password=password&Login=Login"
```

### Successful Response

The server returned:

```text
HTTP/1.1 200 OK
```

The response body contained:

```text
Welcome to the password protected area admin
```

This confirmed that the supplied credentials were accepted.

---

# ❌ Incorrect Password Test

An incorrect password was also tested:

```bash
curl -i \
  -H 'Cookie: PHPSESSID=YOUR_SESSION_ID; security=low' \
  "http://localhost:8080/DVWA/vulnerabilities/brute/?username=admin&password=admin123&Login=Login"
```

The application returned:

```text
HTTP/1.1 200 OK
```

but the response body contained:

```text
Username and/or password incorrect.
```

### Observation

The HTTP status code remained `200 OK` for both successful and failed attempts.

Therefore, the **response body**, rather than the HTTP status code alone, was useful for distinguishing authentication results.

---

# 🤖 Automated Password Testing

A Bash script was used to automate password testing against the local DVWA instance.

Example result:

```text
[-] Failed: 123456
[+] Password found: password
```

### Result

The automated test successfully identified:

```text
Username: admin
Password: password
```

The result demonstrates that the application allowed repeated password attempts without an effective brute-force protection mechanism at the selected security level.

---

# 📸 Evidence

### Evidence 1 — HTTP Request

The browser Developer Tools Network panel showed requests similar to:

```text
GET /DVWA/vulnerabilities/brute/?username=admin&password=admin123&Login=Login
```

The request included the authenticated DVWA session cookie and:

```text
security=low
```

---

### Evidence 2 — Failed Authentication

The application returned:

```text
Username and/or password incorrect.
```

This demonstrated the application's response for an invalid password.

---

### Evidence 3 — Successful Authentication

The valid password produced:

```text
Welcome to the password protected area admin
```

This confirmed successful authentication.

---

### Evidence 4 — Automated Test

The Bash automation produced:

```text
[-] Failed: 123456
[+] Password found: password
```

This is the primary evidence that password guessing was successful.

> Screenshots of the Network panel, successful DVWA response, failed response, and terminal output should be stored in the `screenshots/` directory.

Recommended filenames:

```text
screenshots/
├── 01-network-request.png
├── 02-failed-login.png
├── 03-successful-login.png
└── 04-brute-force-result.png
```

---

# 🔬 Technical Analysis

The vulnerability exists because the authentication mechanism does not provide sufficient protection against repeated password attempts.

An attacker who knows or guesses a valid username can repeatedly submit different passwords.

The observed workflow was:

```text
Attacker
   │
   │ Username + Password Guess
   ▼
DVWA Login Endpoint
   │
   ├── Invalid ──► "Username and/or password incorrect."
   │
   └── Valid ───► "Welcome to the password protected area admin"
                         │
                         ▼
                  Authentication Success
```

The automated script repeated this process until the application's successful response was detected.

---

# ⚠️ Security Impact

A brute-force vulnerability can allow an attacker to compromise accounts when weak or commonly used passwords are present.

Potential impacts include:

* 🔐 **Account compromise**
* 📂 Unauthorized access to protected resources
* 👤 User impersonation
* 🕵️ Unauthorized access to sensitive information
* 🚨 Increased risk when administrative accounts are targeted
* 🔑 Credential reuse attacks against other services

The risk becomes significantly higher when:

* Passwords are weak.
* There is no account lockout.
* There is no rate limiting.
* MFA is not enabled.
* Login attempts are not monitored.
* Authentication failures are not logged or alerted.

---

# 🛡️ Remediation

A production application should implement multiple layers of authentication protection.

### 1. Rate Limiting

Restrict the number of authentication attempts from a user, IP address, or account within a specific time period.

Example:

```text
5 failed attempts
        ↓
Temporary delay / rate limit
        ↓
Additional attempts slowed down
```

---

### 2. Account Lockout / Progressive Delay

Temporarily lock or slow authentication attempts after repeated failures.

Avoid permanent lockouts where possible because attackers can abuse them to cause denial of service against legitimate users.

---

### 3. Multi-Factor Authentication

Require an additional authentication factor such as:

* TOTP
* Security key
* Push authentication

MFA significantly reduces the impact of a stolen or guessed password.

---

### 4. Strong Password Policy

Enforce passwords that are resistant to common password guessing.

Organizations should also prevent the use of:

```text
password
123456
admin
qwerty
```

and other commonly compromised credentials.

---

### 5. Secure Password Storage

Passwords should never be stored in plaintext.

Use a password hashing algorithm designed for password storage, such as:

* Argon2id
* bcrypt
* scrypt

with appropriate parameters.

---

### 6. Monitoring and Detection

Authentication failures should be logged and monitored.

A SOC/SIEM solution can detect patterns such as:

```text
Multiple failed logins
        ↓
Same account
        ↓
Same source IP
        ↓
Short time interval
        ↓
Possible Brute Force Attack
```

Useful detection indicators include:

* High number of failed logins
* Multiple usernames from one source
* Repeated authentication attempts
* Successful login immediately after many failures
* Unusual login locations or source IPs

---

### 7. Avoid GET for Password Submission

The lab uses:

```text
GET /DVWA/vulnerabilities/brute/?username=admin&password=...
```

Production authentication forms should generally use **POST** rather than GET so credentials are not placed in the URL/query string.

HTTPS should also be enforced to protect credentials in transit.

---

# 🧠 SOC Analyst Perspective

From a SOC perspective, a brute-force attack can be investigated by correlating authentication events.

Example detection pattern:

```text
Source IP: 192.168.1.50

10:01:01  Login failure
10:01:02  Login failure
10:01:03  Login failure
10:01:04  Login failure
10:01:05  Login failure
10:01:06  Login success
```

This pattern should generate an alert because a successful authentication occurred immediately after multiple failures.

### Example SIEM Investigation Questions

When investigating a suspected brute-force alert, ask:

1. Which account was targeted?
2. What source IP generated the attempts?
3. How many failures occurred?
4. Over what time period?
5. Was authentication eventually successful?
6. Was the source IP known or suspicious?
7. What activity occurred after successful authentication?
8. Should the account or source IP be contained?

---

# 🧰 Tools Used

| Tool                    | Purpose                    |
| ----------------------- | -------------------------- |
| DVWA                    | Vulnerable web application |
| Firefox Developer Tools | HTTP request inspection    |
| curl                    | Manual HTTP testing        |
| Bash                    | Automation                 |
| Ubuntu                  | Testing platform           |

---

# 📊 Findings Summary

| Finding                                   | Severity | Status       |
| ----------------------------------------- | -------- | ------------ |
| Brute-force protection weakness           | High*    | Confirmed    |
| Repeated authentication attempts possible | High*    | Confirmed    |
| Weak password accepted                    | High*    | Confirmed    |
| GET parameters contain credentials        | Medium*  | Observed     |
| Rate limiting                             | High*    | Not observed |

*Severity is contextual for a real-world application. DVWA is intentionally vulnerable, so this rating is primarily for demonstrating the potential security impact.

---

# ✅ Conclusion

The DVWA Brute Force module successfully demonstrated how an attacker can automate password guessing against an authentication endpoint when adequate brute-force protections are absent.

The test successfully identified the valid password for the `admin` account:

```text
password
```

The exercise demonstrated several important security concepts:

* HTTP request analysis
* Authentication testing
* Session handling
* Automated password testing
* Response-based detection
* Brute-force attack analysis
* Security impact assessment
* Authentication hardening
* SOC detection and investigation

The key defensive lesson is that **authentication should never rely solely on a username and password**. Rate limiting, MFA, strong password controls, secure password storage, logging, and monitoring should be implemented together.

---

# 📚 References

* OWASP — Brute Force Attack
* OWASP — Authentication Security
* DVWA — Damn Vulnerable Web Application

---

## 📁 Evidence Structure

```text
01-Brute-Force/
│
├── README.md
├── commands.md
├── findings.md
├── methodology.md
├── remediation.md
│
└── screenshots/
    ├── 01-network-request.png
    ├── 02-failed-login.png
    ├── 03-successful-login.png
    └── 04-brute-force-result.png
```

---

## ⚖️ Ethical / Legal Note

This exercise was conducted against a deliberately vulnerable DVWA instance running locally for educational purposes.

Brute-force testing should only be performed against systems where you have explicit authorization.
