# JavaScript: Hoisting, TDZ, and Variable Initialization

## 1. Hoisting

Hoisting is the behavior where JavaScript creates bindings for declarations before executing the code in that scope.

A better mental model is:

> JavaScript does not literally move declarations to the top. During the creation phase, it creates bindings before execution starts.

### `var` Hoisting

A `var` binding is created and initialized with `undefined`.

```js
console.log(a);

var a = 10;

console.log(a);
```

Output:

```txt
undefined
10
```

Conceptually:

```text
Creation:
a → undefined

Execution:
console.log(a) → undefined
a = 10
console.log(a) → 10
```

The assignment happens only when execution reaches:

```js
a = 10;
```

---

## 2. Temporal Dead Zone (TDZ)

`let` and `const` declarations are also created during the creation phase, but their bindings are **not initialized immediately**.

The period between:

1. creation of the binding, and
2. initialization of the binding

is called the **Temporal Dead Zone (TDZ)**.

Accessing the variable during this period causes a `ReferenceError`.

### Example

```js
console.log(a);

let a = 10;
```

Result:

```txt
ReferenceError: Cannot access 'a' before initialization
```

Conceptually:

```text
Creation:
a → uninitialized

Execution:
console.log(a)
      ↓
TDZ is still active
      ↓
ReferenceError

let a = 10
```

Important:

> `a` is already known to the JavaScript engine. It is not an "undefined variable". Its binding exists, but it has not been initialized yet.

---

# 3. Initialization of `let` and `const`

The important question is not simply:

> "Was the variable declared?"

The important question is:

> "Has the variable been initialized yet?"

---

## `let` Without an Initial Value

Consider:

```js
let x;

console.log(x);
```

Output:

```txt
undefined
```

Why?

During creation:

```text
x → uninitialized
```

When execution reaches:

```js
let x;
```

the binding becomes initialized with:

```text
x → undefined
```

Therefore, the following access is valid:

```js
console.log(x); // undefined
```

### Important distinction

```js
console.log(x);

let x;
```

❌ `ReferenceError`

But:

```js
let x;

console.log(x);
```

✅ `undefined`

The difference is **whether initialization has happened before the access**.

---

# `const` Must Be Initialized Immediately

A `const` declaration requires an initializer.

Valid:

```js
const x = 10;

console.log(x);
```

Invalid:

```js
const x;
```

This produces:

```txt
SyntaxError
```

A `const` binding remains in the TDZ until its declaration is initialized.

For example:

```js
console.log(x);

const x = 10;
```

Result:

```txt
ReferenceError
```

Conceptually:

```text
Creation:
x → uninitialized

Execution:
console.log(x)
      ↓
TDZ
      ↓
ReferenceError

const x = 10
      ↓
x → 10
```

---

# `var` vs `let` vs `const`

| Declaration | Binding Created Before Execution? | Initial Value Before Declaration | TDZ? | Must Initialize Immediately? |
| ----------- | --------------------------------- | -------------------------------- | ---- | ---------------------------- |
| `var`       | Yes                               | `undefined`                      | No   | No                           |
| `let`       | Yes                               | Uninitialized                    | Yes  | No                           |
| `const`     | Yes                               | Uninitialized                    | Yes  | Yes                          |

---

# Scope + Hoisting Example

Consider:

```js
var x = 10;

function test() {
  console.log(x);

  var x = 20;

  console.log(x);
}

test();
```

Output:

```txt
undefined
20
```

Why doesn't the first `console.log(x)` access the global `x = 10`?

Because `test()` has its own local `x`.

Conceptually, when `test()` starts:

```text
Global:
x → 10

test's local environment:
x → undefined
```

The first lookup finds the local `x` immediately:

```text
console.log(x)
      ↓
test scope
      ↓
x found
      ↓
undefined
```

Then:

```js
x = 20;
```

changes the local binding:

```text
test's x → 20
```

So:

```js
console.log(x);
```

outputs:

```txt
20
```

The global `x = 10` is never reached because the local `x` already exists.

---

# TDZ + Scope Chain Example

Consider:

```js
let x = 10;

function outer() {
  function inner() {
    console.log(x);
  }

  inner();

  let x = 20;
}

outer();
```

The `inner()` function looks for `x` through its lexical scope chain:

```text
inner scope
    ↓
outer scope
    ↓
global scope
```

It finds `x` in the `outer` scope.

However, at the moment `inner()` runs:

```js
inner();

let x = 20;
```

the outer `x` has not been initialized yet.

Therefore:

```txt
ReferenceError
```

The lookup does **not** continue to the global `x = 10`.

Why?

Because the local `x` already exists in `outer`; it is simply still inside its TDZ.

---

# Contrast: Initialization Before Access

Now change the order:

```js
let x = 10;

function outer() {
  let x;

  function inner() {
    console.log(x);
  }

  inner();
}

outer();
```

Output:

```txt
undefined
```

Here:

```js
let x;
```

has already executed before `inner()` accesses `x`.

Therefore:

```text
outer's x → undefined
```

The TDZ has ended.

So `inner()` can safely read the value:

```js
console.log(x); // undefined
```

---

# Key Mental Model

When analyzing `var`, `let`, and `const`, think in this order:

```text
1. Is a binding created?
        ↓
2. Has the binding been initialized?
        ↓
3. What is its current value?
        ↓
4. Which scope contains the binding?
        ↓
5. Does the access happen before or after initialization?
```

The most important distinction is:

```text
var
→ created + initialized to undefined

let
→ created + uninitialized
→ TDZ
→ initialized when declaration executes
→ undefined if no initializer is provided

const
→ created + uninitialized
→ TDZ
→ must be initialized during declaration
```

## Final Rule

> **TDZ is not about whether a variable exists. It is about whether its binding has been initialized.**
