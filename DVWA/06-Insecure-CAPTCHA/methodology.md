# Methodology — Insecure CAPTCHA Lab

## 1. Reconnaissance
Navigated to the "Insecure CAPTCHA" module (`/vulnerabilities/captcha/`), which presents a password-change form gated behind a Google reCAPTCHA widget. Reviewing the page source revealed a warning: `reCAPTCHA API key missing from config file`. This meant the widget itself wasn't rendering, and by extension the standard browser-based flow couldn't be tested as-is.

## 2. Adjusting the test environment
Rather than treat this as a dead end, the config was checked directly inside the container:

```bash
docker exec dvwa cat /var/www/html/config/config.inc.php | grep -i recaptcha
```

This confirmed both `recaptcha_public_key` and `recaptcha_private_key` were empty strings. To properly exercise the vulnerable code path (not just observe a broken widget), Google's publicly documented **test keys** — provided specifically for development/testing and designed to always validate successfully — were appended to the config, followed by a container restart to reload PHP config state.

This step matters methodologically: without functioning reCAPTCHA keys, an assessor might incorrectly conclude the endpoint is unreachable or the form is disabled, when in fact the underlying server-side logic flaw is present regardless of whether the widget renders.

## 3. Understanding the vulnerable logic
Reviewing DVWA's source for this module (via "View Source" / known DVWA source) shows the low-security implementation processes the form in two steps identified by a `step` POST parameter:
- `step=1`: initial submission, expected to include CAPTCHA response, validates it server-side
- `step=2`: expected to run only *after* step 1's CAPTCHA check has passed, and performs the actual password UPDATE query

The flaw: step 2's handler checks `if ($_POST['step'] == 2)` but does not verify that step 1 actually happened or that CAPTCHA validation passed within this request chain. There's no server-side state (session flag, one-time token) linking a validated CAPTCHA to permission for step 2 to execute.

## 4. Exploitation
Sent a single authenticated POST request directly to `/vulnerabilities/captcha/` with:
```
step=2&password_new=hacked123&password_conf=hacked123&Change=Change
```
No `g-recaptcha-response` field was included at all — the request never attempted to interact with reCAPTCHA in any way.

## 5. Validating impact (unplanned but conclusive)
The next login attempt with the original credentials (`admin`/`password`) failed, redirecting back to the login page instead of the dashboard. Testing `admin`/`hacked123` instead succeeded (`302 -> index.php`), which is unambiguous confirmation: the exploit payload had, in fact, changed the live admin account's password — not merely returned a misleading success message, but actually mutated persistent state, with zero CAPTCHA interaction.

## 6. Recovery
The same bypass technique was reused in reverse (`password_new=password`) to restore original credentials, additionally demonstrating the vulnerability is repeatable and not a one-time fluke tied to a specific session or token.
