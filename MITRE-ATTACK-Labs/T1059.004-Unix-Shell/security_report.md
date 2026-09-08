# Security Report – MITRE ATT&CK T1059.004

## 1. Executive Summary

This assessment simulated Unix shell activity on a Linux endpoint and evaluated the ability to collect and investigate Bash execution telemetry.

The activity was successfully detected using Linux Audit (`auditd`) with a dedicated rule targeting execution of `/usr/bin/bash`.

The investigation produced `EXECVE` and `SYSCALL` records containing command-line, process, user, terminal, executable, and execution-status information.

The observed behavior maps to **MITRE ATT&CK T1059.004 – Unix Shell**.

---

## 2. Scope

### In Scope

* Bash execution
* Linux process execution telemetry
* `auditd`
* `auditctl`
* `ausearch`
* `execve`
* T1059.004 mapping
* Detection evidence

### Out of Scope

* Real-world exploitation
* Persistence
* Credential theft
* Privilege escalation
* Destructive activity
* Unauthorized systems

All activity was performed in a controlled lab environment.

---

## 3. Initial Condition

The Linux Audit package was installed, but the audit daemon was initially unavailable.

The service was in a failed state.

Diagnostic execution of `auditd` identified the problem:

```text
Could not open dir /var/log/audit (No such file or directory)
The audit daemon is exiting.
```

---

## 4. Remediation

The missing audit directory was created:

```bash
sudo mkdir -p /var/log/audit
sudo chown root:adm /var/log/audit
sudo chmod 0750 /var/log/audit
```

The service was then restarted:

```bash
sudo systemctl restart auditd
```

Verification showed:

```text
active
enabled 1
lost 0
```

---

## 5. Detection Configuration

The following audit rule was configured:

```text
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/bash -F auid>=1000 -F auid!=-1 -k mitre_t1059_004
```

The rule monitors execution of `/usr/bin/bash` through the `execve` syscall.

---

## 6. Simulation

The following controlled command was executed:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

The command successfully executed through Bash.

---

## 7. Detection Evidence

The audit subsystem generated:

```text
type=EXECVE
argc=3
a0=bash
a1=-c
```

The associated syscall record showed:

```text
syscall=execve
success=yes
comm=bash
exe=/usr/bin/bash
key=mitre_t1059_004
```

This establishes that Bash was executed and that the audit rule successfully captured the process execution.

---

## 8. Investigation

The event provided several useful investigation attributes.

### Process Information

* Process ID
* Parent process ID
* Process name
* Executable path

### User Information

* Audit user ID
* Real user ID
* Effective user ID

### Session Information

* TTY
* Session ID

### Command Information

* Argument count
* Bash executable
* Command-line arguments

### Execution Status

The event reported:

```text
success=yes
exit=0
```

Therefore, the controlled Bash process executed successfully.

---

## 9. MITRE ATT&CK Mapping

| Attribute     | Mapping                   |
| ------------- | ------------------------- |
| Tactic        | Execution                 |
| Technique     | T1059                     |
| Sub-technique | T1059.004                 |
| Name          | Unix Shell                |
| Interpreter   | Bash                      |
| Evidence      | `/usr/bin/bash` execution |
| Telemetry     | Linux Audit               |
| Syscall       | `execve`                  |

---

## 10. Risk Assessment

### Severity

**Context-dependent**

Bash is a normal Linux administration utility. Therefore, the presence of Bash execution by itself should not be classified as malicious.

Risk increases when Bash execution is correlated with suspicious activity.

Examples include:

* Unexpected account activity
* Unusual parent process
* Execution by service accounts
* Commands associated with reconnaissance
* Suspicious outbound network connections
* Privilege escalation
* Access to sensitive files
* Execution from unusual directories

---

## 11. Detection Recommendations

Security monitoring should correlate shell execution with:

* Authentication events
* SSH sessions
* Process ancestry
* Network connections
* File activity
* Privilege changes
* EDR telemetry
* SIEM alerts

A high-confidence detection could combine multiple signals rather than alerting on Bash execution alone.

---

## 12. Defensive Recommendations

* Maintain Linux audit logging.
* Forward audit events to centralized SIEM infrastructure.
* Monitor command interpreters.
* Monitor unusual parent-child process relationships.
* Apply least privilege.
* Restrict unnecessary shell access.
* Monitor privileged accounts.
* Monitor SSH activity.
* Review audit rules regularly.
* Protect audit logs from unauthorized modification.

---

## 13. Evidence Files

```text
screenshots/
├── 01-t1059-004-bash-execution.png
├── 02-t1059-004-audit-rule.png
├── 03-t1059-004-detection-evidence.png
├── 04-auditd-status.png
└── 05-t1059-004-audit-log.png
```

---

## 14. Conclusion

The T1059.004 lab successfully demonstrated a complete MITRE ATT&CK detection workflow.

The exercise progressed from controlled Bash execution to endpoint telemetry collection, troubleshooting, detection, investigation, and MITRE ATT&CK mapping.

The resulting telemetry demonstrated that Linux Audit can provide valuable process-execution context for SOC investigations.

**Assessment Result: Successful**
