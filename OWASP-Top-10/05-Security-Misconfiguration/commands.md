# Commands Log — OWASP A05: Security Misconfiguration (Juice Shop)

## Setup
cd ~/Alrazzaq_Labs/OWASP-Top-10
mkdir -p 05-Security-Misconfiguration/raw-output 05-Security-Misconfiguration/screenshots
cd 05-Security-Misconfiguration
touch README.md commands.md findings.md methodology.md remediation.md

## Environment check (container had stopped between sessions)
docker ps
docker ps -a
docker start owasp-juice-shop-a01
sleep 5
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000/
docker logs owasp-juice-shop-a01 --tail 30

## Check 1: response headers on root page
curl -sI http://localhost:3000/ \
  -o raw-output/headers_root.txt
cat raw-output/headers_root.txt
# Result: present -> X-Content-Type-Options, X-Frame-Options
# missing -> Content-Security-Policy, Strict-Transport-Security, Referrer-Policy, Permissions-Policy
# leaking -> Access-Control-Allow-Origin: *, custom X-Recruiting header disclosing an internal path

## Check 2: CORS behavior on an authenticated endpoint with an arbitrary Origin
curl -s -I -X GET http://localhost:3000/rest/user/whoami \
  -H "Authorization: Bearer $TOKEN" \
  -H "Origin: https://evil-attacker-site.com" \
  -o raw-output/cors_authenticated_check.txt
cat raw-output/cors_authenticated_check.txt
# Result: Access-Control-Allow-Origin: * present, no Access-Control-Allow-Credentials header
# (auth is Bearer-token based, not cookie based, which caps real-world impact)

## Check 3: exposed /ftp directory listing
curl -s http://localhost:3000/ftp/ -o raw-output/ftp_directory_listing.html
cat raw-output/ftp_directory_listing.html | grep -i 'href'
# Result: unauthenticated listing reveals filenames including incident-support.kdbx,
# package.json.bak, package-lock.json.bak, coupons_2013.md.bak, encrypt.pyc, eastere.gg,
# announcement_encrypted.md, suspicious_errors.yml, acquisitions.md, legal.md, quarantine/

## Check 4: file-type restriction on /ftp file serving
curl -s "http://localhost:3000/ftp/package.json.bak" -o raw-output/ftp_file_type_check.html
cat raw-output/ftp_file_type_check.html
# Result: 403 - "Only .md and .pdf files are allowed!" (confirms download restricted to those extensions)

## Check 5: confirmed content disclosure via allowed .md file
curl -s "http://localhost:3000/ftp/acquisitions.md" -o raw-output/ftp_acquisitions.md
cat raw-output/ftp_acquisitions.md
# Result: file explicitly self-labeled "confidential! Do not distribute!", served with zero authentication

curl -s "http://localhost:3000/ftp/legal.md" -o raw-output/ftp_legal.md
cat raw-output/ftp_legal.md
# Result: filler/lorem-ipsum content, kept as a non-sensitive baseline comparison
