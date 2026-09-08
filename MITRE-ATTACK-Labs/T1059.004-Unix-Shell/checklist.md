# T1059.004 – Unix Shell Checklist

## Environment

* [x] Ubuntu/Linux environment verified
* [x] Bash installed
* [x] Bash path identified
* [x] Current user verified
* [x] Lab directory verified

## Simulation

* [x] Bash execution performed
* [x] `bash -c` command execution demonstrated
* [x] System information command executed
* [x] User context verified

## Telemetry

* [x] `auditd` package verified
* [x] Initial `auditd` failure investigated
* [x] Missing `/var/log/audit` identified
* [x] Audit log directory created
* [x] `auditd` restarted
* [x] Kernel auditing enabled
* [x] Audit loss verified as zero

## Detection

* [x] Dedicated T1059.004 audit rule created
* [x] Audit rule loaded
* [x] Audit rule verified with `auditctl`
* [x] Bash execution generated after rule activation
* [x] Audit event discovered with `ausearch`
* [x] `EXECVE` event identified
* [x] `SYSCALL` event identified
* [x] `execve` syscall confirmed
* [x] `/usr/bin/bash` confirmed
* [x] User/process context identified
* [x] Audit key confirmed

## Investigation

* [x] Timestamp identified
* [x] PID identified
* [x] PPID identified
* [x] User attribution identified
* [x] TTY identified
* [x] Command-line arguments identified
* [x] Executable path identified
* [x] Execution result identified

## MITRE Mapping

* [x] T1059 identified
* [x] T1059.004 identified
* [x] Bash execution mapped to Unix Shell
* [x] Mapping supported by endpoint telemetry

## Documentation

* [x] README created
* [x] Commands documented
* [x] Technical notes documented
* [x] Security report prepared
* [x] Checklist completed
* [x] Screenshot directory created

## Evidence

* [x] Bash execution screenshot
* [x] Audit rule screenshot
* [x] Detection evidence screenshot
* [x] Auditd status screenshot
* [x] Audit log screenshot

## Security Hygiene

* [ ] Review screenshots for usernames
* [ ] Review screenshots for IP addresses
* [ ] Review screenshots for hostnames
* [ ] Ensure credentials/secrets are not included
* [ ] Ensure `.git-credentials` or other secret files are not committed
* [ ] Review `git status` before pushing

## Final Verification

* [ ] Review all Markdown files
* [ ] Verify screenshot filenames
* [ ] Verify README image paths
* [ ] Run Git diff
* [ ] Commit lab
* [ ] Push to GitHub
