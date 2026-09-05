# Findings — OWASP A04: Insecure Design

## Finding 1: Inconsistent Quantity Validation Between Create and Update (Basket Items)

**Severity:** Critical
**Endpoint:** `POST /api/BasketItems` (create) vs `PUT /api/BasketItems/:id` (update)
**Status:** Vulnerable

### Description
The basket item creation endpoint (`POST /api/BasketItems`) rejects quantities of `0` or below — but not cleanly. Rather than returning a validation error, it throws an unhandled exception at the database layer (a `500` with a full Sequelize/SQLite stack trace leaked to the client), suggesting there is **no application-level input validation at all** — the rejection is an accidental side effect of a database constraint, not a designed control.

More critically, the corresponding **update endpoint (`PUT /api/BasketItems/:id`) enforces no such constraint**. An item legitimately added with a positive quantity can subsequently be updated to a negative quantity with a clean `200 OK` response.

### Reproduction
1. Add a basket item normally (`POST /api/BasketItems`, quantity: 1) → succeeds, returns item id.
2. Update that same item via `PUT /api/BasketItems/:id` with `quantity: -5` → succeeds (`200`), negative quantity persisted.

### Impact
The negative quantity is not merely stored — it is used in the real basket total calculation:

| Product | Qty | Price | Line Total |
|---|---|---|---|
| Apple Juice (1000ml) | -5 | $1.99 | -$9.95 |
| Orange Juice (1000ml) | 2 | $2.99 | $5.98 |
| **Basket Total** | | | **-$3.97** |

This basket was then submitted through `POST /rest/basket/:id/checkout` and **accepted with a `200 OK`**, producing a real, confirmed order (`orderConfirmation: 4b0f-bcae0efeb01e4d80`) — with **no address, payment method, or delivery selection required at any point**.

The negative total was confirmed to persist on the permanent order record via `GET /rest/track-order/:id`:
```json
"totalPrice": -3.969999999999999,
"paymentId": null,
"addressId": null
```

An attacker can therefore generate a confirmed, trackable order with a negative total and no payment collected, using only basket manipulation — no checkout, payment, or address flow bypass required beyond the single `PUT` request.

### Evidence
- `screenshots/01-negative-quantity-put-accepted.png` — `PUT` request accepting `quantity: -5`
- `screenshots/02-checkout-negative-total-succeeds.png` — checkout completing with `200` and an order confirmation
- `screenshots/03-order-record-negative-total-tracked.png` — persisted order showing `totalPrice: -3.97`, `paymentId: null`

## Finding 2 (Secondary): Unhandled Exceptions Leak Internal Stack Traces

**Severity:** Medium
**Endpoint:** `POST /api/BasketItems` (quantity ≤ 0)
**Status:** Vulnerable

### Description
Invalid quantity values on the create path cause an unhandled database exception to be returned directly to the client as raw HTML, including internal file paths (`/juice-shop/build/routes/basketItems.js:79`), the ORM in use (Sequelize), and the database dialect (SQLite). This does not itself grant access to data, but assists an attacker in fingerprinting the technology stack and locating other exploitable code paths — and it's a symptom of the same root problem as Finding 1: no input validation layer before the database is reached.

### Impact
Information disclosure; contributes to attacker reconnaissance. Overlaps conceptually with A05:2021 (Security Misconfiguration) but is raised here since it was discovered while testing input validation gaps.

## Not Pursued This Session
- Coupon/discount logic abuse (candidate for a follow-up A04 test)
- Password-reset security-question weakness (candidate for a follow-up A04 test)
