# Module 02 — Character Matching & Quantifiers

This module covers how regular expressions match individual characters, character sets, repeated patterns, and variable-length text.

The goal is to understand how regex engines consume characters and how quantifiers control matching behavior.

---

## 1. Character Classes `[]`

A character class matches **exactly one character** from a specified set of characters.

### Basic Character Class

```js
/[abc]/
```

Matches any one of these characters:

- `a`
- `b`
- `c`

```js
/[abc]/.test("apple"); // true
/[abc]/.test("dog");   // false
```

**Important:** `[abc]` does not mean the sequence `"abc"`. It means one character chosen from `a`, `b`, or `c`.

### Character Ranges

We can define ranges inside character classes.

```js
/[a-z]/       // Lowercase English letters
/[A-Z]/       // Uppercase English letters
/[0-9]/       // Digits
/[a-zA-Z]/    // Uppercase or lowercase English letters
/[a-zA-Z0-9]/ // English letters or digits
```

Examples:

```js
/[a-z]/.test("hello"); // true
/[A-Z]/.test("Hello"); // true
/[0-9]/.test("123");   // true
```

### Negated Character Classes `[^]`

When `^` appears at the beginning of a character class, it negates the class.

It matches any single character that is NOT included in the specified set.

```js
/[^abc]/
```

Matches any character except `a`, `b`, or `c`.

```js
/[^0-9]/.test("A"); // true
/[^0-9]/.test("5"); // false
```

Examples:

```js
const text = "123Hello456World";

console.log(/[^0-9]+/.exec(text));
// ["Hello"]
```

The regex skips the initial digits and matches consecutive non-digit characters until another digit appears.

**Important distinction:**

- `^` outside a character class represents the beginning of a string (or line with the `m` flag).
- `^` at the beginning of a character class represents negation.

---

## 2. Shorthand Character Classes

JavaScript provides shorthand patterns for commonly used character sets.

| Shorthand | Meaning | Equivalent |
|---|---|---|
| `\d` | Digit | `[0-9]` |
| `\D` | Non-digit | `[^0-9]` |
| `\w` | Word character | `[a-zA-Z0-9_]` |
| `\W` | Non-word character | `[^a-zA-Z0-9_]` |
| `\s` | Whitespace | Whitespace characters |
| `\S` | Non-whitespace | Non-whitespace characters |

### Examples

```js
/\d/.test("123");       // true
/\D/.test("hello");     // true
/\w/.test("Rafi_123");  // true
/\W/.test("@");         // true
/\s/.test("Hello World"); // true
/\S/.test("Hello");     // true
```

### Important: JavaScript `\w`

In JavaScript, `\w` primarily matches ASCII English letters, digits, and underscore.

It does not represent every Unicode letter.

```js
/\w/.test("A"); // true
/\w/.test("_"); // true
/\w/.test("7"); // true
```

Use explicit Unicode property escapes such as `\p{L}` with the `u` flag when you need to match Unicode letters.

---

## 3. Quantifiers

Quantifiers specify how many times the preceding character, character class, or group should repeat.

| Quantifier | Meaning |
|---|---|
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |
| `{n}` | Exactly n repetitions |
| `{n,m}` | Between n and m repetitions |
| `{n,}` | At least n repetitions |
| `{0,m}` | Zero to m repetitions |

### `*` — Zero or More

```js
/a*/
```

Matches zero or more consecutive `a` characters.

```js
/a*/.exec("aaa"); // ["aaa"]
/a*/.exec("bbb"); // [""]
```

Why does the second example return an empty string?

Because `*` allows zero repetitions. The regex can successfully match an empty string at the beginning.

### `+` — One or More

```js
/a+/
```

Matches at least one consecutive `a`.

```js
/a+/.exec("aaa"); // ["aaa"]
/a+/.exec("bbb"); // null
```

Unlike `*`, `+` cannot match zero characters.

### `?` — Zero or One

```js
/colou?r/
```

The `u` is optional.

```js
/colou?r/.test("color");  // true
/colou?r/.test("colour"); // true
```

