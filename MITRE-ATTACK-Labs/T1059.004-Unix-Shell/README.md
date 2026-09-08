# MITRE ATT&CK T1059.004 — Unix Shell

## 📌 Lab Overview

This laboratory demonstrates the practical simulation, monitoring, detection, investigation, and documentation of **MITRE ATT&CK Technique T1059.004 — Unix Shell**.

The lab uses **Bash** on an Ubuntu Linux system and **Linux Auditd** to collect endpoint telemetry related to shell execution.

A controlled Bash command is executed, an Auditd rule is configured to monitor Bash execution through the `execve` system call, and the resulting audit events are investigated using `ausearch`.

The overall workflow is:

```text
Controlled Bash Execution
        ↓
    execve()
        ↓
   Auditd Rule
        ↓
  Auditd Telemetry
        ↓
/var/log/audit/audit.log
        ↓
  ausearch Analysis
        ↓
MITRE ATT&CK Mapping
        ↓
SOC Detection & Investigation
```

---

# 🎯 Objectives

The main objectives of this lab are:

* Understand MITRE ATT&CK T1059.004 — Unix Shell.
* Understand the security relevance of Unix shell execution.
* Perform controlled Bash execution on Linux.
* Verify the Bash executable and system environment.
* Configure Linux Auditd for Bash execution monitoring.
* Troubleshoot an Auditd startup failure.
* Restore Auditd functionality.
* Create a custom Auditd detection rule.
* Monitor the `execve` system call.
* Generate controlled T1059.004 telemetry.
* Search Auditd events using `ausearch`.
* Analyze `EXECVE` and `SYSCALL` audit records.
* Identify Bash execution evidence.
* Map the observed activity to MITRE ATT&CK.
* Understand SOC investigation requirements.
* Document practical security evidence.
* Apply defensive monitoring and hardening recommendations.

---

# 🧰 Prerequisites

## Knowledge Requirements

The following knowledge is recommended:

* Basic Linux commands
* Basic Bash shell usage
* Linux process fundamentals
* Basic system administration
* Basic cybersecurity concepts
* Basic SOC concepts
* Familiarity with MITRE ATT&CK

## Required Tools

The laboratory uses:

* Bash
* Auditd
* `auditctl`
* `ausearch`
* `augenrules`
* `systemctl`
* `sudo`

---

# 🖥️ Lab Environment

| Component               | Configuration          |
| ----------------------- | ---------------------- |
| Operating System        | Ubuntu 24.04.4 LTS     |
| Architecture            | x86_64                 |
| Kernel                  | Linux 7.0.0-31-generic |
| Interactive Shell       | zsh                    |
| Test Shell              | Bash                   |
| Bash Path               | `/usr/bin/bash`        |
| Monitoring              | Linux Auditd           |
| Investigation Tool      | `ausearch`             |
| Detection Configuration | Auditd Rule            |
| MITRE Technique         | T1059.004              |
| Technique Name          | Unix Shell             |

---

# 🗂️ Folder Structure

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

## Tactic

**Execution**

## Parent Technique

**T1059 — Command and Scripting Interpreter**

## Sub-Technique

**T1059.004 — Unix Shell**

## Technique Description

MITRE ATT&CK T1059.004 represents adversary use of Unix shells to execute commands and scripts.

Common Unix shells include:

* Bash
* sh
* zsh
* ksh
* csh

This laboratory focuses specifically on **Bash execution** on Linux.

---

# 🔐 Cybersecurity Relevance

Unix shells are essential administrative tools on Linux systems. However, adversaries can also use them after obtaining access to a system.

A compromised shell may allow an attacker to:

* Execute commands
* Enumerate users
* Discover system information
* Inspect running processes
* Access files
* Modify files
* Download additional tools
* Execute scripts
* Perform privilege escalation
* Establish persistence
* Perform post-exploitation activities

Because Bash is also heavily used by legitimate administrators and automation systems, Bash execution alone should not automatically be classified as malicious.

SOC analysts should correlate shell execution with additional telemetry such as:

