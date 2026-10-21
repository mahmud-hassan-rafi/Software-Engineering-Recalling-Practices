# TypeScript: Type Annotation, Type Inference & Literal Types

> Notes covering three foundational TypeScript concepts, with examples and practical usage.

## 1. Type Annotation

### What is Type Annotation?

Type Annotation means explicitly telling TypeScript what type a variable, function parameter, object property, or function return value should have.

**Syntax:**

```ts
let variableName: type = value;
```

The colon (`:`) is used to specify a type.

### Variable Annotation

```ts
let username: string = "Rafi";
let age: number = 23;
let isDeveloper: boolean = true;
```

TypeScript checks assignments against the declared type:

```ts
let username: string = "Rafi";

username = "Mahmud"; // Valid
username = 25;       // Error
username = true;     // Error
```

### Function Parameter Annotation

Function parameters should generally have explicit types.

```ts
function greet(name: string, age: number) {
    console.log(`Hello ${name}, age: ${age}`);
}

greet("Rafi", 23); // Valid
greet("Rafi", "23"); // Error
```

### Function Return Type Annotation

A return type is written after the parameter list.

```ts
function add(a: number, b: number): number {
    return a + b;
}

function isAdult(age: number): boolean {
    return age >= 18;
}
```

If the function promises to return a number but returns a string, TypeScript reports an error.

```ts
function getAge(): number {
    return "23"; // Error
}
```

### Array Annotation

Two common syntaxes:

```ts
let numbers: number[] = [10, 20, 30];
let names: string[] = ["Rafi", "Mahmud"];

let scores: Array<number> = [70, 80, 90];
let users: Array<string> = ["Rafi", "Hasan"];
```

`number[]` and `Array<number>` both describe arrays whose elements are numbers.

```ts
let numbers: number[] = [10, 20];

numbers.push(30);      // Valid
numbers.push("thirty"); // Error
```

### Object Annotation

```ts
let user: {
    name: string;
    age: number;
    isActive: boolean;
} = {
    name: "Rafi",
    age: 23,
    isActive: true
};
```

The object must follow the declared structure.

```ts
user.name = "Mahmud"; // Valid
user.age = "23";      // Error
user.email = "x@y.com"; // Error: property is not declared
```

For reusable object shapes, use a `type` alias or an `interface` (covered in a later topic).

```ts
type User = {
    id: string;
    name: string;
    email: string;
};

const user: User = {
    id: "u101",
    name: "Rafi",
    email: "rafi@example.com"
};
```

### When Should You Use Type Annotation?

Use explicit annotations when:

- A variable is declared without an initial value.
- A function parameter needs a known type.
- You want to make a function's return contract explicit.
- An object or API input should follow a known structure.
- A public function or module boundary benefits from a clear contract.

Example:

```ts
let username: string;
username = "Rafi";

function calculateTotal(price: number, quantity: number): number {
    return price * quantity;
}
```

---

## 2. Type Inference

### What is Type Inference?

Type Inference is TypeScript's ability to determine a value's type from its initial value or surrounding context, without requiring the developer to write the type explicitly.

```ts
let username = "Rafi";
let age = 23;
let isDeveloper = true;
```

TypeScript infers:

- `username` → `string`
- `age` → `number`
- `isDeveloper` → `boolean`

Inference still provides type checking:

```ts
let age = 23;

age = 25;       // Valid
age = "twenty"; // Error
```

### Function Return Type Inference

TypeScript can infer a function's return type from its return statements.

```ts
function add(a: number, b: number) {
    return a + b;
}
```

The inferred return type is `number`.

```ts
function getUsername() {
    return "Rafi";
}
```

The inferred return type is `string`.

Parameter types are a different matter: ordinary function parameters generally need annotations or contextual typing.

### Array Inference

```ts
const numbers = [10, 20, 30];
// Inferred type: number[]

const names = ["Rafi", "Mahmud"];
// Inferred type: string[]
```

```ts
numbers.push(40);     // Valid
numbers.push("forty"); // Error
```

### Object Inference

```ts
const user = {
    name: "Rafi",
    age: 23,
    isActive: true
};
```

TypeScript infers an object type with these properties:

