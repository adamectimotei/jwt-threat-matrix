# JKU Header Injection

Some applications support the `jku` (JWK Set URL) JWT header parameter to dynamically fetch a key set from a remote URL. If not properly validated, this behavior allows an attacker to host their own key and trick the server into accepting it for signature verification.

---

## Vulnerability Details

According to the JWS specification, only the `alg` header parameter is mandatory. In practice, however, JWT headers (also known as JOSE headers) often contain several other parameters, such as jku (JWK Set Url).

### JKU Header Example

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
  "jku": "https://target.com/.well-known/jwks.json"
}
```

The `jku` parameter allows a token to reference an external JWK Set (usually hosted at `/.well-known/jwks.json`). If the server accepts any arbitrary URL and doesn't validate its origin or content, it can be tricked into trusting a malicious key hosted by an attacker.

### JWK Set Example

```json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```

In a usual exploit scenario, the attacker:
- Generates their own RSA key pair
- Hosts the public key in a valid JWK Set at a server they control
- Modifies a JWT to include:
  ```json
  {
    "alg": "RS256",
    "kid": "<attacker-key-id>",
    "jku": "https://attacker.com/jwks.json"
  }
  ```
- Signs the token with their private key

If the server fetches and uses the malicious JWK without validation, the signature will be accepted.

---

## Tools

- Burp Suite (JWT Editor extension)
- Exploit server

---

## Exploitation Steps

#### Burp Suite (JWT Editor Extension)
1. Generate a **New RSA Key** in the **JWT Editor Keys** tab.
   <img width="1029" height="654" alt="image" src="https://github.com/user-attachments/assets/9f142a7e-eb93-4d9c-b838-d7614f4cbaa9" />
   
2. **Copy Public Key as JWK**.
   <img width="1029" height="365" alt="image" src="https://github.com/user-attachments/assets/214a8488-a394-4918-a3c6-79861d5a96eb" />

3. Host it in a `jwks.json` file (MUST BE INSIDE `{"keys": []}`!!!).
   ```json
   {
     "keys": [ <PASTE HERE> ]
   }
   ```
   <img width="1004" height="273" alt="image" src="https://github.com/user-attachments/assets/42c562c2-3c6d-4053-8942-5dd4b71e994e" />

4. Modify the JWT header to include:
   ```json
   {
     "alg": "RS256",
     "kid": "<your-kid>",
     "jku": "https://your-server.com/jwks.json"
   }
   ```
5. Sign the modified JWT using your private key without modifying headers.
   <img width="803" height="692" alt="image" src="https://github.com/user-attachments/assets/5ed5ff1a-4660-45f1-bfa3-8b9eec4549af" />

6. Send the token to the application and observe the response.

---

## Bypasses

Some applications implement basic validation for the `jku` URL to restrict where keys can be loaded from. These protections can sometimes be bypassed using one of the following techniques:

- **File upload** (domain-based allowlist):  
  If the application only checks that the domain is trusted, you may upload a `jwks.json` file (e.g., via a user-upload feature) to that domain and point `jku` to it.

- **Open redirect:** (domain-based allowlist):
  If an open redirect exists on a trusted domain, set the `jku` to point through it:
  ```text
  https://trusted.com/redirect?url=https://attacker.com/jwks.json
  ```

- **Path traversal** (path-based check):  
  If the server validates the full path (e.g., expects `/.well-known/jwks.json`), you may bypass it using path traversal:
  ```text
  http://target.com/.well-known/jwks.json/../../uploads/jwks.json
  ```

- **CRLF header injection** (domain-based check):  
  In some cases, it's possible to exploit HTTP header injection by setting the `jku` parameter to a URL that reflects user-controlled headers. If the server does not validate or sanitize the input, you can:
  1. Set `jku` to a URL that accepts HTTP header injection via CRLF (`\r\n`).
  2. Inject a crafted `jwks.json` response into the server’s headers using CRLF.
  3. Cause the server to reflect this header as a fake JWK Set response.
  
  This tricks the JWT validation logic into using your injected public key without needing a file upload or external redirect.

---

## Impact

- Unauthorized access
- Privilege escalation
- Account takeover
- Server-Side Request Forgery (SSRF)

---

## Defense

- Never trust keys from unvalidated `jku` URLs
- Implement an explicit allowlist of trusted domains for JWK loading
- Validate key contents (e.g., `kid`, `alg`)
- Prefer local key storage unless dynamic remote keys are absolutely necessary

---

## Labs

- https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection
- https://pentesterlab.com/exercises/jwt-viii
- https://pentesterlab.com/exercises/jwt-ix
- https://pentesterlab.com/exercises/jwt-x
- https://pentesterlab.com/exercises/jwt-xi
