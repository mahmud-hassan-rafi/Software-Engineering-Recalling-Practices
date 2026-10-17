# Module 03 — Advanced Pattern Construction

This module covers how to combine regex components to build more complex patterns, capture useful data, and assert conditions around a match without consuming characters.

## 1. Grouping — `()`

Parentheses group multiple regex tokens together so that quantifiers and other operations can apply to the entire group.

### Why grouping matters

A quantifier applies only to the immediately preceding token or group.

```js
/ab+/       // a followed by one or more b characters
/(ab)+/     // one or more repetitions of the entire "ab"
```

Example:

```js
const text = "abcabcab";

console.log(/ab+c/.exec(text));
// ["abc"]

console.log(/(abc)+/.exec(text));
// ["abcabc", "abc"]

console.log(/(ab)+c/.exec(text));
// ["abc"]
```

**Key idea:** Use parentheses when you want to treat multiple characters as one unit.

---

## 2. Capturing Groups

A capturing group stores the portion of text matched by its parentheses.

```js
const regex = /(\w+)@(\w+)\.(\w+)/;

const result = regex.exec("rafi@gmail.com");

console.log(result);
```

Output:

```js
[
  "rafi@gmail.com",
  "rafi",
  "gmail",
  "com"
]
```

### Understanding the result

- `result[0]`: Complete match.
- `result[1]`: First capturing group.
- `result[2]`: Second capturing group.
- `result[3]`: Third capturing group.

### Practical example: Extracting date components

```js
const regex = /(\d{4})-(\d{2})-(\d{2})/;

const result = regex.exec("2026-10-17");

console.log(result[1]); // 2026
console.log(result[2]); // 10
console.log(result[3]); // 17
```

**Use capturing groups when you need to extract specific portions of a match.**

---

## 3. Non-Capturing Groups — `(?:...)`

A non-capturing group groups tokens without storing the matched text in the result.

```js
const regex1 = /(abc)+/;
const regex2 = /(?:abc)+/;

console.log(regex1.exec("abcabc"));
// ["abcabc", "abc"]

console.log(regex2.exec("abcabc"));
// ["abcabc"]
```

### Capturing vs Non-Capturing

| Feature | Capturing `()` | Non-Capturing `(?:)` |
|---|---|---|
| Groups tokens | Yes | Yes |
| Stores matched text | Yes | No |
| Appears in result groups | Yes | No |
| Useful for extraction | Yes | No |

### Why use non-capturing groups?

Use them when grouping is necessary but the matched text itself is not needed.

They also prevent unnecessary capturing-group index changes.

```js
const regex = /(?:https?):\/\/(\w+)/;
```

Here, the protocol is grouped but not captured. Only the domain-like portion is captured.

---

## 4. Alternation — `|`

Alternation means OR. It allows a pattern to match one alternative among multiple choices.

```js
/cat|dog/
```

Matches either `cat` or `dog`.

### Examples

```js
const regex = /cat|dog/;

console.log(regex.test("I have a cat")); // true
console.log(regex.test("I have a dog")); // true
console.log(regex.test("I have a bird")); // false
```

### Alternative order matters

```js
/catalog|cat/
```

On `"catalog"`, this matches `"catalog"`.

But:

```js
/cat|catalog/
```

On `"catalog"`, this matches `"cat"` because the first alternative already succeeds at that position.

**Practical rule:** When alternatives overlap, put the more specific or longer alternative first if that is the intended behavior.

---

## 5. Grouping + Alternation

Grouping and alternation can be combined to create controlled choices.

```js
const regex = /^(cat|dog|bird)$/;
```

This matches exactly one of:

- `cat`
- `dog`
- `bird`

### Why grouping is important

Compare:

```js
/^cat|dog|bird$/
```

with:

```js
/^(cat|dog|bird)$/
```

The first pattern is interpreted as:

```text
(^cat) OR (dog) OR (bird$)
```

The second pattern applies both anchors to the entire group of alternatives.

### Example: Title matching

```js
const regex = /(Mrs|Mr|Dr)\.?\s[A-Z][a-z]+/;
```

This can match titles such as:

- Mr Rafi
- Mrs Hasan
- Dr Sakib

When alternatives overlap, ordering them carefully prevents an earlier, shorter alternative from matching prematurely.

---

## 6. Backreferences — `\1`, `\2`

A backreference matches the same text previously captured by a capturing group.

```js
const regex = /(\w+) \1/;

console.log(regex.test("hello hello")); // true
console.log(regex.test("hello world")); // false
console.log(regex.test("rafi rafi"));   // true
```

### How it works

1. `(\w+)` captures a word.
2. `\1` matches the exact same captured text again.

The number refers to the capturing-group index.

```js
const regex = /(\w+) (\w+)-\1-\2/;
```

- `\1` refers to the first capturing group.
- `\2` refers to the second capturing group.

### Practical use cases

- Detecting repeated words.
- Matching duplicated values.
- Checking repeated sequences.

```js
const regex = /\b(\w+)\s+\1\b/i;

console.log(regex.test("hello hello")); // true
console.log(regex.test("hello world")); // false
```

---

## 7. Word Boundaries — `\b`

A word boundary is a position between a word character (`\w`) and a non-word character (`\W`), or at a string edge adjacent to a word character.

It matches a position, not an actual character.

In JavaScript, `\w` primarily represents:

```text
[a-zA-Z0-9_]
```

### Example

