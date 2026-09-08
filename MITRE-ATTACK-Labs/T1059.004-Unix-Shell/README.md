# MITRE ATT&CK T1059.004 — Unix Shell

## 📌 Lab Overview

This lab demonstrates the practical simulation, monitoring, detection, and investigation of **MITRE ATT&CK T1059.004 — Unix Shell** on a Linux system.

The lab specifically focuses on **Bash execution** and uses **Linux Auditd** to generate and investigate endpoint telemetry associated with Bash process execution.

The laboratory follows a complete defensive security workflow:

```text
Environment Verification
        ↓
Controlled Bash Execution
        ↓
Auditd Verification
        ↓
Auditd Troubleshooting
        ↓
Detection Rule Configuration
        ↓
Bash Execution Test
        ↓
Auditd Event Collection
        ↓
Event Investigation
        ↓
MITRE ATT&CK Mapping
        ↓
SOC Detection Analysis
```

---

# 🎯 Objectives

The objectives of this lab are to:

* Understand MITRE ATT&CK T1059.004 — Unix Shell.
* Understand how Bash can be used for command execution.
* Perform a controlled Bash execution.
* Verify the Bash executable and Linux environment.
* Configure Linux Auditd for Bash execution monitoring.
* Troubleshoot an Auditd service failure.
* Create a custom Auditd detection rule.
* Monitor the `execve` system call.
* Generate controlled Bash execution telemetry.
* Search Auditd events using `ausearch`.
* Analyze `EXECVE` and `SYSCALL` records.
* Map observed activity to MITRE ATT&CK T1059.004.
* Understand SOC detection and investigation requirements.
* Document security evidence professionally.

---

# 🧰 Prerequisites

## Knowledge

Recommended knowledge:

* Linux fundamentals
* Bash/Linux shell basics
* Linux process execution
* Basic system administration
* Basic cybersecurity concepts
* Basic SOC concepts
* MITRE ATT&CK fundamentals

## Tools

The following tools were used:

* Bash
* Auditd
* `auditctl`
* `ausearch`
* `augenrules`
* `systemctl`
* `sudo`

---

# 🖥️ Lab Environment

| Component           | Details                |
| ------------------- | ---------------------- |
| Operating System    | Ubuntu 24.04.4 LTS     |
| Architecture        | x86_64                 |
| Kernel              | Linux 7.0.0-31-generic |
| Interactive Shell   | zsh                    |
| Test Shell          | Bash                   |
| Bash Path           | `/usr/bin/bash`        |
| Monitoring          | Linux Auditd           |
| Event Search        | `ausearch`             |
| Detection Mechanism | Auditd rule            |
| MITRE Technique     | T1059.004              |
| Technique           | Unix Shell             |

---

# 📁 Folder Structure

```text
T1059.004-Unix-Shell/
│
├── README.md
├── commands.md
├── notes.md
├── checklist.md
├── security_report.md
│
└── screenshots/
    ├── 01-t1059-004-bash-execution.png
    ├── 02-t1059-004-audit-rule.png
    ├── 03-t1059-004-detection-evidence.png
    └── 04-auditd-status.png
```

---

# 🧩 MITRE ATT&CK Mapping

| Category         | Mapping                                   |
| ---------------- | ----------------------------------------- |
| Tactic           | Execution                                 |
| Parent Technique | T1059 — Command and Scripting Interpreter |
| Sub-Technique    | T1059.004                                 |
| Name             | Unix Shell                                |
| Shell Used       | Bash                                      |
| Telemetry Source | Linux Auditd                              |
| System Call      | `execve`                                  |

---

# 🔐 Cybersecurity Relevance

Unix shells are legitimate and essential components of Linux systems. System administrators, developers, automation systems, and security tools regularly use shells.

However, attackers can also use Unix shells after gaining access to a Linux system.

A compromised shell may be used to:

* Execute commands
* Enumerate the operating system
* Discover users
* Inspect processes
* Access files
* Modify system configuration
* Download tools
* Execute scripts
* Perform privilege escalation
* Establish persistence
* Perform post-exploitation activities

Because shell execution is common in legitimate environments, SOC analysts should not classify every Bash execution as malicious.

Instead, shell execution should be correlated with additional security telemetry.

---

# 🔎 Task 1 — Verify the Linux Environment

The first step was to verify the Linux environment and confirm the Bash executable.

Commands:

