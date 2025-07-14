# Algorithm Confusion (RSA → HMAC)

Algorithm confusion, also called **JWT downgrade attack**, occurs when a server fails to enforce a specific algorithm (`alg`) for JWT validation. An attacker can exploit this by switching the algorithm from an **asymmetric** one (e.g., RS256) to a **symmetric** one (e.g., HS256) and signing the token with the server's **public key**.

---

## Vulnerability Summary

JWTs support multiple algorithms:

- **RS256**: Asymmetric - server signs with private key and verifies with public key
- **HS256**: Symmetric - server signs and verifies with the same secret

If the server blindly trusts the `alg` header:

- Attacker modifies `alg` from `RS256` to `HS256`
- Signs the token using the **server's public key** as the **HMAC secret**
- The server will verify it as valid if it incorrectly uses the same key

---

## Vulnerable Code Example

```javascript
function verify(token, secretOrPublicKey) {
    algorithm = token.getAlgHeader();
    if (algorithm == "RS256") {
        // Use RSA public key
    } else if (algorithm == "HS256") {
        // Use HMAC secret
    }
}

// Vulnerable usage:
publicKey = load("pubkey.pem");
token = getTokenFromCookie();
verify(token, publicKey);  // insecure if alg is HS256
```

---

## Tools

- Burp Suite (JWT Editor extension)
- jwt_tool
- rsa_sign2n

---

## Exploitation Steps

#### 1. Obtain Server's Public Key

- Look for `/.well-known/jwks.json` or `/jwks.json`
- Public key may also be extractable from 2 or more JWTs
```bash
docker run --rm -it portswigger/sig2n <jwt1> <jwt2>
```

- Public key may also be extractable from a TLS certificate:

```bash
openssl s_client -connect example.com:443 2>&1 < /dev/null | sed -n '/-----BEGIN/,/-----END/p' > certificatechain.pem
openssl x509 -pubkey -in certificatechain.pem -noout > pubkey.pem
```

#### 2. Convert Public Key to Symmetric Format

- Base64-encode the PEM-formatted public key
- In Burp's **JWT Editor Keys** tab:
  - Click **New Symmetric Key**
  - Replace the `k` value with the Base64-encoded PEM public key

#### 3. Modify and Sign JWT

- Change `alg` in the JWT header to `HS256`
- Modify the payload (e.g., set `sub=admin`)
- Sign the JWT with the symmetric key created above

---

## Impact

- Unauthorized access
- Privilege escalation
- Account takeover

---

## Mitigation

- Never trust the `alg` field from the token
- Enforce algorithm type on the server (e.g., RS256 only)
- Separate algorithm parsing and validation
- Use libraries that require explicit key types and algorithm enforcement

---

## Labs

- https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion
- https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key
- https://pentesterlab.com/exercises/jwt-algorithm-confusion
- https://pentesterlab.com/exercises/jwt-algorithm-confusion-rsa-key-recovery
