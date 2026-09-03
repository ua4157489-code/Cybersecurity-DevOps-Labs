# Lab 06: Insecure CAPTCHA

## Overview
This lab demonstrates a logic flaw in DVWA's "Insecure CAPTCHA" password-change module. The application splits the password-change process into two steps (`step=1` submits the form with CAPTCHA, `step=2` processes the actual change) but fails to verify that CAPTCHA validation from step 1 actually occurred before executing step 2. An attacker can send `step=2` directly, bypassing CAPTCHA entirely, and change any authenticated user's password.

## Environment
- **Target:** DVWA (Damn Vulnerable Web Application) v1.10 *Development*
- **Deployment:** Docker (`vulnerables/web-dvwa`), mapped to `http://localhost:4280`
- **Security Level:** Low
- **Note:** reCAPTCHA public/private keys were initially blank in `config.inc.php`, which disabled the CAPTCHA widget entirely in the UI. Google's publicly documented test keys were added to the config to properly exercise the intended vulnerable flow (see `commands.md`).

## Objective
Demonstrate that DVWA's server-side logic for the password-change form can be tricked into skipping CAPTCHA verification by submitting `step=2` directly, without ever solving a CAPTCHA challenge.

## Summary of Impact
This vulnerability was validated in the most direct way possible: the bypass was used to change the live `admin` account password from `password` to `hacked123` without solving any CAPTCHA. This is a high-severity finding on any account whose credentials or session cookie an attacker can reach — CAPTCHA is meant to be an anti-automation control here, and it provides zero protection at Low security.

## Files
| File | Description |
|---|---|
| `commands.md` | Exact commands run, including reCAPTCHA config fix and the bypass itself |
| `methodology.md` | Step-by-step approach and reasoning |
| `findings.md` | Vulnerability details, evidence, and severity |
| `remediation.md` | Recommended fixes |
| `screenshots/` | Visual evidence (add browser screenshots of the bypass in action) |

## References
- [OWASP: Testing for CAPTCHA (OWASP-AT-012)](https://owasp.org/www-project-web-security-testing-guide/)
- [CWE-799: Improper Control of Interaction Frequency](https://cwe.mitre.org/data/definitions/799.html)
