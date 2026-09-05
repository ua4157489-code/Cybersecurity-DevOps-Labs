# Commands Log - OWASP A09: Security Logging and Monitoring Failures (Juice Shop)

## Setup
mkdir -p raw-output screenshots
touch README.md commands.md findings.md methodology.md remediation.md

## Finding 1: Verbose error / stack trace exposure

# Trigger 500 on basket PUT (invalid route pattern for this HTTP method)
curl -s -X PUT http://localhost:3000/rest/basket/2 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"coupon":null}' \
  -o raw-output/verbose_error_basket_put.json

# Trigger 500 on products endpoint (wrong path pattern)
curl -s http://localhost:3000/rest/products/1 \
  -H "Authorization: Bearer $TOKEN" \
  -o raw-output/verbose_error_wrong_endpoint.json

# Control: clean 401 with no internal detail (missing auth header)
curl -s http://localhost:3000/rest/basket/999999999999999999999999 \
  -o raw-output/verbose_error_basket.json

## Finding 2: No brute-force detection / throttling

for i in {1..8}; do
  curl -s -o /dev/null -w "Attempt $i: HTTP %{http_code}  Time: %{time_total}s\n" \
    -X POST http://localhost:3000/rest/user/login \
    -H "Content-Type: application/json" \
    -d '{"email":"admin@juice-sh.op","password":"wrongpass'"$i"'"}'
done

## Finding 3: No security notification/logging channel exposed

curl -s http://localhost:3000/rest/admin/application-configuration \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(json.dumps(d['config'].get('ctf', {}), indent=2))"