```js
const text = "cat category wildcat catfish cat";

console.log(text.match(/\bcat\b/g));
// ["cat", "cat"]
```

Without boundaries:

```js
/cat/g
```

This can match `cat` inside:

- cat
- category
- wildcat
- catfish

With boundaries:

```js
/\bcat\b/g
```

It matches standalone `cat` words.

### Important examples

```js
/\bcat\b/.test("cat");       // true
/\bcat\b/.test("cat!");      // true
/\bcat\b/.test("(cat)");     // true
/\bcat\b/.test("cat-dog");   // true
/\bcat\b/.test("cat_dog");   // false
/\bcat\b/.test("123cat");    // false
```

### `\b` vs `^` and `$`

| Pattern | Meaning |
|---|---|
| `^cat$` | Entire string must be exactly `cat` |
| `\bcat\b` | Matches standalone `cat` anywhere in the string |

**Note:** JavaScript's `\b` is based on its regex word-character rules, not full Unicode-aware natural-language word segmentation.

---

## 8. Lookahead and Lookbehind

Lookaround assertions check surrounding context without consuming it.

They are useful when a match should depend on what comes before or after it, but that context should not be included in the returned match.

### 8.1 Positive Lookahead — `(?=...)`

Checks whether a pattern exists immediately after the current position.

```js
const regex = /\d+(?=px)/g;

console.log("10px 20em 30px".match(regex));
// ["10", "30"]
```

The digits are matched only when followed by `px`.

The `px` portion is not included in the match.

### 8.2 Negative Lookahead — `(?!...)`

Checks that a pattern does not exist immediately after the current position.

```js
const regex = /\b\w+\b(?!\!)/g;

console.log("hello! world stop!".match(regex));
// ["world", "stop"]
```

Here, a word is matched only when it is not immediately followed by `!`.

### 8.3 Positive Lookbehind — `(?<=...)`

Checks whether a pattern exists immediately before the current position.

```js
const regex = /(?<=\$)\d+/g;

console.log("$500 BDT 200 $1200".match(regex));
// ["500", "1200"]
```

The number is matched only when preceded by `$`.

The `$` is not included in the match.

### 8.4 Negative Lookbehind — `(?<!...)`

Checks that a pattern does not exist immediately before the current position.

```js
const regex = /(?<!\$)\b\d+/g;

console.log("Price: $500, tax: 30, fee: $20, tip: 10".match(regex));
// ["30", "10"]
```

The number is matched only when it is not immediately preceded by `$`.

### Lookaround quick reference

| Type | Syntax | Meaning |
|---|---|---|
| Positive Lookahead | `(?=...)` | Must be followed by |
| Negative Lookahead | `(?!...)` | Must not be followed by |
| Positive Lookbehind | `(?<=...)` | Must be preceded by |
| Negative Lookbehind | `(?<!...)` | Must not be preceded by |

### Important: Lookaround is zero-width

Lookaround checks a condition but does not consume the characters it checks.

For example:

```js
/\d+(?=px)/g
```

Matches `"10"` in `"10px"`, not `"10px"`.

### Greedy quantifiers and negative lookahead

Consider:

```js
console.log("10px 20em 30px 40rem".match(/\d+(?!px)/g));
```

Output:

```js
["1", "20", "3", "40"]
```

Why does it return `1` and `3`?

1. `\d+` initially consumes all available digits.
2. The negative lookahead fails after `10` because `px` follows.
3. The greedy quantifier backtracks and tries a shorter match.
4. After `1`, the next character is `0`, not `p`, so the assertion succeeds.

**Important:** The lookahead does not backtrack. The preceding greedy quantifier backtracks to allow the complete pattern to succeed.

---

## 9. Practical JavaScript: `matchAll()` and Captured Data

When extracting multiple records with capturing groups, `matchAll()` is especially useful.

```js
const text = `
user:rafi email:rafi@gmail.com amount:$500
user:nabil email:nabil@gmail.com amount:$1200
`;

const regex = /^user:(\w+) email:(\S+) amount:\$(\d+)$/gm;

const users = [...text.matchAll(regex)].map(
    ([, username, email, amount]) => ({
        username,
        email,
        amount
    })
);

console.log(users);
```

Output:

```js
[
    {
        username: "rafi",
        email: "rafi@gmail.com",
        amount: "500"
    },
    {
        username: "nabil",
        email: "nabil@gmail.com",
        amount: "1200"
    }
]
```

### Why use `matchAll()`?

- It returns an iterator of match results.
- Each match includes capturing groups.
- It works well with `map()` after converting the iterator into an array.
- It makes structured data extraction convenient.

---

## 10. Engineering Guidelines

1. Use capturing groups only when you need the captured values.
2. Prefer non-capturing groups when grouping is needed without extraction.
3. Use anchors to enforce complete-string or complete-line matching.
4. Order overlapping alternatives carefully.
5. Use backreferences when the same captured text must appear again.
6. Use word boundaries when matching standalone words.
7. Use lookaround when surrounding context matters but should not be part of the match.
8. Avoid unnecessarily complicated patterns. Readability matters.
9. Test both valid and invalid inputs.
10. Do not use Regex as a replacement for a proper parser when the input format becomes deeply nested or structurally complex.

---

## Module 03 — Final Takeaway

Advanced Regex is about combining smaller, understandable components into a larger pattern.

The goal is not to memorize complicated expressions. The goal is to understand what each part does, why it is needed, and how the entire pattern behaves.

**Think in small pieces. Combine them carefully. Test the edge cases.**