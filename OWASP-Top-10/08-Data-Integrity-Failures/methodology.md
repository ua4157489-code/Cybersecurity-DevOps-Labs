# Methodology — OWASP A08: Software and Data Integrity Failures

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Establish baseline data state** — record the original value of a business-critical field (product price) before any testing, so tampering can be objectively demonstrated and later reverted.
2. **Test with the weakest authenticated context first** — attempt the write operation using a normal, low-privilege customer token rather than jumping straight to unauthenticated requests. This establishes whether the flaw is an authorization gap (any logged-in user can write) versus a full authentication gap.
3. **Escalate to zero-authentication** — repeat the exact same request with no `Authorization` header at all, to determine the true worst-case severity of the flaw rather than stopping at the first successful bypass.
4. **Verify real-world impact, not just API response codes** — a `200 OK` on a write request is not sufficient proof of impact on its own. The tampered state was independently confirmed via the public-facing, unauthenticated search endpoint to prove the change was live and visible to any user of the application, not just an artifact of the direct API call.
5. **Restore original state** — revert all tampered data immediately after evidence capture, to leave the target environment in a clean, unmodified state for any further testing.
6. **Document both the technical finding and its systemic root cause** — rather than treating this as an isolated bug in one endpoint, the finding notes the likely underlying pattern (an auto-scaffolded REST library defaulting to open access) so remediation addresses the class of issue, not just the single reproduced case.

## Tools Used
- `curl` for direct API requests (`PUT`, `GET`)
- `python3 -m json.tool` for JSON pretty-printing
- Terminal screenshots for visual evidence

## Scope
Limited to the Software and Data Integrity Failures (A08:2021) category, specifically data-tampering via improperly-secured write endpoints, as tested against the local Juice Shop instance. No CI/CD pipeline or deserialization testing was performed in this pass (noted as out-of-scope-for-this-session in the findings write-up).
