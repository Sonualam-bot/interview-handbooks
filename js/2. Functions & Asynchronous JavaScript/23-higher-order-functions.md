# Chapter 23 — Higher-Order Functions

> *"A carpenter builds furniture. A master craftsman builds tools that help others build furniture. Higher-order functions are the tools of JavaScript."*

---

# A Mystery

Consider this code.

```js
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(num => num * 2);
```

You never wrote a loop.

Yet every number was processed.

Now look at this.

```js
setTimeout(() => {
    console.log("Hello");
}, 1000);
```

The function isn't executed immediately.

Instead, another function decides **when** it should run.

How can functions control other functions?

---

# Becoming the JavaScript Engine

Imagine you're executing:

```js
numbers.map(fn);
```

The array already knows how to iterate through its elements.

But it doesn't know **what operation** should be performed.

Instead of hardcoding the behavior, it receives another function.

For every element, it simply says:

> "Here's the next value. You decide what to do with it."

The logic and the iteration remain separate.

---

# What Is a Higher-Order Function?

A **Higher-Order Function (HOF)** is a function that:

- accepts one or more functions as arguments,
- returns a function,
- or both.

Higher-order functions don't perform one specific task.

Instead, they coordinate other functions.

---

# Think of a Restaurant

Imagine a restaurant.

The waiter doesn't cook your meal.

The waiter accepts your order and sends it to the chef.

Different customers receive different meals using the same ordering process.

The waiter is like a higher-order function.

The recipe is like the callback function.

The structure stays the same.

Only the behavior changes.

---

# Accepting Functions

Consider:

```js
function repeat(task) {
    task();
    task();
}

function greet() {
    console.log("Hello");
}

repeat(greet);
```

Output:

```text
Hello
Hello
```

`repeat()` never needed to know what `greet()` does.

Its job is simply to execute the function it receives.

---

# Returning Functions

A higher-order function may also create new functions.

```js
function multiplyBy(x) {
    return function (y) {
        return x * y;
    };
}

const triple = multiplyBy(3);

console.log(triple(5));
```

The returned function carries the behavior defined by the outer function.

Closures make this possible.

---

# Common Higher-Order Functions

JavaScript includes many built-in HOFs.

```js
map()
filter()
reduce()
find()
some()
every()
sort()
```

Each one follows the same philosophy.

The method controls **how** iteration happens.

Your callback controls **what** happens for each element.

---

# Why Separate Structure From Behavior?

Imagine writing a custom loop every time you wanted to transform an array.

Your programs would contain repetitive code.

Higher-order functions separate:

- the algorithm,
- from the custom logic.

This makes programs shorter, easier to read, and easier to reuse.

---

# Functional Programming

Higher-order functions are one of the foundations of functional programming.

Instead of repeatedly writing control flow, developers compose behavior by combining small reusable functions.

This leads to code that is often more expressive and easier to test.

---

# React Connection

React relies heavily on higher-order functions.

Examples include:

```jsx
items.map(item => (
    <Card key={item.id} />
));
```

```js
setState(prev => prev + 1);
```

```js
useMemo(() => calculate(), []);
```

Each of these APIs accepts functions rather than raw values, allowing React to decide **when** and **how** those functions execute.

---

# Key Takeaways

- A higher-order function accepts or returns functions.
- It separates reusable structure from custom behavior.
- Array methods like `map()`, `filter()`, and `reduce()` are higher-order functions.
- Closures often power returned functions.
- Higher-order functions are fundamental to modern JavaScript and React.

---

> **Next Chapter:** *Callbacks — Letting One Function Decide When Another Function Runs*
