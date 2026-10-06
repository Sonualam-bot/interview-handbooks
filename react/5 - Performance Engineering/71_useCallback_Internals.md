# Handbook 5 — React Performance Engineering

# Chapter 71 — useCallback Internals

## 1. Core Idea

`useCallback` is about **function reference stability**, not function execution speed.

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

It allows React to reuse the same function reference between renders when dependencies have not changed.

### Mental model

```text
First render
    ↓
Create function
    ↓
Store function + dependencies
    ↓
Later render
    ↓
Compare dependencies
   /          \
same         changed
 ↓              ↓
reuse          create new callback
```

---

## 2. Why Function Identity Matters

Every time a component renders, a normal function declaration creates a new function object.

```jsx
function App() {
  const handleClick = () => {
    console.log("clicked");
  };
}
```

Even though the function's code is identical, its reference can be different:

```js
const fn1 = () => {};
const fn2 = () => {};

Object.is(fn1, fn2); // false
```

This matters when the function is passed as a prop to a memoized child.

---

## 3. What `useCallback` Actually Memoizes

`useCallback` memoizes the **function reference**.

Conceptually, React stores something similar to:

```text
{
  function: previousFunction,
  dependencies: previousDependencies
}
```

On a later render:

```text
Dependencies unchanged
        ↓
Reuse previous function reference
```

or:

```text
Dependencies changed
        ↓
Create a new function reference
```

It does **not** make the function itself execute faster.

---

## 4. Dependency Array

```jsx
const handleClick = useCallback(() => {
  console.log(userId);
}, [userId]);
```

The callback depends on `userId`.

```text
userId unchanged
    ↓
same callback reference

userId changed
    ↓
new callback reference
```

The dependency array determines when React is allowed to reuse the previous callback.

---

## 5. Closures and Stale Closures

Functions created inside components form closures over values from that render.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const logCount = useCallback(() => {
    console.log(count);
  }, []);

  return (
    <button onClick={logCount}>
      Log Count
    </button>
  );
}
```

Here, `count` is captured by the callback but is missing from the dependency array.

The callback can therefore continue referring to the value from the render in which it was created.

This is a **stale closure** problem.

Correct version:

```jsx
const logCount = useCallback(() => {
  console.log(count);
}, [count]);
```

When `count` changes:

```text
count changes
    ↓
dependency changes
    ↓
new callback created
    ↓
new callback captures latest count
```

---

## 6. Why the Functional Updater Can Use `[]`

Consider:

```jsx
const increment = useCallback(() => {
  setCount(prevCount => prevCount + 1);
}, []);
```

This callback does **not** need to capture the current `count`.

Instead, the functional updater receives the latest state value from React.

```text
click
  ↓
setCount(updater)
  ↓
React has current state
  ↓
updater(currentState)
  ↓
currentState + 1
```

Therefore, `[]` can be correct here.

Compare:

### Capturing `count`

```jsx
const increment = useCallback(() => {
  setCount(count + 1);
}, [count]);
```

The callback captures `count`, so `count` belongs in the dependency array.

### Functional updater

```jsx
const increment = useCallback(() => {
  setCount(prevCount => prevCount + 1);
}, []);
```

The callback does not need to capture `count`.

React supplies the latest state to the updater.

---

## 7. `useCallback` + `React.memo`

One of the most important practical use cases:

```jsx
const Child = React.memo(function Child({ onClick }) {
  console.log("Child rendered");

  return <button onClick={onClick}>Child</button>;
});
```

### Without `useCallback`

```text
App renders
    ↓
new handleClick function created
    ↓
new function reference
    ↓
Child receives changed onClick prop
    ↓
React.memo bailout fails
    ↓
Child renders
```

### With `useCallback`

```text
App renders
    ↓
useCallback returns same function reference
    ↓
Child receives same onClick reference
    ↓
React.memo sees unchanged prop
    ↓
Child can bail out
```

This is useful when the child is actually memoized and its render work matters.

---

## 8. `useCallback` Alone Does Not Prevent Child Renders

If the child is not memoized:

```jsx
function Child({ onClick }) {
  return <button onClick={onClick}>Child</button>;
}
```

then keeping `onClick` stable does not prevent normal parent-driven rendering.

The useful relationship is often:

```text
useCallback
    +
React.memo
    ↓
stable function prop
    ↓
possible child bailout
```

---

## 9. `useCallback` Does Not Prevent Parent Renders

When parent state changes:

```text
state update
    ↓
parent renders
    ↓
useCallback checks dependencies
    ↓
