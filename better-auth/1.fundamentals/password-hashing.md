# Password Hashing with bcrypt

## 1. Why should passwords be hashed?

A password should never be stored as plain text in the database.

If an attacker obtains a database backup containing plain-text passwords, those passwords are immediately exposed.

Instead, an application stores a password hash. During login, it verifies the submitted password against the stored hash.

```text
Registration:
Plain password -> bcrypt hash -> Store hash in database

Login:
Submitted password + stored hash -> bcrypt verification -> Match or reject
```

## 2. Hashing vs encryption

- **Encryption** is designed to be reversible when the correct key is available.
- **Password hashing** is designed to be one-way. The application verifies a password rather than decrypting the stored hash.

Passwords should be hashed, not reversibly encrypted for normal login verification.

## 3. What is bcrypt?

bcrypt is a password-hashing algorithm designed to make password guessing expensive.

It has a configurable **cost factor**. Increasing the cost makes each hash calculation more computationally expensive.

This helps slow down brute-force and offline password-guessing attempts if hashes are stolen.

## 4. Salt in bcrypt

bcrypt automatically generates and uses a random salt when creating a hash.

A salt ensures that identical passwords do not normally produce identical stored hashes.

For example, two users may choose the same password:

```text
Password: mySecretPassword

User A -> bcrypt hash with salt A
User B -> bcrypt hash with salt B
```

The resulting hashes will be different because their salts are different.

With bcrypt, the salt and cost information are encoded in the stored hash. You normally do not need to create and store a separate salt field yourself.

## 5. Registration flow

Conceptual TypeScript example using `bcrypt`:

```ts
import bcrypt from "bcrypt";

const password = "user-entered-password";
const saltRounds = 12;

const passwordHash = await bcrypt.hash(password, saltRounds);

// Store passwordHash in the user's database record.
// Never store the original password.
```

The cost factor should be benchmarked on the actual production environment. `12` is an example, not a universal setting for every application.

## 6. Login flow

```ts
import bcrypt from "bcrypt";

const submittedPassword = "user-entered-password";
const storedHash = user.passwordHash;

const isPasswordValid = await bcrypt.compare(submittedPassword, storedHash);

if (!isPasswordValid) {
  throw new Error("Invalid email or password");
}

// Continue the login flow only after successful verification.
```

Notice that we do not decrypt the hash or manually compare two newly generated hashes. `bcrypt.compare()` handles verification using the salt and cost information embedded in the stored hash.

## 7. Important security practices

- Never store plain-text passwords.
- Never log passwords or include them in error messages.
- Use bcrypt's built-in salt generation.
- Choose a cost factor by benchmarking on production-like hardware.
- Use rate limiting and sensible login-attempt protections.
- Use generic login errors so the response does not reveal whether an email exists.
- Require HTTPS in production.
- Keep password hashes out of API responses.
- Consider a password-length policy and understand bcrypt's input-length limitations when designing validation.

## 8. Where bcrypt fits in authentication

bcrypt only handles password hashing and verification. It does not:

- Create user sessions.
- Set browser cookies.
- Generate JWTs.
- Decide user permissions.
- Implement login or logout by itself.

A typical application uses bcrypt to verify credentials, then creates an authenticated session or issues another authentication credential.

## Key takeaways

- Store a bcrypt hash, never the original password.
- bcrypt is intentionally computationally expensive.
- bcrypt generates and incorporates a salt automatically.
- Use `bcrypt.compare()` to verify a submitted password.
- Password hashing is only one part of the complete authentication system.
