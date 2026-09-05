# Remediation — OWASP A06: Vulnerable and Outdated Components

## Finding 1: Critical Algorithm-Confusion Vulnerability in `jsonwebtoken`

**Fix:**
- Upgrade `jsonwebtoken` to a current major version (9.x or later) immediately — this is a critical, actively-exploitable authentication bypass, not a theoretical risk.
- When calling `jwt.verify()`, **always explicitly specify the expected algorithm(s)** via the `algorithms` option (e.g. `algorithms: ['RS256']`), rather than trusting the algorithm declared inside the token's own header. This is the actual root fix — even on a patched library version, omitting this option can reopen similar classes of bugs.
- Never treat a key that is safe to expose for one algorithm family (an RSA/EC **public** key, safe for verification) as if it's safe in all contexts — the vulnerability specifically arises from a public value being reusable as a symmetric secret when algorithm enforcement is missing.
- Add a regression test that specifically attempts this exact attack (forge an HS256 token signed with the known public key, submit it, assert rejection) so this class of bug cannot silently reappear after a future dependency change.

## Finding 2: Authorization Bypass in `express-jwt`

**Fix:**
- Upgrade `express-jwt` to 6.0.0 or later.
- Explicitly configure the `algorithms` option in the `express-jwt` middleware setup — the underlying CVE stems from this option not being enforced by default in old versions, so setting it explicitly is protective even before the upgrade lands.

## Finding 3: Cross-Site Scripting in `sanitize-html`

**Fix:**
- Upgrade `sanitize-html` to the latest 2.x release (well past the 1.4.3 fix for this specific CVE, and past several subsequent related CVEs, e.g. CVE-2021-26539/26540).
- Audit every call site that uses this library to confirm output is still being escaped/sanitized correctly after the upgrade, since sanitizer libraries occasionally change default-allowed tag/attribute behavior between major versions.

## General Principle
All three findings share a single root cause: **dependencies pinned to old exact versions, breaking from the caret-range convention used everywhere else in the manifest.** This is a meaningful signal in its own right — exact pins are sometimes intentional (compatibility reasons) but are just as often the result of a dependency being forgotten during routine updates. Any exact-pinned dependency should be treated as a standing question that needs an explicit answer: "why is this one different, and has anyone re-checked it since it was pinned?"

**Recommended controls:**
- Automated dependency scanning (`npm audit`, Snyk, Dependabot, or equivalent) integrated into CI, failing builds on known-critical CVEs
- A policy requiring justification (in a comment or commit message) for any dependency pinned to an exact version rather than a range
- Regular (e.g. quarterly) manual review of the full dependency tree, not just automated scanning, since some vulnerable-version findings (like this one) are more valuable when demonstrated end-to-end rather than left as an unverified scanner alert
- Secrets/keys review: confirm that anything intentionally exposed (like a public verification key) cannot be repurposed elsewhere in the system due to a missing algorithm/type check
