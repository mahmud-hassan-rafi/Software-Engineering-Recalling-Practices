# JavaScript Promises

> A practical and mental-model-based guide to JavaScript Promises, chaining, error propagation, Promise combinators, microtasks, and async/await.

---

## 1. Why Do We Need Promises?

Before Promises, asynchronous operations were commonly handled with callbacks.

```js
getUser(userId, (user) => {
  getOrders(user.id, (orders) => {
    getPayments(orders, (payments) => {
      // ...
    });
  });
});
```

As the number of dependent operations grows, callbacks become deeply nested.

This is commonly called **Callback Hell**.

Promises provide a structured way to represent and compose asynchronous operations.

---

# 2. What Is a Promise?

A Promise is an object that represents the **eventual result of an asynchronous operation**.

It can be in one of three states:

```text
Pending
   ↓
Fulfilled
   OR
Rejected
```

### States

| State       | Meaning                          |
| ----------- | -------------------------------- |
| `pending`   | Operation is still in progress   |
| `fulfilled` | Operation completed successfully |
| `rejected`  | Operation failed                 |

A Promise can settle only once.

```text
pending
   ↓
fulfilled
```

or

```text
pending
   ↓
rejected
```

It cannot go back to `pending`, and once settled, later `resolve()`/`reject()` calls have no effect.

---

# 3. Creating a Promise

```js
const promise = new Promise((resolve, reject) => {
  // asynchronous work

  resolve("Success");
});
```

`resolve()` means:

> The operation succeeded.

`reject()` means:

> The operation failed.

Example:

```js
const promise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("Done");
  } else {
    reject("Something went wrong");
  }
});
```

---

# 4. Promise Executor Runs Synchronously

This is a very important point.

```js
console.log("A");

const promise = new Promise((resolve) => {
  console.log("B");

  resolve();
});

console.log("C");
```

Output:

```text
A
B
C
```

The function passed to `new Promise()` executes immediately.

### But `.then()` does NOT execute immediately.

```js
const promise = new Promise((resolve) => {
  resolve();
});

promise.then(() => {
  console.log("B");
});

console.log("C");
```

Output:

```text
C
B
```

The Promise reaction is handled asynchronously through the **microtask queue**.

---

# 5. `then()`

```js
promise.then(value => {
  console.log(value);
});
```

The callback passed to `.then()` runs when the Promise fulfills.

Example:

```js
Promise.resolve(10)
  .then(value => {
    console.log(value);
  });
```

Output:

```text
10
```

---

# 6. `.then()` Always Returns a New Promise

This is one of the most important Promise rules.

```js
const p1 = Promise.resolve(10);

const p2 = p1.then(value => {
  return value * 2;
});
```

Here:

```text
p1
 ↓
.then()
 ↓
p2
```

`p2` is a **new Promise**.

What happens to `p2` depends on what the callback returns.

---

# 7. The Three Outcomes of a `.then()` Callback

## Case 1 — Return a normal value

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

Mental model:

```text
.then()
   ↓
return 20
   ↓
new Promise → fulfilled with 20
```

---

## Case 2 — Return another Promise

```js
Promise.resolve(10)
  .then(value => {
    return Promise.resolve(value * 2);
  })
  .then(value => {
    console.log(value);
  });
```

Output:

```text
20
```

The next `.then()` waits for the returned Promise.

A more realistic example:

```js
getUser()
  .then(user => {
    return fetch(`/orders/${user.id}`);
  })
  .then(response => {
    // waits for fetch()
  });
```

Here `fetch()` returns a Promise, so the chain waits for it.

### Mental model

```text
callback returns value
        ↓
new Promise fulfilled with value

callback returns Promise
        ↓
new Promise follows/adopts that Promise
```

---

## Case 3 — Throw an error

```js
Promise.resolve(10)
  .then(value => {
    throw new Error("Oops");
  })
  .then(value => {
    console.log(value);
  });
```

The second `.then()` does not run.

The Promise returned by the first `.then()` becomes rejected.

```text
throw Error
     ↓
new Promise → rejected
     ↓
normal .then() skipped
```

---

# 8. `.catch()`

`.catch()` handles Promise rejection.

```js
Promise.reject("ERROR")
  .catch(error => {
    console.log(error);
  });
```

Output:

```text
ERROR
```

Conceptually:

```js
promise.catch(handler);
```

is equivalent to:

```js
promise.then(null, handler);
```

`catch()` is therefore simply a **rejection handler**.

---

