# Findings — OWASP A01: Broken Access Control (Juice Shop)

## Finding 1: IDOR on Basket Read Endpoint (Horizontal Privilege Escalation)

**Endpoint:** `GET /rest/basket/:id`
**Severity:** High
**Attacker context:** User `attacker@test.com` (id=26, own basket id=7, role=customer)

### Steps to Reproduce
1. Register and log in as `attacker@test.com`, capture JWT
2. Confirm own basket baseline: `GET /rest/basket/7` → correctly returns empty basket, `UserId:26`
3. Request unrelated basket IDs with the same token:
   - `GET /rest/basket/1` → returns full basket for `UserId:1` (Apple Juice, Orange Juice, Eggfruit Juice)
   - `GET /rest/basket/2` → returns full basket for `UserId:2` (Raspberry Juice)

### Evidence
- `raw-output/basket_own_7.json` — attacker's real basket (empty, UserId:26)
- `raw-output/basket_idor_2.json` — UserId 2's basket returned to attacker's token
- `screenshots/01-basket-idor-cross-user-read.png`

### Impact
Any authenticated user can enumerate `/rest/basket/:id` and read other users' basket contents (products, quantities, timestamps). Confidentiality violation; enables profiling of other users' shopping behavior at scale via ID enumeration.

### Root Cause
`GET /rest/basket/:id` checks only that a valid JWT is present — it never verifies the token's `UserId` matches the basket's `UserId` owner field.

---

## Finding 2 (Control/Negative Result): Write Path Correctly Enforces Ownership

**Endpoint:** `POST /api/BasketItems`

### Steps to Reproduce
1. Using attacker's token, attempt to add a product into another user's basket: `{"ProductId":1,"BasketId":2,"quantity":5}`
2. Server responds: `{'error' : 'Invalid BasketId'}` — write rejected
3. Confirmed same token successfully writes to own basket (`BasketId:7`) — `raw-output/basket_own_write_success.json`

### Evidence
- `raw-output/basket_idor_2_write_attempt.json`
- `screenshots/02-basket-write-idor-blocked.png`

### Significance
Unlike the read endpoint, this write endpoint enforces basket ownership correctly. This asymmetry is a common real-world BAC root cause: each route re-implements its own authorization logic instead of relying on a shared, centrally enforced ownership check, so some endpoints on the same resource end up protected and others don't.
