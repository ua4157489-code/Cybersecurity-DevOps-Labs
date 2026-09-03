# Methodology — SQL Injection (Blind) Lab

## 1. Reconnaissance
Navigated to `/vulnerabilities/sqli_blind/`, which presents the same "User ID" input pattern as the regular SQLi lab, but critically the response only ever shows one of two states: **"User ID exists in the database"** or nothing at all. No first name, surname, or any query data is echoed back — this is the defining characteristic of a blind injection point, and it changes the exploitation approach entirely: instead of reading data directly, we have to *infer* it by asking the database true/false questions and observing which produces the "exists" message.

## 2. Confirming the injection point (boolean-based)
Sent two contrasting payloads to check whether the "exists" signal responds to injected SQL logic rather than just the literal `id` value:

- **TRUE case:** `1' AND '1'='1` — should behave identically to a valid ID, since `'1'='1'` is always true
- **FALSE case:** `1' AND '1'='2` — should suppress the "exists" message, since `'1'='2'` is always false

Both behaved exactly as predicted: TRUE preserved the "exists" message, FALSE suppressed it entirely. This confirms the `id` parameter is vulnerable to injection and that we have a reliable binary oracle to build extraction on.

## 3. Establishing an extraction primitive
Blind SQLi extraction relies on converting "is this specific fact about the data true or false?" into the same oracle confirmed in step 2. Two building blocks were used:

- **`LENGTH(expr) = N`** — determines how many characters a target string is, by brute-forcing `N` until the TRUE condition fires
- **`ASCII(SUBSTRING(expr, pos, 1)) = N`** — determines the ASCII code of the character at a specific position, by brute-forcing `N` until the TRUE condition fires

Each of these was wrapped in the same `AND` pattern confirmed in step 2, e.g.:
```
1' AND LENGTH(database())=4-- -
```

## 4. Automating extraction
Manually testing each character would be impractical, so the extraction was scripted in bash: nested loops iterate every candidate length (for the length-detection phase) or every printable ASCII value (for character-detection), firing one `curl` request per guess and checking the response for the "exists" signal via `grep`. When a match is found, the loop breaks and moves to the next position.

This was applied first to `database()` — a low-risk, quick target to validate the automation worked correctly — before moving to the higher-value target: the `admin` account's password hash from the `users` table.

## 5. Optimizing the character-extraction search space
For the database name, the full printable ASCII range (32–126) was searched per position, since the name could theoretically contain any character. For the password hash, the search space was narrowed to ASCII values 48–102 (covering `0`–`9` and `a`–`f`), since MD5 hashes are always lowercase hexadecimal — this significantly reduced the number of requests needed per character without any loss of correctness.

## 6. Cross-validation
The final extracted password hash (`5f4dcc3b5aa765d61d8327deb882cf99`) was compared directly against the hash obtained in Lab 07 via UNION-based (non-blind) injection against the same `users` table. The two values matched exactly, which serves as strong independent confirmation that the blind extraction technique is sound and produces accurate results — not just plausible-looking noise.
