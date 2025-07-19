# Secret Key Brute-Force

JWT implementations using symmetric algorithms like HS256 rely on a shared secret key to sign and verify tokens. If the secret is weak, hardcoded, or predictable, attackers can recover it through offline brute-force and use it to forge arbitrary tokens.

---

## Vulnerability Details

Symmetric algorithms such as HS256 use an arbitrary secret string (like a password) to sign JWTs. If an attacker captures a valid token, they can attempt to brute-force the secret key offline by comparing computed HMAC signatures.

This happens because developers forget to change default or demo secrets (e.g., `"secret"`, `"changeme"`) or weak secrets like `"123456"` or service names are reused across environments.

### Vulnerable Code Example

```javascript
const jwt = require('jsonwebtoken');

const secret = 'secret'; // weak secret key

const token = req.headers.authorization;

try {
  const decoded = jwt.verify(token, secret);
}
catch (err) {
  res.status(401).send('Invalid token');
}
```

---

## Tools

- Hashcat
- jwt_tool
- Burp Suite (JWT Editor extension)

---

## Exploitation Steps

#### Hashcat
```bash
hashcat -a 0 -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
```
- Rule-based attack: `-r rules/best64.rule`
- Brute force mode: `-a 3 -i --increment-min=6 ?l?l?l?l?l?l`

#### jwt_tool
```bash
python3 jwt_tool.py <JWT> -C -d <WORDLIST.TXT>
```

#### Burp Suite (JWT Editor extension)
1. Intercept a request containing a JWT token.
2. Use **Weak HMAC Secret Attack** in **JSON Web Token** tab.
   <img width="718" height="486" alt="image" src="https://github.com/user-attachments/assets/d3aab871-0761-40eb-8fcd-bc0ff93e9b8f" />

3. Re-sign the JWT using the cracked secret.

---

## Impact

- Unauthorized access
- Privilege escalation
- Account takeover

---

## Defense

- Use strong, unpredictable secrets (32+ bytes of entropy)
- Rotate secrets periodically with support for key rollover
- Prefer asymmetric algorithms (e.g., RS256) where possible

---

## Labs
- https://pentesterlab.com/exercises/jwt-v
