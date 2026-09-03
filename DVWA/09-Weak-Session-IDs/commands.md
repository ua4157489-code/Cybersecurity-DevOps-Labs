# 🛠 Lab 09 — Weak Session IDs Commands

## Navigate to Lab

```bash
cd ~/Alrazzaq_Labs/DVWA/09-Weak-Session-IDs
```

---

## Access Weak Session IDs Page

```bash
curl -b cookies.txt -c cookies.txt -s \
  http://localhost:4280/vulnerabilities/weak_id/ \
  -o weak_id_page_1.html
```

---

## Generate Session IDs

```bash
for i in 1 2 3 4 5; do
  curl -b cookies.txt -c cookies.txt -s \
    -X POST http://localhost:4280/vulnerabilities/weak_id/ \
    -d "Generate=Generate" \
    -o weak_id_generate_$i.html

  value=$(awk '$6=="dvwaSession"{print $7}' cookies.txt)
  echo "Request $i: dvwaSession=$value"
done
```

---

## View Cookie

```bash
cat cookies.txt
```

---

## Extract Session ID

```bash
awk '$6=="dvwaSession"{print $7}' cookies.txt
```

---

## View Session Sequence

```bash
cat raw-output/session-sequence.txt
```

---

## View Vulnerable Page Evidence

```bash
cat raw-output/vulnerable-page.txt
```

---

## Inspect Generated Response

```bash
grep -i -A5 -B5 "dvwaSession" weak_id_generate_5.html
```

---

## Remove Sensitive Cookie Evidence

```bash
rm cookies.txt
```

---

## Git Status

```bash
git status
```

---

## Git Add

```bash
git add .
```

## Git Commit

```bash
git commit -m "Add Lab 09 Weak Session IDs"
```

## Git Push

```bash
git push origin main
```
