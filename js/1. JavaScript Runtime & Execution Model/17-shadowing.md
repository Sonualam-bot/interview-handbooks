# Chapter 17 — Shadowing

> *"Sometimes two variables share the same name. JavaScript doesn't get confused—it follows a simple rule: the closest one wins."*

---

# A Mystery

Consider this program.

```js
let language = "JavaScript";

function learn() {
    let language = "TypeScript";

    console.log(language);
}

learn();
```

The output is:

```text
TypeScript
```

But wait.

There is already a variable named `language`.

Why wasn't `"JavaScript"` printed?

Did the second declaration overwrite the first?

No.

Both variables still exist.

JavaScript simply chose one over the other.

---

# Becoming the JavaScript Engine

Imagine you're executing:

```js
console.log(language);
```

inside `learn()`.

Your current Environment Record contains:

```text
language → "TypeScript"
```

Should you continue searching the Scope Chain?

No.

The identifier has already been found.

Searching farther would only create ambiguity.

So JavaScript stops immediately.

The closest matching binding always wins.

---

# What Is Shadowing?

**Shadowing** occurs when an inner scope declares an identifier with the same name as one in an outer scope.

The inner binding temporarily hides the outer binding within that scope.

The outer variable still exists.

It is simply no longer visible from the inner scope.

---

# Think of People With the Same Name

Imagine a company with two employees named Alex.

One works on your team.

The other works in a different office.

When someone in your team says:

> "Ask Alex."

You naturally assume they mean the Alex sitting beside you.

The closer match wins.

JavaScript resolves identifiers in exactly the same way.

---

# Walking Through an Example

```js
const company = "OpenAI";

function engineering() {
    const company = "Anthropic";

    console.log(company);
}

engineering();

console.log(company);
```

Inside `engineering()`:

```text
company → "Anthropic"
```

Outside:

```text
company → "OpenAI"
```

The inner declaration never modified the global one.

It merely shadowed it.

---

# Shadowing Across Blocks

Shadowing isn't limited to functions.

```js
let count = 100;

if (true) {
    let count = 5;

    console.log(count);
}

console.log(count);
```

Output:

```text
5
100
```

The block created a new scope.

Its own `count` hides the outer one only while execution remains inside that block.

---

# Shadowing and the Scope Chain

Remember how JavaScript searches for variables.

1. Current Environment Record.
2. Outer lexical environment.
3. Continue outward.

Shadowing changes only the first step.

If a matching identifier is found immediately, JavaScript never continues searching.

---

# Illegal Shadowing

Not every combination is allowed.

For example:

```js
let value = 10;

{
    var value = 20;
}
```

This results in a syntax error.

Why?

Because `var` belongs to the surrounding function (or global) scope.

Allowing it here would conflict with the existing `let` binding in the same scope.

JavaScript rejects the program before execution begins.

---

# Why Shadowing Exists

Without shadowing, every local variable would need a globally unique name.

Large programs would become difficult to write.

Shadowing allows inner scopes to use meaningful names without affecting outer scopes.

---

# React Connection

Shadowing appears frequently in React.

```jsx
const user = currentUser;

items.map(user => (
  <Profile user={user} />
));
```

The `user` parameter inside `map()` shadows the outer `user` variable.

Inside the callback, `user` refers to the current item—not the component-level variable.

Understanding shadowing helps avoid subtle bugs when naming callback parameters.

---

# Key Takeaways

- Shadowing happens when an inner scope declares an identifier with the same name as an outer scope.
- The inner binding hides the outer one within that scope.
- JavaScript always uses the closest matching identifier.
- The outer binding still exists and becomes visible again after leaving the inner scope.
- Some combinations, such as conflicting `var` and `let` declarations, are illegal.

---

> **Next Chapter:** *Illegal Shadowing — Why Some Shadowing Patterns Are Forbidden*
