# Methodology — XSS (Stored) Lab

## 1. Reconnaissance
Navigated to `/vulnerabilities/xss_s/`, a guestbook feature with `Name` and `Message` fields, "Sign Guestbook" and "Clear Guestbook" buttons. Inspected the form's HTML via `curl` and noted `maxlength` attributes on both inputs (`10` for name, `50` for message) — these are client-side HTML attributes only, easily bypassed by submitting the form data directly rather than through the browser's UI, so they were not treated as a meaningful control during testing.

## 2. Submitting the payload
Rather than testing with a benign value first (as would be standard practice against an unfamiliar target), the script payload was submitted directly, since the goal here was specifically to confirm and document stored XSS behavior, and prior labs (10, 11) had already established that this DVWA instance applies no output encoding at Low security across its XSS modules — a reasonable basis to expect the same here.

```
POST /vulnerabilities/xss_s/
txtName=Tester&mtxMessage=<script>alert('Stored XSS')</script>&btnSign=Sign+Guestbook
```

## 3. Confirming storage (not just reflection)
The critical methodological difference from reflected XSS: after submission, the *same request* returning success doesn't prove persistence — a reflected vulnerability could echo the submitted value back in the confirmation page without ever storing it. To rule this out, a **second, separate GET request** was made to reload the guestbook page fresh, within the same session:

```bash
curl -b cookies.txt -s "http://localhost:4280/vulnerabilities/xss_s/" -o xss_s_after_submit.html
```

Finding the payload embedded in this fresh page load (not just in the immediate POST response) confirmed the data survived a full request/response cycle — i.e., it was actually written to persistent storage (the database), not merely echoed back once.

## 4. Confirming cross-session persistence (the real test of "stored")
Even the previous step could theoretically be explained by session-level state (e.g., a PHP session variable), rather than genuine database storage visible to *other* users. To rule this out definitively, an entirely separate cookie jar was used to perform a fresh login — simulating a different session with zero prior interaction with the guestbook — and the guestbook page was loaded again:

```bash
curl -c cookies2.txt ... # fresh login, new session
curl -b cookies2.txt -s "http://localhost:4280/vulnerabilities/xss_s/" -o xss_s_new_session.html
grep "Stored XSS" xss_s_new_session.html
```

The payload appeared identically in this unrelated session's response. This is the definitive proof of stored XSS: the malicious content is served from shared, persistent storage to any authenticated visitor, not tied to the attacker's own session in any way.

## 5. Browser-based execution confirmation
Finally, opened a fresh Private Browsing window (further ruling out any cached state or cookies from prior testing), logged in normally through the UI, and navigated directly to the guestbook with a clean URL containing no payload whatsoever. The alert fired immediately on page load — visually confirming what the curl-based tests had already established: the stored payload executes automatically for any visitor, with the attacker having no further involvement after the initial submission.

## 6. Why this methodology matters
Distinguishing "stored" from "reflected within the same request" is a common point of confusion in XSS testing. Simply seeing a payload echoed back after submission is not, by itself, proof of stored XSS — it could be a same-request reflection. The two-step verification used here (same-session reload, then cross-session/fresh-login reload) is the rigorous way to confirm genuine persistence before reporting a finding as "stored" rather than "reflected."
