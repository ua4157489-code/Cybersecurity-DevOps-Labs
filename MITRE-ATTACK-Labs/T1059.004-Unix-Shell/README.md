# MITRE ATT&CK T1059.004 — Unix Shell

## 📌 Lab Overview

This lab demonstrates the practical identification, simulation, telemetry collection, detection, investigation, and documentation of **MITRE ATT&CK Technique T1059.004 — Unix Shell**.

The exercise focuses on the use of a Unix shell, specifically **Bash**, and demonstrates how security monitoring can identify shell execution through Linux **Auditd** telemetry.

The lab follows a defensive SOC-oriented workflow:

```text
Controlled Simulation
        ↓
Bash Execution
        ↓
Auditd Telemetry
        ↓
Detection Rule
        ↓
Audit Event
        ↓
Investigation
        ↓
MITRE ATT&CK Mapping
        ↓
Security Analysis
        ↓
Documentation & Remediation
```

---

## 🎯 Objectives

The main objectives of this lab are:

* Understand MITRE ATT&CK **T1059.004 — Unix Shell**.
* Understand how attackers can use Unix shells for command execution.
* Perform a controlled Bash execution in a Linux environment.
* Verify the Bash executable and operating-system environment.
* Configure Linux Auditd to monitor Bash execution.
* Troubleshoot Auditd when the service fails to start.
* Create a custom Auditd detection rule.
* Generate controlled Bash execution telemetry.
* Search and investigate Auditd events.
* Analyze `EXECVE` and `SYSCALL` records.
* Map observed activity to MITRE ATT&CK.
* Document detection evidence.
* Understand how SOC analysts can investigate shell execution.
* Apply appropriate Linux monitoring and hardening practices.

---

# 🧰 Prerequisites

Before performing this lab, the following knowledge and resources are recommended:

### Knowledge

* Basic Linux command-line knowledge
* Basic Bash knowledge
* Understanding of Linux processes
* Basic system administration
* Basic cybersecurity concepts
* Familiarity with MITRE ATT&CK
* Basic SOC and log-analysis concepts

### System Requirements

* Linux system
* `bash`
* `auditd`
* `ausearch`
* `auditctl`
* `augenrules`
* `sudo` privileges

---

# 🖥️ Lab Environment

| Component               | Configuration          |
| ----------------------- | ---------------------- |
| Operating System        | Ubuntu 24.04.4 LTS     |
| Architecture            | x86_64                 |
| Kernel                  | Linux 7.0.0-31-generic |
| Shell                   | zsh / Bash             |
| Bash Path               | `/usr/bin/bash`        |
| Monitoring              | Linux Auditd           |
| Log Analysis            | `ausearch`             |
| Detection Configuration | Auditd rules           |
| ATT&CK Technique        | T1059.004              |
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
    ├── 04-auditd-status.png
    └── 05-t1059-004-audit-log.png
