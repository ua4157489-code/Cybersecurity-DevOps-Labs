# Commands – DVWA File Upload

## 1. Navigate to Lab Directory

```bash
cd ~/Alrazzaq_Labs/DVWA/05-File-Upload
```

---

## 2. Create Screenshots Directory

```bash
mkdir -p screenshots
```

---

## 3. Verify Lab Structure

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

## 4. Create a Harmless Test File

A harmless text file was created for baseline testing:

```bash
echo "DVWA File Upload Test" > test.txt
```

Verify the file:

```bash
cat test.txt
```

Expected output:

```text
DVWA File Upload Test
```

---

## 5. Create a Harmless HTML Test File

A harmless HTML file was created to test file-type validation:

```bash
echo "<h1>DVWA File Upload Test</h1>" > test.html
```

Verify the file:

```bash
cat test.html
```

Expected output:

```text
<h1>DVWA File Upload Test</h1>
```

---

## 6. Verify Test Files

```bash
ls -lh test.txt test.html
```

This confirms that both controlled test files exist locally.

---

## 7. Access DVWA File Upload

Open the following URL in the browser:

```text
http://localhost:8080/DVWA/vulnerabilities/upload/
```

The DVWA security level was configured to:

```text
Low
```

---

## 8. Upload Baseline File

The harmless file:

```text
test.txt
```

was uploaded through the DVWA File Upload form.

The application returned:

```text
../../hackable/uploads/test.txt successfully uploaded!
```

This confirmed that the file was accepted.

---

## 9. Inspect Upload Request

Firefox Developer Tools were used:

```text
F12
→ Network
→ Upload file
→ Select POST request
```

Relevant request information:

```text
Method: POST
Endpoint: /DVWA/vulnerabilities/upload/
Status: 200 OK
Content-Type: multipart/form-data
```

---

## 10. Upload Arbitrary File-Type Test

The harmless HTML file:

```text
test.html
```

was uploaded through the same interface.

The application returned:

```text
../../hackable/uploads/test.html successfully uploaded!
```

This demonstrated insufficient file-type restrictions.

---

## 11. Verify Evidence Screenshots

```bash
ls -lh screenshots/
```

Expected evidence:

```text
01-file-upload-result.png
02-upload-network.png
03-arbitrary-file-upload.png
```

---

## 12. Check Git Status

From the lab directory:

```bash
git status
```

This shows which documentation and evidence files are currently untracked or modified.

---

## 13. Review Lab Files

```bash
find . -maxdepth 2 -type f -print
```

Expected files:

```text
./README.md
./methodology.md
./commands.md
./findings.md
./remediation.md
./screenshots/01-file-upload-result.png
./screenshots/02-upload-network.png
./screenshots/03-arbitrary-file-upload.png
```

---

## 14. Cleanup Test Files

After completing the lab, the local harmless test files can be removed:

```bash
rm -f test.txt test.html
```

Verify:

```bash
ls -la
```

The evidence screenshots and documentation should remain.

---

## 15. Evidence Mapping

| Command / Action     | Evidence                       |
| -------------------- | ------------------------------ |
| Upload `test.txt`    | `01-file-upload-result.png`    |
| Inspect POST request | `02-upload-network.png`        |
| Upload `test.html`   | `03-arbitrary-file-upload.png` |

---

## 16. Security Note

The commands in this document were used only against the intentionally vulnerable local DVWA environment.

No executable web shell, malware, persistence mechanism, or unauthorized target was used.

The purpose of the commands was to demonstrate file-upload behavior and validate insufficient file-type restrictions in a controlled lab.
