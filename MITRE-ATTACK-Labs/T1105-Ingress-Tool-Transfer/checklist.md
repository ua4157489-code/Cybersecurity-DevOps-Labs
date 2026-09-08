# MITRE ATT&CK T1105 — Lab Checklist

## Environment Setup

* [x] Ubuntu Linux host available
* [x] Docker installed and operational
* [x] Docker Compose configuration created
* [x] T1105 project directory created
* [x] Screenshot directory created
* [x] Controlled test file created

---

## Docker Deployment

* [x] Transfer server container created
* [x] Transfer client container created
* [x] Docker Compose deployment completed
* [x] Both containers verified as running
* [x] Nginx server operational
* [x] Curl client operational

---

## Network Configuration

* [x] Docker bridge network created
* [x] Internal network isolation enabled
* [x] Server connected to T1105 network
* [x] Client connected to T1105 network
* [x] Server IP identified as `172.21.0.3`
* [x] Client IP identified as `172.21.0.2`
* [x] Subnet identified as `172.21.0.0/16`

---

## File Transfer

* [x] Server-side test file verified
* [x] HTTP connectivity tested
* [x] HTTP `HEAD` request successful
* [x] HTTP `GET` request successful
* [x] File transferred using `curl`
* [x] 127-byte transfer confirmed
* [x] Destination file created
* [x] Destination file content verified

---

## Evidence Collection

* [x] Docker network evidence collected
* [x] HTTP transfer evidence collected
* [x] File creation evidence collected
* [x] SHA-256 integrity evidence collected
* [x] Evidence text files stored in `screenshots/`
* [x] PNG screenshots captured
* [x] Screenshot filenames standardized
* [x] README screenshot paths verified

---

## Integrity Verification

* [x] Server-side SHA-256 calculated
* [x] Client-side SHA-256 calculated
* [x] Server/client hashes compared
* [x] SHA-256 hashes matched
* [x] File integrity confirmed

---

## MITRE ATT&CK Mapping

* [x] T1105 identified
* [x] Ingress Tool Transfer behavior demonstrated
* [x] Linux transfer utility used
* [x] Network behavior documented
* [x] File creation behavior documented
* [x] Detection opportunities identified
* [x] Defensive considerations documented

---

## Documentation

* [x] `README.md` completed
* [x] `commands.sh` completed
* [x] `notes.md` completed
* [x] `checklist.md` completed
* [x] `security_report.md` completed
* [x] Project structure documented
* [x] Evidence explanations included
* [x] Safety notice included

---

## Final Status

**Lab Status: COMPLETE ✅**

**Technique:** T1105 — Ingress Tool Transfer

**Environment:** Isolated Docker Laboratory

**Transfer Method:** HTTP + curl

**Integrity:** SHA-256 Verified

**Evidence:** Collected and documented
