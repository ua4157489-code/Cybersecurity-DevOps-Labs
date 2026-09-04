# Methodology — OWASP A03: Injection

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Authentication bypass testing** — attempted a well-known SQL comment-injection pattern (`' --`) against the login endpoint's email field. This targets the most common vulnerable pattern in hand-rolled SQL login checks: `WHERE email = '<input>' AND password = '<hash>'`, where a trailing comment removes the password check entirely.
2. **Token verification** — decoded the resulting JWT to confirm the bypass produced a *genuine* authenticated admin session (matching role and real password hash), not a coincidental error response.
3. **Baseline establishment for search injection** — queried the product search endpoint with a normal term (`apple`) to record expected behavior and result shape/columns before attempting injection.
4. **Iterative injection testing** — started with a simple comment-based payload, observed it silently failed (empty result rather than an error or expanded dataset), then escalated to a UNION-based payload once the underlying table's column count could be inferred from the product schema.
5. **Column-count matching** — Juice Shop's product model exposes 9 fields; the UNION payload was constructed to select exactly 9 columns from the `Users` table (mapping `email` and `password` into the first two positions, padding the rest) so the database would accept the UNION without a column-mismatch error.
6. **Evidence capture** — saved raw JSON responses (`raw-output/`) for every request, plus terminal screenshots (`screenshots/`) for the three key results (bypass, token decode, data dump).

## Tools Used
- `curl` for direct API requests
- `python3 -m json.tool` for JSON pretty-printing
- `python3` (base64/json) for manual JWT payload decoding
- Manual URL-encoding of SQL payloads to avoid shell/query-string mangling

## Scope
Limited to the Injection (A03:2021) category as tested against the local Juice Shop instance, focused on classic SQL injection in the login and search endpoints. NoSQL injection was considered but not pursued in this lab; no destructive queries (`DROP`, `DELETE`, `UPDATE`) were attempted.
