# Findings — Insecure CAPTCHA Lab

## Vulnerability
**Type:** Insecure CAPTCHA Implementation / Missing Server-Side Verification
**Location:** `/vulnerabilities/captcha/` (POST, `step` parameter)
**CWE:** CWE-799 — Improper Control of Interaction Frequency (related: CWE-287, Improper Authentication of the step-gated workflow)
**Severity:** High
**CVSS 3.1 (estimated):** 8.1 (AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:N)

## Description
DVWA's password-change form is intended to require a solved CAPTCHA (`step=1`) before the password change is committed (`step=2`). The server-side handler for `step=2` does not verify that the CAPTCHA was actually presented or solved in the current request chain — it only checks that `step` equals `2`. An authenticated attacker (or an attacker with a stolen/fixated session) can submit the `step=2` request directly, changing the account's password with zero CAPTCHA interaction.

## Pre-condition discovered during testing
The reCAPTCHA public/private keys were unset in `config.inc.php` by default, which disabled the CAPTCHA widget in the UI entirely:

```
$_DVWA[ 'recaptcha_public_key' ]  = '';
$_DVWA[ 'recaptcha_private_key' ] = '';
```

Google's public test keys were added to properly exercise the vulnerable flow rather than testing against a non-functional widget.

## Evidence

### 1. Exploit payload
```
POST /vulnerabilities/captcha/
step=2&password_new=hacked123&password_conf=hacked123&Change=Change
```
No `g-recaptcha-response` parameter included. No CAPTCHA challenge solved.

### 2. Impact confirmed via failed/successful login attempts

| Login attempt | Credentials | Result |
|---|---|---|
| Before exploit | `admin` / `password` | `302 -> index.php` (success) |
| After exploit | `admin` / `password` | `302 -> login.php` (**failed**) |
| After exploit | `admin` / `hacked123` | `302 -> index.php` (**success**) |

This is direct, unambiguous proof of impact: the live admin credential was mutated by a single unauthenticated-CAPTCHA request.

### 3. Recovery via the same bypass
```
POST /vulnerabilities/captcha/
step=2&password_new=password&password_conf=password&Change=Change
```
Result: `admin` / `password` login succeeded again (`302 -> index.php`), confirming the bypass is reliably repeatable, not a one-off artifact.

## Impact
- **Confidentiality:** None directly (this flaw doesn't leak data on its own).
- **Integrity:** High — an attacker can silently change another logged-in user's password, potentially locking the legitimate user out or, in a CSRF-chained scenario, taking over the account entirely.
- **Availability:** Moderate — legitimate users can be locked out of their own accounts (denial of service via credential overwrite).
- **Chaining potential:** Since this endpoint doesn't appear to require the original password to set a new one, and only checks session authentication, this is a strong candidate for chaining with CSRF (see Lab 03) — a victim could be tricked into submitting this exact request unknowingly via a malicious page, resulting in full account takeover without ever needing to steal a session cookie.

## Root Cause
Missing server-side state linking successful CAPTCHA validation to authorization for the subsequent action. The `step` parameter is trusted client input used as a workflow gate, with no cryptographic or session-bound proof that the prior step's control (CAPTCHA) was actually satisfied.
