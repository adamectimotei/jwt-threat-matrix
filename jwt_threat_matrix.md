# JWT Threat Matrix

This matrix summarizes which JWT attacks apply to symmetric (HS*) vs asymmetric (RS*) algorithms.

| Attack Name | [Sensitive Data Exposure](attacks/sensitive-data-exposure.md) | [Unverified Signature](attacks/unverified-signature.md) | [None Algorithm](attacks/none-algorithm.md) | [Secret Key Brute-force](attacks/secret-key-brute-force.md) | [Leaked Secret Key](attacks/leaked-secret-key.md) | [JWK Header Injection](attacks/jwk-header-injection.md) | [JKU Header Injection](attacks/jku-header-injection.md) | [KID Header Injection](attacks/kid-header-injection.md) | [Algorithm Confusion (RS ➝ HS)](attacks/algorithm-confusion-rs-to-hs.md) | [HS ➝ RS with jku](attacks/hs-to-rs-with-jku.md) | [Token with no expiration](attacks/token-with-no-expiration.md) |
|-------------|----|----|----|----|----|----|----|----|----|----|----|
| Symmetric | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Asymmetric | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
