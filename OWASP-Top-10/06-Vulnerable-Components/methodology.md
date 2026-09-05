# Methodology — OWASP A06: Vulnerable and Outdated Components

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Category framing** — this category is about proving a *known* vulnerability is present in a running component, not discovering a new one. The approach was therefore: identify actual dependency versions, cross-reference them against public CVE databases, then (where feasible) build a working proof-of-concept rather than stopping at a version-number citation.
2. **Retrieving the real dependency manifest** — rather than inferring versions from error-page banners (as seen leaking `Express ^4.22.1` in earlier labs), direct filesystem access to the running container was used to pull the actual `package.json`. When the container's minimal image lacked a `cat` binary, `docker cp` was used instead to copy the real file to the host for inspection.
3. **Spotting the anomaly, not just reading the list** — nearly every dependency in the manifest uses a caret range (`^x.y.z`), which would auto-update to newer patch/minor versions. Three dependencies stood out by being pinned to old, exact versions instead: `jsonwebtoken` (`0.4.0`), `express-jwt` (`0.1.3`), and `sanitize-html` (`1.4.2`). This pattern — exact pins that break from the surrounding convention — was treated as the signal worth investigating, rather than checking every dependency exhaustively.
4. **Confirming resolved versions, not just declared ranges** — the lockfile was checked directly for each flagged package's actual resolved `version` field, since a declared range and an installed version can differ. The lockfile also surfaced npm's own registry metadata marking `jsonwebtoken@0.4.0` as deprecated with a direct link to its own vulnerability disclosure — a piece of self-documented evidence stronger than any external citation.
5. **Cross-referencing CVEs** — each flagged package/version was checked against public vulnerability databases (CVE/NVD, GitHub Advisory Database) to confirm it fell within an affected range and to identify the specific vulnerability class and fixed version.
6. **Building a working exploit, not stopping at citation** — for the most severe finding (`jsonwebtoken` algorithm confusion, CVE-2015-9235), the actual attack was carried out: the server's signing algorithm was confirmed as `RS256` from a legitimate token's header, the public verification key was retrieved from an exposed endpoint, and a forged token was constructed claiming `alg: HS256` while using the RSA public key's raw bytes as the HMAC secret.
7. **Establishing a clean control before claiming success** — before treating any accepted forged token as proof, a genuinely privilege-gated endpoint was identified and confirmed to reject requests with no token at all (`401`). Only after that control was established was the forged token tested against the same endpoint, avoiding a false positive from an endpoint that might have been open by default (as one earlier candidate endpoint turned out to be).

## Tools Used
- `docker cp` / `docker exec` for direct container filesystem access
- `grep` for extracting dependency entries from `package.json` / `package-lock.json`
- Public CVE/NVD and GitHub Advisory Database lookups for each flagged package
- Custom `python3` script (`hmac`, `hashlib`, `base64`) to manually construct a forged JWT, bypassing the need for any JWT library
- `curl` for all HTTP-level testing

## Scope
Focused on three dependencies flagged by version-pinning anomaly (`jsonwebtoken`, `express-jwt`, `sanitize-html`), with a full working exploit built and demonstrated for the most severe of the three. A broader `npm audit`-style sweep of every dependency was not performed in this session.