# 9. Error Propagation

Suppose:

```js
Promise.resolve(10)
  .then(value => {
    console.log("A", value);

    throw new Error("Oops");
  })
  .then(value => {
    console.log("B", value);
  })
  .catch(error => {
    console.log("C", error.message);
  });
```

Output:

```text
A 10
C Oops
```

Why does `B` not run?

Because:

```text
first .then()
      ↓
throw Error
      ↓
returned Promise becomes rejected
      ↓
next normal .then() is skipped
      ↓
catch() handles rejection
```

### General rule

```text
Rejected Promise
      ↓
skip normal .then()
      ↓
find next rejection handler
      ↓
catch()
```

---

# 10. `catch()` Can Recover the Chain

```js
Promise.reject("ERROR")
  .catch(error => {
    console.log(error);

    return 100;
  })
  .then(value => {
    console.log(value);
  });
```

Output:

```text
ERROR
100
```

Why?

`catch()` returns `100`.

Therefore the Promise returned by `catch()` becomes fulfilled with `100`.

```text
Rejected
   ↓
catch()
   ↓
return 100
   ↓
Fulfilled with 100
   ↓
next .then()
```

### Important

Do not think JavaScript literally does:

```js
Promise.resolve(100);
```

when you write:

```js
return 100;
```

Instead:

> The new Promise returned by the chain becomes fulfilled with `100`.

Thinking of it as "conceptually resolved with 100" is useful, but it is not literally wrapping every normal return in an explicit `Promise.resolve()` call.

---

# 11. `catch()` Can Re-throw

```js
Promise.reject("A")
  .catch(error => {
    console.log("Caught:", error);

    throw new Error("B");
  })
  .then(() => {
    console.log("This will not run");
  })
  .catch(error => {
    console.log("Final:", error.message);
  });
```

Output:

```text
Caught: A
Final: B
```

Mental model:

```text
reject A
   ↓
catch A
   ↓
throw B
   ↓
rejected again
   ↓
next .then() skipped
   ↓
next catch handles B
```

---

# 12. `Promise.resolve()`

```js
const promise = Promise.resolve(10);
```

Creates an already-fulfilled Promise:

```text
fulfilled
value = 10
```

Example:

```js
Promise.resolve(10)
  .then(value => {
    console.log(value);
  });
```

Output:

```text
10
```

---

# 13. `Promise.resolve()` with a Promise

```js
const p = Promise.resolve(
  Promise.resolve(10)
);
```

If `Promise.resolve()` receives a Promise, it **adopts/follows that Promise**.

It does not create a meaningful nested Promise layer.

Mental model:

```text
Promise.resolve(Promise)
        ↓
follow that Promise
        ↓
same eventual result
```

---

# 14. `Promise.reject()`

```js
const p = Promise.reject("ERROR");
```

Creates an already-rejected Promise.

```js
p.catch(error => {
  console.log(error);
});
```

Output:

```text
ERROR
```

---

# 15. Important Difference: `Promise.resolve()` vs `Promise.reject()`

This is a subtle but important distinction.

### `Promise.resolve(Promise)`

It adopts the Promise.

```text
Promise.resolve(P)
       ↓
follows P
```

### `Promise.reject(Promise)`

It does **not** adopt the Promise.

The Promise itself becomes the rejection reason.

```js
const inner = Promise.reject("ERROR");

const outer = Promise.reject(inner);
```

Conceptually:

```text
outer
 ↓
rejected
 ↓
reason = inner Promise
              ↓
           rejected
              ↓
           "ERROR"
```

This distinction is useful for understanding nested Promise behavior.

---

# 16. Promise + Event Loop

Promise callbacks use the **microtask queue**.

Consider:

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

Output:

```text
A
D
C
B
```

Why?

```text
Synchronous code
    ↓
A
D

Microtask queue
    ↓
C

Task/macrotask queue
    ↓
B
```

### Important priority

After the current synchronous task finishes:

```text
Microtasks
   ↓
Next task
```

Microtasks are drained before moving to the next task.

---

# 17. Promise Resolution Inside a Timer

```js
const p = new Promise((resolve) => {
  setTimeout(() => {
    console.log("A");
    resolve("done");
  }, 0);
});

p.then(value => {
  console.log("B", value);
});

console.log("C");
```

Output:

```text
C
A
B done
```

Why?

```text
sync
 ↓
C

timer task
 ↓
A
 ↓
resolve("done")
 ↓
.then() reaction → microtask

microtask
 ↓
B done
```

