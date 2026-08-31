# Findings – DVWA File Upload

## 1. Finding Summary

| Field             | Details                         |
| ----------------- | ------------------------------- |
| Finding           | Unrestricted File Upload        |
| Severity          | High*                           |
| OWASP Category    | Unrestricted File Upload        |
| CWE               | CWE-434                         |
| Affected Endpoint | `/DVWA/vulnerabilities/upload/` |
| HTTP Method       | POST                            |
| Security Level    | Low                             |
| Status            | Confirmed                       |
| Environment       | Local DVWA Lab                  |

*Severity depends on the application's server configuration and whether uploaded files can be executed or otherwise abused.

---

## 2. Description

The DVWA File Upload functionality accepts files without sufficiently restricting the allowed file types.

Although the interface states:

```text
Choose an image to upload:
```

the application successfully accepted a harmless text file:

```text
test.txt
```

and a harmless HTML file:

```text
test.html
```

The application returned successful upload messages for both files.

This demonstrates insufficient server-side file-upload validation.

---

## 3. Affected Functionality

The affected endpoint is:

```text
/DVWA/vulnerabilities/upload/
```

The upload request uses:

```text
POST
```

with:

```text
multipart/form-data
```

The application returned:

```text
HTTP/1.1 200 OK
```

for the upload request.

---

## 4. Evidence

### Evidence 1 – Successful File Upload

The application successfully uploaded:

```text
test.txt
```

and returned:

```text
../../hackable/uploads/test.txt successfully uploaded!
```

Evidence:

```text
screenshots/01-file-upload-result.png
```

![File Upload Result](screenshots/01-file-upload-result.png)

---

### Evidence 2 – Network Request

The browser Network panel captured the upload request.

Observed information:

```text
Method: POST
Endpoint: /DVWA/vulnerabilities/upload/
Status: 200 OK
Content-Type: multipart/form-data
```

Evidence:

```text
screenshots/02-upload-network.png
```

![Upload Network Evidence](screenshots/02-upload-network.png)

---

### Evidence 3 – Arbitrary File Type Accepted

The application also accepted:

```text
test.html
```

and returned:

```text
../../hackable/uploads/test.html successfully uploaded!
```

Evidence:

```text
screenshots/03-arbitrary-file-upload.png
```

![Arbitrary File Upload](screenshots/03-arbitrary-file-upload.png)

---

## 5. Reproduction Summary

The vulnerability was reproduced using the following controlled workflow:

```text
1. Open DVWA File Upload
2. Confirm security level = Low
3. Select test.txt
4. Upload the file
5. Observe successful upload
6. Inspect POST request
7. Select test.html
8. Upload the file
9. Observe successful upload
```

---

## 6. Expected Behavior

Because the application describes the functionality as an image upload, it should restrict uploads to approved image formats.

For example:

```text
photo.jpg
photo.jpeg
photo.png
```

should be accepted after proper server-side validation.

An HTML document such as:

```text
test.html
```

should be rejected.

---

## 7. Observed Behavior

The application accepted:

```text
test.txt
```

and:

```text
test.html
```

Therefore, the application's upload validation was insufficient at the configured security level.

---

## 8. Security Impact

The immediate demonstrated impact is:

```text
Unintended File Type Accepted
```

Depending on server configuration, unrestricted file upload can potentially lead to more serious consequences.

Potential impacts include:

* Unauthorized file storage.
* Malicious content hosting.
* Stored client-side attacks.
* Web application defacement.
* Malware distribution.
* Exposure of sensitive resources.
* Server-side code execution if executable files are accepted and executed.
* Further compromise of the application or host.

The lab did **not** perform server-side code execution.

---

## 9. Root Cause

The primary root cause is inadequate server-side validation of uploaded files.

Possible weaknesses include:

* Insufficient extension validation.
* Lack of MIME-type validation.
* Lack of file-signature validation.
* Trusting user-controlled filenames.
* Insecure upload directory configuration.
* Missing restrictions on executable content.

