# 🔐 Better Auth & Security Learning Roadmap

> A comprehensive tracking repository for mastering Better Auth, Next.js integration, session security, and full-stack authorization.

---

## 📂 Project Directory & Progress

- [x] 📄 **README.md**

### 📁 fundamentals/

- [x] **authentication-vs-authorization.md** — Difference between Authentication & Authorization
- [x] **cookies-sessions-tokens.md** — How Cookies, Sessions, and JWTs work under the hood
- [x] **password-hashing.md** — Secure password hashing mechanisms (Bcrypt, Argon2, Salt)

### 📁 architecture/

- [x] **better-auth-architecture.md** — Core internal architecture of Better Auth
- [x] **mongodb-integration.md** — Connecting Better Auth with MongoDB schema & models

### 📁 email-password/

- [x] **signup.md** — User registration & validation pipeline
- [x] **signin.md** — Secure login flow & credential checks
- [x] **signout.md** — Session destruction & client-side cleanup
- [x] **email-verification.md** — Email verification tokens & SMTP setup
- [x] **password-reset.md** — Password reset flow with time-limited tokens

### 📁 session-management/

- [x] **session-lifecycle.md** — Session creation, validation, and destruction
- [x] **session-expiration.md** — Idle & absolute expiration strategies
- [x] **session-revocation.md** — Force logout & invalidating compromised sessions
- [x] **multiple-sessions.md** — Managing active sessions across multiple devices

### 📁 nextjs-integration/

- [x] **server-components.md** — Accessing session context in React Server Components
- [x] **server-actions.md** — Authenticating & validating Server Actions
- [x] **route-handlers.md** — Handling API route handlers (`/api/auth/*`)
- [x] **protected-routes.md** — Middleware-based & component-level route protection

### 📁 authorization-security/

- [ ] **rbac.md** — Role-Based Access Control (Admin, User, Moderator)
- [ ] **permissions.md** — Fine-grained resource permission checks
- [ ] **csrf.md** — CSRF attack prevention & cookie security flags
- [ ] **rate-limiting.md** — Rate-limiting auth endpoints against brute-force attacks

### 📁 advanced/

- [ ] **oauth.md** — Social authentication (Google, GitHub, Discord, etc.)
- [ ] **custom-fields.md** — Extending default user & session schemas
- [ ] **hooks-and-plugins.md** — Using Better Auth lifecycle hooks and extension plugins

### 📁 production/

- [ ] **environment-configuration.md** — Managing env secrets & security keys
- [ ] **deployment.md** — Deploying to Vercel/Node.js production environments
- [ ] **security-checklist.md** — Final production security review & hardening
- [ ] **testing-and-debugging.md** — Unit testing auth logic & debugging edge cases

### 📁 projects/

└── 📁 workspace-saas/ - [ ] **README.md** — Real-world Multi-tenant SaaS workspace implementation

---

## 🛠️ Stack Overview

- **Framework:** Next.js (App Router)
- **Auth Framework:** Better Auth
- **Database:** MongoDB
- **Primary Focus:** Security, Session Management, RBAC & Server Components