Important:

> While the Promise is pending, `.then()` is registered, but its callback is not yet scheduled as a microtask. The callback becomes runnable after the Promise settles.

---

# 18. Multiple `.then()` Handlers

```js
const p = new Promise(resolve => {
  setTimeout(() => resolve("done"), 0);
});

p.then(value => {
  console.log("A", value);
});

p.then(value => {
  console.log("B", value);
});
```

When `p` resolves, both handlers are scheduled as separate microtasks.

Output:

```text
A done
B done
```

They execute in registration order.

---

# 19. Promise Chaining — Master Mental Model

Always remember:

```text
.then(callback)
      ↓
   NEW Promise
      ↓
What does callback return?

value
  ↓
new Promise fulfilled

Promise
  ↓
new Promise follows/adopts it

throw
  ↓
new Promise rejected
```

This one model explains a huge portion of Promise behavior.

---

# 20. `Promise.all()`

`Promise.all()` waits for all Promises to fulfill.

```js
Promise.all([
  Promise.resolve("A"),
  Promise.resolve("B"),
  Promise.resolve("C")
])
.then(values => {
  console.log(values);
});
```

Output:

```text
["A", "B", "C"]
```

### Important: Result order

The output follows **input order**, not completion order.

Suppose:

```text
B finishes first
C finishes second
A finishes last
```

Still:

```js
["A", "B", "C"]
```

### Mental model

```text
A ──┐
B ──┼──→ Promise.all() → ["A", "B", "C"]
C ──┘
```

---

# 21. `Promise.all()` Rejects if Any Promise Rejects

```js
Promise.all([
  Promise.resolve("A"),
  Promise.reject("ERROR"),
  Promise.resolve("C")
])
.catch(error => {
  console.log(error);
});
```

Output:

```text
ERROR
```

One rejection causes the `Promise.all()` result to reject.

### Important

`Promise.all()` does **not** cancel the other underlying operations.

They may continue running.

---

# 22. `Promise.allSettled()`

`Promise.allSettled()` waits until **every Promise has settled**, regardless of success or failure.

```js
Promise.allSettled([
  Promise.resolve("A"),
  Promise.reject("B"),
  Promise.resolve("C")
])
.then(results => {
  console.log(results);
});
```

Result:

```js
[
  { status: "fulfilled", value: "A" },
  { status: "rejected", reason: "B" },
  { status: "fulfilled", value: "C" }
]
```

It gives you a complete report.

### Useful when

You have independent operations and want all results even if some fail.

Example:

```text
Fetch profile
Fetch orders
Fetch notifications
Fetch recommendations
```

One failing should not prevent you from receiving the others.

---

# 23. `Promise.race()`

`Promise.race()` settles when the **first Promise settles**.

"Settles" means:

```text
fulfilled OR rejected
```

Example:

```js
const p1 = new Promise(resolve =>
  setTimeout(() => resolve("A"), 3000)
);

const p2 = new Promise(resolve =>
  setTimeout(() => resolve("B"), 1000)
);

Promise.race([p1, p2])
  .then(value => {
    console.log(value);
  });
```

Output:

```text
B
```

Because B settles first.

---

## `race()` Can Reject

```js
const p1 = new Promise(resolve =>
  setTimeout(() => resolve("A"), 3000)
);

const p2 = new Promise((resolve, reject) =>
  setTimeout(() => reject("ERROR"), 1000)
);

Promise.race([p1, p2])
  .catch(error => {
    console.log(error);
  });
```

Output:

```text
ERROR
```

Because the rejection happened first.

### Mental model

```text
race()
 ↓
first settlement wins
 ↓
fulfilled OR rejected
```

---

# 24. `Promise.any()`

`Promise.any()` waits for the **first fulfilled Promise**.

Rejections are ignored until all Promises reject.

```js
Promise.any([
  Promise.reject("A"),
  new Promise(resolve =>
    setTimeout(() => resolve("B"), 2000)
  ),
  new Promise(resolve =>
    setTimeout(() => resolve("C"), 1000)
  )
])
.then(value => {
  console.log(value);
});
```

Output:

```text
C
```

Because C is the first successful result.

---

# 25. `Promise.any()` When Everything Rejects

```js
Promise.any([
  Promise.reject("A"),
  Promise.reject("B"),
  Promise.reject("C")
])
.catch(error => {
  console.log(error);
});
```

It rejects with an `AggregateError`.

The individual reasons are available through:

```js
error.errors
```

Result:

```js
["A", "B", "C"]
```

---

# 26. `all()` vs `allSettled()` vs `race()` vs `any()`

| Method                 | What does it wait for? | If one rejects                     |
| ---------------------- | ---------------------- | ---------------------------------- |
| `Promise.all()`        | Everyone fulfills      | Rejects                            |
| `Promise.allSettled()` | Everyone settles       | Still fulfills with report         |
| `Promise.race()`       | First settlement       | Can fulfill or reject              |
| `Promise.any()`        | First fulfillment      | Ignores rejection until all reject |

### Easy memory trick

```text
all
→ সবাই সফল হও

allSettled
→ সবাই শেষ করো

race
→ যে আগে শেষ করুক, তার result

any
→ যে আগে সফল হোক, তার result
```

---

# 27. `async` Functions

An `async` function **always returns a Promise**.

```js
async function getValue() {
  return 10;
}
```

Even though it looks like it returns a normal value, the actual result is:

```text
Promise
   ↓
fulfilled
   ↓
10
```

Therefore:

```js
getValue().then(value => {
  console.log(value);
});
```

Output:

```text
10
```

---

# 28. `async` Function Throwing an Error

```js
async function getValue() {
  throw new Error("Oops");
}
```

The function returns a rejected Promise.

```js
getValue()
  .catch(error => {
    console.log(error.message);
  });
```

Output:

```text
Oops
```

### Rule

```text
async function
      ↓
always returns Promise

return value
      ↓
fulfilled Promise

throw error
      ↓
rejected Promise
```

---

# 29. `await`

`await` waits for a Promise's result inside an `async` function.

```js
async function test() {
  const value = await Promise.resolve(100);

  console.log(value);
}
```

Output:

```text
100
```

Mental model:

```text
await Promise
      ↓
fulfilled → get the value
rejected  → throw the rejection reason
```

---

# 30. `await` Does NOT Block JavaScript

This is one of the most important concepts.

```js
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

console.log("C");

test();

console.log("D");
```

Output:

```text
C
A
D
B
```

Why?

```text
C
 ↓
test()
 ↓
A
 ↓
await
 ↓
function yields
 ↓
D
 ↓
microtask
 ↓
B
```

`await` suspends the async function's continuation.

It does **not** block the JavaScript thread.

---

# 31. `await` + Rejection

```js
async function test() {
  try {
    const value = await Promise.reject("ERROR");

    console.log(value);
  } catch (error) {
    console.log(error);
  }
}
```

Output:

```text
ERROR
```

A rejected Promise being awaited behaves like an exception inside the async function.

This makes `try/catch` very natural with `async/await`.

---

# 32. Promise Chain vs `async/await`

Promise chain:

```js
fetch("/users")
  .then(response => response.json())
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.log(error);
  });
```

Using `async/await`:

```js
async function getUsers() {
  try {
    const response = await fetch("/users");
    const data = await response.json();

    console.log(data);
  } catch (error) {
    console.log(error);
  }
}
```

`async/await` does not replace Promises.

It provides a cleaner syntax for working with Promise-based asynchronous code.

---

# 33. Sequential vs Parallel `await`

This is extremely important for backend development.

### Sequential

```js
const user = await getUser();
const orders = await getOrders();
```

Execution:

```text
getUser
   ↓
wait
   ↓
getOrders
   ↓
wait
```

If these operations are independent, this may be unnecessarily slow.

---

### Parallel

```js
const [user, orders] = await Promise.all([
  getUser(),
  getOrders()
]);
```

Now both operations are started without waiting for the other:

```text
getUser  ───────┐
                ├──→ Promise.all()
getOrders ──────┘
```

This is often the better approach when the operations are independent.

---

# 34. Common Promise Traps

## Trap 1 — Promise executor is asynchronous

Wrong:

```text
new Promise() → asynchronous
```

Correct:

```text
Promise executor → synchronous
.then/.catch callbacks → asynchronous microtasks
```

---

## Trap 2 — `await` blocks JavaScript

Wrong:

```text
await blocks the whole program
```

Correct:

```text
await suspends the current async function
```

The event loop can continue doing other work.

---

## Trap 3 — `Promise.race()` means first success

Wrong.

`race()` means:

```text
first settlement
```

So rejection can win.

---

## Trap 4 — `Promise.any()` means first settlement

Wrong.

`any()` means:

```text
first fulfillment
```

---

## Trap 5 — `Promise.all()` preserves completion order

Wrong.

It preserves **input order**.