```bash
id
hostname
uname -a
command -v bash
bash --version
```

The Bash executable was confirmed as:

```text
/usr/bin/bash
```

The system architecture was confirmed as:

```text
x86_64
```

The interactive shell was `zsh`, so Bash was explicitly launched during the controlled test.

---

# 💻 Task 2 — Controlled Bash Execution

A controlled Bash process was executed using:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

The command produced output similar to:

```text
MITRE T1059.004 test execution
umer
Linux 7.0.0-31-generic x86_64
```

The `bash -c` syntax explicitly starts Bash and executes the supplied command string.

This was used to create a controlled example of Unix Shell execution for the laboratory.

---

## 📸 Evidence 01 — Controlled Bash Execution

![T1059.004 Bash Execution](screenshots/01-t1059-004-bash-execution.png)

**Screenshot:** `01-t1059-004-bash-execution.png`

### Evidence Explanation

This screenshot documents the initial controlled Bash execution.

The evidence demonstrates that:

* Bash was explicitly launched.
* The Bash test command executed successfully.
* The current user was identified.
* Linux kernel information was displayed.
* The architecture was confirmed.
* The execution occurred within the laboratory environment.

This is the **activity-generation stage** of the detection workflow.

---

# 🛡️ Task 3 — Verify Auditd

After demonstrating Bash execution, Linux Auditd was checked because it would be used as the telemetry source.

The Auditd service was checked with:

```bash
systemctl is-active auditd
```

The audit framework was also inspected:

```bash
sudo auditctl -s
```

Auditd is responsible for collecting security-relevant Linux audit events.

For this laboratory, the goal was to monitor Bash execution through the `execve` system call.

---

# 🔧 Task 4 — Troubleshoot Auditd

During the laboratory, Auditd initially failed to start correctly.

A foreground diagnostic was performed:

```bash
sudo timeout 5s auditd -f
```

The diagnostic reported:

```text
Could not open dir /var/log/audit (No such file or directory)
The audit daemon is exiting.
```

The problem was caused by the missing Auditd log directory.

The required directory was created:

```bash
sudo mkdir -p /var/log/audit
```

Ownership was configured:

```bash
sudo chown root:adm /var/log/audit
```

Permissions were configured:

```bash
sudo chmod 0750 /var/log/audit
```

Auditd was then restarted:

```bash
sudo systemctl restart auditd
```

The service was verified again:

```bash
systemctl is-active auditd
```

Expected result:

```text
active
```

The Auditd status was also checked:

```bash
sudo auditctl -s
```

---

# 📸 Evidence 02 — Auditd Service Status

![Auditd Service Status](screenshots/04-auditd-status.png)

**Actual Screenshot File:** `04-auditd-status.png`

### Evidence Explanation

This screenshot documents the successful recovery and verification of Auditd.

The evidence confirms that:

* Auditd is operational.
* The audit subsystem is enabled.
* The monitoring framework is available.
* Auditd can be used for subsequent telemetry collection.

This verification is important because a detection rule is ineffective if the underlying telemetry service is not running correctly.

> **Sequence Note:** This is **Evidence 02 in the documentation workflow**, although the actual filename is `04-auditd-status.png`. The filename is kept unchanged because it is the real screenshot file present in the lab.

---

# 📜 Task 5 — Configure the T1059.004 Auditd Detection Rule

A dedicated Auditd rule was created for Bash execution.

The rule file was:

```text
/etc/audit/rules.d/mitre-t1059-004.rules
```

The rule:

```text
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/bash -F auid>=1000 -F auid!=4294967295 -k mitre_t1059_004
```

The rule components are:

| Option                  | Purpose                                    |
| ----------------------- | ------------------------------------------ |
| `-a always,exit`        | Creates an audit rule for the exit path    |
| `-F arch=b64`           | Targets 64-bit system calls                |
| `-S execve`             | Monitors program execution                 |
| `-F path=/usr/bin/bash` | Focuses on Bash execution                  |
| `-F auid>=1000`         | Focuses on regular user authentication IDs |
| `-F auid!=4294967295`   | Excludes unset authentication IDs          |
| `-k mitre_t1059_004`    | Assigns a searchable audit key             |

The rule was loaded using:

```bash
sudo augenrules --load
```

The active rule was verified using:

```bash
sudo auditctl -l | grep mitre_t1059_004
```

Auditd may normalize:

```text
auid!=4294967295
```

to:

```text
auid!=-1
```

