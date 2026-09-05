# Methodology — OWASP A07: Identification and Authentication Failures

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Category framing** — this category covers failures in verifying and maintaining user identity: missing brute-force protection, weak credential policy, and improper session/token lifecycle management. Testing moved from the login boundary itself (can an attacker guess a password unimpeded?) outward to the full session lifecycle (can a captured token be revoked, and what does it expose?).
2. **Brute-force baseline, then quantified escalation** — a small batch of 10 failed logins established a quick baseline (all `401`, no visible blocking). Rather than stopping there, the test was scaled to 30 attempts with response headers captured and total wall-clock time measured, producing a concrete, citable result (30 attempts / 1.25 seconds / zero blocked) instead of a vague "no rate limiting observed."
3. **Connecting findings across labs** — the app's own dependency manifest (pulled directly in the A06 lab) lists `express-rate-limit` as an installed package. This made the brute-force test more pointed: the question wasn't just "is there rate limiting" but "is the rate-limiting capability that clearly exists in this codebase actually applied to the login endpoint" — and the 30-attempt result answered that directly.
4. **Weak password policy check** — a single-character password was submitted at registration to test for any minimum-length or complexity enforcement, without needing to fuzz a range of weak passwords once the simplest possible case (`"a"`) succeeded.
5. **Session lifecycle investigation** — rather than assuming a "logout" button has real server-side effect, the actual logout endpoint was tested directly. It resolved to a non-existent server route (an Angular-only client path), which was the first signal that logout is purely a client-side/cosmetic action.
6. **Token content and expiry inspection** — a fresh token was obtained, confirmed functional against an authenticated endpoint, and its payload was decoded (JWTs are base64, not encrypted, so this required no cracking) to check for an `exp` claim. Its absence, combined with the missing logout endpoint, was traced through to its full implication: a captured token cannot be invalidated by any mechanism, time-based or user-initiated.
7. **Incidental discovery treated as a first-class finding** — while inspecting the token payload for the expiry claim, the user's password hash was found embedded directly in the payload. This wasn't the original target of the test, but was documented as its own finding rather than being noted only in passing, since it independently qualifies as a real authentication-data-exposure issue.

## Tools Used
- `curl` with `-D -` to capture full response headers, `-w` for status/timing, and shell-level looping (`for`, `time`) for volume/timing tests
- `python3` for JWT payload decoding (`base64`, `json`)
- Cross-referencing with the A06 dependency findings (`express-rate-limit` presence) to sharpen the brute-force test's framing

## Scope
Focused on login brute-force resistance, registration-time password policy, and session/token lifecycle (expiration and logout). Security-question-based password reset flows were identified as a candidate test for this category but not pursued in this session.
