#!/usr/bin/env bash

# ============================================================

# MITRE ATT&CK T1105 — Ingress Tool Transfer

# Controlled Docker Laboratory

# ============================================================

set -e

echo "============================================================"
echo " T1105 — Ingress Tool Transfer Lab"
echo "============================================================"

# ------------------------------------------------------------

# 1. Verify Project Directory

# ------------------------------------------------------------

pwd

# ------------------------------------------------------------

# 2. Start the Docker Laboratory

# ------------------------------------------------------------

docker compose up -d

# ------------------------------------------------------------

# 3. Verify Running Containers

# ------------------------------------------------------------

docker compose ps

# ------------------------------------------------------------

# 4. List T1105 Docker Network

# ------------------------------------------------------------

docker network ls | grep t1105

# ------------------------------------------------------------

# 5. Inspect the Isolated Docker Network

# ------------------------------------------------------------

docker network inspect 
t1105-ingress-tool-transfer_t1105-lab

# ------------------------------------------------------------

# 6. Verify Server-Side Test File

# ------------------------------------------------------------

docker exec t1105-transfer-server 
cat /usr/share/nginx/html/t1105-test.txt

# ------------------------------------------------------------

# 7. Test HTTP Connectivity

# ------------------------------------------------------------

docker exec t1105-transfer-client 
curl -I http://transfer-server/t1105-test.txt

# ------------------------------------------------------------

# 8. Perform Controlled File Transfer

# ------------------------------------------------------------

docker exec t1105-transfer-client 
curl http://transfer-server/t1105-test.txt 
-o /tmp/t1105-test.txt

# ------------------------------------------------------------

# 9. Verify Downloaded File Content

# ------------------------------------------------------------

docker exec t1105-transfer-client 
cat /tmp/t1105-test.txt

# ------------------------------------------------------------

# 10. Verify Downloaded File Metadata

# ------------------------------------------------------------

docker exec t1105-transfer-client 
ls -lh /tmp/t1105-test.txt

docker exec t1105-transfer-client 
stat /tmp/t1105-test.txt

# ------------------------------------------------------------

# 11. Calculate Downloaded File SHA-256

# ------------------------------------------------------------

docker exec t1105-transfer-client 
sha256sum /tmp/t1105-test.txt

# ------------------------------------------------------------

# 12. Analyze Nginx HTTP Transfer Logs

# ------------------------------------------------------------

docker logs t1105-transfer-server --tail 20

# ------------------------------------------------------------

# 13. Save Network Evidence

# ------------------------------------------------------------

docker network inspect 
t1105-ingress-tool-transfer_t1105-lab 
| tee screenshots/01-t1105-network-evidence.txt

# ------------------------------------------------------------

# 14. Save HTTP Transfer Evidence

# ------------------------------------------------------------

docker logs t1105-transfer-server --tail 20 
| tee screenshots/02-t1105-http-transfer-evidence.txt

# ------------------------------------------------------------

# 15. Save File Creation Evidence

# ------------------------------------------------------------

docker exec t1105-transfer-client sh -c '
echo "===== FILE METADATA ====="
ls -lh /tmp/t1105-test.txt
stat /tmp/t1105-test.txt

echo
echo "===== FILE HASH ====="
sha256sum /tmp/t1105-test.txt

echo
echo "===== FILE CONTENT ====="
cat /tmp/t1105-test.txt
' | tee screenshots/03-t1105-file-creation-evidence.txt

# ------------------------------------------------------------

# 16. Verify Original and Downloaded File Integrity

# ------------------------------------------------------------

echo "===== SERVER FILE HASH ====="
sha256sum server-files/t1105-test.txt

echo
echo "===== DOWNLOADED FILE HASH ====="
docker exec t1105-transfer-client 
sha256sum /tmp/t1105-test.txt

# ------------------------------------------------------------

# 17. Final Container Status

# ------------------------------------------------------------

docker compose ps

echo
echo "============================================================"
echo " T1105 Laboratory Verification Complete"
echo "============================================================"

