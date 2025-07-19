# JWK Header Injection

JWTs can optionally include a JWK (JSON Web Key) directly inside the token header using the `jwk` parameter. This allows token issuers to specify the public key that should be used to verify the token. If a server accepts this key without validation, an attacker can forge arbitrary tokens signed with their own private key and bypass authentication.

---

## Vulnerability Details

According to the JWS specification, only the `alg` header parameter is mandatory. In practice, however, JWT headers (also known as JOSE headers) often contain several other parameters, such as jwk (JSON Web Key).

The `jwk` header field lets the sender embed a public key that should be used to verify the JWT. This is intended for advanced scenarios such as key rotation or federated identity. However, if the server naively trusts the embedded key **without validating its origin or issuer**, it opens the door to full token forgery.

This vulnerability was publicly disclosed as [CVE-2018-0114](https://nvd.nist.gov/vuln/detail/CVE-2018-0114) and affected the PyJWT library.

### JWK Header Example

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "prod_key",
  "jwk": {
    "kty": "RSA",
    "e": "AQAB",
    "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
  }
}
```

---

## Tools

- Burp Suite (JWT Editor extension)

---

## Exploitation Steps

#### Burp Suite (JWT Editor extension)
1. Go to the **JWT Editor Keys** tab and generate a **New RSA Key**.
   <img width="1029" height="654" alt="image" src="https://github.com/user-attachments/assets/9f142a7e-eb93-4d9c-b838-d7614f4cbaa9" />

2. Intercept a request and send it to Repeater.
3. In the **JSON Web Token** tab, modify the payload (e.g., `"role": "admin"`).
4. Click **Attack → Embedded JWK** and select your generated key.
   <img width="796" height="699" alt="image" src="https://github.com/user-attachments/assets/d393487a-4dd9-46b6-adea-f3e79a7f1f16" />

5. Send the token to the server and observe the response.

---

## Impact

- Unauthorized access
- Privilege escalation
- Account takeover

---

## Defense

- Never trust keys embedded in JWT headers
- Disable JWK header parsing unless explicitly required
- If JWKs are supported:
  - Validate issuer, key origin, and intended usage
  - Enforce `"use": "sig"` (not `"enc"`)

---

## Labs

- https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection
- https://pentesterlab.com/exercises/cve-2018-0114