when displaying the loaded rule. This is normal Auditd representation.

---

# 📸 Evidence 03 — Auditd Detection Rule

![T1059.004 Audit Rule](screenshots/02-t1059-004-audit-rule.png)

**Actual Screenshot File:** `02-t1059-004-audit-rule.png`

### Evidence Explanation

This screenshot documents the active Auditd rule used for the T1059.004 detection.

The rule specifically monitors:

```text
/usr/bin/bash
```

through:

```text
execve
```

and assigns the custom key:

```text
mitre_t1059_004
```

The key allows analysts to efficiently search for events generated by this detection rule.

This confirms that the detection logic was successfully loaded into the Auditd framework.

---

# ⚡ Task 6 — Generate T1059.004 Detection Telemetry

After the Auditd rule was loaded, another controlled Bash execution was performed:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

The purpose of this execution was to generate an event matching the newly configured Auditd rule.

The custom key was then used to search the resulting events:

```bash
sudo ausearch -k mitre_t1059_004 -i
```

The `-i` option provides human-readable representations of Auditd fields where possible.

---

# 🔎 Task 7 — Investigate the Auditd Event

The relevant events were filtered using:

```bash
sudo ausearch -k mitre_t1059_004 -i \
| grep -E 'type=(EXECVE|SYSCALL)' \
| grep -E 'bash|execve|mitre_t1059_004'
```

The investigation identified the following important event types:

```text
type=EXECVE
type=SYSCALL
```

Important fields included:

```text
comm=bash
exe=/usr/bin/bash
success=yes
key=mitre_t1059_004
```

These fields provide the primary endpoint evidence for the controlled Bash execution.

---

# 📸 Evidence 04 — T1059.004 Detection Evidence

![T1059.004 Detection Evidence](screenshots/03-t1059-004-detection-evidence.png)

**Actual Screenshot File:** `03-t1059-004-detection-evidence.png`

### Evidence Explanation

This screenshot provides the primary Auditd detection evidence.

The event confirms that the controlled Bash execution was captured by the Auditd rule.

### `EXECVE`

The `EXECVE` record provides information about the executed command and its arguments.

Relevant information can include:

* Argument count
* Executable name
* Command arguments

### `SYSCALL`

The `SYSCALL` record provides information about the underlying system call and execution context.

The investigation identified:

```text
syscall=execve
success=yes
comm=bash
exe=/usr/bin/bash
```

### Audit Key

The event also contains:

```text
key=mitre_t1059_004
```

This confirms that the event was generated by the custom detection rule.

---

# 🔄 Complete Detection Workflow

The complete laboratory workflow can be summarized as:

```text
┌──────────────────────────────┐
│ 1. Verify Linux Environment  │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 2. Execute Bash              │
│    bash -c                    │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 3. Verify Auditd             │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 4. Troubleshoot Auditd       │
│    /var/log/audit            │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 5. Configure Auditd Rule     │
│    T1059.004                 │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 6. Generate Bash Telemetry   │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 7. Search with ausearch      │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 8. Analyze EXECVE/SYSCALL    │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ 9. SOC Detection Analysis    │
└──────────────────────────────┘
```

---

# 🕵️ Investigation Findings

The investigation established the following:

| Investigation Item | Result            |
| ------------------ | ----------------- |
| Bash executable    | `/usr/bin/bash`   |
| Execution method   | `bash -c`         |
| System call        | `execve`          |
| Auditd             | Active            |
| Audit rule         | Loaded            |
| Audit key          | `mitre_t1059_004` |
| Execution status   | Successful        |
| `EXECVE` event     | Detected          |
| `SYSCALL` event    | Detected          |
| MITRE mapping      | T1059.004         |

---

# 🧠 Detection Analysis

The detection rule was designed to identify execution of `/usr/bin/bash` through the Linux `execve` system call.

The relationship is:

```text
/usr/bin/bash
      ↓
execve()
      ↓
Auditd Rule
      ↓
mitre_t1059_004
      ↓
EXECVE + SYSCALL
      ↓
SOC Investigation
```

The presence of Bash execution does **not** automatically indicate malicious activity.

A SOC analyst should investigate the surrounding context.

Important contextual fields include:

* User account
* Authentication ID
* Process ID
* Parent process
* Process tree
* Command-line arguments
* Terminal
* Session ID
* Source address
* Authentication activity
* Privilege level
* Network activity
* File activity

---

