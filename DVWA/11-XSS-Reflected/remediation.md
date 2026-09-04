# Remediation — XSS (Reflected) Lab

## Primary Fix: Output encoding on every dynamic value rendered into HTML
The fix is to HTML-encode user-controlled data at the point it's written into the response, converting HTML-significant characters into their safe entity equivalents so browsers render them as literal text instead of parsing them as markup.

**Vulnerable pattern:**
```php
echo '<pre>Hello ' . $_GET['name'] . '</pre>';
```

**Fixed pattern:**
```php
echo '<pre>Hello ' . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8') . '</pre>';
```

`htmlspecialchars()` converts `<`, `>`, `&`, `"`, and (with `ENT_QUOTES`) `'` into their HTML entity equivalents (`&lt;`, `&gt;`, `&amp;`, `&quot;`, `&#039;`), so a payload like `<script>alert('XSS')</script>` renders as visible, inert text — `<script>alert('XSS')</script>` displayed on the page — rather than being parsed and executed as a script tag.

## Defense in Depth

1. **Context-aware encoding** — `htmlspecialchars()` is correct for data placed in HTML body content (as here). If user input were instead placed inside an HTML attribute, a `<script>` block, a URL, or a CSS context, a different, context-specific encoding function is required — using the wrong encoding for the context is a common source of residual XSS even after a "fix" is applied.
2. **Input validation as a secondary layer** — Since a "name" field has no legitimate reason to contain `<`, `>`, or other HTML-significant characters, consider validating/restricting input format server-side (e.g., allow-listing expected characters) in addition to output encoding — defense in depth, not a replacement for it.
3. **Content Security Policy (CSP)** — A strict CSP (`script-src 'self'`, no `unsafe-inline`) would prevent injected inline `<script>` tags from executing even if output encoding were somehow bypassed, providing a second independent layer of protection.
4. **HttpOnly and Secure cookie flags** — Marking session cookies `HttpOnly` prevents any successfully injected script from reading `document.cookie`, meaningfully limiting the impact of session-hijacking attempts even if an XSS vulnerability exists elsewhere.
5. **Use a templating engine with auto-escaping** — Modern frameworks (Twig, Blade, React/JSX, etc.) HTML-escape output by default, making this class of vulnerability much harder to introduce accidentally compared to raw string concatenation as seen in DVWA's vulnerable code.
6. **Automated testing** — Add XSS-focused DAST scanning (e.g., automated payload fuzzing with tools like `sqlmap`'s XSS-focused peers, OWASP ZAP, or Burp Suite's active scanner) to CI/CD or periodic security testing to catch regressions where output encoding is accidentally omitted on new endpoints.

## Verification
After remediation, re-send the payload documented in `commands.md` and confirm the server response now contains the HTML-encoded form of the payload (`&lt;script&gt;alert(&#039;XSS&#039;)&lt;/script&gt;`) rather than a literal `<script>` tag. Load the resulting page in a browser and confirm the text `<script>alert('XSS')</script>` is displayed visibly on the page as plain text, with no alert box firing.
