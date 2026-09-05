# Remediation — OWASP A04: Insecure Design

## Finding 1: Inconsistent Quantity Validation Between Create and Update

**Fix:**
- Apply a single, shared validation function for basket item quantity across **every** endpoint that can set it (create, update, and any bulk/import paths), rather than relying on incidental database constraints:

```js
function validateQuantity(qty) {
  if (!Number.isInteger(qty) || qty <= 0) {
    throw new ValidationError('Quantity must be a positive integer');
  }
}
```

- Call this validator at the top of both the `POST` and `PUT` basket item handlers, before any database interaction, and return a clean `400 Bad Request` with a descriptive (but non-internal) error message on failure.
- Treat basket/order totals as **derived, re-verifiable state**, not client-trusted input. At minimum, recompute and re-validate the total server-side at checkout time (reject or clamp any basket where total < 0 or any line item quantity ≤ 0), rather than trusting whatever value was last written via any prior endpoint.
- Add a database-level `CHECK (quantity > 0)` constraint as defense-in-depth — but only as a backstop, never as the primary validation mechanism, and always paired with proper application-level error handling so a constraint violation never surfaces as a raw exception to the client.
- Add regression tests asserting that **every** mutating endpoint for a resource (not just the "obvious" one) rejects invalid values — the core lesson here is that testing create alone gave a false sense of security while update was completely open.

## Finding 2: Unhandled Exceptions Leak Internal Stack Traces

**Fix:**
- Wrap all database operations in try/catch (or use Express error-handling middleware) so unhandled exceptions never reach the client as raw stack traces.
- Return generic, generic-shaped error responses in production (`{"error": "Invalid request"}`), and log full stack traces server-side only.
- Disable framework/library default error pages (e.g. Express's default error handler) in production configuration — this is often a one-line `NODE_ENV=production` fix that most frameworks respect automatically, worth auditing as part of the A05 (Security Misconfiguration) review too.

## General Principle
This finding is a textbook illustration of **insecure design versus insecure implementation**: no single line of code is "wrong" in isolation — the create endpoint even appears protected. The flaw is architectural: input validation was never designed as a shared, resource-wide concern, so it silently varies by endpoint. Fixing this requires a design-level decision (validate once, consistently, for every mutation path and re-verify at the point of financial impact) rather than a local patch to one route.

**Recommended controls:**
- Threat-model each resource's full CRUD surface during design, not just the "primary" create/read paths
- Server-side recomputation of any value with financial impact (totals, balances, prices) at the point of commitment (checkout/payment), independent of prior client-influenced state
- Centralized validation middleware/schema (e.g. JSON schema or a validation library) applied uniformly across related endpoints
- Production error handling audit to ensure no endpoint leaks stack traces, file paths, or library/version details