```ts
{
    name: string;
    age: number;
    isActive: boolean;
}
```

### `let` vs `const` Inference

The inference behavior of `let` and `const` can differ.

```ts
let role = "admin";
// Inferred type: string

const fixedRole = "admin";
// Inferred type: "admin"
```

A `let` variable can be reassigned, so TypeScript generally infers a wider type such as `string`.

A `const` variable cannot be reassigned, so TypeScript can often infer the exact literal type.

### When Should You Rely on Inference?

Inference is usually a good choice when:

- A variable is initialized with an obvious value.
- A function's return type is straightforward.
- An array or object has an obvious structure.
- An explicit annotation would only repeat information TypeScript already knows.

Avoid adding redundant annotations everywhere. Prefer clear code and add annotations where they improve safety or communicate an important contract.

---

## 3. Literal Types

### What are Literal Types?

A Literal Type restricts a value to one exact literal value, or to a union of specific literal values.

Compare:

```ts
let username: string = "Rafi";

username = "Mahmud"; // Valid
username = "Hasan";  // Valid
```

The type `string` allows any string.

```ts
let fixedName: "Rafi" = "Rafi";

fixedName = "Mahmud"; // Error
```

The type `"Rafi"` allows only that exact string value.

### String Literal Types

```ts
let status: "success";

status = "success"; // Valid
status = "error";   // Error
```

A variable can also accept a limited set of strings using a union:

```ts
type Status = "pending" | "success" | "failed";

let status: Status = "pending";

status = "success"; // Valid
status = "failed";  // Valid
status = "loading"; // Error
```

The `|` symbol is the union operator. Here it means the value must be one of the listed alternatives.

**Real-world example — user roles:**

```ts
type UserRole = "admin" | "user" | "moderator";

let role: UserRole = "admin";

role = "user";       // Valid
role = "moderator";  // Valid
role = "superadmin"; // Error
```

This prevents unsupported values and catches spelling mistakes at compile time.

### Number Literal Types

Number literal types restrict a value to specific numbers.

```ts
type DiceValue = 1 | 2 | 3 | 4 | 5 | 6;

let dice: DiceValue = 4;

dice = 6; // Valid
dice = 7; // Error
```

**Real-world example — ratings:**

```ts
type Rating = 1 | 2 | 3 | 4 | 5;

function giveRating(rating: Rating): void {
    console.log(`User rated: ${rating}`);
}

giveRating(5); // Valid
giveRating(6); // Error
```

**HTTP status example:**

```ts
type HttpStatus = 200 | 201 | 400 | 401 | 403 | 404 | 500;

let responseStatus: HttpStatus = 200;

responseStatus = 404; // Valid
responseStatus = 503; // Error
```

Only the values explicitly listed in `HttpStatus` are accepted. This type does not automatically include every HTTP status code.

### Boolean Literal Types

A boolean literal type can restrict a value to exactly `true` or exactly `false`.

```ts
let alwaysEnabled: true = true;

alwaysEnabled = false; // Error
```

```ts
let alwaysDisabled: false = false;

alwaysDisabled = true; // Error
```

The union `true | false` is equivalent to the broader `boolean` type, so normally use `boolean` when both values are allowed.

**Example — configuration:**

```ts
type RequiredFeatureConfig = {
    featureName: string;
    enabled: true;
};

const config: RequiredFeatureConfig = {
    featureName: "Payment",
    enabled: true
};
```

Here `enabled: false` would be rejected because the type requires the exact value `true`.

### Literal Types in Function Parameters

Literal unions are useful for restricting function arguments.

```ts
type AccountStatus = "active" | "inactive" | "suspended";

function updateAccountStatus(status: AccountStatus): void {
    console.log(`Account status: ${status}`);
}

updateAccountStatus("active");    // Valid
updateAccountStatus("suspended"); // Valid
updateAccountStatus("blocked");   // Error
```

**Payment example:**

```ts
type PaymentMethod = "card" | "bkash" | "nagad";

function processPayment(
    amount: number,
    method: PaymentMethod
): string {
    return `${amount} paid using ${method}`;
}

processPayment(500, "bkash"); // Valid
processPayment(500, "cash");  // Error
```

