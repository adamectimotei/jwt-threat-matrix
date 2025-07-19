# Leaked Secret Key

Some developers use example JWT implementations or online tutorials and forget to replace the default secret key. In some cases, they may also log or expose real JWT tokens in public issue trackers, repositories, or code examples. If such a token is published online, attackers can extract it and use it to recover the secret key or reuse the token directly, leading to full compromise of the associated user session or service.

---

## Vulnerability Details

If a JWT secret key is leaked or left as a known default (e.g., `"secret"`, `"changeme"`, `"my-jwt-secret"`), anyone can forge tokens. These secrets may appear in:
- Public GitHub repositories
- Example code
- Misconfigured `.env` files
- Open-source project defaults

A known list of leaked and common JWT secrets is maintained here:
- https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list

---

## Tools

- Burp Suite (JWT heartbreaker extension)
- jwt_tool
- Hashcat

---

## Exploitation Steps

#### Burp Suite (JWT heartbreaker)
1. Load **JWT heartbreaker** extension.
2. Start intercepting requests containing JWT tokens and check if any of them uses a leaked secret key.
   <img width="757" height="514" alt="image" src="https://github.com/user-attachments/assets/be32886f-1e29-4279-9b72-8128beb2758c" />


#### jwt_tool
```bash
python3 jwt_tool.py <JWT> -C -d jwt.secrets.list
```

#### Hashcat
```bash
hashcat -a 0 -m 16500 jwt.txt jwt.secrets.list
```

---

## Impact

- Unauthorized access
- Privilege escalation
- Account takeover

---

## Defense

- Never hardcode secrets in code or configuration files
- Use strong, unpredictable secrets (32+ bytes of entropy)
- Prefer asymmetric algorithms (e.g., RS256) where possible

---

## Labs

- https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key
