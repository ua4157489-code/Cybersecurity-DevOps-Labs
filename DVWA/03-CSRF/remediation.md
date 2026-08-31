# Remediation – DVWA CSRF

## 1. Overview

The identified Cross-Site Request Forgery (CSRF) vulnerability occurs because a sensitive password-change operation can be submitted without an observed dedicated CSRF token.

The password-change functionality also uses a `GET` request for a state-changing operation.

The primary remediation objective is to ensure that sensitive requests can only be executed when they are intentionally initiated by an authenticated user.

---

## 2. Implement Anti-CSRF Tokens

The application should generate a cryptographically random CSRF token for each authenticated session.

Example:

```text
csrf_token = <random-unpredictable-value>
```

The token should be included with every sensitive state-changing request.

Example:

```text
POST /DVWA/vulnerabilities/csrf/

password_new=<value>
password_conf=<value>
csrf_token=<token>
```

The server must validate the token before performing the password change.

---

## 3. Validate the Token Server-Side

CSRF protection must be enforced on the server.

The application should:

```text
Receive Request
      ↓
Retrieve Session Token
      ↓
Compare Submitted Token
      ↓
Token Valid?
   ↙       ↘
 YES        NO
 ↓           ↓
Process     Reject
Request     Request
```

A missing, invalid, or expired token should result in the request being rejected.

---

## 4. Use POST for Password Changes

The password-change operation should not use `GET`.

Current design:

```text
GET /DVWA/vulnerabilities/csrf/
```

Recommended design:

```text
POST /DVWA/vulnerabilities/csrf/
```

Sensitive state-changing operations should use an appropriate state-changing HTTP method.

---

## 5. Do Not Place Passwords in URLs

The tested request exposed password parameters in the URL:

```text
password_new=<value>
password_conf=<value>
```

Sensitive information should not be placed in URLs because URLs may be recorded in:

* Browser history
* Proxy logs
* Web server logs
* Monitoring systems
* Referer information
* Other infrastructure components

The password should instead be submitted in the HTTPS request body.

---

## 6. Apply Secure Cookie Attributes

Authentication cookies should use appropriate security attributes.

Recommended attributes include:

```text
Secure
HttpOnly
SameSite
```

Example:

```text
Set-Cookie: PHPSESSID=<session>;
Secure;
HttpOnly;
SameSite=Lax
```

For highly sensitive applications, the appropriate `SameSite` policy should be selected based on application requirements.

Cookie protections should supplement CSRF tokens rather than replace them.

---

## 7. Consider Origin Validation

For sensitive operations, the server can validate the `Origin` header.

Expected trusted origin:

```text
http://localhost:8080
```

In a production environment, this would be the application's legitimate HTTPS origin.

Unexpected origins should be rejected when appropriate.

Origin validation should be implemented carefully because legitimate application architectures may involve multiple trusted origins.

---

## 8. Referer Validation as Defense in Depth

The application may also inspect the `Referer` header as an additional security control.

However, Referer validation should not be treated as the primary CSRF defense.

The preferred control remains:

```text
CSRF Token
+
Server-Side Validation
```

---

## 9. Re-Authentication for Sensitive Actions

Password changes are highly sensitive operations.

For important applications, consider requiring the user to re-enter their current password or complete another strong authentication step before changing the password.

Example:

```text
Authenticated Session
        ↓
Re-authentication
        ↓
CSRF Token Validation
        ↓
Password Change
```

This provides additional protection against unauthorized account changes.

---

## 10. Strong Password Handling

Passwords should be handled securely by the application.

The application should:

* Never store plaintext passwords.
* Use a strong password-hashing algorithm.
* Use unique salts.
* Enforce appropriate password policies.
* Avoid logging passwords.
* Avoid exposing passwords in URLs.
* Protect password-change requests with HTTPS.

---

## 11. Defense in Depth

CSRF protection should be combined with additional security controls:

* HTTPS
* Secure cookies
* HttpOnly cookies
* SameSite cookies
* Session management
* Authentication controls
* Authorization checks
* Rate limiting
* Security logging
* Monitoring
* Re-authentication for sensitive operations

No single control should be treated as a complete replacement for secure application design.

---

## 12. Recommended Secure Request Flow

A secure password-change implementation should follow a flow similar to:

```text
User
 ↓
Authenticated Session
 ↓
Password Change Form
 ↓
CSRF Token Generated
 ↓
POST Request
 ↓
HTTPS
 ↓
Server Validates Session
 ↓
Server Validates CSRF Token
 ↓
Server Validates Password
 ↓
Password Hashing
 ↓
Password Updated
 ↓
Security Event Logged
```

---

## 13. Example Secure Design

A simplified secure request could look like:

```text
POST /DVWA/vulnerabilities/csrf/ HTTP/1.1
Host: example.com
Cookie: PHPSESSID=<session>
Content-Type: application/x-www-form-urlencoded

password_new=<new-password>&
password_conf=<new-password>&
csrf_token=<valid-token>
```

The server should verify:

```text
1. Session is valid
2. User is authorized
3. CSRF token exists
4. CSRF token matches the user's session
5. Password values meet validation requirements
6. Request is received over HTTPS
```

Only after successful validation should the password be changed.

---

## 14. Input Validation

Password fields should also be validated server-side.

The application should verify:

```text
password_new
password_conf
```

and ensure:

```text
password_new == password_conf
```

The application should enforce an appropriate password policy without exposing sensitive values in logs or error messages.

---

## 15. Logging and Monitoring

Security-relevant password changes should generate appropriate audit events.

Potential log information:

```text
Timestamp
User/account identifier
Source IP
Action performed
Success/Failure
Authentication context
```

Passwords and CSRF tokens should never be written to logs.

SOC/SIEM monitoring can detect:

* Repeated password-change attempts
* Unusual source locations
* Abnormal account activity
* Multiple password changes
* Suspicious authentication events

---

## 16. Verification After Remediation

After implementing the fixes, the application should be retested.

### Test 1 – Valid CSRF Token

```text
Authenticated session
+
Valid CSRF token
+
POST request
```

Expected:

```text
Password change succeeds
```

---

### Test 2 – Missing CSRF Token

```text
Authenticated session
+
No CSRF token
+
POST request
```

Expected:

```text
Request rejected
```

---

### Test 3 – Invalid CSRF Token

```text
Authenticated session
+
Invalid CSRF token
+
POST request
```

Expected:

```text
Request rejected
```

---

### Test 4 – Reused Token

Attempt to reuse an expired or invalid token where the application's token strategy requires expiration or rotation.

Expected:

```text
Request rejected
```

---

### Test 5 – Cross-Origin Request

Attempt a request originating from an unauthorized origin.

Expected:

```text
Request rejected
```

The exact behavior should depend on the application's legitimate cross-origin requirements.

---

## 17. Security Requirements

The final implementation should satisfy:

```text
┌───────────────────────────────┐
│ Sensitive Password Operation  │
└───────────────┬───────────────┘
                ↓
       Use POST / HTTPS
                ↓
       Validate Session
                ↓
       Validate CSRF Token
                ↓
       Validate Input
                ↓
       Re-authenticate if needed
                ↓
       Update Password Securely
                ↓
       Log Security Event
```

---

## 18. Risk Reduction

Implementing the recommended controls reduces the likelihood of unauthorized password changes caused by forged requests.

The most important controls are:

1. **CSRF token validation**
2. **Server-side validation**
3. **POST for state-changing operations**
4. **HTTPS**
5. **Secure cookie configuration**
6. **Re-authentication for sensitive actions**

---

## 19. Final Recommendation

The DVWA vulnerability exists intentionally for security training. In a production application, the password-change functionality should be redesigned so that authentication cookies alone are not sufficient to authorize a sensitive state-changing request.

The recommended minimum remediation is:

```text
CSRF Token
+
Server-Side Validation
+
POST
+
HTTPS
+
Secure Cookie Attributes
```

Additional controls such as re-authentication, Origin validation, monitoring, and rate limiting should be applied according to the application's risk profile.

---

## 20. Conclusion

The CSRF weakness can be mitigated through a combination of secure request design, server-side CSRF token validation, secure session management, and appropriate authentication controls.

After remediation, requests without a valid CSRF token should be rejected and sensitive password information should no longer be transmitted through URLs.

**Remediation Status:** Recommended

**Target Security State:**

```text
Authenticated User
        ↓
Protected POST Request
        ↓
Valid CSRF Token
        ↓
Server-Side Validation
        ↓
Authorized Password Change
```