# 🚨 Potential SOC Detection Scenarios

## Scenario 1 — SSH Login Followed by Bash

```text
SSH Login
   ↓
Bash Execution
   ↓
Suspicious Commands
```

Unexpected Bash activity following suspicious remote access may require investigation.

---

## Scenario 2 — Privilege Escalation Followed by Bash

```text
Normal User
     ↓
Privilege Escalation
     ↓
Privileged Bash
     ↓
System Changes
```

This sequence may indicate possible post-exploitation activity.

---

## Scenario 3 — Web Application Process Spawning Bash

```text
Web Server
    ↓
Bash
    ↓
Command Execution
```

A web-facing service unexpectedly spawning a shell can be a strong security signal.

---

## Scenario 4 — Bash Followed by Network Activity

```text
Bash Execution
      ↓
Network Connection
      ↓
External Destination
```

Correlation with network telemetry can help determine whether shell activity is suspicious.

---

# ⚠️ False Positive Considerations

Bash execution is common on Linux systems.

Legitimate examples include:

* System administration
* Software installation
* Troubleshooting
* Automation
* CI/CD pipelines
* Configuration management
* Scheduled tasks
* Maintenance activities

Therefore, production detections should use contextual correlation instead of treating every Bash execution as malicious.

---

# 🛡️ Defensive Recommendations

## 1. Monitor Unix Shell Execution

Monitor relevant shells such as:

* Bash
* sh
* zsh
* Other shells used within the environment

## 2. Centralize Security Telemetry

Forward Linux endpoint telemetry to a centralized SIEM such as:

* Wazuh
* Elastic Stack
* Splunk
* Microsoft Sentinel

## 3. Monitor Privileged Shell Activity

Pay particular attention to shell execution involving:

```text
root
sudo
su
```

## 4. Correlate Authentication Activity

Correlate shell execution with:

* SSH logins
* Failed authentication attempts
* Remote administration
* VPN activity
* User sessions

## 5. Apply Least Privilege

Users should only have the permissions required for their responsibilities.

## 6. Protect Auditd Configuration

Restrict unauthorized modification of:

```text
/etc/audit/
/etc/audit/rules.d/
```

## 7. Monitor Auditd Health

Security teams should monitor:

* Auditd service availability
* Audit event loss
* Audit queue status
* Log availability
* Rule configuration
* Unauthorized rule changes

---

# 🔧 Troubleshooting Lesson

A major practical lesson from this lab was the importance of validating the telemetry infrastructure before relying on a detection rule.

Auditd initially failed because:

```text
/var/log/audit
```

was missing.

The problem was diagnosed with:

```bash
sudo timeout 5s auditd -f
```

The required directory was created and configured:

```bash
sudo mkdir -p /var/log/audit
sudo chown root:adm /var/log/audit
sudo chmod 0750 /var/log/audit
```

Auditd was then restarted successfully.

### Key Lesson

A security detection mechanism is only useful when its underlying telemetry pipeline is operational.

---

# 📚 Key Concepts Learned

## MITRE ATT&CK

A knowledge base used to classify and understand adversary behavior.

## T1059.004 — Unix Shell

A sub-technique of Command and Scripting Interpreter representing Unix shell command execution.

## Bash

A commonly used Unix/Linux shell and scripting environment.

## Auditd

The Linux auditing framework used to collect security-relevant system activity.

## `execve`

A Linux system call used to execute a program.

## `EXECVE`

An Auditd event containing information related to program execution and arguments.

## `SYSCALL`

An Auditd event containing information about the system call and execution context.

## `ausearch`

A command-line utility used to search Linux Auditd records.

## Audit Key

A custom key used to identify related Auditd events.

In this lab:

```text
mitre_t1059_004
```

was used as the detection key.

---

# 🌍 Real-World Applications

The techniques demonstrated in this laboratory can be applied to:

* SOC operations
* Linux endpoint monitoring
* Threat hunting
* Incident response
* SIEM detection engineering
* Privilege escalation detection
* SSH investigation
* Post-exploitation detection
* Malware investigation
* MITRE ATT&CK-based detection
* Endpoint security monitoring
* Security telemetry analysis

---

# 🧪 Skills Demonstrated

This lab demonstrates practical skills in:

* Linux administration
* Bash
* Auditd
* Linux security monitoring
* System-call auditing
* Endpoint telemetry
* Log analysis
* Detection engineering
* MITRE ATT&CK
* SOC investigation
* Troubleshooting
* Security evidence collection
* Security documentation

