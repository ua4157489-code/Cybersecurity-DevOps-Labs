# Remediation — XSS (Stored) Lab

## Primary Fix: Output encoding at render time — not just at submission time
The most robust fix treats **every** read from the database as untrusted, applying HTML encoding at the point of output regardless of when or how the data was originally written. This protects against the vulnerability even if sanitization on the write path is ever accidentally skipped, bypassed, or added to via a different code path later.

**Vulnerable pattern:**
```php
// Writing to DB — no sanitization
$name = $_POST['txtName'];
$message = $_POST['mtxMessage'];
$stmt = $db->prepare("INSERT INTO guestbook (name, message) VALUES (?, ?)");
$stmt->execute([$name, $message]);

// Reading from DB — no encoding
foreach ($comments as $comment) {
    echo "<div id='guestbook_comments'>Name: {$comment['name']}<br />Message: {$comment['message']}<br /></div>";
}
```

**Fixed pattern:**
```php
// Writing to DB — parameterized query prevents SQL injection (separate concern), input still stored as-is
$stmt = $db->prepare("INSERT INTO guestbook (name, message) VALUES (?, ?)");
$stmt->execute([$_POST['txtName'], $_POST['mtxMessage']]);

// Reading from DB — encode at output time, every time
foreach ($comments as $comment) {
    $safeName = htmlspecialchars($comment['name'], ENT_QUOTES, 'UTF-8');
    $safeMessage = htmlspecialchars($comment['message'], ENT_QUOTES, 'UTF-8');
    echo "<div id='guestbook_comments'>Name: {$safeName}<br />Message: {$safeMessage}<br /></div>";
}
```

## Defense in Depth

1. **Sanitize on input too, not just encode on output** — For stored content specifically, it's good practice to also validate/reject or strip dangerous content at submission time (e.g., reject any input containing `<script`, or use a proper HTML sanitization library like `HTMLPurifier` if any HTML formatting is meant to be allowed). This reduces the amount of dangerous content sitting in the database at all, limiting exposure if an output-encoding bug is ever introduced elsewhere (e.g., an admin panel or export feature that reads this same data without encoding).
2. **Enforce server-side length/content limits** — The `maxlength="10"` and `maxlength="50"` attributes observed in this lab are client-side HTML only and were trivially bypassed by submitting the POST request directly. Any such limits must also be enforced server-side.
3. **Content Security Policy (CSP)** — A strict CSP (`script-src 'self'`, no `unsafe-inline`) is especially valuable against stored XSS, since it provides protection against every future visitor even if a sanitization gap is later discovered in the codebase, without needing to immediately purge already-stored malicious data.
4. **Rate limiting / moderation on user-generated content** — Since stored XSS often arrives through public-facing submission forms (comments, guestbooks, reviews, forum posts), consider rate-limiting submissions and/or requiring moderation/approval before content is publicly rendered, as a practical mitigation against mass-injection attempts even after the core vulnerability is fixed.
5. **Periodic content auditing** — For any application with historical stored XSS risk, scan existing stored content for suspicious patterns (`<script`, `javascript:`, `onerror=`, etc.) as part of remediation, since fixing the code going forward does not retroactively clean already-poisoned data already sitting in the database.
6. **HttpOnly and Secure cookie flags** — As with reflected/DOM XSS, this limits the impact of any successful injection by preventing injected scripts from reading session cookies directly.

## Verification
After remediation, re-submit the payload documented in `commands.md` and confirm:
1. The guestbook page renders the literal text `<script>alert('Stored XSS')</script>` visibly, rather than executing it
2. This holds true both immediately after submission **and** when reloaded in a completely separate, fresh session
3. If input-side sanitization was also implemented, confirm the payload is rejected or stripped at submission time as an additional check, and audit existing guestbook entries for any previously-stored malicious content that predates the fix.
