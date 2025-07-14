# KID Header Injection

Some applications support multiple signing keys for JWTs. The `kid` (Key ID) header parameter is used to indicate which key should be used to verify the token. If the server naively trusts this value without validation, it can lead to serious vulnerabilities.

---

## Vulnerability Details

According to the JWS specification, only the alg header parameter is mandatory. In practice, however, JWT headers (also known as JOSE headers) often contain several other parameters, such as kid (Key ID).

JWTs often include a `kid` parameter to let the server determine which signing key to use. However, when applications dynamically fetch keys based on this field — especially from filesystems or databases — the kid value becomes a dangerous injection point. If the application uses it insecurely (e.g., directly concatenating it into a file path or SQL query), attackers can manipulate it to point to keys they control.

### Vulnerable Code Example (File Path Traversal)

```javascript
const keyPath = "/keys/" + kid;
const publicKey = fs.readFileSync(keyPath);
```

### Vulnerable Code Example (SQL Injection)

```sql
SELECT key FROM keys WHERE kid = '$KID'
```

### Vulnerable Code Example (Ruby RCE)

```ruby
key = open(kid).read
```

---

## Tools
- Burp Suite (JWT Editor extension)

---

## Exploitation Steps

### Path Traversal using Burp (JWT Editor extension)

1. Generate a new symmetric key in Burp > **JWT Editor Keys**.
   <img width="1028" height="634" alt="image" src="https://github.com/user-attachments/assets/33b33172-47ad-4be8-a1c4-c2f7dd596f24" />

2. Set the `k` property to `AA==` (Base64-encoded null byte).
   <img width="1031" height="634" alt="image" src="https://github.com/user-attachments/assets/3b1070f8-5300-4e75-81ff-fa697832dbab" />

3. In the JWT header, set `kid` to:
   ```
   ../../../../../../../../../dev/null
   ```
4. Click **Sign** and use the symmetric key you created.
   <img width="885" height="697" alt="image" src="https://github.com/user-attachments/assets/391b131f-e966-4d19-8819-0089151bfe52" />

5. Send the forged token and observe the application's response.

### SQL Injection using Burp (JWT Editor extension)

Sign token with `attackerkey`.

### OS Command Injection

In Ruby or similar:

```ruby
open(user_input)
```

Payload:

```json
"kid": "|whoami"
```

Server executes the command and returns output like:

```
"www-data\n"
```

---

## Impact

- Unauthorized access
- Privilege escalation
- Account takeover
- Remote Command Execution (RCE)
- SQL Injection

---

## Defense

- Never trust user-supplied `kid` values without validation.
- Maintain a strict allowlist of valid `kid` values.
- Sanitize and canonicalize paths before use.
- Use parameterized queries for any DB lookups.
- Avoid using user input in OS-level functions like `open()`.

---

## Labs

- https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal
- https://pentesterlab.com/exercises/jwt-iii
- https://pentesterlab.com/exercises/jwt-iv
- https://pentesterlab.com/exercises/jwt-vi

