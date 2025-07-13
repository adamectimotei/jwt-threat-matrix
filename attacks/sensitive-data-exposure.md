# Sensitive Data Exposure

Some applications include sensitive information directly in the JWT payload (e.g., email, internal user IDs, access tokens). Because JWTs are only base64url-encoded (not encrypted), anyone with access to the token can read its contents.

---

## Vulnerability Details

Although signed, the JWT payload is not encrypted. If it contains sensitive or secret information, any party with access to the token can decode and extract the data without needing to tamper with it.

### Vulnerable Example

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "accessToken": "eyJhbGciOi..."
}
```

---

## Tools

- jwt.io
- Burp Suite (JWT Editor)
- jwt_tool

---

## Exploitation Steps

#### Manual Decoding
1. Copy the JWT token from a request.
2. Paste it into [jwt.io](https://jwt.io) or use a local tool.
3. Inspect the decoded contents for sensitive or unexpected data.

#### Burp Suite (JWT Editor)
1. Intercept a request containing a JWT.
2. Switch to the “JSON Web Token” tab.
3. Inspect the decoded contents for sensitive or unexpected data.

#### jwt_tool
```bash
python3 jwt_tool.py <JWT_TOKEN> -d
```

---

## Impact

- Information disclosure (e.g., email, session data)
- Unauthorized data access

---

## Defense

- Never store sensitive information in the JWT payload
