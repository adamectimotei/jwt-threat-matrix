# JWT Threat Matrix

This matrix summarizes which JWT attacks apply to symmetric (HS*) vs asymmetric (RS*) algorithms.

| Attack Name | [Sensitive Data Exposure](attacks/sensitive-data-exposure.md) | [Unverified Signature](attacks/unverified-signature.md) | [None Algorithm](attacks/none-algorithm.md) | [Secret Key Brute-force](attacks/secret-key-brute-force.md) | [Leaked Secret Key](attacks/leaked-secret-key.md) | [JWK Header Injection](attacks/jwk-header-injection.md) | [JKU Header Injection](attacks/jku-header-injection.md) | [KID Header Injection](attacks/kid-header-injection.md) | [Algorithm Confusion (RSA ➝ HMAC)](attacks/algorithm-confusion.md) |
|-------------|----|----|----|----|----|----|----|----|----|
| Symmetric | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Asymmetric | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ |
