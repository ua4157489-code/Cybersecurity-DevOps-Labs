# Remediation — XSS (DOM) Lab

## Primary Fix: Avoid unsafe DOM sinks; use safe DOM APIs instead
The root cause is the use of `document.write()` (and string concatenation) to inject data derived from an untrusted source (`document.location.href`) into the page. Replace this pattern with safe DOM construction methods that treat data as text, never as markup.

**Vulnerable pattern:**
```javascript
var lang = document.location.href.substring(document.location.href.indexOf("default=")+8);
document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
```

**Fixed pattern (using safe DOM APIs):**
```javascript
const params = new URLSearchParams(window.location.search);
const lang = params.get('default');

if (lang) {
  const option = document.createElement('option');
  option.value = lang;
  option.textContent = lang; // textContent, NOT innerHTML — never interpreted as markup
  document.querySelector('select[name="default"]').appendChild(option);
}
```

Key changes:
- `URLSearchParams` for safe, correct URL parameter parsing instead of manual string slicing
- `document.createElement()` + `textContent` instead of `document.write()` + string concatenation — `textContent` never parses its input as HTML, so even a payload containing `<script>` tags is rendered as inert text, not executed

## Defense in Depth

1. **Allow-list validation** — Since this field is meant to hold a small, known set of language codes, validate `lang` against an explicit allow-list (`['English','French','Spanish','German']`) before using it at all, rather than trusting arbitrary URL content even after switching to safe DOM APIs. This adds a second independent layer of protection.
2. **Content Security Policy (CSP)** — A strict CSP (e.g., `script-src 'self'`, disallowing inline scripts and `unsafe-eval`) would prevent injected `<script>` tags from executing even if the DOM-write vulnerability were still present, since inline scripts inserted via `document.write()` would be blocked by the browser. DVWA's own "CSP Bypass" module (visible in the sidebar) explores this control directly.
3. **Avoid `document.write()` entirely** — It's a legacy API with well-known security and performance issues; modern applications should use DOM manipulation methods (`createElement`, `textContent`, `setAttribute`) or a framework's built-in escaping (e.g., React's JSX, which escapes by default) instead.
4. **HttpOnly cookies** — While this doesn't prevent DOM XSS itself, marking session cookies `HttpOnly` prevents `document.cookie` from being read by injected JavaScript, significantly reducing the impact of any XSS that does occur (defense in depth, not a substitute for fixing the injection).
5. **Security awareness for URL-derived data** — Any client-side code that reads from `location.href`, `location.search`, `location.hash`, `document.referrer`, `window.name`, or similar browser-provided-but-attacker-influenceable sources should be treated with the same suspicion as server-side user input — a common blind spot is assuming "the server didn't send this to me, so it's not user input," which this lab directly disproves.

## Verification
After remediation, re-send the payload documented in `commands.md` and confirm the browser renders the literal text `<script>alert(document.domain)</script>` visibly in the dropdown (as inert text) rather than executing it. Additionally test with HTML-entity-encoded and double-URL-encoded variants of the payload to ensure the fix isn't bypassable through encoding tricks.
