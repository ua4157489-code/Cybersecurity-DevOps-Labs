# Remediation — SQL Injection (Blind) Lab

## Primary Fix: Parameterized Queries (Prepared Statements)
The root cause here is identical to Lab 07 — the fix is the same and non-negotiable: never concatenate user input into SQL. Use bound parameters so the database driver treats input strictly as data.

**Vulnerable pattern (conceptual):**
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '" . $_GET['id'] . "'";
$exists = mysqli_num_rows(mysqli_query($conn, $query)) > 0;
```

**Fixed pattern:**
```php
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $_GET['id']);
$stmt->execute();
$exists = $stmt->get_result()->num_rows > 0;
```

Parameterization closes both the standard and blind variants of this vulnerability simultaneously, since the flaw is in how input reaches the query, not in what the application chooses to display afterward.

## Defense in Depth — Specific to Blind SQLi

1. **Minimize oracle signals** — Beyond fixing the query itself, avoid designing application responses that leak binary state distinguishable by an attacker (e.g., different response text, timing, or HTTP status for "record exists" vs "record doesn't exist") wherever this isn't strictly necessary for legitimate functionality. This is defense-in-depth only — it should never be relied upon instead of parameterization, since blind techniques can also exploit timing differences (`SLEEP()`-based) even when response *content* is identical.
2. **Generic error handling** — Ensure application errors (DB exceptions, malformed query errors) don't leak distinguishable signals either; a well-known amplification of blind SQLi is when malformed injection payloads produce a *different* error state than well-formed ones, giving attackers an additional oracle beyond the intended one.
3. **Rate limiting on this class of endpoint** — Blind extraction is inherently request-heavy (this lab needed roughly 500+ requests to extract one password hash). Aggressive rate limiting or anomaly detection on repeated near-identical requests to the same parameterized endpoint can meaningfully slow or block automated blind SQLi tooling (e.g., `sqlmap`), even as a stopgap while the root fix is deployed.
4. **Least privilege database accounts** — As in Lab 07, ensure the web application's DB account cannot read `information_schema`, unrelated tables, or perform writes beyond what the specific feature requires.
5. **Web Application Firewall (WAF)** — A WAF with SQLi detection rules (e.g., OWASP CRS) is particularly valuable against blind SQLi automation, since tools like `sqlmap` generate highly repetitive, signature-matchable payload patterns (`AND ASCII(SUBSTRING(...))=`) that are easier to fingerprint than a single crafted UNION payload.

## Verification
After remediation, re-run the extraction scripts documented in `commands.md` against the patched endpoint. All `LENGTH()`/`ASCII(SUBSTRING())`-based probes should return the exact same "exists" response regardless of the injected condition's truth value — proving the injected SQL is no longer being interpreted as executable logic. Additionally verify that response timing is also consistent across payloads, to rule out a residual time-based blind channel.
