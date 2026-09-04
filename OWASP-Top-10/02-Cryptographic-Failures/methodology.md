# Methodology — OWASP A02: Cryptographic Failures

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Credential testing** — attempt well-documented default/weak credentials against the login endpoint rather than brute-forcing blindly, since Juice Shop is a known intentionally-vulnerable target with public writeups. This mirrors real-world attacks that start with credential-stuffing known defaults before anything more sophisticated.
2. **Token inspection** — decode the JWT issued on successful login without any special tooling (just base64 + JSON), to check what data is exposed to a client that shouldn't need to see it (role claims, hashes, internal IDs).
3. **Hash algorithm identification** — inspect the format/length of the exposed hash (32 hex chars = MD5 signature) and confirm the algorithm by reproducing it locally against a candidate plaintext.
4. **Salt verification** — confirm whether the hash is salted by checking if the same plaintext always produces the same hash across contexts; an unsalted match confirms the weakness.
5. **Supporting checks** — probe adjacent endpoints (`/rest/user/whoami`, `/ftp`) for further sensitive data exposure that compounds the core cryptographic weaknesses.
6. **Evidence capture** — save raw JSON responses (`raw-output/`) for every request, plus terminal screenshots (`screenshots/`) for the three key results.

## Tools Used
- `curl` for direct API requests
- `python3` (base64/json) for manual JWT payload decoding
- `md5sum` for local hash reproduction
- Terminal screenshots for visual evidence

## Scope
Limited to the Cryptographic Failures (A02:2021) category as tested against the local Juice Shop instance. No credential brute-forcing beyond known/documented defaults was performed, and no destructive actions were taken.