### Exact Repetition `{n}`

```js
/\d{4}/
```

Matches exactly four consecutive digits.

```js
/\d{4}/.exec("123456"); // ["1234"]
```

### Range Repetition `{n,m}`

```js
/\d{2,5}/
```

Matches between two and five consecutive digits.

```js
/\d{2,5}/.exec("1234567"); // ["12345"]
```

### Minimum Repetition `{n,}`

```js
/\d{3,}/
```

Matches at least three consecutive digits.

```js
/\d{3,}/.exec("123456"); // ["123456"]
```

---

## 4. Greedy vs Lazy Matching

By default, JavaScript quantifiers are greedy.

A greedy quantifier attempts to match as much text as possible while still allowing the complete pattern to succeed.

A lazy quantifier attempts to match as little text as possible while still allowing the complete pattern to succeed.

### Greedy Quantifiers

```js
*
+
?
{n,m}
```

### Lazy Quantifiers

Add another `?` after the quantifier.

```js
*?
+?
??
{n,m}?
```

### Greedy Example

```js
const text = "a123b456b";

console.log(text.match(/a.*b/));
// ["a123b456b"]
```

The greedy `.*` consumes as much text as possible while allowing the final `b` to match.

### Lazy Example

```js
const text = "a123b456b";

console.log(text.match(/a.*?b/));
// ["a123b"]
```

The lazy `.*?` stops at the earliest position where the entire pattern succeeds.

### Greedy vs Lazy with `+`

```js
const text = "ab123cd456ef";

console.log(/\d+/.exec(text));
// ["123"]

console.log(/\d+?/.exec(text));
// ["1"]
```

The greedy `+` consumes all consecutive digits.

The lazy `+?` consumes the minimum number of digits needed to satisfy the pattern.

However, lazy matching does not always return one character.

```js
console.log(/\d+?3/.exec("123"));
// ["123"]
```

Why?

1. It initially tries to match one digit: `1`.
2. The next required character is `3`, but the next character is `2`.
3. It expands the match to `12`.
4. The next character is still not `3`.
5. It expands again to `123`.
6. Now the complete pattern succeeds.

### Zero-Length Lazy Matching

```js
console.log(/a*?/.exec("aaa"));
// [""]

console.log(/a*/.exec("aaa"));
// ["aaa"]
```

- `*?` allows zero repetitions and prefers the minimum possible match.
- `*` is greedy and consumes all available consecutive `a` characters.

### Greedy vs Lazy with Multiple Matches

```js
const text = "<h1>Hello</h1><h1>World</h1>";

console.log(text.match(/<h1>.*<\/h1>/));
// ["<h1>Hello</h1><h1>World</h1>"]

console.log(text.match(/<h1>.*?<\/h1>/g));
// ["<h1>Hello</h1>", "<h1>World</h1>"]
```

**Important:** These examples demonstrate regex matching behavior. Regular expressions are generally not suitable for parsing arbitrary HTML.

---

## 5. Anchors — `^` and `$`

Anchors define positions rather than matching actual characters.

| Anchor | Meaning |
|---|---|
| `^` | Beginning of input |
| `$` | End of input |

Without anchors, regex searches for a matching pattern anywhere in the input.

### Without Anchors

```js
/\d{3}/.test("abc123xyz");
// true
```

The regex finds three consecutive digits inside the string.

### With Anchors

```js
/^\d{3}$/.test("123");
// true

/^\d{3}$/.test("abc123xyz");
// false
```

The complete input must contain exactly three digits.

### Practical Validation Example

Validate a string containing exactly six digits:

```js
const regex = /^\d{6}$/;

console.log(regex.test("123456"));  // true
console.log(regex.test("12345"));   // false
console.log(regex.test("1234567")); // false
console.log(regex.test("12a456"));  // false
```

**Remember:** `^` and `$` together are essential when validating an entire string rather than simply searching for a matching substring.

---

## 6. Practical Regex Examples

### A. Username Validation

Requirements:

