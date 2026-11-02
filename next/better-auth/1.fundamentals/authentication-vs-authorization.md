# Authentication vs Authorization

## 1. Authentication (AuthN)

Authentication verifies **who a user is**.

The application checks whether the person making a request can prove ownership of an identity.

Common authentication methods:

- Email and password
- Google or GitHub sign-in (OAuth)
- Passkeys
- Multi-factor authentication (MFA)

Examples:

- Are the submitted email and password valid?
- Is this session still valid?
- Has this user successfully signed in?

## 2. Authorization (AuthZ)

Authorization determines **what an authenticated user is allowed to do**.

It checks permissions after the application has established the user's identity.

Examples:

- Can a student create a course?
- Can an instructor edit this course?
- Can an admin delete another user?
- Can this user access a particular workspace?

## 3. The difference

| Concern       | Authentication                                        | Authorization                                       |
| ------------- | ----------------------------------------------------- | --------------------------------------------------- |
| Main question | Who are you?                                          | What can you do?                                    |
| Purpose       | Verify identity                                       | Enforce permissions                                 |
| Happens       | When establishing identity and validating credentials | When deciding whether an action/resource is allowed |
| Example       | Verify a login or session                             | Check whether the user has the `admin` role         |

## 4. How they work together

Imagine a workspace application with students, instructors, and admins.

1. A user signs in with valid credentials.
2. The application establishes the user's identity.
3. The user requests to delete a course.
4. The application checks whether that user has permission to delete it.
5. The application allows or rejects the operation.

A successful login does **not** mean the user can access every feature.

```text
Request
   |
   v
Authentication: Who is making this request?
   |
   v
Authorization: Is this user allowed to perform this action?
   |
   v
Allow or deny
```

## 5. Important security distinction

Do not rely only on hiding buttons or pages in the frontend.

The backend must enforce authorization on every protected operation. A user can manually send requests without using the UI.

For resource-level permissions, check ownership and access rules too. For example, an instructor should not be able to edit another instructor's course just because both have the `instructor` role.

## Key takeaways

- Authentication establishes identity.
- Authorization decides access.
- Authentication and authorization solve different problems.
- A logged-in user is not automatically allowed to perform every action.
- Enforce authorization on the server, not only in the UI.