* User identity
* Parent process
* Process tree
* Command arguments
* SSH authentication
* Source address
* Privilege level
* Network connections
* File modifications
* Execution time
* Host role

---

# 🔎 Task 1 — Environment Verification

The first stage of the laboratory was verifying the Linux environment and Bash executable.

Commands used:

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

The system architecture was also confirmed as:

```text
x86_64
```

The environment verification ensures that the simulation is performed against the expected Linux system.

---

# 💻 Task 2 — Controlled Bash Execution

A controlled Bash process was launched using:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

The command generated output similar to:

```text
MITRE T1059.004 test execution
umer
Linux 7.0.0-31-generic x86_64
```

The use of:

```bash
bash -c
```

explicitly launches Bash for the test.

This is important because the interactive shell in the environment was `zsh`. Explicitly launching Bash provides direct evidence of the Unix Shell technique being exercised through Bash.

---

# 📸 Evidence 01 — Bash Execution

### Screenshot

![T1059.004 Bash Execution](screenshots/01-t1059-004-bash-execution.png)

**File:** `screenshots/01-t1059-004-bash-execution.png`

### Explanation

This screenshot provides visual evidence of the controlled Bash execution.

The evidence demonstrates:

* Bash was explicitly executed.
* The test command completed successfully.
* The current Linux user was identified.
* The Linux kernel was displayed.
* The system architecture was verified.
* The execution occurred inside the dedicated laboratory environment.

This represents the **simulation stage** of the T1059.004 detection workflow.

---

# 🛡️ Task 3 — Auditd Telemetry

Linux Auditd was selected as the endpoint telemetry source for this laboratory.

Auditd can record security-relevant activities, including process execution and system calls.

The service was checked using:

```bash
systemctl is-active auditd
sudo auditctl -s
```

Initially, Auditd was not functioning correctly.

A foreground diagnostic was performed:

```bash
sudo timeout 5s auditd -f
```

The diagnostic identified:

```text
Could not open dir /var/log/audit
The audit daemon is exiting.
```

This indicated that the required Auditd log directory was missing.

---

# 🔧 Task 4 — Auditd Troubleshooting and Recovery

The missing directory was created:

```bash
sudo mkdir -p /var/log/audit
```

The directory ownership was configured:

```bash
sudo chown root:adm /var/log/audit
```

The directory permissions were configured:

```bash
sudo chmod 0750 /var/log/audit
```

Auditd was then restarted:

```bash
sudo systemctl restart auditd
```

The service was verified:

```bash
systemctl is-active auditd
```

Expected result:

```text
active
```

The audit framework was also checked:

```bash
sudo auditctl -s
```

The resulting status confirmed that Auditd was enabled and operational.

---

# 📸 Evidence 02 — Auditd Service Status

### Screenshot

![Auditd Service Status](screenshots/04-auditd-status.png)

**File:** `screenshots/04-auditd-status.png`

### Explanation

This screenshot verifies that the Linux Auditd service is operational.

The evidence demonstrates:

* Auditd is active.
* The Linux audit framework is enabled.
* Auditd has an active process.
* The audit subsystem is functioning.
* The audit framework is available for telemetry collection.
* No audit events were lost during the displayed verification.

This evidence is important because detection rules depend on a functioning telemetry collection service.

The troubleshooting performed earlier in the lab demonstrated that detection infrastructure must be operational before reliable endpoint monitoring can take place.

---

# 📜 Task 5 — Create the T1059.004 Auditd Rule

A dedicated Auditd rule was created to monitor execution of the Bash executable.

The rule was stored in:

```text
/etc/audit/rules.d/mitre-t1059-004.rules
```

The configured rule was:

```text
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/bash -F auid>=1000 -F auid!=4294967295 -k mitre_t1059_004
```

The rule monitors:

| Rule Component       | Purpose                            |
| -------------------- | ---------------------------------- |
| `arch=b64`           | Monitors 64-bit system calls       |
| `-S execve`          | Monitors program execution         |
| `path=/usr/bin/bash` | Focuses on Bash execution          |
| `auid>=1000`         | Focuses on normal user sessions    |
| `auid!=4294967295`   | Excludes unset authentication IDs  |
| `-k mitre_t1059_004` | Assigns a searchable detection key |

