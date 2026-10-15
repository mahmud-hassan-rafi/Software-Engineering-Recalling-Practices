# Module 01 — Regex Fundamentals

> Learn the fundamentals of Regular Expressions in JavaScript, including pattern matching, metacharacters, escaping, and regex flags.

## Table of Contents

1. [What is Regular Expression?](#1-what-is-regular-expression)
2. [How Regex Matching Works](#2-how-regex-matching-works)
3. [Regex Literal vs RegExp Constructor](#3-regex-literal-vs-regexp-constructor)
4. [Literal Characters](#4-literal-characters)
5. [Metacharacters](#5-metacharacters)
6. [Escaping Special Characters](#6-escaping-special-characters)
7. [Regex Flags](#7-regex-flags)
8. [Important Concepts to Remember](#8-important-concepts-to-remember)

---

## 1. What is Regular Expression?

A Regular Expression (Regex) is a pattern-matching language used to search, validate, extract, and replace text.

JavaScript provides the `RegExp` object to work with regular expressions.

### Example

```js
const regex = /hello/;

console.log(regex.test("hello world")); // true
console.log(regex.test("good morning")); // false
```

The pattern `/hello/` searches for the substring `hello` anywhere in the input string.

### Common Use Cases

- Validating user input.
- Searching for specific patterns.
- Extracting information from text.
- Replacing or modifying text.
- Parsing structured strings.

---

## 2. How Regex Matching Works

JavaScript's Regex engine examines input text and attempts to find a sequence of characters that satisfies the given pattern.

For example:

```js
const regex = /cat/;

console.log(regex.test("I have a cat")); // true
console.log(regex.test("I have a dog")); // false
```

### Important Behaviours

**1. Regex searches anywhere by default.**

```js
/cat/.test("I have a cat"); // true
/cat/.test("cat is cute");  // true
```

The entire string does not need to match.

**2. Character order matters.**

```js
/abc/.test("abc"); // true
/abc/.test("acb"); // false
```

**3. Matching is case-sensitive by default.**

```js
/dev/.test("developer"); // true
/dev/.test("DEV");       // false
```

---

## 3. Regex Literal vs RegExp Constructor

JavaScript provides two ways to create regular expressions.

### A. Regex Literal

```js
const regex = /hello/;
```

The pattern is written directly between two forward slashes.

Use this approach when the pattern is known in advance.

### B. RegExp Constructor

```js
const regex = new RegExp("hello");
```

The pattern is passed as a string to the constructor.

Use this approach when the pattern needs to be created dynamically.

### Dynamic Pattern Example

```js
const keyword = "javascript";

const regex = new RegExp(keyword);

console.log(regex.test("I love javascript")); // true
```

The pattern changes according to the value of `keyword`.

### Adding Flags

Both approaches support flags.

```js
const regex1 = /hello/i;

const regex2 = new RegExp("hello", "i");
```

Both perform case-insensitive matching.

### Important: Escaping Difference

The constructor receives a JavaScript string first, so backslashes must also follow JavaScript string escaping rules.

```js
// Regex literal
const regex1 = /\.com/;

// RegExp constructor
const regex2 = new RegExp("\\.com");
```

Both patterns match a literal dot followed by `com`.

---

## 4. Literal Characters

Literal characters are ordinary characters that match themselves.

```js
const regex = /cat/;

console.log(regex.test("cat"));       // true
console.log(regex.test("concatenate")); // true
console.log(regex.test("dog"));       // false
```

The regex searches for the exact sequence:

```text
c → a → t
```

The characters must appear consecutively and in the correct order.

### Case Sensitivity

```js
/cat/.test("CAT"); // false
/cat/.test("cat"); // true
```

Without the `i` flag, uppercase and lowercase characters are different.

---

## 5. Metacharacters

Metacharacters are characters that have special meanings in regular expressions.

### The Dot (`.`)

The dot matches any single character except line terminators by default.

```js
const regex = /c.t/;

console.log(regex.test("cat")); // true
console.log(regex.test("cut")); // true
console.log(regex.test("c9t")); // true
console.log(regex.test("coat")); // false
```

The pattern `/c.t/` means:

- `c` — literal character.
- `.` — any single character.
- `t` — literal character.

### Important: Dot and Line Terminators

By default, `.` does not match these four line terminators:

| Character | Description |
|---|---|
| `\n` | Line Feed |
| `\r` | Carriage Return |
| `\u2028` | Line Separator |
| `\u2029` | Paragraph Separator |

However, it can match spaces, tabs, digits, letters, and other ordinary characters.

```js
/a.b/.test("a b");  // true
/a.b/.test("a\tb"); // true
/a.b/.test("a\nb"); // false
```

---

## 6. Escaping Special Characters

Some characters have special meanings in Regex.

To match those characters literally, escape them using a backslash (`\`).

### Example: Literal Dot

```js
const regex1 = /.com/;
const regex2 = /\.com/;

console.log(regex1.test("googleXcom")); // true
console.log(regex2.test("googleXcom")); // false

console.log(regex2.test("google.com")); // true
```

Explanation:

- `.` matches any single character.
- `\.` matches an actual dot character.

### Example: Dollar Sign

```js
const regex = /\$/;

console.log(regex.test("$100")); // true
console.log(regex.test("100"));  // false
```

The escaped dollar sign matches the actual `$` character.

### Common Regex Special Characters

These characters have special meanings in regular expressions:

```text
^  $  \  .  *  +  ?  (  )  [  ]  {  }  |
```

When you need to match them literally, escaping is generally required.

---

## 7. Regex Flags

Flags modify how a regular expression performs matching.

JavaScript supports seven standard flags:

| Flag | Name | Purpose |
|---|---|---|
| `g` | Global | Find multiple matches |
| `i` | Ignore Case | Case-insensitive matching |
| `m` | Multiline | Change `^` and `$` behaviour across lines |
| `s` | Dot All | Allow `.` to match line terminators |
| `u` | Unicode | Enable Unicode-aware matching |
| `y` | Sticky | Match only at the current `lastIndex` |
| `d` | Has Indices | Provide match start/end indices |

### 7.1 Global Flag (`g`)

The `g` flag enables global matching.

Without `g`, `match()` returns only the first match.

```js
const text = "apple banana apple mango apple";

console.log(text.match(/apple/));
// First match information
```

With `g`:

```js
console.log(text.match(/apple/g));
// ["apple", "apple", "apple"]
```

#### Important: `lastIndex`

Global Regex objects maintain a `lastIndex` property.

It determines where the next matching attempt starts when using methods such as `test()` and `exec()`.

```js
const regex = /cat/g;
const text = "cat cat";

console.log(regex.lastIndex);  // 0

console.log(regex.test(text)); // true
console.log(regex.lastIndex);  // 3

console.log(regex.test(text)); // true
console.log(regex.lastIndex);  // 7

console.log(regex.test(text)); // false
console.log(regex.lastIndex);  // 0
```

Important behaviours:

- Successful matching updates `lastIndex` to the position immediately after the match.
- Failed matching resets `lastIndex` to `0`.
- Reusing a global Regex can produce different results across repeated calls.

Reset manually when necessary:

```js
regex.lastIndex = 0;
```

**Engineering Note:** Be careful when reusing global Regex objects, especially in repeated validation or shared utility functions.

### 7.2 Ignore Case Flag (`i`)

The `i` flag makes matching case-insensitive.

```js
const regex = /hello/i;

console.log(regex.test("hello")); // true
console.log(regex.test("HELLO")); // true
console.log(regex.test("Hello")); // true
```

Without `i`, matching is case-sensitive.

### 7.3 Multiline Flag (`m`)

The `m` flag changes the behaviour of two anchors:

- `^` matches the beginning of each line.
- `$` matches the end of each line.

Without `m`, these anchors refer to the beginning and end of the entire input string.

#### Using `^`

```js
const text = `I love coding
Hello world
I love JavaScript`;

console.log(/^Hello/.test(text));  // false
console.log(/^Hello/m.test(text)); // true
```

The second line starts with `Hello`.

#### Using `$`

```js
const text = `Hello world
I love coding
Hello`;

console.log(/Hello$/m.test(text)); // true
```

The last line ends with `Hello`.

**Important:** The `m` flag does not create new lines. It changes anchor behaviour for existing line boundaries.

### 7.4 Dot All Flag (`s`)

By default, the dot does not match line terminators.

The `s` flag changes that behaviour.

```js
console.log(/a.b/.test("a\nb"));  // false
console.log(/a.b/s.test("a\nb")); // true
```

With `s`, the dot matches line terminators as well.

### 7.5 Unicode Flag (`u`)

JavaScript strings use UTF-16 encoding.

Some Unicode characters, such as certain emoji, require two UTF-16 code units.

The `u` flag enables Unicode-aware matching.

```js
console.log(/^.$/.test("😊"));  // false
console.log(/^.$/u.test("😊")); // true
```

Use Unicode-aware patterns when working with international text and Unicode characters.

### 7.6 Sticky Flag (`y`)

The sticky flag requires matching at the exact position specified by `lastIndex`.

Unlike `g`, it does not search ahead for the next match.

```js
const text = "cat dog";

const globalRegex = /dog/g;
globalRegex.lastIndex = 0;

console.log(globalRegex.test(text)); // true
```

The global Regex searches forward and finds `dog`.

With sticky matching:

```js
const stickyRegex = /dog/y;
stickyRegex.lastIndex = 0;

console.log(stickyRegex.test(text)); // false
```

It fails because `dog` does not begin at index `0`.

### `g` vs `y`

| Global (`g`) | Sticky (`y`) |
|---|---|
| Searches forward | Matches only at `lastIndex` |
| Can skip characters | Cannot skip characters |
| Useful for finding multiple matches | Useful for position-based parsing |

### 7.7 Has Indices Flag (`d`)

The `d` flag provides the start and end positions of matched substrings.

```js
const regex = /cat/d;

const result = regex.exec("I love cat");

console.log(result[0]);      // "cat"
console.log(result.indices); // [[7, 10]]
```

The indices represent:

- Start: `7`
- End: `10` (exclusive)

This is useful when the exact position of matched text matters.

---

## 8. Important Concepts to Remember

### Regex Literal vs Special Meaning

```js
/cat/  // Searches for "cat"
/./    // Matches any one character
/\./   // Matches a literal dot
```

### Important Behaviour Differences

| Concept | Behaviour |
|---|---|
| Default matching | Case-sensitive |
| Default searching | Finds a substring anywhere |
| `.` | Matches one character except line terminators |
| `\.` | Matches a literal dot |
| `^` | Beginning of input or line with `m` |
| `$` | End of input or line with `m` |
| `g` | Stateful when used with `test()` and `exec()` |
| `s` | Makes dot match line terminators |

### Final Mental Model

Think of Regex as a small language for describing text patterns.

- Literal characters describe exactly what to find.
- Metacharacters describe matching rules.
- Escaping removes a special character's Regex meaning.
- Flags modify the matching behaviour.
- The JavaScript `RegExp` object provides methods to execute patterns.

---

## Next Module

**Module 02 — Character Matching & Quantifiers**

Topics:

1. Character Classes: `[abc]`
2. Character Ranges: `[a-z]`, `[0-9]`
3. Negated Character Classes: `[^abc]`
4. Shorthand Character Classes: `\d`, `\w`, `\s`
5. Quantifiers: `*`, `+`, `?`, `{n}`
6. Greedy vs Lazy Matching

> Goal: Build reliable Regex patterns by understanding how characters and repetition rules work together.

---

**Reference:** [MDN — Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions) · [MDN — RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
