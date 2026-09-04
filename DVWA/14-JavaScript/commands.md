# Commands — JavaScript Attacks Lab

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

## 2. Retrieve and read the full page source

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/javascript/" -o js_baseline.html
cat js_baseline.html
```

**Key logic found in the inline `<script>` block:**
```javascript
function generate_token() {
    var phrase = document.getElementById("phrase").value;
    document.getElementById("token").value = md5(rot13(phrase));
}
```
(An MD5 implementation and a `rot13()` function are both included inline in the page — no server round-trip is involved in token generation at all.)

## 3. Independently compute the expected token

```bash
python3 -c "
import codecs, hashlib
phrase = 'success'
rot13_phrase = codecs.encode(phrase, 'rot_13')
print('ROT13(success):', rot13_phrase)
token = hashlib.md5(rot13_phrase.encode()).hexdigest()
print('MD5(ROT13(success)):', token)
"
```

**Result:**
```
ROT13(success): fhpprff
MD5(ROT13(success)): 38581812b435834ebf84ebcc2c6424d6
```

## 4. Cross-verify against the page's own JavaScript (browser console)

Opened DevTools Console on the live page and ran the page's own functions directly:
```javascript
md5(rot13("success"))
```
**Result:** `"38581812b435834ebf84ebcc2c6424d6"` — exact match with the independently computed Python value, confirming the reverse-engineered logic is correct.

## 5. Submit the forged token via curl (bypassing the browser entirely)

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/javascript/" \
  -d "phrase=success&token=38581812b435834ebf84ebcc2c6424d6&send=Submit" \
  -o js_bypass_result.html

grep -io "well done\|invalid token\|incorrect" js_bypass_result.html
```

**Result:** `Well done`

```bash
grep -B2 -A2 "Well done" js_bypass_result.html
```

Confirms successful submission using a token computed entirely outside the browser, with the actual page JavaScript never having executed for this specific request.