The rules were loaded using:

```bash
sudo augenrules --load
```

The active rule was verified using:

```bash
sudo auditctl -l | grep mitre_t1059_004
```

Auditd may normalize the unset authentication ID value during rule loading and display:

```text
auid!=-1
```

instead of:

```text
auid!=4294967295
```

This is expected normalization.

---

# 📸 Evidence 03 — Auditd Detection Rule

### Screenshot

![T1059.004 Audit Rule](screenshots/02-t1059-004-audit-rule.png)

**File:** `screenshots/02-t1059-004-audit-rule.png`

### Explanation

This screenshot shows the active Auditd rule created specifically for the T1059.004 laboratory.

The rule is designed to detect execution of:

```text
/usr/bin/bash
```

through the Linux:

```text
execve
```

system call.

The custom key:

```text
mitre_t1059_004
```

provides an efficient way to search for events generated by this rule.

This confirms that the detection logic was successfully loaded into the active Auditd configuration.

---

# ⚡ Task 6 — Generate Detection Telemetry

After the detection rule was loaded, a controlled Bash execution was performed again:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

The purpose was to generate an Auditd event matching the newly configured rule.

The generated events were searched using:

```bash
sudo ausearch -k mitre_t1059_004 -i
```

The `-i` option makes the output easier to interpret by converting available numeric values into human-readable representations.

---

# 🔎 Task 7 — Investigate Auditd Events

The relevant Bash execution records were filtered using:

```bash
sudo ausearch -k mitre_t1059_004 -i \
| grep -E 'type=(EXECVE|SYSCALL)' \
| grep -E 'bash|execve|mitre_t1059_004'
```

The investigation identified records including:

```text
type=EXECVE
```

and:

```text
type=SYSCALL
```

Important fields included:

```text
comm=bash
exe=/usr/bin/bash
success=yes
key=mitre_t1059_004
```

These fields provide the endpoint evidence needed to identify the Bash execution.

---

# 📸 Evidence 04 — Detection Evidence

### Screenshot

![T1059.004 Detection Evidence](screenshots/03-t1059-004-detection-evidence.png)

**File:** `screenshots/03-t1059-004-detection-evidence.png`

### Explanation

This screenshot provides the primary detection evidence collected during the laboratory.

The Auditd output confirms that the controlled Bash execution generated security telemetry.

## `EXECVE` Record

The `EXECVE` record provides information about the executed command and its arguments.

Relevant information can include:

* Argument count
* Executable name
* Command arguments

## `SYSCALL` Record

The `SYSCALL` record provides information about the underlying system call and execution context.

Important fields observed during the investigation include:

```text
syscall=execve
success=yes
comm=bash
exe=/usr/bin/bash
```

## Detection Key

The event contains:

```text
key=mitre_t1059_004
```

This connects the event to the custom Auditd detection rule.

From a SOC perspective, this demonstrates how endpoint telemetry can be used to detect and investigate Unix shell execution.

---

# 🔄 Detection Workflow

The complete detection workflow demonstrated in this lab is:

```text
┌─────────────────────────────┐
│ Controlled Bash Execution   │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ execve System Call          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Auditd Detection Rule       │
│ key=mitre_t1059_004         │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ EXECVE + SYSCALL Records    │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Auditd Log                  │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ ausearch Investigation      │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ SOC Detection & Analysis    │
└─────────────────────────────┘
```

---

# 🕵️ Investigation Findings

The investigation established the following:

| Finding          | Result            |
| ---------------- | ----------------- |
| Bash executable  | `/usr/bin/bash`   |
| Execution method | `bash -c`         |
| System call      | `execve`          |
| Auditd           | Active            |
| Audit rule       | Loaded            |
| Detection key    | `mitre_t1059_004` |
| Execution status | Successful        |
| `EXECVE` event   | Detected          |
| `SYSCALL` event  | Detected          |
| MITRE mapping    | T1059.004         |