callback may be reused
```

The parent still renders.

`useCallback` controls the identity of the callback value; it does not prevent the component's render.

---

## 10. `useCallback` vs `useMemo`

### `useMemo`

Memoizes a **value**:

```jsx
const value = useMemo(() => calculate(a), [a]);
```

### `useCallback`

Memoizes a **function reference**:

```jsx
const fn = useCallback(() => doSomething(a), [a]);
```

Conceptually:

```text
useMemo
    ↓
memoized calculated value

useCallback
    ↓
memoized function reference
```

`useCallback(fn, deps)` is conceptually similar in intent to:

```jsx
useMemo(() => fn, deps);
```

---

## 11. Functional State Updates Can Reduce Dependencies

Instead of:

```jsx
const increment = useCallback(() => {
  setCount(count + 1);
}, [count]);
```

we can often write:

```jsx
const increment = useCallback(() => {
  setCount(prevCount => prevCount + 1);
}, []);
```

Now:

```text
No count captured
      ↓
No count dependency
      ↓
Callback can remain stable
```

This is useful when callback identity matters.

Do not force this pattern when the callback genuinely needs the current value.

---

## 12. `useCallback` Has a Cost

`useCallback` is not free.

There is still:

- dependency tracking
- dependency comparison
- retaining the callback
- additional code complexity
- additional cognitive overhead

Therefore:

> Do not use `useCallback` because a function exists.

Use it when stable function identity provides a meaningful benefit.

---

## 13. When `useCallback` Is a Good Candidate

A strong candidate usually has:

```text
Memoized child
      +
Callback passed as prop
      +
Parent renders frequently
      +
Callback dependencies often unchanged
      +
Child render is meaningful/expensive
```

---

## 14. When `useCallback` Is Probably Unnecessary

For a simple component:

```jsx
function Button() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <button onClick={handleClick}>Click</button>;
}
```

There may be no meaningful benefit from:

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

If the function is only used by a DOM element in the same component and its identity is not relevant to memoization or dependency optimization, the ordinary function is usually simpler.

---

## 15. Interview Mental Model

```text
useCallback(() => {...}, [deps])
                 ↓
          First render
                 ↓
         create function
                 ↓
       store function + deps
                 ↓
          Later render
                 ↓
       compare dependencies
            /        \
         same       changed
          ↓            ↓
       reuse        new callback
```

---

## 16. Interview Questions

### What problem does `useCallback` solve?

> `useCallback` memoizes a function reference between renders while its dependencies remain unchanged. It is useful when function identity matters, especially when callbacks are passed to memoized children or used as dependencies.

### Does `useCallback` make functions faster?

> No. It preserves function identity; it does not make the function's execution faster.

### Why use `React.memo` with `useCallback`?

> `React.memo` compares props. `useCallback` can keep a function prop's reference stable, allowing the memoized child to bail out when its other props are also unchanged.

### What is a stale closure?

> A callback captures values from a particular render. If those values are used inside the callback but are incorrectly omitted from its dependencies, the callback can continue using an outdated value.

### Why can a functional updater allow `[]`?

```jsx
const increment = useCallback(() => {
  setCount(prevCount => prevCount + 1);
}, []);
```

> The callback does not need to capture `count`. React supplies the latest state value to the functional updater, so `count` does not need to be a dependency.

---

## 17. Example Trace

```jsx
const Child = React.memo(function Child({ onIncrement }) {
  console.log("Child rendered");

  return (
    <button onClick={onIncrement}>
      Increment
    </button>
  );
});

function App() {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []);

  return (
    <>
      <p>{count}</p>
      <Child onIncrement={increment} />
    </>
  );
}
```

Clicking the button:

```text
User clicks
    ↓
increment callback executes
    ↓
setCount(updater)
    ↓
React calculates latest state
    ↓
count: 0 → 1
    ↓
App renders
    ↓
useCallback checks []
    ↓
dependencies unchanged
    ↓
same increment reference
    ↓
Child receives same onIncrement prop
    ↓
React.memo can bail out
```

The important insight:

> The callback can remain the same function while the state it updates continues to change.

---

# Golden Rules

1. **`useCallback` is about function identity, not function speed.**
2. **Normal component renders can create new function references.**
3. **`React.memo` + stable callback references can enable child render bailouts.**
4. **`useCallback` alone does not prevent child renders.**
5. **`useCallback` does not prevent parent renders.**
6. **Dependencies must reflect values captured by the callback.**
7. **Missing dependencies can create stale closures.**
8. **Functional state updates can remove unnecessary state dependencies.**
9. **Do not use `useCallback` for every function.**
10. **Use it when stable function identity has a meaningful performance or correctness benefit.**
