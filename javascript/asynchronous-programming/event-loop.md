# JavaScript Event Loop

The **Event Loop** is the mechanism that allows JavaScript to handle asynchronous operations while keeping JavaScript execution single-threaded.

It coordinates:

* Call Stack
* Runtime APIs
* Task Queue
* Microtask Queue
* Promise callbacks
* `async/await` continuations

---

## 1. The Big Picture

JavaScript itself executes code on a **Call Stack**.

But operations such as timers, network requests, and DOM events may take time.

The runtime handles those asynchronous operations and later makes their callbacks/jobs available to JavaScript.

A simplified model:

```text
                  JavaScript Runtime
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
    ┌───────────┐    ┌──────────────┐   Runtime APIs
    │ Call Stack│    │Microtask Queue│   / Async work
    └─────┬─────┘    └──────────────┘
          │
          │          ┌──────────────┐
          └─────────►│  Task Queue  │
                     └──────────────┘

                    ┌─────────────┐
                    │  Event Loop │
                    └─────────────┘
                          │
                          │ coordinates
                          ▼
                    Send queued-work
                      to Call Stack
```

The Event Loop continuously checks when JavaScript can execute queued work.

---

# 2. Synchronous vs Asynchronous Code

### Synchronous

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```text
A
B
C
```

Each statement runs immediately on the Call Stack.

### Asynchronous

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

```text
A
C
B
```

`setTimeout()` does not put its callback directly onto the Call Stack.

The callback becomes eligible later and is eventually handled by the Event Loop.

---

# 3. Call Stack

The Call Stack keeps track of currently executing JavaScript functions.

Example:

```js
function one() {
  two();
}

function two() {
  three();
}

function three() {
  console.log("Hello");
}

one();
```

Execution:

```text
one()
  ↓
two()
  ↓
three()
  ↓
console.log()
```

The stack follows **LIFO**:

> Last In, First Out.

When a function finishes, its execution context is removed from the stack.

---

# 4. Runtime APIs

The JavaScript language itself does not provide everything required for interacting with the outside world.

The runtime/environment provides additional capabilities.

In a browser, these include things such as:

* `setTimeout`
* `fetch`
* DOM APIs
* `addEventListener`
* `localStorage`

These are commonly referred to as **Web APIs**.

For example:

```js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

The timer is handled by the runtime rather than by the JavaScript Call Stack itself.

> **Web APIs are browser runtime capabilities, not JavaScript language features.**

Node.js provides its own runtime APIs instead of browser Web APIs.

---

# 5. Task Queue

Callbacks from many asynchronous operations eventually become tasks waiting for execution.

Example:

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

```text
A
C
B
```

Conceptually:

```text
1. console.log("A")
2. setTimeout() registers timer
3. console.log("C")
4. timer callback becomes eligible
5. callback waits for execution
6. Event Loop eventually allows it to run
```

### Important

```js
setTimeout(fn, 0);
```

does **not** mean:

> Run `fn` immediately.

It means the callback can become eligible after the minimum delay.

The callback may execute later if JavaScript is busy.

---

# 6. Microtask Queue

Microtasks have higher priority than normal tasks.

Common microtasks include:

* Promise reactions (`then`, `catch`, `finally`)
* `queueMicrotask()`

Example:

```js
console.log("A");

Promise.resolve().then(() => {
  console.log("B");
});

console.log("C");
```

Output:

```text
A
C
B
```

The `.then()` callback becomes a microtask.

---

# 7. Task vs Microtask

Consider:

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

console.log("4");
```

Output:

```text
1
4
3
2
```

Why?

```text
Synchronous code
    ↓
1
4
    ↓
Drain microtasks
    ↓
3
    ↓
Run next task
    ↓
2
```

### Core Rule

> After the current synchronous task finishes, JavaScript drains the microtask queue before moving to the next task.

---

# 8. Microtasks Are Fully Drained

Consider:

```js
Promise.resolve().then(() => {
  console.log("A");

  Promise.resolve().then(() => {
    console.log("B");
  });

  console.log("C");
});

Promise.resolve().then(() => {
  console.log("D");
});
```

Output:

```text
A
C
D
B
```

Why?

Initially:

```text
Microtask Queue:
[ first .then, second .then ]
```

First microtask runs:

```text
A
C
```

During it, another microtask is added:

```text
Queue:
[ D, B ]
```

Therefore:

```text
D
B
```

### Important

A newly created microtask does not interrupt the currently executing microtask.

It goes to the end of the microtask queue.