---

# 🧠 Detection Analysis

The detection logic focuses on Bash execution through the Linux `execve` system call.

The relationship can be represented as:

```text
/usr/bin/bash
     +
  execve
     +
Authenticated User
     +
mitre_t1059_004
     ↓
T1059.004 Detection Evidence
```

However, Bash execution alone is not enough to determine malicious activity.

A SOC analyst should investigate additional context before classifying the event.

Useful contextual information includes:

* User account
* Parent process
* Process tree
* Command-line arguments
* SSH authentication
* Source address
* Privilege level
* File activity
* Network connections
* Execution time
* Host role

---

# 🚨 Potential SOC Detection Scenarios

## Scenario 1 — SSH Login + Bash Execution

```text
Successful SSH Login
        ↓
Bash Execution
        ↓
Suspicious Commands
```

Unexpected Bash execution following suspicious remote access may warrant investigation.

---

## Scenario 2 — Privilege Escalation + Bash

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

## Scenario 3 — Web Server + Shell Execution

```text
Web Server Process
        ↓
Shell Execution
        ↓
Command Execution
```

Unexpected shell execution originating from a web-service process can be a strong investigation signal.

---

## Scenario 4 — Shell + Network Activity

```text
Bash Execution
      ↓
Network Connection
      ↓
External Host
```

Correlation with network telemetry can provide additional evidence of potentially suspicious activity.

---

# ⚠️ False Positive Considerations

Bash is a legitimate Linux administrative tool.

Potential legitimate Bash execution includes:

* System administration
* Software installation
* Configuration management
* Automation
* CI/CD pipelines
* Scheduled jobs
* System maintenance
* Troubleshooting

Therefore, production detection should use contextual correlation instead of treating every Bash execution as malicious.

---

# 🛠️ Defensive Recommendations

## 1. Monitor Shell Execution

Monitor relevant Unix shells such as:

* Bash
* sh
* zsh
* Other shells used in the environment

## 2. Centralize Audit Logs

Forward endpoint telemetry to a centralized SIEM such as:

* Wazuh
* Elastic Stack
* Splunk
* Microsoft Sentinel

## 3. Monitor Privileged Shell Activity

Pay additional attention to shell execution involving:

```text
root
sudo
su
```

## 4. Correlate Authentication Events

Correlate shell execution with:

* SSH authentication
* VPN sessions
* Remote administration
* Login events

## 5. Apply Least Privilege

Users should receive only the permissions required for their responsibilities.

## 6. Protect Audit Configuration

Restrict unauthorized modification of:

```text
/etc/audit/
```

and related Auditd configuration files.

## 7. Monitor Auditd Health

Continuously monitor:

* Auditd service status
* Audit queue
* Lost events
* Log availability
* Detection-rule configuration

---

# 🔧 Troubleshooting Lesson

One of the important practical lessons from this lab was troubleshooting Auditd.

Auditd initially failed because:

```text
/var/log/audit
```

did not exist.

The diagnostic command:

```bash
sudo timeout 5s auditd -f
```

identified the issue.

The problem was resolved using:

```bash
sudo mkdir -p /var/log/audit
sudo chown root:adm /var/log/audit
sudo chmod 0750 /var/log/audit
```

Auditd was then restarted and successfully verified.

### Key Lesson

Security detection depends on reliable telemetry.

A detection rule is ineffective if the underlying monitoring infrastructure cannot collect or retain security events.

---

# 📚 Key Concepts Learned

## MITRE ATT&CK

A knowledge base used to understand and classify adversary tactics and techniques.

## T1059.004

The MITRE ATT&CK sub-technique representing **Unix Shell** command execution.

## Bash

A widely used Unix/Linux command shell and scripting environment.

## Auditd

The Linux auditing framework used to record security-relevant system activity.

## `execve`

A Linux system call used to execute a program.

## `EXECVE`

An Auditd record containing information about command execution and arguments.

## `SYSCALL`

An Auditd record containing information about the system call and execution context.

## `ausearch`

A command-line utility used to search Auditd records.

