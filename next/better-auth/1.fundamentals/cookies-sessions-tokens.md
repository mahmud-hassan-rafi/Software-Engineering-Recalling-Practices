# Cookies, Sessions, and Tokens

These concepts are related, but they are **not the same thing**.

## 1. Cookie

A cookie is a small piece of data that a browser stores for a website and can automatically attach to matching HTTP requests.

A server can send a cookie using the `Set-Cookie` response header:

```http
Set-Cookie: session_token=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
```

The browser may later send it back:

```http
GET /dashboard
Cookie: session_token=abc123
```

A cookie is a browser storage and transport mechanism. It does not automatically mean that the application uses server-side sessions.

### Important cookie attributes

| Attribute             | Purpose                                                                                |
| --------------------- | -------------------------------------------------------------------------------------- |
| `HttpOnly`            | Prevents JavaScript from reading the cookie through APIs such as `document.cookie`     |
| `Secure`              | Sends the cookie only over HTTPS                                                       |
| `SameSite`            | Controls when the browser sends the cookie in cross-site contexts; helps mitigate CSRF |
| `Path`                | Limits the request paths where the cookie is sent                                      |
| `Expires` / `Max-Age` | Controls the cookie's browser-side lifetime                                            |

`HttpOnly` does not prevent a stolen cookie from being copied into another browser. It mainly reduces cookie theft through JavaScript access.

## 2. Session

A session represents an authenticated state maintained by the application.

In a common server-side session design, the server stores a session record associated with a user.

Example:

```ts
{
  id: "session_123",
  userId: "user_456",
  token: "secret_random_value",
  expiresAt: new Date("2026-11-09T12:00:00Z")
}
```

The browser usually receives a session credential, often through a cookie. When a protected request arrives, the server uses that credential to find and validate the session.

### Typical session request flow

```text
1. User submits credentials
          |
          v
2. Server verifies credentials
          |
          v
3. Server creates a session record
          |
          v
4. Server sends a session credential in a cookie
          |
          v
5. Browser sends the cookie on later requests
          |
          v
6. Server validates the session and identifies the user
```

### Multiple devices

A user can have multiple independent sessions:

```text
User: user_456
  |
  +-- Chrome session
  |
  +-- Firefox session
  |
  +-- Mobile session
```

Logging out one session does not necessarily log out the others.

- **Delete the browser cookie:** Removes that browser's copy of the credential. It does not necessarily remove the server-side session.
- **Revoke the server-side session:** Makes that session invalid even if a browser still has a copy of its cookie.
- **Log out all devices:** Revoke all active sessions belonging to the user.

## 3. Token

A token is a value used to represent or prove something in an authentication flow.

A session token and a JWT are both tokens, but they work differently.

### Session token

A session token is commonly an opaque, unpredictable value. The server uses it to look up a session record.

```text
Browser cookie
      |
      v
Session token
      |
      v
Server looks up session
      |
      v
Session record -> User
```

The token itself does not need to contain the user's profile or permissions.

### JWT (JSON Web Token)

A JWT is a structured token containing claims. A signed JWT can be validated using its signature and the relevant verification rules.

A JWT commonly has three Base64URL-encoded parts:

```text
header.payload.signature
```

Conceptually:

- **Header:** Identifies the token type and signing algorithm.
- **Payload:** Contains claims such as subject (`sub`) and expiration (`exp`).
- **Signature:** Allows the receiver to verify that the signed content has not been modified.

A signed JWT is generally not encrypted. Do not put passwords, secrets, or sensitive private data in its payload.

Example of a decoded payload:

```json
{
  "sub": "user_456",
  "role": "student",
  "exp": 1794225600
}
```

The receiver must validate the signature and relevant claims, including expiration. Merely decoding a JWT does not verify it.

## 4. Server-side sessions vs JWT

| Concern              | Server-side session              | JWT                                                                |
| -------------------- | -------------------------------- | ------------------------------------------------------------------ |
| Common credential    | Opaque session token             | Signed JWT                                                         |
| Validation           | Look up and validate session     | Verify signature and claims                                        |
| Server-side state    | Usually stores session records   | Can be stateless for access-token validation                       |
| Immediate revocation | Revoke/delete the session record | Requires an additional revocation strategy or short token lifetime |
| Cookie usage         | Can be stored in a cookie        | Can also be stored in a cookie                                     |

JWT does not mean cookie-less authentication. A JWT can be transported in a cookie or an `Authorization` header.

## 5. Session theft / hijacking

Suppose a valid session cookie is copied from Chrome and placed into Firefox.

If the copied credential is still valid and the server accepts it, Firefox may be authenticated as the same user. The server generally cannot trust the browser name as proof of identity.

This is why session credentials must be protected.

Useful defenses include:

- HTTPS and the `Secure` cookie attribute.
- `HttpOnly` to reduce JavaScript-based cookie theft.
- Appropriate `SameSite` settings and CSRF defenses.
- Strong, unpredictable session tokens.
- Reasonable expiration and server-side revocation.
- XSS prevention and careful handling of untrusted input.
- Re-authentication or MFA for sensitive operations.

IP address or User-Agent checks can be useful signals, but should not be treated as reliable proof by themselves.

## Key takeaways

- A **cookie** is a browser mechanism for storing and sending data.
- A **session** represents authenticated state maintained by the application.
- A **session token** identifies or grants access to a session.
- A **JWT** is a structured token containing claims and a signature.
- Cookie deletion and server-side session revocation are different operations.
- A valid stolen session credential may be replayed by another browser.