### Literal Types and Type Inference

The initial value and declaration style affect inference.

```ts
let status = "pending";
// Inferred type: string

const statusValue = "pending";
// Inferred type: "pending"
```

Similarly:

```ts
let age = 25;       // number
const ageLimit = 25; // 25 (literal type)

let isActive = true;  // boolean
const isEnabled = true; // true (literal type)
```

An explicit annotation can also preserve a literal type with `let`:

```ts
let role: "admin" = "admin";

role = "admin"; // Valid
role = "user";  // Error
```

### `as const`

`as const` tells TypeScript to preserve literal values as narrowly as possible and make object properties or array elements readonly.

Without `as const`:

```ts
const roles = ["admin", "user", "moderator"];
// Inferred type: string[]

roles.push("guest"); // Valid
```

With `as const`:

```ts
const roles = ["admin", "user", "moderator"] as const;
// Inferred type:
// readonly ["admin", "user", "moderator"]

// roles.push("guest"); // Error
// roles[0] = "user";   // Error
```

`as const` is useful for fixed configuration values and constant collections. It does not validate external runtime data.

### Common Mistakes

**1. Confusing `string` with a string literal:**

```ts
let role: string = "admin";   // Any string is allowed
let fixedRole: "admin" = "admin"; // Only "admin" is allowed
```

**2. Assuming `let` cannot use literal types:**

```ts
let role: "admin" = "admin"; // Valid
```

An explicit literal annotation works with `let` too.

**3. Ignoring case sensitivity:**

```ts
type Status = "pending" | "success";

let status: Status = "Pending"; // Error
```

`"Pending"` and `"pending"` are different string values.

---

## 4. Putting the Three Concepts Together

These concepts commonly work together in the same code.

```ts
type CreateUserInput = {
    name: string;
    email: string;
    role: "admin" | "user" | "moderator";
};

function createUser(input: CreateUserInput) {
    const user = {
        id: crypto.randomUUID(),
        name: input.name,
        email: input.email,
        role: input.role,
        isActive: true
    };

    return user;
}
```

What is happening?

| Code | Concept |
|---|---|
| `input: CreateUserInput` | Type annotation |
| `name: string` | Property type annotation |
| `role: "admin" \| "user" \| "moderator"` | String literal union |
| `id` | Type inference |
| `isActive` | Type inference |
| Function return type | Type inference |
| `user` object structure | Type inference |

The goal is not to annotate every expression. Use annotations to define important boundaries and contracts, and let TypeScript infer obvious types.

> Important: TypeScript types are primarily compile-time checks. They do not automatically validate incoming HTTP request bodies, database records, or other runtime data. Runtime validation requires a validation mechanism (for example, Zod).

---

## 5. Quick Revision

| Concept | Meaning | Example |
|---|---|---|
| Type Annotation | Developer explicitly declares a type | `let age: number = 23` |
| Type Inference | TypeScript determines the type | `let age = 23` |
| String Literal | One exact string value | `"admin"` |
| Number Literal | One exact number value | `200` |
| Boolean Literal | One exact boolean value | `true` |
| Union Literal | One value from a fixed set | `"admin" \| "user"` |
| Const Assertion | Preserves narrow literal types and readonly structure | `as const` |

## 6. Practice Exercises

Try to solve these without looking at the examples above.

1. **String Literal:** Create a variable that accepts only `"success"`.
2. **Union Literal:** Create an `OrderStatus` type with `"pending"`, `"processing"`, `"shipped"`, `"delivered"`, and `"cancelled"`.
3. **Number Literal:** Create a `Rating` type that accepts only `1`, `2`, `3`, `4`, or `5`.
4. **Boolean Literal:** Create a variable whose type accepts only `false`.
5. **Function Challenge:** Create a `setUserRole(role)` function. It should accept only `"admin"`, `"editor"`, or `"viewer"` and return `void`.
6. **Inference Challenge:** Create a function that accepts two numbers and returns their product. Do not explicitly annotate its return type.
7. **Combined Challenge:** Create a `Payment` type with `amount: number`, `method: "card" | "bkash" | "nagad"`, and `status: "pending" | "paid" | "failed"`. Create a valid payment object.
