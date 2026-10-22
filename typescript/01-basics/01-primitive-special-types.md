# TypeScript: Primitive Types & Special Types

> Complete reference notes for TypeScript's primitive types and commonly used special types. Examples assume a modern TypeScript project with `strict: true` unless a section says otherwise.

## Contents

1. [Primitive Types](#1-primitive-types)
   - `string`
   - `number`
   - `boolean`
   - `bigint`
   - `symbol`
   - `null`
   - `undefined`
2. [Special Types](#2-special-types)
   - `any`
   - `unknown`
   - `never`
   - `void`
   - `object`
3. [Important Comparisons](#3-important-comparisons)
4. [Practical Examples](#4-practical-examples)
5. [Common Mistakes](#5-common-mistakes)
6. [Quick Revision](#6-quick-revision)
7. [Practice Exercises](#7-practice-exercises)

---

# 1. Primitive Types

Primitive values are values that are not objects and do not have methods or properties of their own. JavaScript has seven primitive types:

- `string`
- `number`
- `boolean`
- `bigint`
- `symbol`
- `null`
- `undefined`

TypeScript uses the corresponding lowercase type names.

## 1.1 `string`

The `string` type represents textual data.

```ts
let username: string = "Rafi";
let message: string = 'Welcome!';
let description: string = `Hello, ${username}`;
```

All three string syntaxes are valid:

- Single quotes: `'text'`
- Double quotes: `"text"`
- Template literals: `` `text ${value}` ``

### Common string operations

```ts
const firstName: string = "Mahmud";
const lastName: string = "Rafi";

const fullName = `${firstName} ${lastName}`;

console.log(fullName.toUpperCase());
console.log(fullName.toLowerCase());
console.log(fullName.length);
console.log(fullName.includes("Rafi"));
```

TypeScript knows the result types of these operations. For example, `toUpperCase()` returns a `string`, while `length` returns a `number`.

### Important points

```ts
let name: string = "Rafi";

name = "Mahmud"; // Valid
name = 100;      // Error
```

- `string` is the primitive type.
- Prefer `string`, not the JavaScript wrapper object type `String`.
- A string can contain an empty value: `""`.
- A string containing only spaces is still a non-empty string: `"   "`.

**Avoid:**

```ts
let username: String = "Rafi"; // Avoid the wrapper type
```

**Prefer:**

```ts
let username: string = "Rafi";
```

## 1.2 `number`

The `number` type represents both integer and floating-point numbers.

```ts
let age: number = 23;
let price: number = 499.99;
let temperature: number = -5;
let score: number = 0;
```

JavaScript does not have separate TypeScript types for `int`, `float`, or `double`. They are all represented by `number`.

### Special numeric values

```ts
const positiveInfinity: number = Infinity;
const negativeInfinity: number = -Infinity;
const invalidNumber: number = NaN;
```

`NaN` means “Not a Number”, but its JavaScript type is still `number`.

```ts
console.log(typeof NaN); // "number"
console.log(Number.isNaN(NaN)); // true
```

Do not check for `NaN` using `value === NaN`; that comparison is always false. Use `Number.isNaN(value)`.

### Common operations

```ts
const price = 100;
const quantity = 3;

const total = price * quantity;
const remainder = 10 % 3;
const rounded = Math.round(10.6);
```

### Important points

- `number` supports positive, negative, decimal and special numeric values.
- JavaScript numbers use IEEE 754 double-precision floating-point representation.
- Very large integers may lose precision when represented as `number`; use `bigint` when exact large-integer arithmetic is required.
- Prefer `number`, not the wrapper type `Number`.

```ts
let amount: number = 500;
amount = "500"; // Error
```

## 1.3 `boolean`

The `boolean` type represents logical values:

- `true`
- `false`

```ts
let isLoggedIn: boolean = true;
let hasPermission: boolean = false;
```

### Boolean expressions

Comparison and logical expressions commonly produce boolean values.

```ts
const age = 20;

const isAdult: boolean = age >= 18;
const canVote: boolean = isAdult && true;
const isGuest: boolean = !isAdult;
```

### Important distinction: Boolean vs truthy/falsy values

JavaScript has truthy and falsy values, but those values are not all of type `boolean`.

```ts
const username = "";       // string
const count = 0;           // number
const emptyValue = null;   // null
const isActive = false;    // boolean
```

A condition can convert a truthy/falsy value into a decision, but the original value keeps its own type.

```ts
if (username) {
    console.log("Username exists");
}
```

Use `boolean`, not the wrapper type `Boolean`.

## 1.4 `bigint`

The `bigint` type represents integers larger than the safe integer range of JavaScript's `number` type.

A BigInt literal is written with an `n` suffix.

```ts
const bigNumber: bigint = 9007199254740993n;
const anotherBigNumber = BigInt("9007199254740993");
```

### Why do we need `bigint`?

JavaScript's maximum safe integer is:

```ts
Number.MAX_SAFE_INTEGER;
// 9007199254740991
```

Beyond this range, `number` cannot reliably represent every integer exactly.

```ts
const regularNumber = 9007199254740993;
console.log(regularNumber);
// Precision may be lost
```

Use `bigint` for exact integer arithmetic beyond the safe integer range.

```ts
const a = 9007199254740993n;
const b = 10n;

const result = a + b;

console.log(result);
```

### BigInt rules

BigInt and number cannot be mixed directly in arithmetic:

```ts
const result = 10n + 5; // Error
```

Convert intentionally:

```ts
const result = 10n + BigInt(5); // 15n
```

Division between BigInts truncates the fractional part:

```ts
console.log(7n / 2n); // 3n
```

BigInt is not directly supported by `JSON.stringify()`:

```ts
JSON.stringify({ amount: 10n }); // Throws TypeError
```

For JSON APIs, convert BigInt to a string or define a deliberate serialization strategy.

### Compiler target

BigInt requires an appropriate JavaScript target/runtime. In a TypeScript project, use an ES2020-or-newer target and a runtime that supports BigInt.

```json
{
  "compilerOptions": {
    "target": "ES2020"
  }
}
```

## 1.5 `symbol`

The `symbol` type represents unique identifiers.

A symbol is created with `Symbol()`.

```ts
const id: symbol = Symbol("id");
const anotherId: symbol = Symbol("id");

console.log(id === anotherId); // false
```

Even when both symbols have the same description, they are different and unique values.

### Why use symbols?

Symbols can be used as object property keys when you need a key that will not accidentally collide with ordinary string keys.

```ts
const userId = Symbol("userId");

const user = {
    name: "Rafi",
    [userId]: 101
};

console.log(user[userId]); // 101
```

The symbol key is not returned by ordinary string-key enumeration methods such as `Object.keys()`.

```ts
console.log(Object.keys(user)); // ["name"]
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(userId)]
```

### `symbol` vs `unique symbol`

`symbol` is the general symbol type. `unique symbol` represents the identity of one specific symbol declaration.

```ts
const appId: unique symbol = Symbol("appId");

type AppId = typeof appId;
```

A `unique symbol` declaration must be `const` or `static readonly` in the relevant declaration context. You will encounter it less often than ordinary `symbol`.

## 1.6 `null`

The `null` type represents an intentional absence of a value.

```ts
let selectedUser: null = null;
```

A common use is to represent a value that is currently unavailable.

```ts
type User = {
    id: string;
    name: string;
};

let currentUser: User | null = null;

currentUser = {
    id: "u101",
    name: "Rafi"
};

currentUser = null; // Valid
```

### `null` and `strictNullChecks`

When `strictNullChecks` is enabled, `null` is not assignable to ordinary types such as `string`, `number`, or `User`.

```ts
let username: string = "Rafi";

username = null; // Error with strictNullChecks enabled
```

If null is a valid possibility, include it explicitly:

```ts
let username: string | null = null;

username = "Rafi"; // Valid
username = null;   // Valid
```

### JavaScript quirk

```ts
console.log(typeof null); // "object"
```

This is a historical JavaScript behavior. In TypeScript, the type of `null` is still `null`, not `object`.

## 1.7 `undefined`

The `undefined` type represents a value that has not been assigned or is absent.

```ts
let result: undefined = undefined;
```

A variable declared without an initial value receives `undefined` at runtime when it is a local variable that is not definitely assigned before use.

```ts
let username: string | undefined;

username = "Rafi";
username = undefined;
```

### Optional object properties

An optional property may be absent.

```ts
type User = {
    name: string;
    nickname?: string;
};
```

With `strictNullChecks`, reading `nickname` produces a type that includes `undefined`, because the property may not exist.

```ts
const user: User = {
    name: "Rafi"
};

console.log(user.nickname); // string | undefined
```

### Optional function parameters

```ts
function greet(name?: string): void {
    console.log(name);
}

greet("Rafi");
greet();
```

An optional parameter can be omitted, so inside the function its value may be `undefined`.

### `null` vs `undefined`

| `null` | `undefined` |
|---|---|
| Usually represents intentional absence | Usually represents missing or not-yet-assigned value |
| Must be assigned explicitly in typical application code | Commonly appears when a value is omitted or not initialized |
| Type is `null` | Type is `undefined` |
| Often used in explicit API/domain contracts | Often appears with optional properties and parameters |

These are conventions, not strict rules imposed by JavaScript. Choose a consistent convention for your application.

---

# 2. Special Types

The following types are not all primitive types. They have special roles in TypeScript's type system:

- `any`
- `unknown`
- `never`
- `void`
- `object`

## 2.1 `any`

`any` disables most type checking for a value.

```ts
let data: any = "Hello";

data = 100;
data = true;
data = {};
```

You can perform operations on an `any` value without TypeScript checking whether those operations are safe.

```ts
let value: any = "hello";

value.toUpperCase(); // No type error
value.toFixed(2);    // No type error, but fails at runtime
value.notExisting(); // No type error, but may fail at runtime
```

### Why is `any` risky?

`any` effectively opts out of type safety. It allows invalid operations to pass compilation.

```ts
function processData(data: any) {
    return data.user.profile.name;
}

processData(null); // Can fail at runtime
```

### When might `any` appear?

- Migrating a legacy JavaScript codebase gradually.
- Working temporarily with an untyped library.
- A short-lived escape hatch when a better type is not yet available.

Prefer a proper type or `unknown` whenever possible.

**Rule:** Avoid using `any` as a shortcut. It removes useful compiler checks.

## 2.2 `unknown`

`unknown` can hold a value of any type, but TypeScript does not allow you to use that value until you narrow or check its type.

```ts
let value: unknown;

value = "Hello";
value = 100;
value = true;
value = { name: "Rafi" };
```

Unlike `any`, TypeScript protects you from using the value unsafely.

```ts
let value: unknown = "Hello";

value.toUpperCase(); // Error
```

First check the type:

```ts
if (typeof value === "string") {
    console.log(value.toUpperCase()); // Valid
}
```

### Type narrowing with `unknown`

```ts
function printValue(value: unknown): void {
    if (typeof value === "string") {
        console.log(value.toUpperCase());
    } else if (typeof value === "number") {
        console.log(value.toFixed(2));
    } else if (typeof value === "boolean") {
        console.log(value ? "Yes" : "No");
    } else {
        console.log("Unsupported value");
    }
}
```

TypeScript narrows `value` inside each branch.

### `unknown` vs `any`

| `any` | `unknown` |
|---|---|
| Accepts values of any type | Accepts values of any type |
| Allows operations without checks | Requires narrowing before most operations |
| Can hide type errors | Preserves type safety |
| Avoid when possible | Prefer for values whose type is not yet known |

### Real-world use: external data

Data from an API, request body, file, or other external source should not be trusted merely because it exists.

```ts
function handleExternalData(data: unknown): void {
    if (
        typeof data === "object" &&
        data !== null &&
        "name" in data &&
        typeof data.name === "string"
    ) {
        console.log(data.name.toUpperCase());
    }
}
```

This is a basic runtime check. For complex request payloads, use a runtime validation library such as Zod. TypeScript types alone do not validate incoming JSON.

## 2.3 `never`

`never` represents a value that never occurs.

It is used for code paths that cannot complete normally by returning a value, or for values that cannot exist after type narrowing.

### Example 1: Function that never returns normally

```ts
function throwError(message: string): never {
    throw new Error(message);
}
```

The function always throws, so it never returns a normal value.

An infinite loop is another example:

```ts
function runForever(): never {
    while (true) {
        // Keeps running
    }
}
```

### Example 2: Exhaustive checking

`never` is especially useful when handling every possible member of a union.

```ts
type Status = "pending" | "success" | "failed";

function handleStatus(status: Status): string {
    switch (status) {
        case "pending":
            return "Please wait";

        case "success":
            return "Completed";

        case "failed":
            return "Something went wrong";

        default: {
            const exhaustiveCheck: never = status;
            return exhaustiveCheck;
        }
    }
}
```

Because all possible values are handled, TypeScript narrows `status` to `never` in the `default` branch.

Now imagine someone adds another value:

```ts
type Status = "pending" | "success" | "failed" | "cancelled";
```

The exhaustive check produces a compile-time error until `"cancelled"` is handled. This helps prevent missing cases when a union grows.

### Example 3: Impossible intersection

```ts
type Impossible = string & number;
// never
```

A value cannot be both a string and a number at the same time, so the intersection reduces to `never`.

### Important distinction: `never` vs `void`

- `void`: a function does not provide a useful return value to its caller.
- `never`: a function does not complete normally with a return value at all.

More on `void` below.

## 2.4 `void`

`void` is commonly used as a function return type when the caller should not expect a meaningful return value.

```ts
function logMessage(message: string): void {
    console.log(message);
}
```

The function performs an action but does not return a useful result.

```ts
const result = logMessage("Hello");
```

The result is `undefined` at runtime.

### `void` vs `undefined`

They are related, but they are not interchangeable in every type position.

```ts
function logMessage(): void {
    console.log("Hello");
}

function getNothing(): undefined {
    return undefined;
}
```

- `void` describes a return contract where the returned value is not intended to be used.
- `undefined` is a specific value and can be explicitly returned.

A function explicitly annotated with `undefined` must return `undefined` on every reachable return path (or otherwise satisfy the function's return requirements).

### Important callback behavior

A callback typed as returning `void` means the caller will ignore its return value. A function that returns a value can still often be used where a `void`-returning callback is expected.

```ts
const numbers = [1, 2, 3];

numbers.forEach((number): void => {
    console.log(number);
});
```

For example, `forEach` expects a callback whose result is ignored.

### When to use `void`

Use `void` for functions whose purpose is an action rather than producing a result:

- Logging
- Updating state
- Sending a notification
- Writing to a stream
- Performing a side effect

Do not use `void` to mean that a function can never finish; use `never` for that.

## 2.5 `object`

The `object` type represents non-primitive values.

It accepts objects, arrays, and functions, but rejects primitive values such as strings, numbers, booleans, symbols, BigInts, `null`, and `undefined`.

```ts
let value: object;

value = { name: "Rafi" }; // Valid
value = [1, 2, 3];        // Valid
value = new Date();       // Valid
value = () => {};         // Valid

value = "Rafi"; // Error
value = 100;    // Error
value = true;   // Error
```

### `object` vs `{}` vs `Object`

These types are easy to confuse.

- `object`: non-primitive values.
- `{}`: any non-nullish value in TypeScript; this includes primitives such as strings and numbers.
- `Object`: the broad JavaScript object wrapper type; generally avoid using it as a type.

For example:

```ts
let a: object;
a = { id: 1 }; // Valid
a = "hello";   // Error

let b: {};
b = { id: 1 }; // Valid
b = "hello";   // Valid
b = 100;       // Valid
b = null;      // Error with strictNullChecks

let c: Object;
c = "hello";   // Allowed, but generally avoid Object
```

### `object` does not describe an object's shape

This is valid:

```ts
const user: object = {
    name: "Rafi",
    age: 23
};
```

But the `object` type does not tell TypeScript that `name` and `age` exist.

```ts
user.name; // Error: property 'name' is not known on type 'object'
```

If you need to describe properties, use an object type, `type` alias, or `interface`.

```ts
type User = {
    name: string;
    age: number;
};

const user: User = {
    name: "Rafi",
    age: 23
};

console.log(user.name); // Valid
```

---

# 3. Important Comparisons

## 3.1 `any` vs `unknown` vs `never`

These three types have very different purposes.

| Type | Main meaning | Can accept arbitrary values? | Can use without narrowing? |
|---|---|---:|---:|
| `any` | Opt out of type checking | Yes | Yes |
| `unknown` | Value is not yet known | Yes | No, generally requires checks |
| `never` | Impossible value / no normal completion | No ordinary value can be assigned | Not applicable |

A useful mental model:

- `any`: “Do not check this for me.”
- `unknown`: “I do not know what this is yet; make me prove it.”
- `never`: “There is no possible value here.”

## 3.2 `void` vs `never`

```ts
function saveData(): void {
    console.log("Saved");
}

function stopApplication(): never {
    throw new Error("Application stopped");
}
```

| `void` | `never` |
|---|---|
| No meaningful result is provided | No normal result can be produced |
| Function can finish normally | Function does not finish normally |
| Common for side-effect functions | Common for throwing or non-terminating functions |

## 3.3 `null` vs `undefined` vs `void`

| Type | Meaning |
|---|---|
| `null` | Explicitly absent value |
| `undefined` | Missing or undefined value |
| `void` | Function return contract where the result is not useful to the caller |

## 3.4 Primitive types vs `object`

```ts
let primitive: string = "Rafi";
let nonPrimitive: object = { name: "Rafi" };
```

Use a specific primitive type when representing primitive data. Use a structured object type when you need to describe an object's properties.

---

# 4. Practical Examples

## 4.1 User model

```ts
type User = {
    id: string;
    name: string;
    age: number;
    isActive: boolean;
    nickname?: string;
};

const user: User = {
    id: "u101",
    name: "Rafi",
    age: 23,
    isActive: true
};
```

Here:
- `string`, `number`, and `boolean` are primitive types.
- `nickname?` is optional and may be `undefined` when read.
- `User` is a structured object type.

## 4.2 Safe handling of unknown input

```ts
function getStringLength(value: unknown): number {
    if (typeof value === "string") {
        return value.length;
    }

    throw new Error("Expected a string");
}
```

The function accepts an unknown value, checks it, and only then uses it as a string.

## 4.3 Function that may return a user or no user

```ts
type User = {
    id: string;
    name: string;
};

function findUser(id: string): User | null {
    if (id === "u101") {
        return {
            id: "u101",
            name: "Rafi"
        };
    }

    return null;
}

const user = findUser("u101");

if (user !== null) {
    console.log(user.name);
}
```

`User | null` makes the possibility of “not found” explicit.

## 4.4 Exhaustive state handling

```ts
type PaymentState = "pending" | "paid" | "failed";

function getPaymentMessage(state: PaymentState): string {
    switch (state) {
        case "pending":
            return "Payment is processing";

        case "paid":
            return "Payment completed";

        case "failed":
            return "Payment failed";

        default: {
            const impossible: never = state;
            return impossible;
        }
    }
}
```

This pattern helps catch missing cases when the union type changes.

---

# 5. Common Mistakes

## Mistake 1: Using `String`, `Number`, or `Boolean`

Avoid wrapper types:

```ts
let username: String = "Rafi";
let age: Number = 23;
let active: Boolean = true;
```

Prefer primitive types:

```ts
let username: string = "Rafi";
let age: number = 23;
let active: boolean = true;
```

## Mistake 2: Treating `unknown` like `any`

```ts
let data: unknown = "Hello";

data.toUpperCase(); // Error
```

Narrow first:

```ts
if (typeof data === "string") {
    data.toUpperCase(); // Valid
}
```

## Mistake 3: Using `any` for external data

Avoid:

```ts
function handleRequest(body: any) {
    console.log(body.user.name);
}
```

Prefer `unknown` plus runtime validation:

```ts
function handleRequest(body: unknown) {
    if (
        typeof body === "object" &&
        body !== null &&
        "user" in body
    ) {
        // Continue with appropriate validation of nested data.
    }
}
```

For production request validation, a schema validator is usually more maintainable than manually checking deeply nested structures.

## Mistake 4: Thinking `object` means any JSON-like object with known properties

```ts
const user: object = { name: "Rafi" };

user.name; // Error
```

Use a shape:

```ts
type User = {
    name: string;
};

const user: User = { name: "Rafi" };
user.name; // Valid
```

## Mistake 5: Confusing `void` and `never`

A logging function usually returns `void`; a function that always throws can return `never`.

```ts
function log(): void {
    console.log("Hello");
}

function fail(): never {
    throw new Error("Failed");
}
```

## Mistake 6: Forgetting `strictNullChecks`

With `strictNullChecks: true`, `null` and `undefined` must be handled explicitly.

```ts
let name: string = null; // Error with strictNullChecks
```

Prefer:

```ts
let name: string | null = null;
```

A project using `"strict": true` enables `strictNullChecks`.

---

# 6. Quick Revision

| Type | Category | What it represents | Example |
|---|---|---|---|
| `string` | Primitive | Text | `"Rafi"` |
| `number` | Primitive | Integers and decimals | `23`, `19.5` |
| `boolean` | Primitive | Logical value | `true` |
| `bigint` | Primitive | Arbitrary-precision integer | `123n` |
| `symbol` | Primitive | Unique identifier | `Symbol("id")` |
| `null` | Primitive | Intentional absence | `null` |
| `undefined` | Primitive | Undefined/missing value | `undefined` |
| `any` | Special | Disables most type checking | `let x: any` |
| `unknown` | Special | Unknown value that must be narrowed | `let x: unknown` |
| `never` | Special | Impossible value / no normal return | `throw new Error()` |
| `void` | Special | No useful function return value | `function log(): void` |
| `object` | Special | Non-primitive values | `{ id: 1 }` |

## `typeof` at a glance

JavaScript's `typeof` operator is useful for runtime checks, but remember these results:

```ts
typeof "hello";     // "string"
typeof 42;          // "number"
typeof true;        // "boolean"
typeof 10n;         // "bigint"
typeof Symbol();    // "symbol"
typeof undefined;   // "undefined"
typeof null;        // "object" (historical quirk)
typeof {};          // "object"
typeof [];          // "object"
typeof (() => {});  // "function"
```

`typeof` is a runtime JavaScript operator. It does not validate complex object structures.

---

# 7. Practice Exercises

Try these without copying the examples.

1. **Primitive Types:** Create variables for a username, account balance, login state, a large integer ID, a unique symbol key, an explicitly empty selected item, and an undefined optional result. Give each an appropriate type.
2. **Number Precision:** Explain why `number` may not safely represent every integer above `Number.MAX_SAFE_INTEGER`. Demonstrate the same calculation with `bigint`.
3. **Unknown Input:** Write a function that accepts `unknown` and returns the uppercase value only when the input is a string. Decide what it should do for other input types.
4. **Any vs Unknown:** Write two short examples showing how `any` permits an unsafe operation while `unknown` requires narrowing.
5. **Never:** Write a function that always throws an error and annotate its return type correctly.
6. **Exhaustive Check:** Create a union called `OrderStatus` with `"pending"`, `"paid"`, and `"cancelled"`. Handle every case in a `switch` and add a `never` check.
7. **Void:** Create a function that accepts a message and logs it. Give it the correct return type.
8. **Object vs Object Shape:** Show why a variable typed as `object` cannot safely access a `name` property. Then define a proper `User` type and fix it.
9. **Nullability:** Write a `findUser()` function that returns either a `User` object or `null`. Handle both outcomes safely.
10. **Runtime Validation:** Explain why annotating an Express request body with a TypeScript type does not validate the actual incoming JSON at runtime.

---

## Recommended Configuration

For a TypeScript application, enable strict checking:

```json
{
    "compilerOptions": {
        "strict": true,
        "target": "ES2020"
    }
}
```

`strict: true` enables a family of stronger type checks, including `strictNullChecks` and `noImplicitAny`. The ES2020 target is appropriate when using BigInt syntax, provided the runtime supports it.

## Key Takeaways

- Use lowercase primitive types: `string`, `number`, `boolean`, `bigint`, `symbol`, `null`, and `undefined`.
- Prefer `unknown` over `any` when a value's type is not known.
- Use `never` for impossible states and exhaustive checking.
- Use `void` for functions whose return value is not intended to be consumed.
- Use `object` only when you specifically mean a non-primitive value; use a `type` or `interface` to describe properties.
- Enable strict TypeScript settings.
- TypeScript checks types at compile time; use runtime validation for untrusted external data.