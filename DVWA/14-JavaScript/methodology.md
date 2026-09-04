# Methodology — JavaScript Attacks Lab

## 1. Reconnaissance
Navigated to `/vulnerabilities/javascript/`, which presents a simple form: a "Phrase" text field, a hidden `token` field, and a submit button, with instructions to "Submit the word 'success' to win." Retrieved the full page source via `curl` rather than just viewing it in-browser, since the goal here is specifically to determine whether server-side validation exists independently of the client-side JavaScript.

## 2. Reading the obfuscated logic
The page source revealed:
1. A full, minified MD5 implementation (a well-known open-source library, unminified for readability during analysis but present as minified/obfuscated code in the actual page)
2. A small `rot13()` function performing a classic Caesar-cipher-style letter substitution
3. A `generate_token()` function that computes `md5(rot13(phrase))` and writes it into the hidden `token` field, called automatically on page load and (implicitly, in higher security levels not tested here) potentially on each keystroke

The presence of minified/obfuscated code is itself a signal worth noting: obfuscation is sometimes mistaken for a security control, but it only raises the effort required to read the logic — it does not prevent it, and browser DevTools trivially deobfuscate/format such code for readability.

## 3. Reimplementing the logic independently
Rather than only reading the code, the exact algorithm was reimplemented in Python, entirely outside the browser: ROT13 the target phrase (`success`), then MD5-hash the result. This produced a computed token (`38581812b435834ebf84ebcc2c6424d6`) without ever loading the DVWA page or executing any of its JavaScript.

## 4. Cross-validation against the live page's own logic
To rule out any implementation discrepancy (e.g., character encoding differences, a non-standard ROT13 variant, or an MD5 library quirk), the exact same computation was also run **inside the actual page's browser console**, directly invoking the live `md5()` and `rot13()` functions the page itself uses:
```javascript
md5(rot13("success"))
```
This returned the identical value, confirming the independent Python implementation was correct and not coincidentally matching.

## 5. Submitting the forged token without browser execution
The core proof of the vulnerability: submitted the phrase and computed token directly via `curl`, entirely bypassing the actual page and its JavaScript:
```bash
curl ... -d "phrase=success&token=38581812b435834ebf84ebcc2c6424d6&send=Submit"
```
The server accepted this and returned `"Well done"` — proving the server performs no validation of its own beyond checking that the submitted `token` matches `MD5(ROT13(phrase))`, a check that provides no actual security since the entire algorithm (and the ability to compute it) is available to anyone, independent of the browser.

## 6. Why this demonstrates a general principle, not just a DVWA quirk
This lab is less about a specific bug and more about a broader security principle: **any validation logic that runs entirely client-side can be read, understood, and replicated by an attacker who never runs the intended client at all.** The specific algorithm (MD5+ROT13) is almost incidental — the real lesson is that client-side JavaScript should be treated as fully attacker-visible and attacker-controllable, useful for improving user experience (e.g., instant feedback) but never as the sole enforcement point for anything security-relevant.
