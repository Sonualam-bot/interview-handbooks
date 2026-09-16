# Chapter 37 — `useMemo`

## 1. Core Mental Model

```jsx
const result = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

`useMemo` memoizes the result of a calculation.

Think:

```text
Render
 ↓
Dependencies changed?
 ↓
Yes → calculate again
No  → reuse previous result
```

> **`useMemo` caches a computed value between renders.**

---

## Deep Dive `[NEW]`

### The Actual Storage: One Slot Holding `[value, deps]`

`useMemo` takes a node in the same per-fiber Hook list as every other
Hook (see Rules of Hooks), storing a two-element tuple: the last
computed value and the dependency array used to produce it. On each
render, React walks to that slot and compares the new dependency array
against the stored one, index by index, using `Object.is`. If every
entry matches, React returns the stored value **without ever calling
the factory function** — the calculation genuinely does not run, not
just "runs but gets ignored." If any entry differs, React calls the
factory, then overwrites both the stored value and the stored
dependency array with this render's versions. This is why `useMemo`'s
cache only ever holds the single most recent result — there's no
history, no LRU, just one slot being overwritten.

### Why `useMemo` Is a Hint, Not a Guarantee — And Why That Matters

React's own documentation reserves the right to discard a memoized
value and recompute it even when dependencies haven't changed — for
example, to free memory for offscreen components. This is a
deliberate design decision with a real consequence: you should never
rely on `useMemo` for *correctness* (e.g. assuming a factory function
with a side effect only runs once per dependency change, or that
identity is permanently guaranteed) — only for *performance*. If code
would break when a memoized calculation happens to re-run an extra
time, that's a sign the value needs `useState`/`useReducer` (a real
guarantee) rather than `useMemo` (a best-effort optimization).

## 2. Why Use It?

Suppose:

```jsx
const filteredProducts = filterProducts(products, search);
```

This calculation happens whenever the component renders.

If the calculation is expensive and:

```text
products didn't change
search didn't change
```

there may be unnecessary work.

With `useMemo`:

```jsx
const filteredProducts = useMemo(() => {
  return filterProducts(products, search);
}, [products, search]);
```

React can reuse the previous result when the dependencies haven't changed.

---

## 3. `useMemo` Does NOT Prevent Rendering

This is critical.

```jsx
const result = useMemo(
  () => expensiveCalculation(data),
  [data]
);
```

If the component renders:

```text
Component renders
 ↓
useMemo checks dependencies
 ↓
Calculation may be reused
```

The component **still rendered**.

So:

```text
useMemo
→ optimizes a calculation

NOT

useMemo
→ prevents component rendering
```

---

## 4. Dependency Array

```jsx
const result = useMemo(
  () => expensiveCalculation(a, b),
  [a, b]
);
```

Think:

```text
a/b unchanged
    ↓
reuse cached result

a or b changed
    ↓
calculate again
```

If the calculation depends on a value, that value needs to be accounted for in the dependencies.

---

## 5. Example

```jsx
const doubled = useMemo(() => {
  return count * 2;
}, [count]);
```

If:

```text
count = 5
```

React calculates:

```text
10
```

Next render:

```text
count = 5
```

Dependency unchanged:

```text
reuse 10
```

Then:

```text
count = 6
```

Dependency changed:

```text
calculate again
→ 12
```

---

## 6. `useMemo` vs `useState`

### `useState`

```jsx
const [count, setCount] = useState(0);
```

Stores application state.

### `useMemo`

```jsx
const result = useMemo(
  () => calculate(data),
  [data]
);
```

Caches a derived calculation.

Mental model:

```text
useState
→ persistent application state

useMemo
→ cached computed value
```

---

## 7. `useMemo` vs `useRef`

### `useRef`

```jsx
const ref = useRef(value);
```

Stores a mutable value:

```text
ref.current
```

Changing it does not cause a render.

### `useMemo`

```jsx
const value = useMemo(() => calculate(), []);
```

Caches the result of a calculation.

Therefore:

```text
useRef
→ mutable container

