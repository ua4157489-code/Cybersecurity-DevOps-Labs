# Remediation — OWASP A05: Security Misconfiguration

## Finding 1: Unauthenticated Disclosure of a Confidential Document

**Fix:**
- Remove any genuinely confidential or non-public documents from any publicly reachable path entirely — a file server should never be the storage location for sensitive business documents.
- If a file-sharing feature is required, gate it behind authentication and authorization (role-based access, e.g. only `admin` or `legal` roles can access legal/business documents), not just an extension filter.
- Add a content classification / pre-deployment review step so files containing terms like "confidential" or "do not distribute" cannot ship to a public-facing directory in the first place — this can be automated as a CI check (grep for known sensitivity markers in files under public web roots).

## Finding 2: Unauthenticated Directory Listing Discloses Sensitive Filenames

**Fix:**
- Disable directory listing entirely on any publicly exposed path (in Express/`serve-index`, remove or restrict the middleware serving `/ftp`, or set `index: false` equivalent configuration).
- If a public file-drop feature is genuinely required, serve an explicit allow-list of intended-public files rather than exposing the full directory contents — never let unrelated files (backups, credential stores, old configs) sit reachable in the same served directory.
- Remove backup files (`.bak`), leftover data files (`.kdbx`, `.pyc`), and old/deprecated data (`coupons_2013.md.bak`) from any directory reachable by the web server, public or not — these should never be deployed alongside application code in the first place, addressed via `.gitignore`/build-pipeline hygiene rather than access control alone.

## Finding 3: Wildcard CORS Policy on Authenticated Endpoints

**Fix:**
- Replace `Access-Control-Allow-Origin: *` with an explicit allow-list of trusted origins (the application's own known frontend domain(s)), even though current token-based auth limits practical exploitability today.
- Never combine a wildcard or overly permissive CORS policy with `Access-Control-Allow-Credentials: true` in the future if authentication is changed to cookie/session-based — that combination is what turns this into a critical finding, so it should be explicitly disallowed via configuration and covered by a regression test.
- Treat CORS configuration as part of the security baseline reviewed whenever the authentication mechanism changes, not a one-time setting.

## General Principle
Every finding in this lab stems from the same root cause: a helpful **default** (broad CORS, permissive directory listing, verbose errors seen in the previous A04 lab) was left in its most permissive state rather than deliberately configured for production. Security Misconfiguration findings are rarely about a single dangerous line of code — they're about defaults nobody revisited before shipping.

**Recommended controls:**
- A documented, version-controlled security baseline configuration (headers, CORS policy, error verbosity) applied identically across environments, reviewed before each release
- Automated security header scanning (e.g. `securityheaders.com`-style checks) as part of CI/CD
- Regular audits of any directory served by the web server for unintended file exposure (backups, credential stores, stale data)
- Least-privilege review of every "convenience" feature (file servers, debug endpoints, permissive CORS) before it reaches production, with an explicit owner sign-off
