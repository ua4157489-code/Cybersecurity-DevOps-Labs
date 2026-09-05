# Findings — OWASP A05: Security Misconfiguration

## Finding 1: Unauthenticated Disclosure of a Confidential Document

**Severity:** Critical
**Endpoint:** `GET /ftp/acquisitions.md`
**Status:** Vulnerable

### Description
A file explicitly labeled inside its own content as **"confidential! Do not distribute!"** is served by the application with **no authentication or access control whatsoever**. Any unauthenticated user (or automated scanner) can retrieve it directly.

### Reproduction
```
curl -s "http://localhost:3000/ftp/acquisitions.md"
```
Returns the full document content, including the confidentiality notice, with a plain `200 OK` and no login required.

### Impact
Direct, unambiguous information disclosure — the application itself asserts the sensitivity of this content, removing any ambiguity about severity. In a real deployment, a file with this label being world-readable would represent exposure of non-public business information (e.g. M&A plans) to anyone who finds the URL.

### Evidence
- `screenshots/03-confidential-acquisitions-md-exposed.png`

---

## Finding 2: Unauthenticated Directory Listing Discloses Sensitive Filenames

**Severity:** Medium
**Endpoint:** `GET /ftp/`
**Status:** Vulnerable

### Description
The `/ftp/` path returns a full, unauthenticated directory listing of files on the server, including names that strongly imply sensitive content: `incident-support.kdbx` (a KeePass password database), `package.json.bak` / `package-lock.json.bak` (full dependency manifests, useful for identifying outdated/vulnerable library versions), `coupons_2013.md.bak` (implies historical discount codes that may still be valid — a lead worth following up against A04-style coupon abuse), and a `quarantine/` subdirectory.

A follow-up test confirmed the file-serving route itself enforces an extension allow-list (`Only .md and .pdf files are allowed!`, `403`), meaning these specific files **cannot be downloaded directly through this endpoint**. The finding is therefore reconnaissance-value disclosure — an attacker learns exactly what exists and what to target next — rather than direct arbitrary file read.

### Reproduction
```
curl -s http://localhost:3000/ftp/ | grep -i href
```
Returns filenames and sizes for every file in the directory, unauthenticated.

### Impact
Reveals internal file naming and probable content categories (credentials store, dependency manifests, old promotional data) that materially assist an attacker in prioritizing further attacks, even though direct download of non-`.md`/`.pdf` files is blocked.

### Evidence
- `screenshots/02-ftp-directory-listing-sensitive-files.png`

---

## Finding 3: Wildcard CORS Policy on Authenticated Endpoints

**Severity:** Medium
**Endpoint:** All tested endpoints, including `GET /rest/user/whoami`
**Status:** Vulnerable (limited real-world exploitability given current auth model)

### Description
Every response, including authenticated ones, returns `Access-Control-Allow-Origin: *` — an arbitrary hostile `Origin` header (`https://evil-attacker-site.com`) was reflected/allowed without restriction. However, no `Access-Control-Allow-Credentials: true` header was present.

### Reproduction
```
curl -s -I -X GET http://localhost:3000/rest/user/whoami \
  -H "Authorization: Bearer $TOKEN" \
  -H "Origin: https://evil-attacker-site.com"
```
Returns `Access-Control-Allow-Origin: *` on a request carrying an authenticated Bearer token.

### Impact
Because Juice Shop authenticates via a `Bearer` token that must be explicitly attached by client-side JavaScript (not a cookie automatically sent by the browser), a malicious cross-origin page cannot silently ride an authenticated session the way it could with cookie-based auth plus wildcard CORS. This significantly limits real-world exploitability today. However, the wildcard policy is still a misconfiguration: it removes a layer of defense that would matter if authentication ever changes to cookie-based sessions, or if a separate vulnerability (e.g. an XSS bug) allowed an attacker to first exfiltrate the token.

### Evidence
- `screenshots/01-cors-wildcard-headers.png`

---

## Not Pursued This Session
- `robots.txt` / other exposed metadata endpoints
- Outdated dependency version disclosure via `package.json.bak` metadata (blocked from direct download by extension filter; would require an alternate access path)
- Admin/debug endpoint discovery
