# Unverified Signature

Some applications decode JWTs without verifying their signature, usually due to incorrect use of JWT libraries. This allows attackers to modify the token payload and bypass authentication or authorization controls.

---

## Vulnerability Details
Most JWT libraries offer two separate functions:
- `verify()` – which validates the token’s signature
- `decode()` – which only decodes the payload without verifying the signature

In Node.js, the `jsonwebtoken` library is a common example where this confusion happens. If developers use `decode()` instead of `verify()`, **any token with a valid-looking structure will be accepted**, even if the signature is invalid or removed entirely.

Developers might also disable signature verification for testing and then forget to re-enable it.

### Vulnerable Code Example
```javascript
const jwt = require('jsonwebtoken');
const token = req.headers.authorization;
const user = jwt.decode(token);  // no signature verification!
```

---

## Tools

- Burp Suite (JWT Editor extension)
- jwt_tool

---

## Exploitation Steps

#### Burp Suite (JWT Editor extension)
1. Intercept a request containing a JWT token.
2. Switch to the "JSON Web Token" tab.
3. Modify the payload (e.g., change `"role": "user"` to `"role": "admin"`).
4. First, try keeping the original signature (`header.payload.signature`).
5. If that fails, try removing the signature entirely (`header.payload.`).
6. Forward the modified request and observe the response.

#### jwt_tool
Removing the signature entirely:
```bash
python3 jwt_tool.py <JWT_TOKEN> -X a
```

---

## Impact

This issue effectively renders JWT-based authentication useless.
- Unauthorized access
- Privilege escalation
- Account takeover

---

## Defense

- Always use `verify()` (not just `decode()`) in JWT libraries.
- Reject unsigned tokens or those with malformed signatures.
- Implement strict JWT validation middleware.

---

## Labs
- https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature
- https://pentesterlab.com/exercises/jwt-vii