---

# 📸 Evidence Summary

The laboratory contains four confirmed screenshots.

| Documentation Evidence | Actual File                           | Purpose                      |
| ---------------------- | ------------------------------------- | ---------------------------- |
| Evidence 01            | `01-t1059-004-bash-execution.png`     | Controlled Bash execution    |
| Evidence 02            | `04-auditd-status.png`                | Auditd service verification  |
| Evidence 03            | `02-t1059-004-audit-rule.png`         | Active T1059.004 Auditd rule |
| Evidence 04            | `03-t1059-004-detection-evidence.png` | Bash detection telemetry     |

## Screenshot Documentation Sequence

The screenshots are intentionally documented according to the **actual laboratory workflow**:

```text
Evidence 01
Bash Execution
        ↓
Evidence 02
Auditd Status
        ↓
Evidence 03
Auditd Detection Rule
        ↓
Evidence 04
Detection Evidence
```

The actual filenames remain unchanged:

```text
01-t1059-004-bash-execution.png
04-auditd-status.png
02-t1059-004-audit-rule.png
03-t1059-004-detection-evidence.png
```

This means the README sequence is correct even though the original screenshot filenames were created in a different order.

---

# 🔐 Security Evidence Hygiene

Before publishing the repository to GitHub, review all screenshots and logs for sensitive information.

Check for:

* Passwords
* API keys
* Access tokens
* SSH private keys
* Credentials
* Personal information
* Sensitive host information
* Internal infrastructure details

Never commit files containing secrets such as:

```text
.env
.git-credentials
private keys
password files
API tokens
```

Only the evidence necessary to demonstrate the security technique should be published.

---

# 📋 Lab Completion Checklist

```text
[✓] Linux environment verified
[✓] Bash executable verified
[✓] Controlled Bash execution performed
[✓] Auditd status checked
[✓] Auditd startup issue diagnosed
[✓] /var/log/audit directory created
[✓] Auditd service restored
[✓] Auditd detection rule created
[✓] Detection rule loaded
[✓] Bash telemetry generated
[✓] Auditd events searched
[✓] EXECVE event analyzed
[✓] SYSCALL event analyzed
[✓] T1059.004 mapping completed
[✓] SOC detection scenarios documented
[✓] False-positive considerations documented
[✓] Defensive recommendations documented
[✓] Evidence screenshots documented
```

---

# 🏁 Lab Outcome

The laboratory successfully demonstrated a complete defensive detection workflow for:

> **MITRE ATT&CK T1059.004 — Unix Shell**

A controlled Bash execution was performed and monitored using Linux Auditd.

The Auditd rule successfully captured the Bash execution through the `execve` system call.

The investigation identified relevant evidence including:

```text
type=EXECVE
type=SYSCALL
comm=bash
exe=/usr/bin/bash
success=yes
key=mitre_t1059_004
```

The lab also demonstrated real-world troubleshooting by diagnosing and resolving an Auditd startup problem caused by the missing `/var/log/audit` directory.

The final workflow demonstrates practical knowledge of:

```text
Linux
  ↓
Bash
  ↓
Auditd
  ↓
System Calls
  ↓
Endpoint Telemetry
  ↓
Detection
  ↓
Investigation
  ↓
MITRE ATT&CK
  ↓
SOC Analysis
```

---

# 📁 Documentation Files

| File                 | Description                         |
| -------------------- | ----------------------------------- |
| `README.md`          | Complete laboratory documentation   |
| `commands.md`        | Commands used during the laboratory |
| `notes.md`           | Technical notes and concepts        |
| `checklist.md`       | Laboratory completion checklist     |
| `security_report.md` | Security assessment and findings    |
| `screenshots/`       | Practical evidence screenshots      |

---

# 👨‍💻 Author

**Umer Ali**

Cybersecurity / SOC Learning Portfolio

### Focus Areas

* SOC Operations
* Linux Security
* SIEM
* Threat Detection
* Incident Response
* MITRE ATT&CK
* Vulnerability Assessment
* Security Automation

---

## ⭐ Portfolio Note

This laboratory is part of a hands-on cybersecurity portfolio demonstrating practical experience with Linux security monitoring, Auditd, endpoint telemetry, detection engineering, MITRE ATT&CK mapping, SOC investigation, troubleshooting, and professional security documentation.
