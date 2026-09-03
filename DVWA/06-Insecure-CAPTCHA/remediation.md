# Remediation — Insecure CAPTCHA Lab

## Primary Fix: Bind CAPTCHA validation to server-side session state
Never trust a client-supplied `step` parameter as proof that a prior control was satisfied. Instead, validate the CAPTCHA response server-side within the *same request* that performs the sensitive action, or set a short-lived, server-side session flag only after successful CAPTCHA verification, and check that flag (not `step`) before executing the password change.

**Vulnerable pattern (conceptual):**
```php
if ($_POST['step'] == 2) {
    // directly updates password, no re-check of CAPTCHA result
    update_password($_POST['password_new']);
}
```

**Fixed pattern:**
```php
if (isset($_POST['g-recaptcha-response'])) {
    $captcha_valid = verify_recaptcha($_POST['g-recaptcha-response']);
    if ($captcha_valid) {
        update_password($_POST['password_new']);
    } else {
        show_error("CAPTCHA verification failed.");
    }
} else {
    show_error("CAPTCHA response required.");
}
```

The key principle: CAPTCHA verification and the sensitive action it protects must happen atomically in the same server-side flow — never split across a multi-step process gated only by a client-controlled parameter.

## Defense in Depth

1. **Require current password for password changes** — Even with CAPTCHA fixed, a logged-in-session attacker (e.g., via XSS or session fixation) shouldn't be able to change a password without re-confirming the *current* password. This limits the blast radius of any session-based attack, CAPTCHA-related or not.
2. **CSRF protection on the password-change form** — Ensure a valid, single-use `user_token` is required and validated server-side for this specific action, closing the chaining risk noted in `findings.md` where this endpoint could be combined with a CSRF attack for full account takeover.
3. **Rate limiting / account lockout** — Even with CAPTCHA correctly enforced, add server-side rate limiting on sensitive account-mutation endpoints as a second layer, since CAPTCHA alone is a weak anti-automation control and is frequently bypassed in the wild (OCR solvers, CAPTCHA farms, logic flaws like this one).
4. **Notify users of credential changes** — Send an email/notification whenever a password change occurs, so the legitimate user has an out-of-band signal if their account was compromised, independent of whether the technical control held.
5. **Secure configuration management** — The missing reCAPTCHA keys observed here is itself a minor finding: a security control (CAPTCHA) was silently non-functional due to configuration, with no monitoring or startup check to flag that the control was disabled. Add a startup/health check that alerts if security-critical configuration values are unset.

## Verification
After remediation, re-send the payload documented in `commands.md` (`step=2` with no CAPTCHA response) and confirm the server rejects it with an explicit CAPTCHA-required error, rather than processing the password change. Additionally verify that submitting `step=2` with a *valid* CAPTCHA response, but for a *different* session than the one that solved it, is also rejected (protects against CAPTCHA response replay across sessions).