```

---

# 🧩 MITRE ATT&CK Mapping

## Technique

**T1059 — Command and Scripting Interpreter**

### Sub-Technique

**T1059.004 — Unix Shell**

### Description

Unix Shell is a command and scripting interpreter technique in which adversaries use Unix shells to execute commands and scripts.

Examples of Unix shells include:

* Bash
* sh
* zsh
* ksh
* csh

In this lab, **Bash** is used to demonstrate the technique in a controlled environment.

---

# 🔐 Cybersecurity Relevance

Unix shells are legitimate administrative tools, but they can also be abused after an attacker gains access to a Linux system.

An attacker may use a shell to:

* Execute commands
* Discover system information
* Enumerate users
* Inspect running processes
* Modify files
* Download tools
* Establish persistence
* Perform privilege escalation activities
* Execute scripts
* Move through a compromised environment

Because shell execution is common in legitimate administration, detection requires good telemetry and contextual analysis.

This makes **T1059.004** particularly relevant to Linux endpoint monitoring and SOC operations.

---

# 🔬 Task 1 — Verify the Linux Environment

The first step is to verify the operating-system environment and the available Bash executable.

Useful commands:

```bash
id
hostname
uname -a
command -v bash
bash --version
```

The lab environment confirmed:

```text
Bash: 5.2.21(1)-release
Bash Path: /usr/bin/bash
Architecture: x86_64
```

The environment verification ensures that the subsequent simulation is performed against the expected Linux shell.

---

# 💻 Task 2 — Controlled Bash Execution

A controlled Bash execution was performed using:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

The command produced evidence similar to:

```text
MITRE T1059.004 test execution
umer
Linux 7.0.0-31-generic x86_64
```

The use of:

```bash
bash -c
```

ensures that an explicit Bash process is launched instead of relying on the current interactive shell.

This is important because the interactive environment may use another shell such as `zsh`.

---

# 📸 Evidence 01 — Bash Execution

### Screenshot

![T1059.004 Bash Execution](screenshots/01-t1059-004-bash-execution.png)

**File:** `screenshots/01-t1059-004-bash-execution.png`

### Explanation

This screenshot demonstrates the controlled execution of Bash associated with **MITRE ATT&CK T1059.004 — Unix Shell**.

The screenshot provides evidence that:

* Bash was successfully executed.
* The Bash environment was available.
* The command executed successfully.
* The current Linux user was identified.
* The operating-system kernel and architecture were verified.
* The activity occurred inside the dedicated laboratory environment.

This represents the **simulation stage** of the detection workflow.

---

# 🛡️ Task 3 — Configure Auditd Telemetry

Linux Auditd provides security auditing capabilities that can record system-level activities.

The Auditd service was checked using:

```bash
systemctl is-active auditd
sudo auditctl -s
```

Initially, Auditd was not functioning correctly.

The diagnostic investigation showed:

```text
Could not open dir /var/log/audit
The audit daemon is exiting.
```

This indicated that the required Auditd log directory was missing.

---

# 🔧 Task 4 — Troubleshoot and Repair Auditd

The missing directory was created:

```bash
sudo mkdir -p /var/log/audit
```

Ownership and permissions were configured:

```bash
sudo chown root:adm /var/log/audit
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

The audit framework was also verified:

```bash
sudo auditctl -s
```

The resulting configuration confirmed that Auditd was enabled and operational.

---

# 📸 Evidence 02 — Auditd Service Status

### Screenshot

![Auditd Service Status](screenshots/04-auditd-status.png)

**File:** `screenshots/04-auditd-status.png`

### Explanation

This screenshot verifies that the Linux Auditd service is operating correctly.

The evidence demonstrates:

* Auditd is active.
* The Linux audit framework is enabled.
* Auditd has an active process.
* The audit subsystem is accepting events.
* No audit events were lost during the verification.

This evidence is particularly important because detection rules cannot provide reliable telemetry if the underlying auditing service is not operational.

---

# 📜 Task 5 — Create the T1059.004 Audit Rule

After restoring Auditd functionality, a dedicated rule was created to monitor execution of `/usr/bin/bash`.

The rule was configured with:

```text
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/bash -F auid>=1000 -F auid!=4294967295 -k mitre_t1059_004
```

The rule monitors:

* 64-bit system calls
* `execve`
* `/usr/bin/bash`
* User sessions with authenticated user IDs
* A custom detection key

The rule was stored in:

```text
/etc/audit/rules.d/mitre-t1059-004.rules
```

The configuration was loaded using:

```bash
sudo augenrules --load
```

The active rule was verified using:

```bash
sudo auditctl -l | grep mitre_t1059_004
```

---

# 📸 Evidence 03 — Auditd Detection Rule

### Screenshot

![T1059.004 Audit Rule](screenshots/02-t1059-004-audit-rule.png)

**File:** `screenshots/02-t1059-004-audit-rule.png`

### Explanation

This screenshot shows the active Auditd rule created specifically for the MITRE ATT&CK T1059.004 lab.

The important components are:

| Rule Component        | Purpose                              |
| --------------------- | ------------------------------------ |
| `arch=b64`            | Monitors 64-bit system calls         |
| `-S execve`           | Monitors process execution           |
| `path=/usr/bin/bash`  | Focuses on Bash execution            |
| `auid>=1000`          | Focuses on normal user sessions      |
| `auid!=-1`            | Excludes unset authentication IDs    |
| `key=mitre_t1059_004` | Provides an investigation/search key |

