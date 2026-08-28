# Chapter 38 — `useCallback`

## 1. Core Mental Model

```jsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

`useCallback` memoizes a **function reference** between renders.

Think:

```text
Render
 ↓
Dependencies changed?
 ↓
Yes → create new function
No  → reuse previous function reference
```

> **`useCallback` is about preserving function identity between renders.**

---

## 2. Why Function Identity Matters

In JavaScript:

```js
() => {}
```

creates a new function object.

So:

```jsx
function Component() {
  const handleClick = () => {
    console.log("click");
  };
}
```

can produce:

```text
Render 1 → Function A
Render 2 → Function B
Render 3 → Function C
```

Even though the code is identical:

```text
Function A !== Function B
```

---

## 3. `useCallback` + `React.memo`

Suppose:

```jsx
const Child = React.memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});
```

Without `useCallback`:

```jsx
function Parent() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <Child onClick={handleClick} />;
}
```

When the parent renders again:

```text
Parent renders
 ↓
new handleClick function
 ↓
Child receives new function reference
 ↓
React.memo sees changed prop
 ↓
Child can render again
```

With:

```jsx
function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  return <Child onClick={handleClick} />;
}
```

the function reference can remain stable:

```text
Render 1 → Function A
Render 2 → Function A
Render 3 → Function A
```

As long as dependencies don't change.

This can allow `React.memo` to bail out of unnecessary child rendering.

---

## 4. `useCallback` Does NOT Prevent Parent Rendering

Important:

```jsx
const handleClick = useCallback(...);
```

does **not** mean:

```text
Parent won't render
```

The parent can still render normally.

`useCallback` only helps keep the function reference stable.

---

## 5. Dependencies

Example:

```jsx
const handleClick = useCallback(() => {
  console.log(userId);
}, [userId]);
```

If:

```text
userId unchanged
```

React can reuse the previous function.

If:

```text
userId changes
```

React creates a new function that captures the new value.

Conceptually:

```text
userId = 1
 ↓
Function A captures 1

userId = 2
 ↓
dependency changed
 ↓
Function B captures 2
```

---

## 6. `useCallback` and Closures

```jsx
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);
```

The function closes over the `count` from the render that created it.

Conceptually:

```text
count = 0
 ↓
Function A captures 0

count = 1
 ↓
dependency changed
 ↓
Function B captures 1
```

If `count` is incorrectly omitted:

```jsx
const handleClick = useCallback(() => {
  console.log(count);
}, []);
```

the callback can capture a stale value.

---

## 7. `useCallback` vs `useMemo`

### `useMemo`

```jsx
const value = useMemo(
  () => calculate(),
  [deps]
);
```

Memoizes a **value**.

### `useCallback`

```jsx
const fn = useCallback(
  () => doSomething(),
  [deps]
);
```

Memoizes a **function reference**.

Mental model:

```text
useMemo
→ memoize result

useCallback
→ memoize function
```

Conceptually:

```jsx
useCallback(fn, deps)
```

can be thought of similarly to:

```jsx
useMemo(() => fn, deps)
```

The important distinction is the intent of the APIs.

---

## 8. `useCallback` vs `useRef`

### `useRef`

```jsx
const ref = useRef(value);
```

Gives you:

```text
ref.current
```

and changing `.current` doesn't trigger a render.

### `useCallback`

```jsx
const fn = useCallback(() => {}, []);
```

Keeps a function reference stable when dependencies don't change.

Therefore:

```text
useRef
→ mutable container

useCallback
→ stable function reference
```

---

## 9. When Should You Actually Use `useCallback`?

A strong use case is:

```text
Memoized child
+
Function passed as prop
+
Parent re-renders
+
Function doesn't need a new identity
```

Example:

```jsx
const Child = React.memo(ChildComponent);

function Parent() {
  const handleClick = useCallback(() => {
    // ...
  }, []);

  return <Child onClick={handleClick} />;
}
```

This can allow the child to skip unnecessary rendering.

---

## 10. Don't Wrap Every Function

Avoid blindly doing:

```jsx
const add = useCallback((a, b) => a + b, []);
const subtract = useCallback((a, b) => a - b, []);
const multiply = useCallback((a, b) => a * b, []);
```

if stable identity isn't useful.

`useCallback` itself has overhead and adds complexity.

The goal is:

```text
Avoid meaningful unnecessary work
```

not:

```text
Memoize everything
```

---

## 11. `useCallback` + `useEffect`

Example:

```jsx
const createOptions = useCallback(() => {
  return {
    roomId
  };
}, [roomId]);

useEffect(() => {
  const options = createOptions();

  connect(options);
}, [createOptions]);
```

Without `useCallback`, `createOptions` is recreated every render, potentially causing the effect dependency to change every render.

With `useCallback`:

```text
roomId unchanged
 ↓
same function reference
 ↓
effect dependency unchanged
```

When:

```text
roomId changes
 ↓
new function
 ↓
effect runs
```

Don't use `useCallback` merely to silence dependency warnings. Understand why the function is a dependency.

---

## 12. Interview Questions

### Q1. What does `useCallback` do?

> `useCallback` memoizes a function reference so React can reuse the same function between renders while its dependencies remain unchanged.

### Q2. Does `useCallback` prevent a component from rendering?

> No. It only stabilizes the function reference.

### Q3. Why use it with `React.memo`?

> `React.memo` compares props, and a newly created function is a different reference. `useCallback` can keep that function reference stable so the memoized child can bail out.

### Q4. `useMemo` vs `useCallback`?

> `useMemo` memoizes a value; `useCallback` memoizes a function reference.

### Q5. What happens when a dependency changes?

> A new callback is created so it can capture the latest dependency values.

### Q6. What happens if dependencies are missing?

> The callback can capture stale props or state.

---

# Quick Revision

```text
useCallback
→ memoize function reference
```

```text
Dependency unchanged
→ reuse function reference
```

```text
Dependency changed
→ new function
```

```text
useMemo
→ memoize value
```

```text
React.memo + function prop
→ common use case
```

```text
useCallback
≠
prevent component rendering
```

```text
Missing dependency
→ possible stale closure
```

---

# Final Mental Model

```text
             dependencies
                  ↓
Function creation
       ↓
 ┌───────────────┐
 │               │
same          changed
 │               │
 ↓               ↓
reuse          new function
reference      reference
```

> **Use `useCallback` when function identity matters. Don't use it simply because a function exists.**

**Chapter 38 — COMPLETE**
