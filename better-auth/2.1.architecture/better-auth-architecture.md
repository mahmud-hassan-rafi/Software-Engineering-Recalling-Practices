# Better Auth Architecture

> Module 1 — Better Auth Architecture

## 1. What is Better Auth?

Better Auth is a TypeScript-first authentication and authorization framework/library. It provides reusable authentication workflows so an application does not need to implement every auth mechanism from scratch.

It is **not** a hosted authentication service. In a typical self-hosted setup, Better Auth runs inside your application/server and stores authentication data in your configured database.

### Mental model

```text
Browser / Client
      |
      v
Next.js Route Handler (/api/auth/*)
      |
      v
Better Auth
  ├── Core authentication logic
  ├── Database adapter
  └── Plugins
      |
      v
Database
```

## 2. Main architectural pieces

### Core

The core handles central authentication workflows and rules, such as processing sign-up/sign-in, validating credentials, managing authentication state, and creating, validating, or revoking sessions.

### Database adapter

The adapter connects Better Auth's core logic to a particular database. It translates Better Auth's database operations into operations supported by that database.

An adapter is **not** the same thing as an ORM. The adapter integrates Better Auth with a database; an ORM such as Mongoose is a separate application data tool.

### Plugins

Plugins extend Better Auth with additional capabilities, such as two-factor authentication, organizations, passkeys, or administrative user management.

A plugin may add configuration, APIs, or database models/collections. Therefore, the core models are not necessarily the only collections Better Auth will use.

### Framework integration

Framework integration connects Better Auth to the framework's request/response lifecycle. In Next.js, the route-handler integration receives the HTTP request and passes it to Better Auth, then returns the appropriate response.

## 3. Simplified sign-up request flow

1. The client sends a sign-up request to the Better Auth endpoint.
2. The Next.js route handler receives the request.
3. Better Auth's core validates and processes it.
4. Better Auth uses its database adapter to read or write auth data.
5. Depending on configuration, Better Auth may create a session.
6. The response may include a session cookie for the browser.

This is a conceptual flow. Exact behavior depends on configuration; for example, email-verification settings can affect whether a session is created immediately.

## 4. Core authentication data models

Better Auth commonly works with four core model concepts. Exact fields and storage details depend on configuration, adapter, and enabled features.

### User

Represents the person/account in your application. Typical information includes name, email, email verification status, profile image, and configured additional fields.

### Account

Represents an authentication account or provider connection associated with a user.

Examples:

- Email/password credentials.
- Google account.
- GitHub account.

One user may have multiple linked accounts. Account linking must follow a secure, verified flow. Do not automatically link accounts merely because their email strings match.

### Session

Represents an active login session. A single user can have multiple sessions:

```text
User
 ├── Session A (Laptop)
 ├── Session B (Phone)
 └── Session C (Another browser)
```

Each session is independently managed. Revoking Session A does not automatically delete the user or linked accounts, and does not necessarily revoke Sessions B and C.

### Verification

Stores temporary verification-related information, such as email-verification tokens or password-reset tokens.

A verification record may expire or be consumed. This differs from a persistent user field such as `emailVerified`, which records the user's current verification status.

### Relationship overview

```text
User
 ├── Account (one or more)
 └── Session (zero or more)

Verification
 └── Temporary verification records
```

Plugins and custom configuration can add more models and relationships.

## 5. Use purpose-specific Better Auth APIs

Prefer documented Better Auth APIs for authentication-related operations instead of treating auth collections like ordinary application tables.

Operation categories include updating permitted user information, listing linked accounts, listing sessions, revoking a session, and administrative user management when the relevant feature is configured.

Exact method names, availability, and arguments depend on the installed Better Auth version, client/server context, and enabled plugins. Check the documentation for your installed version before implementing an API call.

## 6. Trust boundaries and authorization data

Authentication establishes who the user is. Authorization decides what that user is allowed to do.

A role such as `student`, `instructor`, or `admin` may be represented as a configured additional user field or managed through an authorization feature.

Important rules:

- Never trust a role sent directly by an untrusted client during sign-up.
- Sensitive fields should not be client-writable.
- Set privileged roles through trusted server-side logic.
- Make authorization decisions on the server.
- Consider whether role changes must take effect immediately or whether existing session data may be stale.

Hiding a button in the UI is not authorization. The backend must enforce permissions.

## 7. What belongs in Better Auth vs the application?

Better Auth should own authentication data and workflows:

- User identity fields managed by Better Auth.
- Provider/credential account records.
- Login sessions.
- Verification records.
- Auth-specific operations.

Your application should own business/domain data:

- Student or instructor profile details.
- Skills, biography, and preferences.
- Courses and enrollments.
- Orders and payments.

For example, a learning platform may store basic identity in Better Auth and extended instructor details in a Mongoose `Profile` collection.

## 8. Key takeaways

- Better Auth is an authentication framework/library running as part of your application, not a hosted auth service.
- Core, adapter, plugins, and framework integration have separate responsibilities.
- The adapter connects Better Auth to the database; it is not the same as an ORM.
- `User`, `Account`, `Session`, and `Verification` represent different concepts.
- One user can have multiple linked accounts and independent sessions.
- Verification tokens are temporary; a user's verification status is persistent state.
- Use Better Auth's documented APIs for auth workflows.
- Enforce roles and permissions on the server.
- Keep authentication-owned data separate from application business data.
