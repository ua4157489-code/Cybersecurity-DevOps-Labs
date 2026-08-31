# Remediation – DVWA File Upload

## 1. Overview

The identified File Upload vulnerability occurs because the application accepts files without sufficiently restricting their type and content.

During testing, the application accepted:

```text
test.txt
```

and:

```text
test.html
```

even though the functionality is intended for image uploads.

The primary remediation objective is to ensure that only legitimate, approved files can be uploaded and that uploaded content cannot be executed or abused.

---

## 2. Implement Server-Side Allowlisting

The application should maintain an explicit allowlist of permitted file types.

For an image-upload feature, the allowlist could include only approved formats such as:

```text
.jpg
.jpeg
.png
.gif
```

Everything else should be rejected.

Do not rely on a denylist such as:

```text
.php
.html
.exe
```

because attackers may use alternative extensions or bypass techniques.

---

## 3. Validate File Extension

The server should validate the uploaded file extension before accepting it.

For example:

```text
photo.jpg
```

may be allowed.

While:

```text
test.html
```

should be rejected.

However, extension validation alone is not sufficient because the filename is controlled by the user.

---

## 4. Validate MIME Type

The server should inspect the actual MIME type of the uploaded file.

For example:

```text
image/jpeg
image/png
image/gif
```

could be allowed for an image-upload feature.

The application should not blindly trust the MIME type supplied by the browser because it can be manipulated.

---

## 5. Validate File Signature / Content

The application should inspect the actual contents of the uploaded file.

For image uploads, server-side image parsing libraries can be used to verify that the file is actually a valid image.

Security flow:

```text
Uploaded File
      ↓
Extension Check
      ↓
MIME Check
      ↓
File Signature / Content Check
      ↓
Valid Image?
   ↙       ↘
 Yes        No
 ↓           ↓
Accept      Reject
```

Using multiple validation layers provides stronger protection.

---

## 6. Generate Safe Server-Side Filenames

The application should not use the original user-supplied filename as the final storage filename.

Instead, generate a random server-side name.

Example:

```text
User File:
profile.jpg
```

could become:

```text
8f31c9a7.jpg
```

This reduces filename-based attacks and prevents users from controlling storage names.

---

## 7. Store Files Outside the Web Root

Uploaded files should preferably be stored outside the publicly accessible web directory.

For example:

```text
Application
     ↓
Upload Handler
     ↓
Secure Storage
     ↓
Access Controlled File
```

rather than:

```text
Application
     ↓
Web Root
     ↓
Publicly Accessible File
```

If files must be publicly accessible, access should be mediated through a controlled application endpoint.

---

## 8. Disable Script Execution

The upload directory should not allow uploaded files to execute as server-side scripts.

For example, a web server should be configured so that uploaded content cannot be interpreted as:

```text
PHP
CGI
ASP
JSP
```

or other executable server-side content.

This is an important defense-in-depth control.

---

## 9. Apply File Size Limits

The application should enforce a maximum upload size.

For example:

```text
Maximum File Size
        ↓
Validation
        ↓
Too Large?
   ↙       ↘
 Yes        No
 ↓           ↓
Reject      Continue
```

This reduces the risk of resource exhaustion and denial-of-service conditions.

---

## 10. Restrict Filesystem Permissions

The account used by the web application should have only the permissions required to store uploaded files.

Apply the principle of least privilege:

```text
Web Application
      ↓
Minimum Required Permissions
      ↓
Upload Storage
```

The web application should not have unnecessary access to sensitive system resources.

---

## 11. Rename and Sanitize Filenames

User-controlled filenames should not be trusted.

The application should reject or sanitize dangerous path characters and patterns.

Examples of problematic input include:

```text
../
..\
/
\
```

The safest approach is to generate a server-side filename rather than attempting to sanitize every possible malicious filename.

---

## 12. Authentication and Authorization

Only authorized users should be permitted to upload files.

The application should verify:

```text
User Authentication
        ↓
Authorization
        ↓
Upload Permission
        ↓
File Validation
        ↓
Upload
```

Upload functionality should not automatically be available to every user.

---

## 13. Malware and Content Scanning

For applications handling untrusted files, additional security controls may include:

* Malware scanning.
* Antivirus integration.
* Content inspection.
* File reputation checks.
* Quarantine before final storage.

This provides another security layer for uploaded content.

---

## 14. Logging and Monitoring

All upload activity should be logged.

Useful events include:

