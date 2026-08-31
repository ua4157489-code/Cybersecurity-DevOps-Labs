# DVWA – Local File Inclusion (LFI)

## 1. Lab Overview

This lab demonstrates a **Local File Inclusion (LFI)** vulnerability in Damn Vulnerable Web Application (DVWA).

The vulnerability occurs when an application uses user-controlled input to determine which server-side file should be included without applying adequate validation or access controls.

In this lab, the vulnerable parameter is:

```text
page
```

The application initially loads a legitimate local page:

```text
page=file1.php
```

A controlled LFI test was then performed by supplying:

```text
page=/etc/hostname
```

The application returned the contents of the requested local file, confirming that user-controlled input could be used to access a local server-side file.

---

## 2. Lab Environment

| Item                 | Details                                |
| -------------------- | -------------------------------------- |
| Application          | Damn Vulnerable Web Application (DVWA) |
| Vulnerability        | Local File Inclusion (LFI)             |
| Security Level       | Low                                    |
| Target               | `localhost:8080`                       |
| Vulnerable Endpoint  | `/DVWA/vulnerabilities/fi/`            |
| HTTP Method          | GET                                    |
| Vulnerable Parameter | `page`                                 |
| Testing Environment  | Local / Isolated Lab                   |

---

## 3. Objective

The objectives of this assessment were to:

* Identify the vulnerable File Inclusion functionality.
* Identify the user-controlled parameter.
* Determine whether local files could be included.
* Validate the vulnerability using a harmless local system file.
* Capture HTTP/network evidence.
* Assess the potential security impact.
* Document appropriate remediation measures.

---

## 4. Vulnerability Identification

The File Inclusion page was accessed through:

```text
http://localhost:8080/DVWA/vulnerabilities/fi/
```

The normal application request contained:

```text
page=file1.php
```

The request was successfully processed with:

```text
HTTP/1.1 200 OK
```

This indicated that the `page` parameter controlled which resource was loaded by the application.

---

## 5. Normal Request

The original request was:

```text
GET /DVWA/vulnerabilities/fi/?page=file1.php HTTP/1.1
Host: localhost:8080
```

The application returned the expected page content.

This established the normal behavior before performing the controlled security test.

---

## 6. LFI Test

A controlled local-file inclusion test was performed by changing the `page` parameter to a harmless local system file:

```text
page=/etc/hostname
```

The resulting request was:

```text
GET /DVWA/vulnerabilities/fi/?page=/etc/hostname HTTP/1.1
Host: localhost:8080
```

The server responded with:

```text
HTTP/1.1 200 OK
```

The application processed the supplied local file path and returned its contents.

This confirmed the presence of Local File Inclusion.

---

## 7. Evidence

### Evidence 1 – LFI Result

The first screenshot documents the application behavior during the LFI test.

![LFI Result](screenshots/01-lfi-result.png)

**Evidence:** The application processes the supplied local file path and displays the resulting file content.

---

### Evidence 2 – Network Request

The second screenshot documents the HTTP request observed through the browser's Network Developer Tools.

![LFI Network Proof](screenshots/02-lfi-proof.png)

**Observed request:**

```text
GET /DVWA/vulnerabilities/fi/?page=/etc/hostname
```

**HTTP Status:**

```text
200 OK
```

**Security Level:**

```text
low
```

This provides network-level evidence that the `page` parameter accepted the local file path.

---

## 8. Attack Flow

The observed attack flow was:

```text
Attacker
   |
   |  page=/etc/hostname
   ↓
DVWA File Inclusion Endpoint
   |
   |  Processes user-controlled page parameter
   ↓
Local File Inclusion
   |
   ↓
/etc/hostname
   |
   ↓
File Content Returned
```

The important security issue is that the application trusted user-controlled input when determining which local resource should be included.

---

## 9. Technical Analysis

The vulnerable functionality accepts a value through the `page` parameter.

Normal input:

```text
page=file1.php
```

Controlled test input:

```text
page=/etc/hostname
```

The difference demonstrates that the application does not sufficiently restrict the value of the `page` parameter to approved application resources.

A secure application should not allow arbitrary filesystem paths to control server-side file inclusion.

---

## 10. Impact

The impact of an LFI vulnerability depends on the application's configuration, filesystem permissions, PHP/runtime behavior, and available files.

Potential consequences can include:

* Unauthorized access to local files.
* Disclosure of application configuration.
* Exposure of sensitive information.
* Disclosure of source or configuration files in vulnerable scenarios.
* Exposure of credentials or secrets stored in accessible files.
* Information gathering about the server environment.
* Potential escalation into more serious attacks when combined with other vulnerabilities.

In this lab, the demonstrated impact was **local file disclosure**.

---

## 11. Severity Assessment

**Suggested Severity: Medium**

The vulnerability allows an attacker to influence server-side file inclusion and retrieve local file content.

The actual severity in a production environment would depend on:

