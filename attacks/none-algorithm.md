# None Algorithm (alg: none)

Some applications accept JWTs with `alg: none`, meaning no signature is required to verify the token. If improperly implemented, an attacker can craft an unsigned token and bypass authentication or authorization controls entirely.

---

## Vulnerability Details

JWTs include an `alg` field in the header to specify the signing algorithm. The special value `none` indicates that the token is **not signed**. Most libraries reject this by default, but some applications may:
- Accept `alg: none` due to misconfiguration
- Fail to validate the `alg` field at all
- Be vulnerable to obfuscated values like `None` or mixed casing

If signature verification is skipped based on this value, the attacker can supply any payload they want and still pass authentication.

### Vulnerable Example

```json
{ "alg": "none", "typ": "JWT" }
```

---

## Tools

- Burp Suite (JWT Editor extension)
- jwt_tool

---

## Exploitation Steps

#### Burp Suite (JWT Editor extension)
1. Intercept a request containing a JWT token.
2. Modify the header to use `"alg": "none"` and try other variants as well:
   - `"alg": "None"`
   - `"alg": "NONE"`
   - `"alg": "nOnE"`
  <img width="796" height="690" alt="image" src="https://github.com/user-attachments/assets/d8fc43c7-97d8-4bb5-9bb8-31cb1fc40d12" />

3. Forward the modified request and observe the response.

#### jwt_tool
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

- Explicitly disable the `none` algorithm in your JWT library configuration
- Validate the algorithm server-side using strict whitelist (e.g., always enforce HS256 or RS256)

---

## Labs

- https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification
- https://pentesterlab.com/exercises/jwt