---

## Trap 6 — `Promise.all()` cancels the other Promises

Wrong.

It rejects its own resulting Promise, but underlying operations may continue.

---

## Trap 7 — `return Promise.resolve(value)` is always necessary

Wrong.

These normally produce the same eventual result:

```js
.then(() => {
  return 10;
});
```

and:

```js
.then(() => {
  return Promise.resolve(10);
});
```

The second form is useful when explicitly demonstrating Promise adoption, but normal values do not need to be manually wrapped.

---

# 35. The Complete Promise Mental Model

When debugging a Promise chain, think in this order:

```text
1. What is the current Promise state?
        ↓
2. Is this `.then()` or `.catch()`?
        ↓
3. Will this handler run or be skipped?
        ↓
4. What does the handler return?
        ↓
5. Does it return:
      value?
      Promise?
      throw?
        ↓
6. What is the state of the NEW Promise?
        ↓
7. Which handler receives that result?
```

This mental model is much more powerful than memorizing syntax.

---

# 36. Promise + Event Loop Master Model

Keep this picture in your head:

```text
                 JavaScript Runtime
                        │
                        ▼
                  Call Stack
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
       Web APIs / Runtime      Microtask Queue
       timers, I/O, etc.       Promise reactions
             │                     │
             ▼                     │
        Task Queue                 │
             │                     │
             └──────────┬──────────┘
                        ▼
                  Event Loop
                        │
                        ▼
                Execute callbacks
```

The simplified priority to remember:

```text
Current synchronous code
        ↓
Microtasks
        ↓
Next task
```

And Promise callbacks such as `.then()` / `.catch()` are microtasks.

---

# 37. Final Cheat Sheet

```text
Promise
→ represents an eventual result

new Promise(executor)
→ executor runs synchronously

resolve(value)
→ fulfills the Promise

reject(reason)
→ rejects the Promise

.then()
→ handles fulfillment
→ always returns a NEW Promise

.catch()
→ handles rejection
→ equivalent to .then(null, handler)

return value
→ next Promise becomes fulfilled

return Promise
→ next Promise follows/adopts it

throw error
→ next Promise becomes rejected

Promise.resolve(value)
→ fulfilled Promise

Promise.resolve(Promise)
→ adopts/follows the Promise

Promise.reject(reason)
→ rejected Promise

Promise.all()
→ all fulfill required
→ result preserves input order
→ one rejection rejects the result

Promise.allSettled()
→ waits for everyone
→ gives status/value/reason for each

Promise.race()
→ first settlement wins
→ fulfillment or rejection

Promise.any()
→ first fulfillment wins
→ all reject → AggregateError

async function
→ always returns a Promise

return inside async
→ fulfilled Promise

throw inside async
→ rejected Promise

await Promise
→ fulfilled → value
→ rejected → throws

await
→ does not block the JS thread

Independent async operations
→ consider Promise.all()
```

---

# 38. The One Diagram to Remember

```text
                 PROMISE
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      fulfilled            rejected
          │                   │
        .then()             .catch()
          │                   │
          └─────────┬─────────┘
                    ▼
              NEW PROMISE
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       value      Promise    throw
          │         │         │
          ▼         ▼         ▼
      fulfilled   adopts    rejected
```

And for multiple Promises:

```text
             PROMISE COMBINATORS

all          → সবাই সফল
allSettled   → সবাই শেষ
race         → প্রথম settled
any          → প্রথম fulfilled
```

---

## 🚀 Promise Chapter Status

You should now be comfortable with:

* [x] Promise states
* [x] Creating Promises
* [x] `resolve()` / `reject()`
* [x] Promise executor
* [x] `.then()`
* [x] Promise chaining
* [x] Returning values
* [x] Returning Promises
* [x] Error propagation
* [x] `.catch()`
* [x] Recovery vs re-throw
* [x] Promise microtasks
* [x] Multiple `.then()` handlers
* [x] `Promise.resolve()`
* [x] `Promise.reject()`
* [x] `Promise.all()`
* [x] `Promise.allSettled()`
* [x] `Promise.race()`
* [x] `Promise.any()`
* [x] `async`
* [x] `await`
* [x] `try/catch` with async/await
* [x] Sequential vs parallel async operations
* [x] Promise + Event Loop
* [x] Promise interview traps

> **Core takeaway:** Don't memorize Promise behavior as isolated rules. Follow the Promise chain: **what state is it in → which handler runs → what does that handler return → what happens to the new Promise?**
