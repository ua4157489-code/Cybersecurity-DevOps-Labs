# Methodology – DVWA File Upload

## 1. Purpose

This document describes the methodology used to assess the File Upload functionality in Damn Vulnerable Web Application (DVWA).

The objective was to determine whether the application properly restricted uploaded files to the intended image formats or whether unintended file types could be uploaded.

Testing was performed in a controlled local environment:

```text
http://localhost:8080/DVWA/
```

---

## 2. Scope

### Target Application

```text
Damn Vulnerable Web Application (DVWA)
```

### Target Functionality

```text
File Upload
```

### Target Endpoint

```text
/DVWA/vulnerabilities/upload/
```

### HTTP Method

```text
POST
```

### Security Level

```text
Low
```

### Testing Environment

```text
Local / Isolated Lab
```

---

## 3. Testing Objectives

The assessment had the following objectives:

1. Identify the file upload functionality.
2. Identify the upload endpoint.
3. Establish normal upload behavior.
4. Upload a harmless baseline file.
5. Test whether unintended file types were accepted.
6. Inspect the HTTP upload request.
7. Capture evidence screenshots.
8. Determine whether file-type validation was sufficient.
9. Assess potential security impact.
10. Document remediation recommendations.

---

## 4. Methodology Overview

The following workflow was followed:

```text
Reconnaissance
      ↓
Identify File Upload Functionality
      ↓
Identify Upload Endpoint
      ↓
Establish Baseline
      ↓
Upload Harmless File
      ↓
Inspect HTTP Request
      ↓
Test Unintended File Type
      ↓
Observe Server Response
      ↓
Capture Evidence
      ↓
Assess Security Impact
      ↓
Document Remediation
```

---

## 5. Step 1 – Identify File Upload Functionality

The DVWA File Upload module was accessed through:

```text
http://localhost:8080/DVWA/vulnerabilities/upload/
```

The application displayed:

```text
Choose an image to upload:
```

This established that the intended functionality was image uploading.

---

## 6. Step 2 – Identify the Upload Endpoint

The browser Network Developer Tools were used to inspect the upload request.

The observed request used:

```text
POST /DVWA/vulnerabilities/upload/
```

The request was sent to:

```text
localhost:8080
```

The upload operation used:

```text
multipart/form-data
```

which is commonly used for browser-based file uploads.

---

## 7. Step 3 – Establish Baseline Behavior

A harmless text file was selected for the initial controlled test.

The file was:

```text
test.txt
```

The application successfully processed the upload and returned:

```text
../../hackable/uploads/test.txt successfully uploaded!
```

This established that the application accepted the uploaded file.

---

## 8. Step 4 – Inspect the Network Request

Firefox Developer Tools → Network was used to inspect the HTTP request generated during the upload.

The observed request contained:

```text
Method:
POST
```

```text
Endpoint:
/DVWA/vulnerabilities/upload/
```

```text
Status:
200 OK
```

```text
Content-Type:
multipart/form-data
```

This provided network-level evidence of the upload operation.

---

## 9. Step 5 – Test File-Type Validation

A second harmless file was used to determine whether the application enforced its intended image-only restriction.

The file used was:

```text
test.html
```

The application successfully accepted the file and returned:

```text
../../hackable/uploads/test.html successfully uploaded!
```

This demonstrated that the application accepted an HTML file despite presenting an image-upload interface.

---

## 10. Step 6 – Validate the Finding

The finding was considered confirmed based on the following observations:

```text
1. The application requested an image.
2. A text file was successfully uploaded.
3. An HTML file was successfully uploaded.
4. The server returned successful upload responses.
5. The uploaded files were stored in the application upload directory.
```

Therefore:

```text
Finding:
Unrestricted / Insufficiently Restricted File Upload

Status:
Confirmed
```

---

## 11. Step 7 – Evidence Collection

Three screenshots were captured during testing.

### Evidence 1 – Successful Upload

```text
screenshots/01-file-upload-result.png
```

Purpose:

```text
Demonstrates successful file upload.
```

---

### Evidence 2 – Network Request

```text
screenshots/02-upload-network.png
```

Purpose:

```text
Documents the POST request and HTTP response.
```

---

### Evidence 3 – Arbitrary File Type

```text
screenshots/03-arbitrary-file-upload.png
```

Purpose:

```text
Demonstrates successful upload of test.html.
```

---

## 12. Step 8 – Impact Assessment

The demonstrated security issue was the acceptance of unintended file types.

Potential real-world impact depends on how uploaded files are handled by the server.

Potential risks include:

* Unauthorized file storage.
* Malicious content hosting.
* Stored client-side attacks.
* Application defacement.
* Malware distribution.
* Server-side code execution if executable files are accepted and executed.
* Additional attacks through uploaded content.

No server-side code execution was performed during this lab.

---

## 13. Step 9 – Root Cause Analysis

The likely root cause is insufficient server-side validation of uploaded files.

A secure upload mechanism should validate multiple properties:

```text
Filename
   ↓
Extension
   ↓
MIME Type
   ↓
File Signature / Content
   ↓
File Size
   ↓
Storage Location
   ↓
Execution Permissions
```

The application should not rely solely on the client-side file-selection interface.

---

## 14. Step 10 – Remediation Assessment

Recommended security controls include:

* Strict extension allowlisting.
* Server-side MIME validation.
* File signature/content validation.
* Safe server-generated filenames.
* File size restrictions.
* Secure storage locations.
* Disabled script execution.
* Restricted filesystem permissions.
* Authentication and authorization checks.
* Upload logging and monitoring.

---

## 15. Secure Testing Principles

### Controlled Testing

All testing was performed against the local DVWA environment.

### Harmless Files

Only harmless files such as:

```text
test.txt
test.html
```

were used.

### No Code Execution

No executable web shell or server-side payload was deployed.

### Evidence-Based Validation

The finding was confirmed using application responses and browser Network evidence.

### Minimal Impact

Testing was limited to demonstrating file-type validation behavior.

---

## 16. Evidence Chain

The evidence chain was:

```text
File Upload Page
       ↓
Image Upload Expected
       ↓
Upload test.txt
       ↓
Upload Successful
       ↓
Inspect POST Request
       ↓
Upload test.html
       ↓
Upload Successful
       ↓
Insufficient File-Type Validation Confirmed
```

---

## 17. Final Assessment

The DVWA File Upload functionality was successfully assessed in the controlled local environment.

The application accepted unintended file types, including:

```text
test.txt
test.html
```

The successful upload of `test.html` demonstrated that the application's file-type restrictions were insufficient at the configured Low security level.

The assessment was documented using three evidence screenshots and supporting technical documentation.

---

## 18. Ethical Testing Statement

This assessment was performed exclusively against a deliberately vulnerable DVWA instance running in a controlled local environment.

No external, third-party, or unauthorized systems were targeted.

The methodology is intended for cybersecurity education, authorized vulnerability assessment, and defensive security research.
