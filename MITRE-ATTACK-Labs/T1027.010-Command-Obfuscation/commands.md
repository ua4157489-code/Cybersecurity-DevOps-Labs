#!/usr/bin/env bash

# ============================================================
# MITRE ATT&CK T1027.010 — Command Obfuscation
# Lab Commands
# ============================================================

set -e

LAB_DIR="$HOME/Alrazzaq_Labs/MITRE-ATTACK-Labs/T1027.010-Command-Obfuscation"

cd "$LAB_DIR"

echo "===== T1027.010 COMMAND OBFUSCATION LAB ====="

# ------------------------------------------------------------
# 1. Environment
# ------------------------------------------------------------

echo
echo "[1] Docker containers"
docker ps --filter name=t1027 \
  --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"

# ------------------------------------------------------------
# 2. Network verification
# ------------------------------------------------------------

echo
echo "[2] Docker network"
docker network inspect t1027010-command-obfuscation_t1027-lab \
  --format 'Name={{.Name}}
Driver={{.Driver}}
Internal={{.Internal}}
Subnet={{range .IPAM.Config}}{{.Subnet}}{{end}}
Gateway={{range .IPAM.Config}}{{.Gateway}}{{end}}'

# ------------------------------------------------------------
# 3. Container IPs
# ------------------------------------------------------------

echo
echo "[3] Container IP addresses"

docker inspect -f \
  '{{.Name}} -> {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  t1027-command-lab \
  t1027-log-monitor

# ------------------------------------------------------------
# 4. Linux environment
# ------------------------------------------------------------

echo
echo "[4] Linux environment"

docker exec t1027-command-lab bash -c '
echo "OS:"
cat /etc/os-release | grep PRETTY_NAME

echo
echo "Kernel:"
uname -a

echo
echo "Shell:"
echo "$SHELL"

echo
echo "Bash:"
command -v bash
'

# ------------------------------------------------------------
# 5. Create harmless test script
# ------------------------------------------------------------

echo
echo "[5] Creating harmless test script"

docker exec t1027-command-lab bash -c '
cat > /tmp/t1027_010_test.sh << "EOF"
#!/usr/bin/env bash

echo "MITRE T1027.010 controlled test"
echo "Command Obfuscation laboratory"
whoami
uname -s
EOF

chmod +x /tmp/t1027_010_test.sh
'

# ------------------------------------------------------------
# 6. Baseline execution
# ------------------------------------------------------------

echo
echo "[6] Baseline execution"

docker exec t1027-command-lab \
  bash /tmp/t1027_010_test.sh

# ------------------------------------------------------------
# 7. String splitting
# ------------------------------------------------------------

echo
echo "[7] String splitting"

docker exec t1027-command-lab bash -c '
a="MITRE"
b="T1027.010"
echo "$a $b controlled test"
'

# ------------------------------------------------------------
# 8. Environment variable
# ------------------------------------------------------------

echo
echo "[8] Environment variable substitution"

docker exec t1027-command-lab bash -c '
export TECHNIQUE="T1027.010"
printf "%s\n" "MITRE $TECHNIQUE Command Obfuscation"
'

# ------------------------------------------------------------
# 9. Character construction
# ------------------------------------------------------------

echo
echo "[9] Character construction"

docker exec t1027-command-lab bash -c '
c1="M"
c2="I"
c3="TRE"
printf "%s\n" "$c1$c2$c3 T1027.010 controlled test"
'

# ------------------------------------------------------------
# 10. Command reconstruction
# ------------------------------------------------------------

echo
echo "[10] Command reconstruction"

docker exec t1027-command-lab bash -c '
cmd="ec""ho"
$cmd "MITRE T1027.010 reconstructed command"
'

# ------------------------------------------------------------
# 11. Bash execution trace
# ------------------------------------------------------------

echo
echo "[11] Bash execution trace"

docker exec t1027-command-lab bash -x -c '
a="MITRE"
b="T1027.010"
echo "$a $b controlled test"

export TECHNIQUE="T1027.010"
printf "%s\n" "MITRE $TECHNIQUE Command Obfuscation"

c1="M"
c2="I"
c3="TRE"
printf "%s\n" "$c1$c2$c3 T1027.010 controlled test"

cmd="ec""ho"
$cmd "MITRE T1027.010 reconstructed command"
' 2>&1 | tee screenshots/02-t1027-010-bash-trace.txt

# ------------------------------------------------------------
# 12. Test script hash
# ------------------------------------------------------------

echo
echo "[12] Test script hash"

docker exec t1027-command-lab \
  sha256sum /tmp/t1027_010_test.sh

# ------------------------------------------------------------
# 13. Process telemetry
# ------------------------------------------------------------

echo
echo "[13] Process telemetry"

docker exec t1027-command-lab ps -ef

# ------------------------------------------------------------
# 14. Docker isolation
# ------------------------------------------------------------

echo
echo "[14] Outbound connectivity"

docker exec t1027-command-lab bash -c '
if timeout 3 bash -c "</dev/tcp/1.1.1.1/80" 2>/dev/null; then
    echo "UNEXPECTED: outbound connectivity available"
else
    echo "EXPECTED: outbound connectivity blocked"
fi
'

echo
echo "===== LAB COMPLETE ====="