* Which files are accessible.
* Application privileges.
* Sensitive information stored on the host.
* Server configuration.
* Whether the vulnerability can be chained with another weakness.

---

## 12. Security Risk

The core security risk is:

```text
Untrusted User Input
        ↓
File Path
        ↓
Server-Side File Inclusion
        ↓
Local File Disclosure
```

The application should instead use:

```text
User Request
     ↓
Strict Validation
     ↓
Allowlisted Resource
     ↓
Safe File Handling
```

---

## 13. Detection Indicators

Security monitoring should look for suspicious values in file inclusion parameters.

Potential indicators include:

```text
page=/etc/hostname
page=/etc/...
page=../../...
page=../...
```

Other suspicious patterns may include:

* Directory traversal sequences.
* Unexpected filesystem paths.
* Encoded traversal sequences.
* Requests for operating-system configuration files.
* Requests for application configuration files.

Web server and application logs should be monitored for abnormal requests targeting file inclusion functionality.

---

## 14. Remediation Summary

The vulnerability should be addressed by preventing user input from directly determining arbitrary filesystem paths.

Recommended controls include:

1. Use an allowlist of permitted pages.
2. Avoid dynamic filesystem inclusion where possible.
3. Never trust user-controlled file paths.
4. Validate input server-side.
5. Normalize and canonicalize paths when appropriate.
6. Restrict filesystem permissions.
7. Run the application with least privilege.
8. Keep sensitive configuration files outside publicly accessible locations.
9. Implement security monitoring.
10. Add automated security testing.

---

## 15. Secure Allowlist Approach

Instead of allowing the user to provide an arbitrary filename:

```text
page=<user-controlled-path>
```

the application should map approved identifiers to known resources.

Conceptually:

```text
User Input
    ↓
Validate Against Allowlist
    ↓
Approved?
  /     \
YES      NO
 |        |
 ↓        ↓
Load     Reject
Known    Request
File
```

For example:

```text
home  → file1.php
about → file2.php
help  → help.php
```

The user should not be able to supply an arbitrary filesystem path.

---

## 16. Path Validation

If dynamic paths are unavoidable, the application should:

* Validate the requested resource.
* Canonicalize the path.
* Restrict access to an approved directory.
* Prevent directory traversal.
* Reject paths outside the permitted directory.
* Avoid trusting encoded or obfuscated path representations.

However, allowlisting known resources is generally preferable to attempting to block individual malicious patterns.

---

## 17. Least Privilege

The web application should operate with the minimum filesystem permissions required.

This reduces the potential impact if an inclusion vulnerability is discovered.

For example:

```text
Web Application
      ↓
Limited User Permissions
      ↓
Limited File Access
```

The web server should not have unnecessary access to sensitive system or application files.

---

## 18. Verification After Remediation

After remediation, the application should be retested.

### Valid Resource

```text
page=file1.php
```

Expected:

```text
Approved resource is loaded
```

### Unauthorized Local Path

```text
page=/etc/hostname
```

Expected:

```text
Request rejected
```

### Traversal Attempt

```text
page=../../some-file
```

Expected:

```text
Request rejected
```

The application should never return unauthorized local file contents.

---

## 19. Lessons Learned

This lab demonstrated several important web application security concepts:

* User-controlled parameters can become security-sensitive.
* File inclusion vulnerabilities can lead to local file disclosure.
* HTTP 200 responses can still represent security vulnerabilities.
* Network Developer Tools can provide valuable attack evidence.
* Input validation should be performed server-side.
* Allowlisting is generally safer than blacklist-based filtering.
* Least privilege helps reduce vulnerability impact.
* Security testing should include both application behavior and network-level evidence.

---

## 20. Portfolio Evidence Summary

| Evidence            | Description                                                    |
| ------------------- | -------------------------------------------------------------- |
| `01-lfi-result.png` | LFI result showing local file inclusion behavior               |
| `02-lfi-proof.png`  | Network request showing `page=/etc/hostname` and HTTP `200 OK` |

---

## 21. Conclusion

The DVWA File Inclusion functionality was successfully assessed at the **Low** security level.

Testing identified the `page` parameter as user-controlled and demonstrated that it could be manipulated to request a local file:

```text
page=/etc/hostname
```

The server returned:

```text
HTTP/1.1 200 OK
```

and the requested local file content was processed by the application.

This confirms a **Local File Inclusion (LFI)** vulnerability resulting in local file disclosure within the controlled DVWA environment.

The primary remediation is to eliminate arbitrary file selection through user input and implement strict allowlisting, secure file handling, server-side validation, and least-privilege access controls.

---

## 22. Lab Status

**Vulnerability:** Local File Inclusion (LFI)

**Status:** Confirmed

**Environment:** Controlled DVWA Lab

**Security Level:** Low

**Evidence:** 2 screenshots

**Primary Impact:** Local File Disclosure

**Recommended Fix:** Strict allowlisting and secure server-side file handling
