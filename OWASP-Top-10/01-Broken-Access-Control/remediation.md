# Remediation — OWASP A01: Broken Access Control

## Finding 1: Basket Read IDOR

**Fix:** Enforce object-level authorization on every resource-fetching endpoint. Before returning basket data, the server must verify that the `UserId` associated with the requested `:id` matches the `id`/`bid` claim in the authenticated user's JWT. Example (pseudocode):

```js
const basket = await Basket.findByPk(req.params.id)
if (basket.UserId !== req.user.data.id) {
  return res.status(403).json({ error: 'Forbidden' })
}
```

**General principle:** Never trust a resource identifier supplied in the URL/path alone — always re-derive "which resources this user may access" server-side from the authenticated session, not from client input.

## Finding 2: Inconsistent Enforcement Across Endpoints

**Fix:** Centralize authorization logic (e.g., a shared middleware or policy layer applied to all routes touching a given resource type) rather than re-implementing ownership checks ad hoc per endpoint. This prevents the scenario observed here, where the write endpoint enforces ownership but the read endpoint on the same resource does not.

**Recommended controls:**
- Adopt a consistent authorization middleware pattern (e.g., attribute-based or role-based access control checked at a single choke point)
- Add automated tests that specifically assert cross-user access is denied for every CRUD operation on every resource type
- Conduct regular access-control-focused code review/audits, since BAC bugs are logic errors that static analysis tools often miss
- Apply the principle of least privilege and "deny by default" — access should be explicitly granted, not implicitly allowed by omission
