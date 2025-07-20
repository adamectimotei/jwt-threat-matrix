# Algorithm Confusion (RSA → HMAC)

Algorithm confusion, also called **JWT downgrade attack**, occurs when a server fails to enforce a specific algorithm (`alg`) for JWT validation. An attacker can exploit this by switching the algorithm from an **asymmetric** one (e.g., RS256) to a **symmetric** one (e.g., HS256) and signing the token with the server's **public key**.

---

## Vulnerability Details

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

1. Obtain the server's public key from:
   - JWK endpoint
   - existing JWTs
   - TLS certificate
2. Convert the public key to a symmetric format.
3. Modify the JWT header to use HS256, sign it using the public key, and send it.

#### 1. Obtain Server's Public Key

- Look for `/.well-known/jwks.json` or `/jwks.json`
  
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

  1. Copy only one of the keys, for example:
     
  ```json
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        }
  ```

  2. In **JWT Editor Keys** tab, click **New RSA Key**, paste the copied JWK key and generate a **New RSA Key**.
  <img width="1034" height="657" alt="image" src="https://github.com/user-attachments/assets/fbce9973-a120-4e99-a990-429efc9b0b3e" />

  3. You can now generate the PEM version by using **Copy Public Key as PEM**.
  <img width="1031" height="345" alt="image" src="https://github.com/user-attachments/assets/44cec567-c45f-4665-b243-028861b018eb" />
  
- Public key may also be extractable from 2 or more JWTs
  1. Run sig2n with two JWT tokens.
     
     ```bash
     docker run --rm -it portswigger/sig2n <JWT_TOKEN1> <JWT_TOKEN2>
     ```
     <img width="885" height="507" alt="image" src="https://github.com/user-attachments/assets/42dd95fb-4629-4dfe-9cb7-2e9272cf3b56" />

  2. Try each of the forged JWT tokens and find out which one gets accepted by the server.
     <img width="884" height="505" alt="image" src="https://github.com/user-attachments/assets/a9905fcb-112e-41b1-a62f-0406e53081bc" />

  3. The corresponding base64-encoded x509 key is the server's public key.
     <img width="882" height="507" alt="image" src="https://github.com/user-attachments/assets/2b69f1fc-6d00-4624-b1ea-f955fadbcaa5" />

- Public key may also be extractable from a TLS certificate:

```bash
openssl s_client -connect example.com:443 2>&1 < /dev/null | sed -n '/-----BEGIN/,/-----END/p' > certificatechain.pem
openssl x509 -pubkey -in certificatechain.pem -noout > pubkey.pem
```

#### 2. Convert Public Key to Symmetric Format

- Base64-encode the PEM-formatted public key if extracted from `jwks.json` file or a TLS certificate
- Alternatively, use base64-encoded public key generated from sig2n directly
- In Burp's **JWT Editor Keys** tab, create a **New Symmetric Key** and replace the `k` value with the Base64-encoded PEM public key
  <img width="1031" height="637" alt="image" src="https://github.com/user-attachments/assets/038e5579-9f98-4762-aa02-898478021e54" />

#### 3. Modify and Sign JWT

- Change the JWT header to:
  
  ```json
  {
    "typ": "JWT",
    "alg": "HS256"
  }
  ```

- Sign the JWT with the symmetric key created above
  <img width="767" height="696" alt="image" src="https://github.com/user-attachments/assets/1bba082b-9a63-47c3-8ec5-c5b4aa2c887b" />

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
