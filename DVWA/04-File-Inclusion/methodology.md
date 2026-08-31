# Methodology – DVWA Local File Inclusion (LFI)

## 1. Purpose

This document describes the methodology followed to identify, validate, document, and assess the Local File Inclusion (LFI) vulnerability in Damn Vulnerable Web Application (DVWA).

Testing was performed in a controlled local lab environment against:

```text
http://localhost:8080/DVWA/
```

The purpose was to understand how user-controlled file parameters can lead to unauthorized local file inclusion and disclosure.

---

## 2. Scope

### Target Application

```text
Damn Vulnerable Web Application (DVWA)
```

### Target Endpoint

```text
/DVWA/vulnerabilities/fi/
```

### HTTP Method

```text
GET
```

### Parameter Tested

```text
page
```

### Security Level

```text
Low
```

### Environment

```text
Local / Isolated Lab
```

---

## 3. Testing Objectives

The assessment focused on the following objectives:

1. Identify the File Inclusion functionality.
2. Identify user-controlled parameters.
3. Establish normal application behavior.
4. Test whether the file parameter could reference a local file.
5. Verify the server response.
6. Capture browser/network evidence.
7. Determine the security impact.
8. Document remediation recommendations.

---

## 4. Methodology Overview

The following workflow was followed:

```text
Reconnaissance
      ↓
Identify File Inclusion Functionality
      ↓
Identify User-Controlled Parameter
      ↓
Establish Normal Behavior
      ↓
Controlled LFI Test
      ↓
Observe Server Response
      ↓
Capture Evidence
      ↓
Assess Impact
      ↓
Document Remediation
```

---

## 5. Step 1 – Identify the Target Functionality

The DVWA File Inclusion module was opened through:

```text
http://localhost:8080/DVWA/vulnerabilities/fi/
```

The page displayed a file inclusion interface and application content.

The functionality indicated that the application was loading a page based on a URL parameter.

---

## 6. Step 2 – Identify the Parameter

The normal request was observed through Firefox Developer Tools.

The request contained:

```text
page=file1.php
```

Therefore, the parameter identified for testing was:

```text
page
```

This parameter became the primary input point for the security assessment.

---

## 7. Step 3 – Establish Baseline Behavior

Before performing the security test, the normal application behavior was established.

The baseline request was:

```text
GET /DVWA/vulnerabilities/fi/?page=file1.php
```

The server returned:

```text
HTTP/1.1 200 OK
```

The expected application page was displayed.

This baseline was important because it provided a reference for comparing the result of the controlled test.

---

## 8. Step 4 – Controlled LFI Testing

A harmless local system file was selected for testing.

The `page` parameter was changed from:

```text
page=file1.php
```

to:

```text
page=/etc/hostname
```

The resulting request was:

```text
GET /DVWA/vulnerabilities/fi/?page=/etc/hostname
```

This test was performed only against the controlled local DVWA environment.

---

## 9. Step 5 – Analyze the Response

The server returned:

```text
HTTP/1.1 200 OK
```

The application processed the supplied local path.

The response behavior demonstrated that the `page` parameter could reference a local filesystem resource.

This behavior was inconsistent with secure file-selection practices.

---

## 10. Step 6 – Verify Through Network Evidence

Firefox Developer Tools → Network was used to inspect the request.

The following information was captured:

```text
Method:
GET

Host:
localhost:8080

Endpoint:
/DVWA/vulnerabilities/fi/

Parameter:
page=/etc/hostname

Status:
200 OK
```

This provided independent network-level evidence of the test.

---

## 11. Step 7 – Capture Screenshots

Two screenshots were captured as portfolio evidence.

### Evidence 1

```text
screenshots/01-lfi-result.png
```

Purpose:

```text
Document the LFI result and application behavior.
```

### Evidence 2

```text
screenshots/02-lfi-proof.png
```

Purpose:

```text
Document the HTTP request and response observed through Network Developer Tools.
```

---

## 12. Step 8 – Determine Vulnerability Status

The vulnerability was considered confirmed because:

1. The `page` parameter was user-controlled.
2. The parameter accepted a local filesystem path.
3. A local file was successfully processed.
4. The server returned a successful HTTP response.
5. The application exposed the resulting local file content.

Therefore:

```text
Finding:
Local File Inclusion (LFI)

Status:
Confirmed
```

---

## 13. Step 9 – Impact Assessment

The demonstrated impact was:

```text
Local File Disclosure
```

Potential impact in a real application may depend on filesystem permissions and application configuration.

Possible consequences include:

* Disclosure of sensitive local files.
* Exposure of application configuration.
* Information leakage.
* Exposure of credentials or secrets.
* Server environment discovery.
* Chaining with other vulnerabilities.

Only local file disclosure was demonstrated during this lab.

---

## 14. Step 10 – Remediation Assessment

The recommended security controls were identified based on the root cause.

Primary recommendations:

```text
1. Avoid dynamic file inclusion.
2. Implement strict allowlisting.
3. Validate user input server-side.
4. Prevent directory traversal.
5. Canonicalize paths where required.
6. Restrict filesystem permissions.
7. Apply least privilege.
8. Monitor suspicious file inclusion requests.
```

---

## 15. Testing Principles

The following principles were followed during the assessment:

### Controlled Testing

Testing was performed only against the local DVWA environment.

### Minimal Impact

A harmless local file was used to demonstrate the vulnerability.

### Evidence-Based Validation

The finding was confirmed using both application behavior and network-level evidence.

### Repeatability

The test can be repeated using the same endpoint and parameter.

### Documentation

Each stage was documented for use in the security portfolio.

---

## 16. Evidence Chain

The evidence chain for this finding is:

```text
Normal Request
     ↓
page=file1.php
     ↓
Identify User-Controlled Parameter
     ↓
Controlled Input
     ↓
page=/etc/hostname
     ↓
HTTP 200 OK
     ↓
Local File Content Processed
     ↓
LFI Confirmed
```

---

## 17. Final Assessment

The assessment successfully demonstrated that the DVWA File Inclusion functionality allowed user-controlled input to influence local file selection.

The vulnerability was confirmed at the configured **Low** security level.

The finding was documented with:

```text
README.md
methodology.md
commands.md
findings.md
remediation.md
```

and supported with two screenshots:

```text
01-lfi-result.png
02-lfi-proof.png
```

---

## 18. Ethical Testing Statement

This assessment was conducted exclusively against a deliberately vulnerable application running in a controlled local environment.

No external systems, third-party applications, or unauthorized infrastructure were targeted.

The techniques documented in this lab are intended for authorized security testing, education, and defensive security research.
