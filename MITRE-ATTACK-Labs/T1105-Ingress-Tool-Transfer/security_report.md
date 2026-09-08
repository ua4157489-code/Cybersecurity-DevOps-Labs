# Security Report — MITRE ATT&CK T1105

## Ingress Tool Transfer

**Technique ID:** T1105
**Technique Name:** Ingress Tool Transfer
**Environment:** Isolated Docker Laboratory
**Platform:** Linux
**Status:** Successfully Demonstrated

---

# 1. Executive Summary

This security assessment demonstrates the behavior associated with **MITRE ATT&CK T1105 — Ingress Tool Transfer** in a controlled Docker environment.

A harmless test file was hosted on an internal Nginx server and transferred to a Docker client using the `curl` command-line utility.

The activity was successfully observed through:

* Docker network configuration
* Nginx HTTP access logs
* Destination file metadata
* File content
* SHA-256 hash verification

The experiment successfully reproduced the core observable behavior of an ingress file transfer without using malware or targeting an external system.

---

# 2. Scope

The assessment was limited to the following controlled components:

```text
t1105-transfer-server
t1105-transfer-client
```

The laboratory network was:

```text
t1105-ingress-tool-transfer_t1105-lab
```

Network subnet:

```text
172.21.0.0/16
```

No external or unauthorized systems were included in the test.

---

# 3. Technique Mapping

| Attribute        | Value                 |
| ---------------- | --------------------- |
| MITRE ATT&CK     | T1105                 |
| Technique        | Ingress Tool Transfer |
| Platform         | Linux                 |
| Transfer Utility | curl                  |
| Protocol         | HTTP                  |
| Server           | Nginx                 |
| Destination      | `/tmp/t1105-test.txt` |

---

# 4. Attack Simulation

The controlled activity followed this sequence:

```text
Internal Client
      ↓
HTTP Connection
      ↓
GET /t1105-test.txt
      ↓
Nginx Returns File
      ↓
127 Bytes Transferred
      ↓
File Created in /tmp
      ↓
SHA-256 Verified
```

The activity represents the core behavior of transferring a file into a target environment.

---

# 5. Evidence Analysis

## 5.1 Network Evidence

The Docker network inspection confirmed:

```text
Driver: bridge
Internal: true
Subnet: 172.21.0.0/16
Client: 172.21.0.2
Server: 172.21.0.3
```

This confirms controlled communication between the laboratory containers.

---

## 5.2 HTTP Evidence

The Nginx access log recorded:

```text
172.21.0.2 ... "GET /t1105-test.txt HTTP/1.1" 200 127
```

This demonstrates:

* Client communication
* Requested resource
* Successful response
* Data transfer
* Transfer size

The `curl/8.22.0` user agent further identifies the command-line transfer client.

---

## 5.3 File Creation Evidence

The destination file was:

```text
/tmp/t1105-test.txt
```

Metadata showed:

```text
Size: 127 bytes
Permissions: 0644
Owner: curl_user
Group: curl_group
```

The presence of the file after the HTTP transfer provides destination-side evidence.

---

## 5.4 Integrity Evidence

The original and downloaded files produced the same SHA-256 hash:

```text
f6bb28ca837cdd942dd96c576288bfa1c649afb2f643d378d29cb8bff51458a4
```

This confirms that the downloaded laboratory file matches the original.

---

# 6. Security Impact

In a real compromise, ingress tool transfer can enable an adversary to introduce additional resources into a compromised environment.

Potential impacts include:

* Introduction of unauthorized tools
* Delivery of scripts
* Retrieval of additional payloads
* Delivery of configuration files
* Staging of malicious resources
* Preparation for subsequent execution
* Supporting later attack stages

The severity of an actual T1105 event depends heavily on context, transferred content, source, destination, user, and subsequent activity.

---

# 7. Detection Opportunities

Potential detection sources include:

### Network Telemetry

Monitor:

* New outbound connections
* HTTP requests
* Unusual destinations
* Unexpected download activity
* Suspicious user agents

### Process Telemetry

Monitor:

* `curl`
* `wget`
* `scp`
* `sftp`
* Other transfer utilities

### File Telemetry

Monitor:

* New files in `/tmp`
* New executable files
* Recently downloaded files
* Unexpected file extensions
* File hashes

