# Methodology — OWASP A05: Security Misconfiguration

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Category framing** — Security Misconfiguration (A05:2021) covers a wide surface: missing hardening headers, verbose defaults, unnecessary exposed features, and permissive access policies. Rather than testing one specific bug, the approach was a breadth-first sweep of common misconfiguration categories, escalating into whichever produced a real signal.
2. **Environment recovery** — the target container had stopped between sessions. Before treating early connection failures as findings, the environment was verified (`docker ps -a`, container logs) and restarted, confirming the earlier failures were an environment issue, not a discovery.
3. **Header inspection baseline** — a plain `HEAD` request against the root page established which security headers were present, missing, or unusually informative (e.g. a non-standard `X-Recruiting` header), without needing authentication.
4. **CORS depth-check, not just presence-check** — a wildcard `Access-Control-Allow-Origin` was visible immediately, but rather than reporting it at face value, the test was repeated against an **authenticated** endpoint with a hostile `Origin` header, and the response was checked specifically for `Access-Control-Allow-Credentials`. This distinction matters: wildcard CORS combined with credentialed (cookie-based) requests is a critical cross-origin account-takeover primitive, but Juice Shop uses Bearer-token auth, which a malicious origin cannot silently attach the way a browser would with cookies. The severity assigned reflects this nuance rather than treating "CORS wildcard exists" as automatically critical.
5. **Exposed-endpoint discovery** — Juice Shop's `/ftp` directory-listing feature was checked directly. Rather than assuming impact from the listing alone, a follow-up request attempted to download a `.bak` file to test whether the file-serving route enforced any restriction — it did (extension allow-list, `.md`/`.pdf` only), which shaped how the finding is framed (reconnaissance-value disclosure of filenames, not arbitrary file read).
6. **Confirming real content impact** — rather than stopping at "filenames are visible," an allowed file type (`.md`) was actually fetched. One file (`acquisitions.md`) turned out to be self-labeled as confidential inside its own content, which converts the finding from theoretical to a concrete, self-evident disclosure. A second file (`legal.md`) was fetched as a control comparison and found to be non-sensitive filler content, avoiding overstating the impact of every file in the listing.

## Tools Used
- `curl` (with `-I` for header-only requests, custom `Origin` headers for CORS testing)
- `grep` for extracting file listings from raw HTML
- `docker` commands for environment/container verification

## Scope
Focused on HTTP security headers, CORS configuration, and the `/ftp` exposed-directory feature. Additional candidate checks (outdated dependency disclosure via `package.json.bak` metadata, `robots.txt`, admin-endpoint discovery) were identified but not pursued in this session.