The custom key makes the events easy to retrieve with `ausearch`.

---

# ⚡ Task 6 — Generate Detection Telemetry

After loading the Auditd rule, another controlled Bash execution was performed:

```bash
bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'
```

This execution was designed to generate an Auditd event.

The resulting event was then searched using:

```bash
sudo ausearch -k mitre_t1059_004 -i
```

The `-i` option makes the audit output easier to interpret by converting numeric fields into human-readable values where possible.

---

# 🔎 Task 7 — Investigate Auditd Events

The relevant events were filtered using:

```bash
sudo ausearch -k mitre_t1059_004 -i \
| grep -E 'type=(EXECVE|SYSCALL)' \
| grep -E 'bash|execve|mitre_t1059_004'
```

The investigation produced important telemetry including:

```text
type=EXECVE
```

and:

```text
type=SYSCALL
```

The event also contained fields such as:

```text
comm=bash
exe=/usr/bin/bash
success=yes
key=mitre_t1059_004
```

---

# 📸 Evidence 04 — T1059.004 Detection Evidence

### Screenshot

![T1059.004 Detection Evidence](screenshots/03-t1059-004-detection-evidence.png)

**File:** `screenshots/03-t1059-004-detection-evidence.png`

### Explanation

This is the primary detection evidence for the lab.

The screenshot demonstrates that Auditd successfully detected the controlled Bash execution.

Important fields include:

### `type=EXECVE`

The `EXECVE` record provides information about the executed command and its arguments.

It can reveal:

* Executable name
* Command arguments
* Number of arguments

### `type=SYSCALL`

The `SYSCALL` record provides information about the underlying system call.

The important values include:

```text
syscall=execve
success=yes
comm=bash
exe=/usr/bin/bash
```

### `key=mitre_t1059_004`

This field connects the event to the custom Auditd detection rule.

From a SOC perspective, this provides a strong correlation point for searching and investigating T1059.004 activity.

---

# 🧾 Task 8 — Audit Log Verification

Auditd stores collected security events in its configured audit log.

The log was verified using:

```bash
sudo ls -lh /var/log/audit/audit.log
```

The T1059.004 events can be searched using:

```bash
sudo ausearch -k mitre_t1059_004 -i
```

This confirms that the detection telemetry is not only generated but also available for later investigation.

---

# 📸 Evidence 05 — Audit Log Verification

### Screenshot

![T1059.004 Audit Log](screenshots/05-t1059-004-audit-log.png)

**File:** `screenshots/05-t1059-004-audit-log.png`

### Explanation

This screenshot verifies the presence of the Auditd log and the recorded T1059.004 activity.

The evidence demonstrates that:

1. Bash execution generated an audit event.
2. The event was associated with the custom detection key.
3. Auditd retained the event in its logging system.
4. The event can be retrieved during an investigation.
5. The telemetry can support SOC-level detection and analysis.

This represents the **persistence and investigation stage** of the workflow.

---

# 🔄 Complete Detection Workflow

The complete lab workflow can be represented as:

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
│ Auditd Security Event       │
│ EXECVE + SYSCALL            │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ /var/log/audit/audit.log    │
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

The investigation confirmed the following:

| Finding          | Result            |
| ---------------- | ----------------- |
| Bash executable  | `/usr/bin/bash`   |
| Bash execution   | Successful        |
| System call      | `execve`          |
| Auditd           | Active            |
| Audit rule       | Loaded            |
| Detection key    | `mitre_t1059_004` |
| Execution event  | Detected          |
| `EXECVE` record  | Present           |
| `SYSCALL` record | Present           |
| Audit log        | Available         |
| MITRE mapping    | T1059.004         |

---

# 🧠 Detection Analysis

The detection logic is based on monitoring execution of the Bash binary.

The important telemetry relationship is:

```text
/usr/bin/bash
      +
execve
      +
authenticated user session
      +
mitre_t1059_004
      ↓
Potential T1059.004 Activity
```