### Correlation

A stronger detection can correlate:

```text
Network Connection
        +
Transfer Utility Execution
        +
File Creation
        +
Subsequent Execution
```

---

# 8. Recommended Defensive Controls

Organizations should consider:

1. Centralized endpoint logging.
2. Network monitoring.
3. SIEM correlation rules.
4. EDR process monitoring.
5. File integrity monitoring.
6. Application allowlisting.
7. Network segmentation.
8. Restriction of unnecessary outbound traffic.
9. Monitoring temporary directories.
10. Malware analysis for suspicious downloaded files.

---

# 9. SOC Investigation Workflow

If a T1105 alert is generated, an analyst should investigate:

### Step 1 — Identify the Source

Determine:

* Source host
* Source IP
* User
* Process
* Parent process

### Step 2 — Identify the Destination

Determine:

* Remote IP/domain
* Destination port
* Protocol
* Requested resource

### Step 3 — Analyze the File

Determine:

* Filename
* Location
* File type
* Size
* SHA-256
* Reputation

### Step 4 — Check Follow-On Activity

Determine whether the file was:

* Executed
* Renamed
* Modified
* Moved
* Used for persistence
* Followed by additional suspicious activity

### Step 5 — Determine Legitimacy

Compare the activity with expected:

* Administrative operations
* Software deployment
* Automation
* CI/CD activity
* User behavior

---

# 10. MITRE ATT&CK Detection Logic

A useful behavioral detection concept is:

```text
IF

A process establishes network communication

AND

A file is retrieved from the remote endpoint

AND

A new file is subsequently created locally

THEN

Generate a potential T1105 detection
```

Additional context such as process identity, user, destination reputation, file type, and subsequent execution can improve detection accuracy.

---

# 11. False Positive Considerations

Legitimate administrative tools may use the same mechanisms.

Examples include:

* Software updates
* Package installation
* Configuration management
* DevOps automation
* Backup systems
* Remote administration
* File synchronization

Therefore, detection rules should use contextual information rather than treating every `curl` or file download as malicious.

---

# 12. Risk Assessment

| Category               | Assessment |
| ---------------------- | ---------- |
| Technique Demonstrated | T1105      |
| Actual Malware Used    | No         |
| External Target        | No         |
| Unauthorized Activity  | No         |
| Network Isolation      | Yes        |
| File Transfer          | Successful |
| File Creation          | Successful |
| Integrity Verification | Successful |
| Security Risk of Lab   | Low        |
| Real-World Relevance   | High       |

---

# 13. Evidence Summary

| Evidence                   | Result                    |
| -------------------------- | ------------------------- |
| Docker network             | Confirmed                 |
| Internal isolation         | Confirmed                 |
| Client/server connectivity | Confirmed                 |
| HTTP GET                   | Confirmed                 |
| File transfer              | Confirmed                 |
| File creation              | Confirmed                 |
| File metadata              | Captured                  |
| SHA-256                    | Captured                  |
| Hash match                 | Confirmed                 |
| T1105 behavior             | Successfully demonstrated |

---

# 14. Recommendations

For enterprise environments:

* Monitor command-line download utilities.
* Correlate network activity with file creation.
* Monitor suspicious temporary-directory activity.
* Maintain centralized endpoint telemetry.
* Integrate endpoint and network events into a SIEM.
* Investigate downloaded executables before execution.
* Use file hashes as investigation pivots.
* Apply least-privilege principles.
* Restrict unnecessary network access.
* Maintain strong network segmentation.

---

# 15. Conclusion

The laboratory successfully reproduced the observable behavior associated with **MITRE ATT&CK T1105 — Ingress Tool Transfer**.

A controlled file was transferred from an internal Nginx server to a Docker client using `curl`. The HTTP request was recorded, the destination file was created, metadata was captured, and SHA-256 verification confirmed file integrity.

The exercise demonstrates that effective detection of T1105 should focus on **behavioral correlation** rather than relying on a single indicator.

---

## Analyst Conclusion

**Assessment Result: SUCCESSFUL DEMONSTRATION**

The laboratory provides a reproducible example of how a SOC analyst can identify, investigate, and document ingress file-transfer behavior in a controlled Linux environment.
