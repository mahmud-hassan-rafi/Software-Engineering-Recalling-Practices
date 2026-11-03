# MongoDB Integration with Better Auth

## 1. The main idea

Better Auth can use MongoDB through its MongoDB database adapter.

You do **not** have to use Mongoose just because your application uses MongoDB. Better Auth can communicate with MongoDB through its adapter, while your application independently uses Mongoose for business/domain data.

### Mental model

```text
                         Next.js application
                                  |
                  ┌───────────────┴────────────────┐
                  |                                |
             Better Auth                        App services
                  |                                |
          MongoDB adapter                       Mongoose
                  |                                |
                  └───────────────┬────────────────┘
                                  |
                              MongoDB
                  ┌───────────────┴────────────────┐
                  |                                |
           Auth-owned data                  App-owned data
       User / Account / Session          Profile / Course /
          / Verification                   Enrollment / Order
```

Both tools can connect to the same MongoDB database while owning different parts of the application's data.

## 2. Native MongoDB driver vs Mongoose

### Native MongoDB driver

The official MongoDB Node.js driver is the lower-level JavaScript/TypeScript interface for communicating with MongoDB.

It provides objects such as:

- `MongoClient` — manages connections to a MongoDB deployment.
- `Db` — represents a selected database.
- `Collection` — represents a MongoDB collection.

You work more directly with MongoDB documents and operations.

### Mongoose

Mongoose is an ODM (Object Data Modeling) library built on top of the MongoDB Node.js driver.

It adds features such as:

- Schemas and models.
- Document validation.
- Middleware/hooks.
- Model-based query APIs.
- Population through configured model references.

### Key distinction

| Native MongoDB driver                       | Mongoose                                          |
| ------------------------------------------- | ------------------------------------------------- |
| Direct driver-level API                     | ODM built on the driver                           |
| Works with collections and documents        | Works mainly with schemas and models              |
| No Mongoose schema/model layer              | Provides schema/model layer                       |
| More direct control over MongoDB operations | More application-level structure and conveniences |

Neither is automatically better for every task. They solve related but different problems.

## 3. Can Better Auth and Mongoose work together?

Yes. They can be used in the same application and can connect to the same MongoDB database.

A sensible ownership boundary:

| Responsibility                      | Recommended owner |
| ----------------------------------- | ----------------- |
| Auth user identity                  | Better Auth       |
| Provider/credential account records | Better Auth       |
| Login sessions                      | Better Auth       |
| Verification tokens                 | Better Auth       |
| Extended student/instructor profile | Mongoose          |
| Courses and enrollments             | Mongoose          |
| Orders and payment records          | Mongoose          |

The tools do not need separate databases. Each system needs a clear responsibility.

## 4. Avoid directly managing Better Auth collections with Mongoose

Better Auth maintains rules and relationships across its authentication data.

For example, creating or revoking a session may involve more than inserting or deleting an arbitrary document. Directly writing to Better Auth's internal collections through Mongoose can bypass expected behavior and create inconsistent auth data.

Therefore:

- Use Better Auth's documented APIs for auth lifecycle operations.
- Use Mongoose for your application's own collections.
- Avoid directly inserting, updating, or deleting Better Auth-owned documents unless the documentation explicitly describes that operation as supported.

Reading auth data directly for debugging is different, but production application logic should generally use supported Better Auth APIs.

## 5. Referencing a Better Auth user from a Mongoose document

Suppose your application has a `Profile` collection. Each profile belongs to a Better Auth user.

```text
Better Auth User
       |
       | user ID
       v
Mongoose Profile
  ├── userId
  ├── bio
  ├── skills
  └── ...
```

The Mongoose document should store the identifier of the corresponding Better Auth user.

Do not generate an unrelated Mongoose `ObjectId` and assume it represents the same user. The value and type of the Better Auth identifier depend on the configured adapter and ID strategy. Confirm the actual ID format in your configuration and stored documents before defining the Mongoose field.

### Does Mongoose `populate()` automatically work?

No—not automatically.

Mongoose `populate()` works through Mongoose model references that Mongoose knows how to resolve. A Better Auth user collection is not automatically a Mongoose model just because both use the same MongoDB database.

For a relationship between app-owned data and a Better Auth user, you can:

- Store the Better Auth user ID in the app document.
- Fetch user information through Better Auth's supported API when needed.
- Define a carefully configured Mongoose model for read-only access if there is a strong reason and the schema/ID behavior is understood.

Do not use `populate()` as a reason to create a second, unrelated user identity.

## 6. Where should a role be stored?

A simple role such as `student`, `instructor`, or `admin` can be represented as an additional Better Auth user field or managed through an appropriate authorization feature.

If you choose an additional field:

- Configure it deliberately.
- Do not allow an untrusted client to assign privileged values during sign-up.
- Assign sensitive roles using trusted server-side logic.
- Enforce role checks on the backend.

For richer profile information—such as instructor biography, skills, education, or student preferences—an application-owned Mongoose `Profile` collection is usually a cleaner fit.

## 7. MongoDB adapter and connection setup: concepts

The MongoDB integration requires a MongoDB connection and a Better Auth adapter configured to use it.

Conceptually:

1. Create or obtain a MongoDB client connection.
2. Select the database Better Auth should use.
3. Configure Better Auth with the MongoDB adapter and selected database.
4. Keep the connection reusable rather than creating unnecessary new connections for every request.

The exact package name, import path, configuration options, and connection pattern can change between Better Auth versions. Verify the official MongoDB adapter documentation for the version installed in your project before copying setup code.

This note focuses on architecture and data ownership. Detailed native-driver connection lifecycle and hands-on adapter setup belong in the MongoDB driver module.

## 8. Example: a learning management system

Imagine a learning platform with students and instructors.

**Better Auth owns:**

- The user's basic identity.
- Email/password or Google account records.
- Login sessions.
- Email verification and password-reset records.

**Mongoose owns:**

- Extended student/instructor profile.
- Courses.
- Course enrollment.
- Payment/order records.
- Other LMS business data.

A user signs in through Better Auth. Your application then uses the authenticated user's ID to locate related profile or business records in Mongoose.

## 9. Key takeaways

- Better Auth supports MongoDB through its MongoDB adapter.
- Mongoose is not required for Better Auth's MongoDB integration.
- Better Auth and Mongoose can coexist in one application and use the same database.
- Better Auth should own its authentication data and lifecycle.
- Mongoose should own your application's business/domain data.
- Store the real Better Auth user ID when referencing a user from a Mongoose document.
- Do not assume Mongoose `populate()` automatically resolves Better Auth collections.
- Do not let untrusted clients assign privileged roles.
- Verify exact adapter imports and configuration against the installed Better Auth version.

## Review checklist

- [x] I can explain the difference between the native MongoDB driver and Mongoose.
- [x] I understand that Mongoose is an ODM built on the MongoDB driver.
- [x] I know Better Auth does not require Mongoose.
- [x] I can explain how Better Auth and Mongoose can use the same database.
- [x] I know which data belongs to Better Auth and which belongs to my application.
- [x] I understand why I should not directly mutate Better Auth's internal collections.
- [x] I understand how a Mongoose document can reference a Better Auth user.
- [x] I know why `populate()` does not automatically work with Better Auth collections.
- [x] I know to verify the configured Better Auth user ID format before defining references.
