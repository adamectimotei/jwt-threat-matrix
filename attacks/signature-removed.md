# Signature Removed

## 🧠 Description
Some applications decode JWTs without verifying their signature, usually due to incorrect use of JWT libraries. This allows attackers to modify the token payload and bypass authentication or authorization controls.

---

## 🔍 Vulnerability Details
Most JWT libraries offer two separate functions:
- `verify()` – which validates the token’s signature
- `decode()` – which only decodes the payload without verifying the signature

In Node.js, the `jsonwebtoken` library is a common example where this confusion happens. If developers use `decode()` instead of `verify()`, **any token with a valid-looking structure will be accepted**, even if the signature is invalid or removed entirely.

### ❌ Vulnerable Code Example
```javascript
const jwt = require('jsonwebtoken');
const token = req.headers.authorization;
const user = jwt.decode(token);  // BAD: no signature verification!
```

### 🔁 Example of Exploitation
Original payload:
```json
{ "username": "carlos", "isAdmin": false }
```

Modified payload:
```json
{ "username": "carlos", "isAdmin": true }
```

---

## 🛠 Exploitation Steps

1. Obtain a valid JWT (by logging in as a normal user).
2. Base64URL-decode the token to extract header and payload.
3. Modify the payload values.
    - e.g., change `"role": "user"` to `"role": "admin"`
4. Base64URL-encode the modified header and payload.
5. Reassemble token:
    - Either reuse original signature: `header.payload.signature`
    - Or omit the signature completely: `header.payload.`
6. Send the modified JWT in a request header:
    - `Authorization: Bearer <forged_token>`

---

## 🧪 Tools

### 🔧 Burp Suite (JWT Editor extension)
1. Intercept a request containing a JWT.
2. Switch to the "JSON Web Token" tab.
3. Modify the payload (e.g., escalate privileges).
4. Optionally remove the signature or keep it.
5. Forward the modified request and observe the response.

---

## 🛡️ Defense

- Always use `verify()` (not just `decode()`) in JWT libraries.
- Reject unsigned tokens or those with malformed signatures.
- Implement strict JWT validation middleware.
- Monitor for anomalies in token structure or usage (e.g., missing signature parts).