---

# 9. Promise and the Event Loop

A Promise itself does not automatically mean its callback runs immediately.

Consider:

```js
Promise.resolve().then(() => {
  console.log("Hello");
});
```

The Promise may already be fulfilled, but the `.then()` callback is scheduled as a microtask.

Therefore:

> **Promise settlement and Promise callback execution are different things.**

---

# 10. Multiple `.then()` Calls

```js
const promise = Promise.resolve();

promise.then(() => console.log("A"));
promise.then(() => console.log("B"));
promise.then(() => console.log("C"));
```

The callbacks are queued in registration order:

```text
A
B
C
```

---

# 11. Promise Chaining

Consider:

```js
Promise.resolve()
  .then(() => console.log("A"))
  .then(() => console.log("B"));
```

The second `.then()` does not immediately enter the microtask queue.

The first `.then()` must execute and settle the Promise returned by it.

Conceptually:

```text
First .then()
    ↓
executes
    ↓
returned Promise settles
    ↓
Second .then() becomes eligible
    ↓
Second microtask executes
```

---

# 12. Returning Values Through Promises

```js
Promise.resolve(10)
  .then(value => {
    return value * 2;
  })
  .then(value => {
    console.log(value);
  });
```

Output:

```text
20
```

Each `.then()` returns a new Promise.

The returned value becomes the fulfillment value of that new Promise.

---

# 13. `async` Functions

An `async` function always returns a Promise.

```js
async function test() {
  return 10;
}

const result = test();

console.log(result);
```

`result` is a Promise, not the raw number `10`.

Conceptually:

```text
test()
 ↓
Promise
 ↓
fulfilled with 10
```

---

# 14. `await`

`await` pauses the current `async` function until the awaited Promise settles.

It does **not** block the entire JavaScript program.

Example:

```js
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

console.log("1");

test();

console.log("2");
```

Output:

```text
1
A
2
B
```

Execution:

```text
console.log("1")
    ↓
test()
    ↓
A
    ↓
await
    ↓
test() pauses
    ↓
console.log("2")
    ↓
microtask
    ↓
B
```

### Key Rule

> `await` pauses the current async function, not the entire JavaScript runtime.

---

# 15. `await` and Returned Values

Consider:

```js
async function test() {
  const result = await Promise.resolve(15);
  return result;
}
```

`test()` returns:

```text
Promise
```

That Promise eventually fulfills with:

```text
15
```

Therefore:

```js
const value = await test();
```

gives:

```text
15
```

So:

```text
test()
    ↓
Promise
    ↓
fulfilled with 15
    ↓
await test()
    ↓
15
```

---

# 16. Tricky Example

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");

  Promise.resolve().then(() => {
    console.log("4");
  });
});

console.log("5");
```

Output:

```text
1
5
3
4
2
```

Execution:

```text
Synchronous:
1
5

Microtasks:
3
4

Task:
2
```

---

# 17. `async/await` + Promise + Timer

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

async function test() {
  console.log("4");

  await Promise.resolve();

  console.log("5");
}

test();

console.log("6");
```

Output:

```text
1
4
6
3
5
2
```

Why?

Initial synchronous execution:

```text
1
4
6
```

Microtask queue:

```text
3
5
```

Then timer:

```text
2
```

Therefore:

```text
1 → 4 → 6 → 3 → 5 → 2
```

---

# 18. Microtask Starvation

A microtask can continuously create another microtask:

```js
function spam() {
  queueMicrotask(() => {
    console.log("M");
    spam();
  });
}

spam();

setTimeout(() => {
  console.log("T");
}, 0);
```

Conceptually:

```text
M
M
M
M
M
...
```

The timer may never get a chance to execute because the microtask queue is continuously replenished.

This is called:

> **Microtask starvation**

---

# 19. Synchronous Recursion vs Asynchronous Recursion

### Synchronous recursion

```js
function hello() {
  hello();
}

hello();
```

Each call creates another stack frame:

```text
hello()
  ↓
hello()
  ↓
hello()
  ↓
...
```

Eventually:

```text
Stack Overflow
```

### Microtask recursion

```js
function spam() {
  queueMicrotask(() => {
    spam();
  });
}

spam();
```

Here each function schedules future work and returns.

Therefore the Call Stack can repeatedly become empty.

The problem becomes:

```text
Microtask starvation
```

rather than traditional stack overflow.

---

# 20. Browser vs Node.js

JavaScript needs a runtime environment.

### Browser

