#!/usr/bin/env bash

# ============================================================

# MITRE ATT&CK T1059.004 - Unix Shell

# Lab Command Reference

# ============================================================

set -u

echo "===== ENVIRONMENT VERIFICATION ====="

echo "[OS]"
cat /etc/os-release | grep -E 'PRETTY_NAME|VERSION='

echo
echo "[CURRENT SHELL]"
echo "$SHELL"

echo
echo "[BASH VERSION]"
bash --version | head -n 1

echo
echo "[BASH PATH]"
command -v bash

echo
echo "[CURRENT USER]"
whoami

echo
echo "[LAB DIRECTORY]"
pwd

# ============================================================

# BASH EXECUTION SIMULATION

# ============================================================

echo
echo "===== BASH EXECUTION SIMULATION ====="

bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'

# ============================================================

# AUDITD STATUS

# ============================================================

echo
echo "===== AUDITD STATUS ====="

systemctl is-active auditd
sudo auditctl -s

# ============================================================

# AUDIT LOG DIRECTORY

# ============================================================

echo
echo "===== AUDIT LOG DIRECTORY ====="

sudo ls -ld /var/log/audit
sudo ls -lh /var/log/audit/audit.log

# ============================================================

# CREATE AUDIT LOG DIRECTORY

# Use only if auditd is not starting because the directory

# does not exist.

# ============================================================

echo
echo "===== AUDIT LOG DIRECTORY SETUP ====="

sudo mkdir -p /var/log/audit
sudo chown root:adm /var/log/audit
sudo chmod 0750 /var/log/audit

# ============================================================

# RESTART AUDITD

# ============================================================

echo
echo "===== RESTART AUDITD ====="

sudo systemctl restart auditd

echo
echo "AUDITD STATE:"
systemctl is-active auditd

echo
echo "KERNEL AUDIT STATUS:"
sudo auditctl -s

# ============================================================

# CREATE MITRE T1059.004 AUDIT RULE

# ============================================================

echo
echo "===== CREATE T1059.004 AUDIT RULE ====="

sudo tee /etc/audit/rules.d/mitre-t1059-004.rules > /dev/null <<'EOF'
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/bash -F auid>=1000 -F auid!=-1 -k mitre_t1059_004
EOF

sudo augenrules --load

# ============================================================

# VERIFY AUDIT RULE

# ============================================================

echo
echo "===== VERIFY T1059.004 AUDIT RULE ====="

sudo auditctl -l | grep mitre_t1059_004

# ============================================================

# GENERATE CONTROLLED BASH EVENT

# ============================================================

echo
echo "===== GENERATE BASH TELEMETRY ====="

bash -c 'echo "MITRE T1059.004 test execution"; whoami; uname -srm'

# ============================================================

# SEARCH AUDIT EVENTS

# ============================================================

echo
echo "===== T1059.004 AUDIT EVENTS ====="

sudo ausearch -k mitre_t1059_004 -i | tail -n 40

# ============================================================

# CLEAN DETECTION EVIDENCE

# ============================================================

echo
echo "===== T1059.004 DETECTION EVIDENCE ====="

sudo ausearch -k mitre_t1059_004 -i 
| grep -E 'type=(EXECVE|SYSCALL)' 
| grep -E 'bash|execve|mitre_t1059_004'

# ============================================================

# AUDIT LOG VERIFICATION

# ============================================================

echo
echo "===== AUDIT LOG VERIFICATION ====="

sudo ls -lh /var/log/audit/audit.log

echo
echo "===== RECENT T1059.004 EVENTS ====="

sudo ausearch -k mitre_t1059_004 -i | tail -n 25

# ============================================================

# END

# ============================================================

echo
echo "===== LAB COMMAND SEQUENCE COMPLETE ====="
