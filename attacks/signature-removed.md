JWT libraries typically provide one method for verifying tokens and another that just decodes them. For example, the Node.js library `jsonwebtoken` has `verify()` and `decode()`.

Occasionally, developers confuse these two methods and only pass incoming tokens to the `decode()` method. This effectively means that the application doesn't verify the signature at all.

{
"username": "carlos",
"isAdmin": false
}

→ isAdmin změníme na True

The steps are trivial:

1. Obtain a valid token (e.g., by registering or logging in as a normal user).
2. Base64URL-decode the token to view the header and payload.
3. Modify the payload, for example changing:to:
    
    ```json
    {"user": "bob", "role": "user"}
    ```
    
    ```json
    {"user": "admin", "role": "admin"}
    ```
    
4. Base64URL-encode the modified header and payload.
5. Reassemble the token. You can:
    - Keep the original signature (most likely to work), or
    - Remove it completely and just send `header.payload.` (less likely to work)
6. Send the token in a request (e.g., as a cookie or `Authorization` header).
