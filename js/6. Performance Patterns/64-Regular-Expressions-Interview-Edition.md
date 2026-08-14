# Chapter 64 — Regular Expressions (Interview Edition)

> **Performance Patterns Handbook**

---

# What You'll Learn

- What Regular Expressions Are
- Creating Regular Expressions
- Common Regex Methods
- Character Classes
- Quantifiers
- Anchors
- Groups
- Common Patterns
- Interview Questions

---

# Introduction

Regular Expressions (Regex) are patterns used to search, validate, extract, and replace text.

They are commonly used for:

- Form validation
- Password validation
- Email validation
- Search functionality
- Parsing strings
- Log processing

Although Regex can look intimidating, understanding the basic building blocks is enough for most frontend interviews.

---

# Creating a Regular Expression

There are two ways to create a regex.

## Regex Literal

```js
const regex = /hello/;
```

## RegExp Constructor

```js
const regex = new RegExp("hello");
```

The literal syntax is more commonly used.

---

# Common Regex Methods

## test()

Returns `true` if the pattern matches.

```js
const regex = /hello/;

console.log(regex.test("hello world"));
console.log(regex.test("javascript"));
```

Output:

```text
true
false
```

---

## match()

Returns the matched text.

```js
const str = "JavaScript";

console.log(str.match(/Script/));
```

---

## replace()

Replaces matching text.

```js
const text = "Hello World";

console.log(
  text.replace("World", "JavaScript")
);
```

Output:

```text
Hello JavaScript
```

---

# Character Classes

| Pattern | Meaning |
|---------|---------|
| `.` | Any character except newline |
| `\d` | Digit |
| `\D` | Non-digit |
| `\w` | Letter, digit or underscore |
| `\W` | Not a word character |
| `\s` | Whitespace |
| `\S` | Non-whitespace |

Example:

```js
/\d/.test("7");
```

Returns:

```text
true
```

---

# Quantifiers

| Pattern | Meaning |
|---------|---------|
| `*` | 0 or more |
| `+` | 1 or more |
| `?` | 0 or 1 |
| `{n}` | Exactly n |
| `{n,}` | At least n |
| `{n,m}` | Between n and m |

Example:

```js
/\d{3}/.test("123");
```

---

# Anchors

Anchors define where matching should occur.

| Pattern | Meaning |
|---------|---------|
| `^` | Start of string |
| `$` | End of string |

Example:

```js
/^Hello/.test("Hello World");
```

```js
/World$/.test("Hello World");
```

---

# Groups

Groups allow multiple expressions to be treated as one unit.

```js
/(cat|dog)/.test("dog");
```

Output:

```text
true
```

---

# Useful Patterns

## Email (Simple)

```js
/^[^\s@]+@[^\s@]+\.[^\s@]+$/
```

---

## Digits Only

```js
/^\d+$/
```

---

## Alphabet Only

```js
/^[A-Za-z]+$/
```

---

## Remove Extra Spaces

```js
text.replace(/\s+/g, " ");
```

---

# Flags

| Flag | Meaning |
|------|---------|
| `g` | Global search |
| `i` | Case-insensitive |
| `m` | Multiline |

Example:

```js
/hello/i.test("HELLO");
```

Output:

```text
true
```

---

# Common Mistakes

## Forgetting Anchors

Without:

```js
^
$
```

the pattern may match only part of the string instead of the entire value.

---

## Making Patterns Too Complex

Prefer small, readable regex patterns over extremely complicated ones whenever possible.

---

# Interview Questions

### What is a regular expression?

A pattern used to search, validate, and manipulate text.

---

### What does `\d` represent?

A digit (`0–9`).

---

### What is the difference between `test()` and `match()`?

- `test()` returns a boolean.
- `match()` returns the matched result.

---

### What do the `g` and `i` flags do?

- `g` performs a global search.
- `i` ignores letter casing.

---

# Key Takeaways

- Regular expressions are powerful text-processing tools.
- Learn the common character classes and quantifiers first.
- Use `test()` for validation and `match()` or `replace()` for extraction and modification.
- Keep regex patterns readable and maintainable.
- Basic regex knowledge is sufficient for most frontend interviews.

---

# Handbook Complete ✅

You have completed the **Performance Patterns Handbook**.

## JavaScript Interview Notes Complete 🎉

You have now completed all JavaScript handbooks:

- ✅ JavaScript Runtime & Execution
- ✅ Objects & Memory
- ✅ Prototypes & OOP
- ✅ Collections
- ✅ Performance Patterns

Next Phase:

**React Handbook**
