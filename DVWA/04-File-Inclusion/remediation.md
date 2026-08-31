# Remediation – DVWA Local File Inclusion (LFI)

## 1. Overview

The Local File Inclusion (LFI) vulnerability exists because user-controlled input is used to determine which server-side file is processed by the application.

The vulnerable parameter identified during testing was:

```text
page
```

The application accepted a normal resource:

```text
page=file1.php
```

but also processed a local filesystem path:

```text
page=/etc/hostname
```

This demonstrates insufficient validation and access control over file selection.

The primary remediation objective is to ensure that users cannot control arbitrary server-side filesystem paths.

---

## 2. Eliminate Dynamic File Inclusion

The preferred remediation is to avoid dynamically including files based directly on user input.

Instead of:

```text
User Input
    ↓
File Path
    ↓
include()
```

the application should use predefined application resources.

Recommended architecture:

```text
User Input
    ↓
Validation
    ↓
Allowlisted Resource
    ↓
Known Application File
```

This prevents arbitrary filesystem paths from being processed.

---

## 3. Implement Strict Allowlisting

The strongest application-level control is to allow only explicitly approved resources.

For example:

```text
home  → file1.php
about → file2.php
help  → help.php
```

The user should submit an identifier such as:

```text
page=home
```

rather than an arbitrary filesystem path.

The server then maps that identifier to a predefined file.

Conceptually:

```text
Input: home
      ↓
Allowlist
      ↓
home → file1.php
      ↓
Load approved file
```

An input such as:

```text
/etc/hostname
```

should not exist in the allowlist and therefore should be rejected.

---

## 4. Perform Server-Side Validation

Validation must be performed on the server.

Client-side validation alone is insufficient because an attacker can modify requests directly.

The application should validate:

```text
page
```

before using it for any file operation.

Invalid input should result in a controlled error:

```text
Invalid page requested.
```

The application should not disclose filesystem information through error messages.

---

## 5. Avoid Blacklist-Based Filtering

A weak remediation strategy would be attempting to block specific strings such as:

```text
../
/etc/
/var/
/tmp/
```

Blacklist filtering is not sufficient because attackers may use alternative encodings or path representations.

The preferred approach is:

```text
Allow known-good values
```

rather than:

```text
Block known-bad values
```

---

## 6. Prevent Directory Traversal

If file paths must be accepted for legitimate application functionality, directory traversal must be prevented.

Suspicious traversal patterns include:

```text
../
../../
..\
```

Encoded variations should also be considered.

The application should verify that the final resolved path remains inside the intended directory.

Conceptually:

```text
Requested Path
      ↓
Canonicalize / Resolve
      ↓
Expected Directory?
   /          \
 YES           NO
 ↓              ↓
Allow          Reject
```

---

## 7. Canonicalize Paths

When an application legitimately handles filesystem paths, paths should be normalized or canonicalized before security decisions are made.

The security check should be performed on the resolved path rather than relying only on the raw user input.

The final path should be verified to remain within the intended application directory.

---

## 8. Use Safe File Mapping

A safer implementation uses a fixed mapping between user input and application files.

Example:

```text
Allowed Input       Server File

home                /var/www/dvwa/pages/file1.php
about               /var/www/dvwa/pages/file2.php
help                /var/www/dvwa/pages/help.php
```

The user cannot select arbitrary paths because only predefined identifiers are accepted.

---

## 9. Restrict Filesystem Permissions

The web application should run with the minimum filesystem privileges required.

The web server account should not have unnecessary access to sensitive system files.

Recommended model:

```text
Web Application
      ↓
Least-Privilege Account
      ↓
Limited Filesystem Access
      ↓
Reduced Impact
```

Filesystem permissions should follow the principle of least privilege.

---

## 10. Protect Sensitive Files

Sensitive configuration files, credentials, secrets, and internal application data should not be stored in locations unnecessarily accessible to the web application.

Where possible:

* Keep secrets outside web-accessible directories.
* Restrict filesystem permissions.
* Use secure secret-management mechanisms.
* Avoid storing credentials in source code.
* Prevent sensitive files from being directly exposed through the web application.

---

## 11. Secure Error Handling

The application should avoid revealing filesystem information in errors.

Avoid responses containing details such as:

```text
/var/www/html/...
/etc/...
/home/...
```

Instead, return a generic message:

```text
Invalid resource requested.
```

Detailed diagnostic information should be available only through secured server-side logs.

---

## 12. Apply Least Privilege

The web application process should operate with only the privileges necessary for normal functionality.

This limits the potential impact if an LFI vulnerability or another application vulnerability is exploited.

Least privilege should be applied to:

* Filesystem permissions
* Database permissions
* Service accounts
* Container permissions
* Network access
* Application processes

---

## 13. Implement Security Monitoring

Web server and application logs should be monitored for suspicious File Inclusion activity.

Potential indicators include:

```text
page=/etc/...
page=../...
page=../../...
page=%2e%2e%2f...
```

Other indicators include:

* Repeated requests against the `page` parameter.
* Requests for system configuration files.
* Requests containing traversal sequences.
* Unusual file extensions.
* Encoded filesystem paths.
* Repeated failed file requests.

These events can be forwarded to a SIEM for detection and investigation.

---

## 14. Web Application Firewall

A Web Application Firewall (WAF) can provide an additional defensive layer.

A WAF may detect or block suspicious requests containing:

* Directory traversal patterns.
* Known LFI payload patterns.
* Encoded traversal sequences.
* Suspicious filesystem paths.

However, a WAF should be treated as **defense in depth**.

It should not replace secure application-level validation.

---

## 15. Secure Application Architecture

A secure implementation should follow:

```text
                 User Request
                      ↓
              Input Validation
                      ↓
               Allowlist Check
                      ↓
              Resource Mapping
                      ↓
             Authorized File Only
                      ↓
                Safe Processing
```

The vulnerable architecture is:

```text
                 User Request
                      ↓
               page parameter
                      ↓
              Arbitrary File Path
                      ↓
               File Inclusion
```

The second design should be avoided.

---

## 16. Example Secure Logic

A simplified secure approach is:

```text
Receive page parameter

IF page is not in allowlist:
    Reject request

ELSE:
    Map page to predefined server-side file
    Load approved file
```

Example:

```text
page=home
        ↓
Allowed?
        ↓
YES
        ↓
file1.php
```

Whereas:

```text
page=/etc/hostname
        ↓
Allowed?
        ↓
NO
        ↓
Reject
```

---

## 17. Use Safe Defaults

The application should fail securely.

If the `page` parameter is:

* Missing
* Empty
* Invalid
* Unexpected
* Unauthorized

the application should load a safe default or reject the request.

It should never interpret unexpected input as an arbitrary filesystem path.

---

## 18. Verification After Remediation

After implementing remediation, security testing should be repeated.

### Test 1 – Valid Resource

Request:

```text
page=file1.php
```

Expected:

```text
Approved resource is loaded.
```

---

### Test 2 – Local File

Request:

```text
page=/etc/hostname
```

Expected:

```text
Request rejected.
```

The application must not display the local file contents.

---

### Test 3 – Directory Traversal

Request:

```text
page=../../some-file
```

Expected:

```text
Request rejected.
```

---

### Test 4 – Encoded Traversal

Test appropriate encoded representations of traversal sequences.

Expected:

```text
Request rejected.
```

The application should validate the canonicalized path rather than relying only on raw string matching.

---

## 19. Regression Testing

The remediation should not break legitimate application functionality.

Test:

```text
Valid Resource
     ↓
Expected Page Loads
```

and:

```text
Invalid Resource
     ↓
Request Rejected
```

Automated security tests should be added to prevent the vulnerability from being reintroduced during future development.

---

## 20. Secure Development Recommendations

Developers should follow secure coding practices when implementing file-related functionality.

Recommended practices:

* Avoid unnecessary dynamic includes.
* Never trust client-controlled file paths.
* Use allowlists.
* Validate input server-side.
* Canonicalize paths when necessary.
* Apply least privilege.
* Use secure error handling.
* Keep secrets outside accessible locations.
* Conduct regular security testing.
* Include security tests in CI/CD pipelines.

---

## 21. Defense-in-Depth Strategy

A layered security approach should include:

```text
Application Allowlisting
          ↓
Server-Side Validation
          ↓
Safe File Mapping
          ↓
Path Canonicalization
          ↓
Filesystem Permissions
          ↓
Least Privilege
          ↓
WAF / Network Controls
          ↓
Logging & SIEM Monitoring
```

Each layer reduces the likelihood or impact of exploitation.

---

## 22. Remediation Priority

| Priority             | Recommendation                               |
| -------------------- | -------------------------------------------- |
| Critical             | Remove arbitrary dynamic file inclusion      |
| High                 | Implement strict allowlisting                |
| High                 | Validate input server-side                   |
| High                 | Prevent directory traversal                  |
| Medium               | Apply least-privilege filesystem permissions |
| Medium               | Protect sensitive files                      |
| Medium               | Improve error handling                       |
| Medium               | Implement security monitoring                |
| Low/Defense-in-depth | WAF protections                              |

---

## 23. Remediation Verification Criteria

The vulnerability should be considered remediated only when:

```text
✓ Arbitrary file paths cannot be supplied
✓ Only approved resources can be loaded
✓ Directory traversal is prevented
✓ Server-side validation is enforced
✓ Sensitive files cannot be accessed
✓ Invalid requests are rejected
✓ Errors do not disclose filesystem information
✓ Security events are logged
```

---

## 24. Expected Secure Behavior

Before remediation:

```text
page=file1.php
        ↓
Application loads file

page=/etc/hostname
        ↓
Application loads local file
        ↓
LFI confirmed
```

After remediation:

```text
page=file1.php
        ↓
Allowlist validation
        ↓
Approved
        ↓
File loads


page=/etc/hostname
        ↓
Allowlist validation
        ↓
Rejected
```

---

## 25. Conclusion

The DVWA Local File Inclusion vulnerability can be effectively mitigated by preventing user-controlled input from directly determining server-side filesystem paths.

The most important remediation is to replace arbitrary file selection with a strict allowlist and predefined resource mapping.

Additional protections such as server-side validation, path canonicalization, directory traversal prevention, least-privilege filesystem permissions, secure error handling, WAF controls, and SIEM monitoring provide defense in depth.

The final security objective is:

```text
User Input
    ↓
Strict Validation
    ↓
Allowlist
    ↓
Known-Safe Resource
    ↓
Authorized File Processing
```

rather than:

```text
User Input
    ↓
Arbitrary File Path
    ↓
Server-Side File Inclusion
```

**Remediation Status:** Recommended

**Primary Fix:** Eliminate arbitrary file inclusion and implement strict allowlisting.