```text
JavaScript
    ↓
JavaScript Engine
    ↓
Browser Runtime
    ↓
DOM / Web APIs / Network / Timers
```

### Node.js

```text
JavaScript
    ↓
V8
    ↓
Node.js Runtime
    ↓
OS / File System / Network / Timers / etc.
```

V8 is the JavaScript engine used by Node.js.

Node.js provides the runtime environment and additional capabilities needed to run JavaScript outside the browser.

---

# 21. Node.js Event Loop — Basic Understanding

For backend development, it is useful to understand the basic relationship between Node.js and the Event Loop.

For example:

```js
setTimeout(() => {
  console.log("Hello");
}, 0);
```

Node registers the timer.

The callback does not execute immediately.

It becomes eligible for execution through Node's Event Loop and timers processing.

A useful beginner mental model is:

```text
setTimeout()
    ↓
Timer registered
    ↓
Delay threshold
    ↓
Callback becomes eligible
    ↓
Event Loop
    ↓
Callback executes
```

### Important

```js
setTimeout(fn, 1000);
```

does not guarantee:

```text
exactly 1000ms → fn()
```

It means the callback cannot execute before the timer's delay threshold has been reached, and actual execution can happen later depending on the runtime's state.

---

# 22. The Most Important Mental Model

When tracing JavaScript asynchronous code, use this order:

```text
1. Execute current synchronous code
            ↓
2. Microtask Queue
            ↓
3. Drain ALL microtasks
            ↓
4. Next Task
            ↓
5. Drain ALL microtasks again
            ↓
6. Next Task
            ↓
        Repeat
```

The key is:

> **Microtasks are drained before moving on to the next task.**

---

# 23. Interview Checklist

When you see an Event Loop problem, ask:

### Step 1

What runs synchronously?

### Step 2

What gets scheduled?

### Step 3

Which callbacks become microtasks?

### Step 4

Which callbacks become tasks?

### Step 5

What is the order of the microtask queue?

### Step 6

Do any microtasks create additional microtasks?

### Step 7

Does `await` pause the current async function?

### Step 8

When does the Promise returned by an `async` function settle?

### Step 9

Could microtasks starve tasks?

---

# 24. Common Interview Traps

### Trap 1

```js
setTimeout(fn, 0);
```

does not mean:

```text
run immediately
```

It means the callback becomes eligible after the timer threshold.

---

### Trap 2

A fulfilled Promise does not mean its `.then()` callback runs synchronously.

```js
Promise.resolve().then(fn);
```

`fn` runs as a microtask.

---

### Trap 3

`await` does not block the entire program.

It pauses the current async function.

---

### Trap 4

A microtask created inside another microtask does not interrupt the current microtask.

It is added to the queue.

---

### Trap 5

An `async` function does not mean its entire body is asynchronous.

Code before the first `await` executes synchronously when the function is called.

---

### Trap 6

A Promise object is not itself "the microtask".

The Promise reaction/callback is what gets scheduled as a microtask.

---

# 25. One-Line Mental Model

> **Call Stack executes JavaScript, the runtime handles asynchronous operations, queues hold callbacks/jobs waiting for execution, and the Event Loop coordinates when that work can return to JavaScript—with microtasks drained before the next task.**

---

# 26. Final Execution Pattern

For most interview problems, visualize:

```text
                 ┌───────────────┐
                 │  Call Stack   │
                 └───────┬───────┘
                         │
                         ▼
                 Current JS finishes
                         │
                         ▼
              ┌────────────────────┐
              │  Microtask Queue   │
              └─────────┬──────────┘
                        │
                        ▼
                 Drain ALL of them
                        │
                        ▼
                 ┌──────────────┐
                 │  Next Task   │
                 └──────┬───────┘
                        │
                        ▼
              Drain microtasks again
                        │
                        ▼
                      Repeat
```

This mental model is enough to solve a large class of JavaScript asynchronous execution-order problems.

---

## Summary

The Event Loop is not responsible for executing JavaScript itself.

The **Call Stack executes JavaScript**.

The runtime handles asynchronous operations.

The Event Loop coordinates when queued work can be executed.

The most important priority rule is:

```text
Synchronous code
      ↓
Microtasks
      ↓
Next task
      ↓
Microtasks
      ↓
Next task
      ↓
...
```

And the most important concepts to remember are:

* Call Stack
* Runtime APIs
* Task Queue
* Microtask Queue
* Promise reactions
* `async/await`
* Microtask starvation
* Browser vs Node.js runtime