A secure implementation should use multiple validation layers rather than relying on a single filename check.

---

## 10. CWE Classification

### CWE-434 – Unrestricted Upload of File with Dangerous Type

The vulnerability is consistent with CWE-434 when an application allows users to upload files without adequately restricting dangerous or unintended file types.

Reference classification:

```text
CWE-434
Unrestricted Upload of File with Dangerous Type
```

---

## 11. Risk Assessment

### Likelihood

**Medium to High**

The upload functionality accepts unintended file types, making abuse possible if an attacker can reach the upload functionality.

### Impact

**Potentially High**

The ultimate impact depends on whether uploaded files can be executed, interpreted, publicly accessed, or used to attack other users.

### Overall Risk

```text
High*
```

*The actual production severity must be determined based on the application's architecture and server configuration.

---

## 12. Security Controls Missing or Insufficient

The following controls were insufficient or absent in the vulnerable configuration:

* Strict file-extension allowlisting.
* Strong server-side MIME validation.
* File-content/signature validation.
* Safe filename generation.
* Secure upload storage.
* Execution restrictions.
* File-size restrictions.

---

## 13. Recommended Security Controls

The application should:

1. Allow only explicitly approved file types.
2. Validate file type on the server.
3. Validate MIME type.
4. Validate the file's actual content/signature.
5. Generate a random server-side filename.
6. Prevent user-controlled filenames from determining storage paths.
7. Store uploaded files outside the web root where possible.
8. Disable script execution in upload directories.
9. Apply restrictive filesystem permissions.
10. Enforce maximum file sizes.
11. Log upload activity.
12. Monitor suspicious upload behavior.

---

## 14. Detection Opportunities

Security monitoring can look for:

* Unexpected file extensions.
* Multiple failed upload attempts.
* Uploads of executable or script files.
* Unusual upload volume.
* Requests accessing newly uploaded files.
* Suspicious child processes originating from web-server processes.
* Unexpected changes inside upload directories.

These events can be forwarded to a SIEM for correlation and investigation.

---

## 15. Evidence-to-Finding Mapping

| Evidence                       | Observation                       | Security Significance                           |
| ------------------------------ | --------------------------------- | ----------------------------------------------- |
| `01-file-upload-result.png`    | `test.txt` accepted               | Demonstrates unintended file type acceptance    |
| `02-upload-network.png`        | POST upload request with HTTP 200 | Confirms server-side upload transaction         |
| `03-arbitrary-file-upload.png` | `test.html` accepted              | Demonstrates insufficient file-type restriction |

---

## 16. Validation Result

```text
Test: test.txt
Result: ACCEPTED
```

```text
Test: test.html
Result: ACCEPTED
```

Final assessment:

```text
File-Type Restriction: INSUFFICIENT
Finding: CONFIRMED
```

---

## 17. Limitations

This assessment was performed against a deliberately vulnerable DVWA instance configured at Low security.

The assessment does not claim that arbitrary file upload would automatically result in remote code execution on a production server.

Actual impact depends on:

* Web server configuration.
* Upload directory permissions.
* Script execution settings.
* Application architecture.
* File-access controls.
* Network exposure.

---

## 18. Conclusion

The DVWA File Upload vulnerability was successfully confirmed.

The application accepted unintended file types, including `test.txt` and `test.html`, despite presenting an image-upload interface.

The finding demonstrates insufficient file-upload validation and highlights the importance of server-side allowlisting, MIME/content validation, safe storage, filename randomization, and execution restrictions.

No executable payload or server-side web shell was used during testing.

---

## 19. Ethical Testing Statement

Testing was performed exclusively against a deliberately vulnerable DVWA instance running in a controlled local environment.

No unauthorized or third-party systems were targeted.

The evidence and findings are intended for cybersecurity education, authorized security testing, and portfolio documentation.