A SOC analyst should not automatically treat every Bash execution as malicious because Bash is a legitimate administrative tool.

Instead, the alert should be investigated using additional context.

Useful contextual information includes:

* User account
* Parent process
* Process tree
* Command arguments
* Source of the session
* SSH activity
* Authentication events
* Network connections
* File modifications
* Privilege changes
* Timing
* Host role

---

# 🚨 Potential SOC Detection Scenarios

The same telemetry can become more valuable when correlated with other events.

Examples include:

### Scenario 1 — SSH Login + Bash Execution

```text
Successful SSH Login
        ↓
Bash Execution
        ↓
Suspicious Commands
```

This may warrant investigation depending on the user and command context.

### Scenario 2 — Privilege Escalation + Bash

```text
Normal User
    ↓
Privilege Escalation
    ↓
Root Bash
    ↓
System Changes
```

This could indicate post-exploitation activity.

### Scenario 3 — Web Server + Shell

```text
Web Server Process
       ↓
Shell Execution
       ↓
Command Execution
```

Unexpected shell execution from a web-service process can be a strong investigation signal.

### Scenario 4 — Shell + Network Activity

```text
Bash Execution
      ↓
Network Connection
      ↓
External Host
```

Correlation with network telemetry can help identify potentially malicious command execution.

---

# ⚠️ False Positive Considerations

Bash execution by itself is not necessarily malicious.

Legitimate examples include:

* System administrators
* Automation scripts
* Configuration management
* Software installation
* Maintenance tasks
* CI/CD pipelines
* Scheduled jobs
* Troubleshooting

Therefore, a production detection should include contextual conditions such as:

* User identity
* Parent process
* Host type
* Command line
* Session source
* Time of execution
* Privilege level
* Network activity

---

# 🛠️ Remediation & Hardening

Organizations should consider the following defensive controls:

### 1. Monitor Shell Execution

Maintain endpoint telemetry for:

* Bash
* sh
* zsh
* Other relevant shells

### 2. Centralize Logs

Forward Auditd events to a central SIEM such as:

* Wazuh
* Elastic Stack
* Splunk
* Microsoft Sentinel

### 3. Monitor Privileged Shells

Pay particular attention to:

```text
root
sudo
su
```

shell execution.

### 4. Monitor Remote Sessions

Correlate shell execution with:

* SSH authentication
* VPN activity
* Remote administration
* Source IP addresses

### 5. Apply Least Privilege

Users should only have the permissions necessary for their role.

### 6. Protect Audit Configuration

Restrict unauthorized modification of:

```text
/etc/audit/
```

### 7. Monitor Audit Service Health

Ensure that Auditd remains active and that audit events are not being lost.

---

# 🔧 Troubleshooting Lesson

During this lab, Auditd initially failed to start.

The diagnostic command:

```bash
sudo timeout 5s auditd -f
```

identified:

```text
Could not open dir /var/log/audit
```

The issue was resolved by creating the missing directory:

```bash
sudo mkdir -p /var/log/audit
```

and applying the appropriate ownership and permissions:

```bash
sudo chown root:adm /var/log/audit
sudo chmod 0750 /var/log/audit
```

After restarting Auditd, the service became operational.

### Lesson Learned

Security monitoring depends not only on detection rules but also on the health of the underlying telemetry infrastructure.

A properly configured detection rule is ineffective if the logging service cannot start or store events.

---

# 📚 Key Concepts Learned

## MITRE ATT&CK

A globally used knowledge base for understanding adversary tactics, techniques, and procedures.

## T1059.004

MITRE ATT&CK sub-technique representing **Unix Shell** command execution.

## Bash

A commonly used Unix/Linux command shell and scripting environment.

## Auditd

Linux auditing framework used to record security-relevant system activity.

## `execve`

A Linux system call used to execute a program.

## `EXECVE`

Audit record containing information about the command and arguments used during execution.

## `SYSCALL`

Audit record containing information about the system call and execution context.

## `ausearch`

Command-line utility used to search Linux Auditd logs.

## Detection Key

A custom identifier such as:

```text
mitre_t1059_004
```

used to efficiently search related Auditd events.

---

# 🌍 Real-World Applications