- Must start with an English letter.
- Remaining characters can contain letters, digits, or underscores.
- Minimum length: 4 characters.
- Maximum length: 12 characters.

```js
const usernameRegex = /^[a-z]\w{3,11}$/i;

console.log(usernameRegex.test("Rafi123")); // true
console.log(usernameRegex.test("rafi_01")); // true
console.log(usernameRegex.test("1rafi"));   // false
console.log(usernameRegex.test("_rafi"));   // false
console.log(usernameRegex.test("raf"));     // false
console.log(usernameRegex.test("rafi@123"));// false
```

Explanation:

- `^` — Beginning of input.
- `[a-z]` — First character must be an English letter.
- `\w{3,11}` — Next 3–11 characters must be word characters.
- `$` — End of input.
- `i` — Case-insensitive matching.

Total length: 4–12 characters.

### B. Bangladeshi Mobile Number Validation

Local format:

```js
const phoneRegex = /^01[3-9]\d{8}$/;

console.log(phoneRegex.test("01570954580")); // true
console.log(phoneRegex.test("01912345678")); // true
console.log(phoneRegex.test("12345678901")); // false
```

International and local format:

```js
const phoneRegex = /^(?:01[3-9]\d{8}|\+8801[3-9]\d{8})$/;

console.log(phoneRegex.test("01570954580"));   // true
console.log(phoneRegex.test("+8801570954580")); // true
```

### C. Extract All Numbers

```js
const text = "Order 123 costs 450 taka, quantity 25";

const numbers = text.match(/\d+/g);

console.log(numbers);
// ["123", "450", "25"]
```

- `\d+` — Matches one or more consecutive digits.
- `g` — Finds all matches throughout the string.

**JavaScript note:** `String.prototype.match()` with the `g` flag returns `null` when no matches are found.

```js
"Hello World".match(/\d+/g); // null
```

---

## 7. Important JavaScript Behaviors

### Greedy and Lazy Matching Depend on the Complete Pattern

A quantifier does not work independently. It attempts to satisfy the entire regex pattern.

```js
/\d+?3/.exec("123883");
// ["123"]
```

Even though the quantifier is lazy, it must expand until the next required character can match.

### Quantifiers Repeat the Immediately Preceding Token

```js
/a+/
```

Repeats `a`.

```js
/[a-z]+/
```

Repeats lowercase letters.

```js
/\d{3}/
```

Repeats digits exactly three times.

### Global Matching

```js
const text = "cat CAT Cat cAt";

console.log(text.match(/cat/gi));
// ["cat", "CAT", "Cat", "cAt"]
```

- `g` — Global matching.
- `i` — Case-insensitive matching.

---

## 8. Quick Reference

| Pattern | Meaning |
|---|---|
| `[abc]` | One of a, b, or c |
| `[a-z]` | One lowercase English letter |
| `[^abc]` | Any character except a, b, or c |
| `\d` | One digit |
| `\D` | One non-digit |
| `\w` | One ASCII word character |
| `\W` | One non-word character |
| `\s` | One whitespace character |
| `\S` | One non-whitespace character |
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |
| `{n}` | Exactly n |
| `{n,m}` | Between n and m |
| `{n,}` | At least n |
| `*?` | Lazy zero or more |
| `+?` | Lazy one or more |
| `^` | Beginning of input |
| `$` | End of input |

---

## 9. Key Takeaways

1. Character classes define which individual characters can match.
2. Negated character classes exclude specified characters.
3. Shorthand classes make common patterns shorter and easier to read.
4. Quantifiers control how many times the preceding token repeats.
5. Greedy quantifiers consume as much as possible while allowing the complete pattern to succeed.
6. Lazy quantifiers consume as little as possible while allowing the complete pattern to succeed.
7. Anchors help enforce whole-string validation.
8. The `g` flag allows extraction of multiple matches.
9. Understanding how a regex consumes characters is more important than memorizing patterns.


**Next**: Module 03 — Advanced Pattern Construction (Grouping, Capturing Groups, Non-Capturing Groups, Alternation, and Backreferences).