# MITRE ATT&CK T1105 — Lab Notes

## 1. Technique Overview

**MITRE ATT&CK Technique:** T1105 — Ingress Tool Transfer

T1105 describes the transfer of tools, files, or other resources into an environment during an intrusion.

In this laboratory, the behavior was safely reproduced using:

* Docker
* Nginx
* `curl`
* A harmless text file
* An isolated Docker bridge network

No malicious payload was used.

---

## 2. Laboratory Architecture

The laboratory contains two containers:

### Transfer Server

```text
Container: t1105-transfer-server
Image: nginx:alpine
IP: 172.21.0.3
```

The server hosts:

```text
t1105-test.txt
```

### Transfer Client

```text
Container: t1105-transfer-client
Image: curlimages/curl:latest
IP: 172.21.0.2
```

The client downloads the file to:

```text
/tmp/t1105-test.txt
```

---

## 3. Network Isolation

The containers communicate through:

```text
t1105-ingress-tool-transfer_t1105-lab
```

Network properties:

```text
Driver: bridge
Internal: true
Subnet: 172.21.0.0/16
Gateway: 172.21.0.1
```

The `Internal: true` configuration was used to keep the laboratory isolated from external network access.

---

## 4. Transfer Method

The transfer was performed with:

```bash
curl http://transfer-server/t1105-test.txt \
-o /tmp/t1105-test.txt
```

This produced an HTTP `GET` request to the Nginx server.

---

## 5. HTTP Evidence

The Nginx logs recorded:

```text
"GET /t1105-test.txt HTTP/1.1" 200 127
```

Interpretation:

| Field             | Meaning                     |
| ----------------- | --------------------------- |
| GET               | HTTP file retrieval request |
| `/t1105-test.txt` | Requested file              |
| `200`             | Successful HTTP response    |
| `127`             | Transferred response size   |
| `curl/8.22.0`     | Client user agent           |

---

## 6. Destination File

The downloaded file was created at:

```text
/tmp/t1105-test.txt
```

Observed properties:

```text
Size: 127 bytes
Permissions: 0644
Owner: curl_user
Group: curl_group
```

---

## 7. File Integrity

The SHA-256 hash of both files was:

```text
f6bb28ca837cdd942dd96c576288bfa1c649afb2f643d378d29cb8bff51458a4
```

The identical hashes demonstrate that the downloaded file matches the original test file.

---

## 8. Detection Observations

The most useful behavioral indicators demonstrated by this lab are:

1. Network connection to an internal file server.
2. HTTP `GET` request.
3. Use of a command-line transfer utility.
4. Successful file transfer.
5. Creation of a new file on the destination.
6. File metadata and hash becoming available.

A defensive detection rule could correlate network activity with subsequent file creation.

---

## 9. Possible False Positives

T1105-like behavior is not automatically malicious.

Legitimate examples include:

* Software installation
* Package management
* System administration
* Configuration retrieval
* Automated deployment
* CI/CD operations
* Backup and synchronization tasks

Detection should therefore consider:

* Source
* Destination
* User
* Process
* Command line
* File type
* File location
* Timing
* Network reputation
* Expected administrative behavior

---

## 10. Investigation Questions

When investigating possible T1105 activity, a SOC analyst can ask:

* Which process performed the transfer?
* Which user initiated it?
* What remote host was contacted?
* What file was transferred?
* Where was the file written?
* Was the file subsequently executed?
* Does the file hash match a known malicious indicator?
* Was the destination directory unusual?
* Was the network connection expected?
* Did additional suspicious activity occur afterward?

---

## 11. Key Learning

The major lesson from this laboratory is that **file transfer alone is not sufficient to determine malicious activity**.

Stronger detection comes from correlating:

```text
Process
+
Network Activity
+
File Creation
+
File Metadata
+
File Hash
+
Execution
```

This provides better context for SOC investigation and reduces false positives.

---

## 12. Lab Safety

This exercise used only a harmless text file inside an isolated Docker environment.

The laboratory was designed for authorized cybersecurity education and defensive analysis.