The techniques demonstrated in this lab can be applied in real SOC environments.

Examples include:

* Linux endpoint monitoring
* Threat hunting
* Incident response
* Malware investigation
* Insider-threat monitoring
* SSH attack investigation
* Privilege escalation detection
* Post-exploitation detection
* SIEM correlation
* MITRE ATT&CK-based detection engineering

---

# 🧪 Skills Demonstrated

This lab demonstrates practical skills in:

* Linux administration
* Bash
* Auditd
* Linux security monitoring
* System-call auditing
* Log analysis
* Detection engineering
* MITRE ATT&CK
* SOC investigation
* Troubleshooting
* Security documentation
* Evidence collection
* Incident-analysis methodology

---

# 📸 Evidence Summary

All screenshots are stored in the `screenshots/` directory.

| Evidence | Screenshot                            | Demonstrates              |
| -------- | ------------------------------------- | ------------------------- |
| 01       | `01-t1059-004-bash-execution.png`     | Controlled Bash execution |
| 02       | `02-t1059-004-audit-rule.png`         | T1059.004 Auditd rule     |
| 03       | `03-t1059-004-detection-evidence.png` | Detected Bash execution   |
| 04       | `04-auditd-status.png`                | Auditd service health     |
| 05       | `05-t1059-004-audit-log.png`          | Persistent audit evidence |

---

# 🔐 Security Evidence Hygiene

Before committing this lab to GitHub, screenshots and logs should be reviewed for unnecessary sensitive information.

Check for:

* Passwords
* API keys
* Access tokens
* Private credentials
* SSH private keys
* Personal information
* Unnecessary internal IP addresses
* Session identifiers
* Sensitive host information

Do not commit files such as:

```text
.git-credentials
.env
private keys
password files
API tokens
```

Only the evidence required to demonstrate the cybersecurity technique should be published.

---

# ✅ Lab Outcome

The lab successfully demonstrated a complete defensive workflow for **MITRE ATT&CK T1059.004 — Unix Shell**.

The workflow included:

```text
✔ Linux environment verification
✔ Explicit Bash execution
✔ Auditd troubleshooting
✔ Auditd service recovery
✔ Detection rule creation
✔ Audit rule loading
✔ Controlled telemetry generation
✔ Audit event investigation
✔ EXECVE analysis
✔ SYSCALL analysis
✔ Audit log verification
✔ MITRE ATT&CK mapping
✔ SOC detection analysis
✔ False-positive consideration
✔ Security hardening recommendations
✔ Evidence documentation
```

---

# 🏁 Conclusion

This lab demonstrated how Unix shell execution can be simulated and monitored in a Linux environment using Auditd.

The exercise went beyond simply executing Bash commands by implementing a complete security-monitoring workflow. Auditd was configured to monitor Bash execution through the `execve` system call, and the resulting events were investigated using `ausearch`.

The collected telemetry demonstrated how fields such as:

```text
EXECVE
SYSCALL
comm=bash
exe=/usr/bin/bash
success=yes
key=mitre_t1059_004
```

can provide valuable endpoint evidence for SOC analysts.

The troubleshooting phase also demonstrated an important operational lesson: **security detection depends on reliable telemetry collection**.

Overall, the lab provides practical experience with Linux auditing, detection engineering, MITRE ATT&CK mapping, SOC investigation, and security documentation.

---

# 📁 Documentation Files

| File                 | Purpose                                        |
| -------------------- | ---------------------------------------------- |
| `README.md`          | Complete lab documentation and visual evidence |
| `commands.md`        | Commands used throughout the lab               |
| `notes.md`           | Technical notes and concepts                   |
| `checklist.md`       | Lab completion checklist                       |
| `security_report.md` | Formal security assessment/report              |
| `screenshots/`       | Practical evidence collected during the lab    |

---

# 👨‍💻 Author

**Umer Ali**

Cybersecurity / SOC Learning Portfolio

**Focus Areas:**

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

This lab is part of a hands-on cybersecurity portfolio focused on demonstrating practical security operations, Linux security monitoring, detection engineering, and MITRE ATT&CK-based analysis.
