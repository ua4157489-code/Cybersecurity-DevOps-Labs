# Methodology — OWASP A04: Insecure Design

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Category framing** — Insecure Design (A04:2021) is distinct from the other categories tested so far: it targets flaws in application *logic and workflow*, present even when individual code paths are "correctly" implemented. The test plan was chosen accordingly — probing business rules (basket quantity, checkout totals) rather than a single injectable parameter.
2. **Baseline and boundary testing** — before assuming a vulnerability, established what "normal" looked like (positive quantity add) and tested boundary values (`0`, `-1`, `-5`) on the item-creation path to see how the API responded at each edge.
3. **Distinguishing real findings from test noise** — the first attempts on the `POST` (create) path returned `500` errors. Rather than treating this as the finding, the errors were investigated: stack traces were read to confirm the failure point (database layer), and a control test (adding a *different*, not-yet-present product) was run to isolate whether the 500 was caused by negative quantity itself or an unrelated unique-constraint collision from a leftover row. This confirmed the 500s were a side effect of test setup, not the actual security boundary.
4. **Testing the corresponding update path** — since create appeared to have an (accidental, badly-handled) constraint, the logical next step was checking whether the `PUT` (update) endpoint enforced the same rule. It did not — this asymmetry between create and update validation is the core Insecure Design finding.
5. **Chasing impact, not just acceptance** — an accepted negative number alone isn't proof of harm. The negative quantity was traced through to (a) the computed basket total, and (b) a full checkout attempt, to establish whether the flaw was contained to display/state or actually reachable at the business-transaction level.
6. **Confirming persistence** — after checkout succeeded, the resulting order was looked up via the order-tracking endpoint to confirm the negative total was written to a permanent, queryable record — not just a transient checkout response — which is the strongest evidence of real business impact.
7. **Evidence capture** — three screenshots were taken covering the three decisive moments: unauthorized negative-quantity acceptance, successful checkout with a negative total, and the persisted order record proving impact.

## Tools Used
- `curl` for direct API requests against Juice Shop's REST/API layer
- `python3 -m json.tool` for JSON pretty-printing
- Inline `python3` for computing basket totals the same way the application would (quantity × price, summed)
- JWT payload decoding (reused from the A03 lab) to recover user id and basket id without needing to re-authenticate

## Scope
Focused on basket quantity manipulation and its downstream effect on checkout/order totals. Coupon/discount logic abuse and password-reset security-question weaknesses were identified as candidate tests for this category but not pursued in this lab session.
