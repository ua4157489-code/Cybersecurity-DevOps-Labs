# Methodology — OWASP A01: Broken Access Control

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Establish baseline identity** — register a fresh, low-privilege ("customer" role) attacker account and authenticate to obtain a valid JWT. This ensures all tests are performed as an authenticated but non-privileged user, which is the most realistic BAC threat model (horizontal privilege escalation between same-tier users).
2. **Decode the JWT** to confirm role and identity claims (`role: customer`, `id`, `bid`) before testing, establishing what the attacker *should* be limited to.
3. **Capture own-resource baseline** — request the attacker's own basket (`/rest/basket/<own-id>`) to confirm expected, authorized behavior before attempting unauthorized access.
4. **Horizontal IDOR testing** — sequentially request adjacent/foreign resource IDs (other basket IDs) using the same valid token, checking whether the server enforces object-level ownership or only checks for a valid session.
5. **Cross-verify read vs write** — test both `GET` (read) and `POST` (write) operations on the same resource type (baskets) to check for inconsistent enforcement across HTTP methods/endpoints.
6. **Document both positive and negative findings** — a blocked exploitation attempt is still valuable evidence (shows what correct enforcement looks like) and helps characterize the inconsistency in the app's access control implementation.
7. **Evidence capture** — save raw JSON responses (`raw-output/`) for every request/response pair, plus terminal screenshots (`screenshots/`) for the two key exploitation results.

## Tools Used
- `curl` for direct API requests
- `python3 -m json.tool` for JSON pretty-printing
- `python3 base64`/`json` for manual JWT payload decoding
- Terminal screenshots for visual evidence

## Scope
Limited to the Broken Access Control (A01:2021) category as tested against the local Juice Shop instance. No destructive testing (delete/mass data modification) was performed.
