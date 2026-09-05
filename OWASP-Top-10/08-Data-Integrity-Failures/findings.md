# Findings — OWASP A08: Software and Data Integrity Failures

## Finding 1: Unauthenticated / Unauthorized Product Price Tampering (Critical)

**Endpoint:** `PUT /api/Products/:id`
**Severity:** Critical
**Access required:** None (unauthenticated requests succeed)

### Steps to Reproduce
1. Capture baseline product data: `GET /api/Products/1` → Apple Juice, price `$1.99`
2. Send a price-overwrite request using a valid **customer**-role token:
   `PUT /api/Products/1` with body `{"price":0.01}` → `200 OK`, price updated
3. Confirm the tampered price is live in the **public, unauthenticated** storefront search:
   `GET /rest/products/search?q=apple` → returns `$0.01`
4. Repeat the tamper on a second product with **no `Authorization` header at all**:
   `PUT /api/Products/2` with body `{"price":0.01}` → `200 OK`, price updated
5. Confirm this second, fully unauthenticated tamper is also publicly visible via search

### Evidence
- `raw-output/product_1_tamper_customer_token.json` — 200 OK with customer token
- `raw-output/product_2_tamper_no_auth.json` — 200 OK with **no auth header**
- `raw-output/search_apple_tampered.json`, `raw-output/search_orange_tampered.json` — tampered prices confirmed live in public search
- `screenshots/01-product-price-tampered-customer-token.png`
- `screenshots/02-tampered-price-visible-in-public-search.png`
- `screenshots/03-price-tamper-with-zero-authentication.png`

### Impact
Any party — including one with zero credentials — can rewrite core business data (product pricing) storefront-wide. This is a complete data integrity failure: the application accepts and persists writes to authoritative business records without verifying the source is trusted or authorized to make that change. Real-world impact would include financial loss (products sold below cost), reputational damage, and loss of trust in displayed pricing data.

### Root Cause
`PUT /api/Products/:id` is exposed with no authentication check and no role/authorization check. This endpoint pattern is consistent with an auto-generated REST scaffold (e.g. a library like `finale-rest`, seen in this app's dependency manifest during the A06 lab) that defaults to open CRUD access unless explicitly restricted per-route by the developer.

### Severity Justification
Rated **Critical** rather than High because:
- No authentication of any kind is required (worse than a broken authorization check on an authenticated endpoint)
- The affected data (pricing) is core, customer-facing business data with direct financial impact
- The tampered state is immediately and publicly visible with no additional step required
