# T1059.004 – Unix Shell Notes

## Technique

**MITRE ATT&CK:** T1059.004
**Name:** Unix Shell
**Parent Technique:** T1059 – Command and Scripting Interpreter

---

## Core Concept

Unix shells provide command-line interfaces through which users and processes can execute commands.

Common Unix shells include:

* Bash
* sh
* zsh
* ksh
* dash

This lab focused specifically on Bash.

---

## Why Bash Matters to Security

Bash is a legitimate administration tool, but it can also be used after an attacker gains access to a Linux system.

Examples of suspicious shell activity include:

```text
bash → reconnaissance
bash → credential access
bash → privilege escalation
bash → file discovery
bash → network activity
bash → payload execution
```

Bash execution alone does not establish malicious activity. Context and correlation are required.

---

## Telemetry

Linux Audit can capture process execution through the `execve` syscall.

The lab used the audit key:

```text
mitre_t1059_004
```

This made searching for relevant events easier:

```bash
sudo ausearch -k mitre_t1059_004 -i
```

---

## Important Event Types

### EXECVE

Provides command-line execution information.

Example fields:

```text
argc
a0
a1
a2
```

For this lab:

```text
a0=bash
a1=-c
```

---

### SYSCALL

Provides process execution context.

Important fields include:

```text
syscall=execve
success=yes
pid=
ppid=
auid=
uid=
tty=
comm=bash
exe=/usr/bin/bash
key=mitre_t1059_004
```

---

## Auditd Troubleshooting

The initial `auditd` failure was caused by:

```text
/var/log/audit
```

not existing.

The diagnostic output showed:

```text
Could not open dir /var/log/audit
The audit daemon is exiting.
```

The issue was resolved by creating the directory and restarting `auditd`.

---

## Detection Principle

A useful detection should not simply alert on every Bash process.

Instead, investigate combinations such as:

```text
Bash execution
+
Unexpected user
+
Unusual parent process
+
Suspicious command
+
Network connection
```

This reduces false positives and provides stronger SOC context.

---

## Key Commands

Check Bash:

```bash
command -v bash
```

Check audit status:

```bash
sudo auditctl -s
```

List audit rules:

```bash
sudo auditctl -l
```

Search technique events:

```bash
sudo ausearch -k mitre_t1059_004 -i
```

---

## Key Takeaway

The most important lesson from this lab is that MITRE ATT&CK mapping becomes significantly stronger when supported by actual endpoint telemetry.

The Bash execution was observed through:

```text
EXECVE
+
SYSCALL
+
/usr/bin/bash
+
execve
+
audit key
```

This provides a reproducible detection workflow rather than a theoretical technique demonstration.
