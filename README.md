# JWT Threat Matrix

**JWT Threat Matrix** is a curated cheatsheet to help **identify, exploit, and mitigate** vulnerabilities in JSON Web Token (JWT) implementations. It aims to:

- Map known attacks against JWTs
- Clarify which attacks apply to symmetric vs. asymmetric JWT algorithms
- Help testers quickly identify and exploit JWT vulnerabilities

---

| Attack Name | [Sensitive Data Exposure](attacks/sensitive-data-exposure.md) | [Unverified Signature](attacks/unverified-signature.md) | [None Algorithm](attacks/none-algorithm.md) | [Secret Key Brute-force](attacks/secret-key-brute-force.md) | [Leaked Secret Key](attacks/leaked-secret-key.md) | [JWK Header Injection](attacks/jwk-header-injection.md) | [JKU Header Injection](attacks/jku-header-injection.md) | [KID Header Injection](attacks/kid-header-injection.md) | [Algorithm Confusion (RSA ➝ HMAC)](attacks/algorithm-confusion.md) |
|-------------|----|----|----|----|----|----|----|----|----|
| Symmetric | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Asymmetric | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ |

---

## Basic JWT Concepts

#### JWT token

A JWT (JSON Web Token) is a compact token typically used for authentication and authorization. It looks like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.  ← Header (Base64URL)
eyJzdWIiOiJhZG1pbiIsImlhdCI6MTYwNDA0Njg0Mn0. ← Payload (Base64URL)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature
```

Each token has three parts:
1. **Header** – defines the algorithm and token type (`alg`, `typ`)
2. **Payload** – contains claims (user ID, expiration, roles, etc.)
3. **Signature** – validates the token's integrity using a secret or key

#### Algorithm Types

| Type       | Algorithm | Signing Key | Verification Key |
|------------|-----------|-------------|------------------|
| Symmetric  | HS256     | Secret      | Same as signing  |
| Asymmetric | RS256     | Private Key | Public Key       |

---

## Catch

In modern microservice-based applications:

- Services may use different JWT libraries
- Services may have different keys, parsing logic, and expiration settings
- Third-party services may handle tokens insecurely

Ideally, you should test **EVERY ENDPOINT**!

---

## First Checks

Before manual testing, try to run jwt_tool with **playbook** or **all** tests

```bash
python3 jwt_tool.py -t <URL> -M pb -rh "Authorization: Bearer <JWT>"
```

```bash
python3 jwt_tool.py -t <URL> -M at -rh "Authorization: Bearer <JWT>"
```

---

## Resources

- [HackTricks: JWT Vulnerabilities](https://book.hacktricks.wiki/en/pentesting-web/hacking-jwt-json-web-tokens.html)
- [JWT Tool Wiki](https://github.com/ticarpi/jwt_tool/wiki)
- [PortSwigger JWT Labs](https://portswigger.net/web-security/jwt)

---

## Contributions  
Feel free to open issues or pull requests to improve examples or add new attacks and tools.