```text
Username
Timestamp
Source IP
Original Filename
Validated File Type
File Size
Upload Result
Stored Filename
```

Security monitoring can identify suspicious upload behavior.

---

## 15. SIEM Detection Opportunities

Upload logs can be forwarded to a SIEM such as Wazuh or an ELK-based monitoring platform.

Potential detection rules could identify:

```text
Multiple failed uploads
        ↓
Suspicious file extensions
        ↓
Unusual upload frequency
        ↓
Access to uploaded files
        ↓
Unexpected web-server child processes
```

These events can be correlated to detect possible exploitation attempts.

---

## 16. Defense in Depth

File-upload security should not depend on a single control.

A layered approach should include:

```text
Authentication
      ↓
Authorization
      ↓
Extension Allowlist
      ↓
MIME Validation
      ↓
Content Validation
      ↓
File Size Limit
      ↓
Safe Filename
      ↓
Secure Storage
      ↓
Execution Disabled
      ↓
Logging & Monitoring
```

If one control fails, additional controls should still reduce the risk.

---

## 17. Secure Upload Behavior

### Valid File

```text
photo.jpg
     ↓
Extension Allowed
     ↓
MIME Valid
     ↓
Content Valid
     ↓
Safe Filename Generated
     ↓
Stored Securely
```

### Invalid File

```text
test.html
     ↓
Extension Not Allowed
     ↓
REJECT
```

The application should never allow an unauthorized file simply because the browser selected it successfully.

---

## 18. Verification After Remediation

After implementing remediation, the application should be retested.

### Test 1 – Valid Image

Example:

```text
photo.jpg
```

Expected:

```text
Accepted
```

after all server-side validation checks pass.

### Test 2 – HTML File

Example:

```text
test.html
```

Expected:

```text
Rejected
```

### Test 3 – Invalid Extension

Example:

```text
test.exe
```

Expected:

```text
Rejected
```

### Test 4 – Oversized File

Expected:

```text
Rejected
```

if it exceeds the configured upload limit.

---

## 19. Remediation Verification Matrix

| Test                                   | Expected Result |
| -------------------------------------- | --------------- |
| Valid JPEG                             | Accepted        |
| Valid PNG                              | Accepted        |
| HTML file                              | Rejected        |
| Executable file                        | Rejected        |
| Oversized file                         | Rejected        |
| Suspicious filename                    | Rejected        |
| Invalid image content                  | Rejected        |
| Script execution from upload directory | Disabled        |

---

## 20. Security Objective

The final security objective is:

```text
User
 ↓
Authenticated Request
 ↓
Authorized Upload
 ↓
Strict Validation
 ↓
Safe Filename
 ↓
Secure Storage
 ↓
Execution Disabled
 ↓
Logging
```

rather than:

```text
User
 ↓
Upload Arbitrary File
 ↓
Server Stores File
 ↓
Potential Abuse
```

---

## 21. Priority

Recommended remediation priority:

### High Priority

* Implement server-side file-type allowlisting.
* Validate actual file content.
* Disable script execution in upload directories.
* Store uploaded files securely.
* Generate server-side filenames.

### Medium Priority

* Add file-size restrictions.
* Implement malware/content scanning.
* Add detailed upload logging.
* Integrate security monitoring.

### Ongoing

* Perform regular vulnerability assessments.
* Review web-server configuration.
* Monitor upload activity.
* Keep application and security components updated.

---

## 22. Conclusion

The File Upload vulnerability can be mitigated by implementing strong server-side validation and secure file-handling practices.

The most important controls are:

```text
Allowlist
+
MIME Validation
+
Content Validation
+
Safe Filename
+
Secure Storage
+
Execution Restrictions
+
Monitoring
```

These controls significantly reduce the likelihood and impact of malicious or unintended file uploads.

The remediation should be verified by retesting both valid and invalid file types after the security controls have been implemented.

---

## 23. Evidence Reference

The original vulnerability evidence is stored in:

```text
screenshots/
├── 01-file-upload-result.png
├── 02-upload-network.png
└── 03-arbitrary-file-upload.png
```

These screenshots document the successful upload behavior and network-level request observed during the assessment.

---

## 24. Ethical Testing Statement

This remediation assessment relates exclusively to the deliberately vulnerable DVWA instance running in a controlled local environment.

No unauthorized external systems were tested.

The recommendations are intended for defensive security, secure application development, and authorized vulnerability assessment.
