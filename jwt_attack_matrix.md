# JWT Attack Matrix

This matrix summarizes which JWT attacks apply to symmetric (HS*) vs asymmetric (RS*) algorithms.

| Attack Name | [Unverified Signature](attacks/unverified-signature.md) | [None Algorithm](attacks/none-algorithm.md) | [Brute-force HS256 Secret](attacks/brute-force-hs256-secret.md) | [jwk header injection](attacks/jwk-header-injection.md) | [jku header injection](attacks/jku-header-injection.md) | [kid directory traversal / LFI](attacks/kid-directory-traversal---lfi.md) | [kid SQLi / Command Injection](attacks/kid-sqli---command-injection.md) | [Algorithm Confusion (RS ➝ HS)](attacks/algorithm-confusion-rs-to-hs.md) | [HS ➝ RS with jku](attacks/hs-to-rs-with-jku.md) | [JWT with weak key](attacks/jwt-with-weak-key.md) | [Token with no expiration](attacks/token-with-no-expiration.md) |
|-------------|----|----|----|----|----|----|----|----|----|----|----|
| Symmetric | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Asymmetric | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ |
