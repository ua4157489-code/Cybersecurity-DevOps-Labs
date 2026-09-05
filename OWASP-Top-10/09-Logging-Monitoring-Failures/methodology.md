# Methodology - OWASP A09: Security Logging and Monitoring Failures

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach

Unlike prior labs in this series, A09 is not about finding a single exploitable endpoint - it is about demonstrating that malicious activity goes undetected and unlogged. The testing approach therefore focused on three independent angles, each proving an absence of a control rather than the presence of a bug:

1. **Error handling behavior** - Deliberately triggered server-side errors (via malformed/unsupported route+method combinations) to check whether the application catches exceptions and returns a generic, sanitized response, or leaks full internal stack traces to the client. A leaked stack trace is treated as evidence that server-side logging/error-handling is either absent or misconfigured to double as the client response.

2. **Brute-force response testing** - Sent a burst of rapid, consecutive failed login attempts against a known account and measured both the HTTP status code and response time per attempt. Consistent status codes and flat response times across all attempts indicate no rate-limiting, lockout, or anomaly-detection response exists.

3. **Security event/audit trail discovery** - Inspected the full application configuration (as an authenticated admin) for any reference to a security notification system, audit log, or SIEM/alerting integration. Absence of any such reference, combined with the lack of any dedicated log-retrieval endpoint discovered across the entire lab series, was treated as evidence of a systemic logging/monitoring gap.

## Tools Used
- `curl` with `-w` timing/status format strings for precise response measurement
- `python3 -m json.tool` / inline `json` parsing for configuration inspection
- Terminal screenshots for visual evidence

## Scope
Limited to observable, black-box evidence of missing logging/monitoring/alerting behavior against the local Juice Shop instance. No access to server-side logs, source code, or infrastructure was used - all findings are based purely on client-observable behavior, consistent with an external attacker's perspective.