## Audit Key

A custom identifier such as:

```text
mitre_t1059_004
```

used to locate related audit events.

---

# 🌍 Real-World Applications

The concepts demonstrated in this laboratory can be applied to:

* Linux endpoint monitoring
* SOC operations
* Threat hunting
* Incident response
* SIEM detection engineering
* Privilege escalation detection
* SSH attack investigation
* Post-exploitation detection
* Malware investigation
* Insider-threat monitoring
* MITRE ATT&CK-based detection
* Security telemetry analysis

---

# 🧪 Skills Demonstrated

This laboratory demonstrates practical experience with:

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
* Evidence collection
* Security documentation
* Defensive security practices

---

# 📸 Evidence Summary

The laboratory currently contains **four practical screenshots**.

| Evidence | Screenshot                            | Purpose                       |
| -------- | ------------------------------------- | ----------------------------- |
| 01       | `01-t1059-004-bash-execution.png`     | Controlled Bash execution     |
| 02       | `02-t1059-004-audit-rule.png`         | Auditd detection rule         |
| 03       | `03-t1059-004-detection-evidence.png` | T1059.004 detection telemetry |
| 04       | `04-auditd-status.png`                | Auditd service status         |

---

# 🔐 Security Evidence Hygiene

Before publishing this laboratory to GitHub, screenshots and logs should be reviewed for unnecessary sensitive information.

Check for:

* Passwords
* API keys
* Access tokens
* SSH private keys
* Credentials
* Personal information
* Unnecessary internal IP addresses
* Sensitive host information
* Session identifiers

Do not commit sensitive files such as:

```text
.git-credentials
.env
private keys
password files
API tokens
```

Only the evidence required to demonstrate the security technique should be published.

---

# 📋 Lab Completion Checklist

```text
[✓] Linux environment verified
[✓] Bash executable verified
[✓] Controlled Bash execution performed
[✓] Auditd troubleshooting completed
[✓] Auditd service restored
[✓] Auditd detection rule created
[✓] Audit rule loaded
[✓] T1059.004 telemetry generated
[✓] Audit events investigated
[✓] EXECVE event analyzed
[✓] SYSCALL event analyzed
[✓] MITRE ATT&CK mapping completed
[✓] SOC detection considerations documented
[✓] False-positive considerations documented
[✓] Defensive recommendations documented
[✓] Evidence screenshots documented
```

---

# 🏁 Lab Outcome

The laboratory successfully demonstrated a complete defensive workflow for **MITRE ATT&CK T1059.004 — Unix Shell**.

The exercise included:

```text
Simulation
    ↓
Telemetry Collection
    ↓
Detection Rule
    ↓
Bash Execution Detection
    ↓
Audit Event Investigation
    ↓
MITRE ATT&CK Mapping
    ↓
SOC Analysis
    ↓
Security Documentation
```

Bash execution was performed in a controlled Linux environment and monitored using Auditd.

The resulting telemetry included important fields such as:

```text
EXECVE
SYSCALL
comm=bash
exe=/usr/bin/bash
success=yes
key=mitre_t1059_004
```

The laboratory also demonstrated practical troubleshooting when Auditd initially failed because its log directory was missing.

Overall, this lab provides hands-on experience with Linux security monitoring, endpoint telemetry, Auditd, detection engineering, MITRE ATT&CK mapping, SOC investigation, and security documentation.

---

# 📁 Documentation Files

| File                 | Description                                    |
| -------------------- | ---------------------------------------------- |
| `README.md`          | Complete lab documentation and visual evidence |
| `commands.md`        | Commands used during the laboratory            |
| `notes.md`           | Technical notes and key concepts               |
| `checklist.md`       | Lab completion checklist                       |
| `security_report.md` | Formal security assessment                     |
| `screenshots/`       | Practical laboratory screenshots               |

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

This laboratory is part of a hands-on cybersecurity portfolio demonstrating practical skills in Linux security monitoring, endpoint telemetry, detection engineering, SOC investigation, MITRE ATT&CK mapping, and security documentation.