useMemo
→ cached computed value
```

---

## 8. Referential Equality

`useMemo` can also be useful for keeping an object's reference stable.

Without memoization:

```jsx
const options = {
  sort: "price"
};
```

A new object is created on every render:

```text
Render 1 → Object A
Render 2 → Object B
Render 3 → Object C
```

Even though the contents are identical:

```text
Object A !== Object B
```

You can memoize it:

```jsx
const options = useMemo(() => ({
  sort: "price"
}), []);
```

Now the value can retain the same reference between renders.

This can matter when passing the value to memoized child components.

---

## 9. `useMemo` + `React.memo`

Suppose:

```jsx
const Child = React.memo(function Child({ data }) {
  return <div>{data.name}</div>;
});
```

Parent:

```jsx
function Parent() {
  const data = {
    name: "Sonu"
  };

  return <Child data={data} />;
}
```

Every parent render creates a new object.

So the child may receive:

```text
old data !== new data
```

and render again.

With:

```jsx
const data = useMemo(() => ({
  name: "Sonu"
}), []);
```

the reference can remain stable, allowing `React.memo` to bail out when appropriate.

---

## 10. Don't Memoize Everything

This is a common mistake:

```jsx
const result = useMemo(() => count + 1, [count]);
```

The calculation:

```text
count + 1
```

is trivial.

Prefer:

```jsx
const result = count + 1;
```

`useMemo` itself has some overhead.

The goal is:

```text
Avoid meaningful unnecessary work
```

not:

```text
Memoize everything
```

---

## 11. `useMemo` Is a Performance Optimization

Important interview point:

> **`useMemo` should be treated as a performance optimization, not a correctness mechanism.**

Your application should remain logically correct if the memoization is removed.

The difference is that `useMemo` may avoid recalculating a value when dependencies haven't changed.

---

## 12. `useMemo` and Rendering

Suppose:

```jsx
function App({ count }) {
  const expensiveValue = useMemo(
    () => expensiveCalculation(),
    []
  );

  console.log("App rendered");

  return <div>{expensiveValue}</div>;
}
```

If `count` changes:

```text
App renders again
 ↓
useMemo checks []
 ↓
reuse cached value
```

So:

```text
Component render = YES
Calculation = NO
```

This distinction is frequently tested in interviews.

---

## 13. `useMemo` vs `React.memo`

### `useMemo`

Memoizes a **value**:

```jsx
const value = useMemo(
  () => calculate(),
  [deps]
);
```

### `React.memo`

Can skip a **child component's render** when its props are considered unchanged.

Mental model:

```text
useMemo
→ cache a value

React.memo
→ can skip child rendering
```

---

## 14. `useMemo` vs `useCallback`

### `useMemo`

```jsx
const value = useMemo(
  () => calculate(),
  [deps]
);
```

Returns a computed value.

### `useCallback`

```jsx
const fn = useCallback(() => {
  doSomething();
}, [deps]);
```

Returns a function reference.

Mental model:

```text
useMemo
→ memoize result

useCallback
→ memoize function
```

---

## 15. Good Candidates for `useMemo`

### Expensive calculation

```jsx
const sorted = useMemo(() => {
  return expensiveSort(items);
}, [items]);
```

### Expensive filtering

```jsx
const filtered = useMemo(() => {
  return expensiveFilter(items, query);
}, [items, query]);
```

### Stable object identity

```jsx
const options = useMemo(() => ({
  theme,
  language
}), [theme, language]);
```

Use this when stable identity is actually useful to downstream memoization.

---

## 16. Interview Questions

### Q1. What does `useMemo` do?

> `useMemo` memoizes the result of a calculation and can reuse that result when its dependencies haven't changed.

### Q2. Does `useMemo` prevent a component from rendering?

> No. The component can still render; `useMemo` only avoids recalculating the memoized value when appropriate.

### Q3. When should you use `useMemo`?

> Primarily for expensive calculations or when stable value identity is useful for downstream memoization.

### Q4. Is `useMemo` required for correctness?

> No. It is a performance optimization.

### Q5. Difference between `useMemo` and `useCallback`?

> `useMemo` memoizes a value; `useCallback` memoizes a function reference.

### Q6. Difference between `useMemo` and `React.memo`?

> `useMemo` caches a computed value inside a component, while `React.memo` can skip rendering a child when its props are considered unchanged.

---

# Quick Revision

```text
useMemo
→ memoize a computed value
```

```text
Dependencies unchanged
→ reuse cached result
```

```text
Dependencies changed
→ calculate again
```

```text
useMemo
≠
prevent component rendering
```

```text
useMemo
→ value

useCallback
→ function

React.memo
→ component render optimization
```

```text
useMemo
→ performance optimization
→ not required for correctness
```

```text
Good use
→ expensive calculations
→ useful stable references
```

---

# Final Mental Model

```text
Component renders
       ↓
useMemo checks dependencies
       ↓
 ┌───────────────┐
 │               │
same          changed
 │               │
 ↓               ↓
reuse          calculate
value          again
```

> **`useMemo` does not stop rendering. It can stop an expensive calculation from being repeated unnecessarily.**

**Chapter 37 — COMPLETE**
