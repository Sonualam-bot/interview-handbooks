# Handbook 5 — React Performance Engineering

# Chapter 70 — useMemo Internals

## Core Idea

`useMemo` lets React reuse the result of a calculation between renders when its dependencies have not changed.

```text
First render → calculate → store value
Later render → compare dependencies
                  ↓
          unchanged → reuse value
          changed   → calculate again
```

## 1. What Does useMemo Memoize?

It memoizes the **returned value**.

```jsx
const value = useMemo(
  () => expensiveCalculation(),
  dependencies
);
```

It does not memoize the component, DOM, or calculation function itself.

For objects, the useful thing being preserved is the **object reference**.

---

## 2. Conceptual Internal Model

Think of React as maintaining something conceptually similar to:

```text
{
  value: previousValue,
  dependencies: previousDependencies
}
```

On a later render:

```text
useMemo()
   ↓
Get previous value
   ↓
Compare dependencies
   ↓
Changed?
 ┌────┴────┐
 NO        YES
 ↓          ↓
reuse     calculate
value      again
```

---

## 3. First Render

Given:

```jsx
const result = useMemo(
  () => expensiveCalculation(a, b),
  [a, b]
);
```

There is no previous memoized value.

```text
calculate
   ↓
store value + dependencies
```

Conceptually:

```js
{
  value: calculatedResult,
  deps: [a, b]
}
```

---

## 4. Dependencies Unchanged

Previous:

```text
[a, b]
```

Next:

```text
[a, b]
```

React compares the dependencies using `Object.is` semantics.

If all dependencies are equal:

```text
dependencies unchanged
        ↓
reuse previous value
        ↓
don't run calculation again
```

---

## 5. Dependency Changed

Previous:

```text
[a, b]
```

Next:

```text
[a, c]
```

If the second dependency changed:

```text
dependency changed
      ↓
run calculation again
      ↓
store new value + dependencies
```

Basic rule:

```text
dependencies unchanged → reuse
dependencies changed   → recalculate
```

---

## 6. Referential Equality

For primitives:

```js
Object.is(10, 10);       // true
Object.is("a", "a");     // true
```

For objects:

```js
const a = { id: 1 };
const b = { id: 1 };

Object.is(a, b); // false
```

Therefore:

```jsx
const result = useMemo(
  () => calculate(user),
  [user]
);
```

can recalculate when `user` receives a new object reference, even if the contents look identical.

---

## 7. useMemo Does Not Mean “Calculate Only Once”

This is incorrect:

> "`useMemo` calculates something only once."

Correct:

> "`useMemo` reuses the previous result when its dependencies remain unchanged."

With:

```jsx
const value = useMemo(
  () => calculate(a),
  [a]
);
```

if `a` changes:

```text
a changes
   ↓
calculate again
```

---

## 8. Empty Dependency Array

```jsx
const value = useMemo(() => {
  return expensiveCalculation();
}, []);
```

Conceptually:

```text
First render
    ↓
calculate
    ↓
store result

Later renders
    ↓
dependencies unchanged
    ↓
reuse result
```

However, `useMemo` is a **performance optimization**, not a permanent-cache or semantic guarantee that a value can never be recalculated.

---

## 9. useMemo Is Not a General-Purpose Cache

Do not think:

```text
useMemo = permanent cache
```

Think:

```text
useMemo = React-managed performance optimization
```

Do not use it as a replacement for:

- application state
- server/data cache
- persistent storage
- data-fetching cache

---

## 10. useMemo vs useRef

### useMemo

```jsx
const value = useMemo(
  () => calculateSomething(data),
  [data]
);
```

Purpose:

> Cache a calculated value for performance.

### useRef

```jsx
const ref = useRef(value);
```

Purpose:

> Hold a mutable value across renders without causing a render when it changes.

Mental model:

```text
useMemo → computed value optimization
useRef  → persistent mutable container
```

---

## 11. useMemo + React.memo

This is one of the most important connections.

Without `useMemo`:

```jsx
const Child = React.memo(function Child({ data }) {
  return <div>{data.length}</div>;
});

function Parent({ items }) {
  const data = items.filter(item => item.active);

  return <Child data={data} />;
}
```

Even if `items` hasn't changed:

```text
Parent renders
     ↓
filter() runs
     ↓
new array created
     ↓
Child receives new reference
     ↓
React.memo sees changed prop
     ↓
Child renders
```

With `useMemo`:

```jsx
const data = useMemo(
  () => items.filter(item => item.active),
  [items]
);
```

If `items` hasn't changed:

```text
Parent renders
     ↓
useMemo reuses data
     ↓
same array reference
     ↓
Child receives same prop reference
     ↓
React.memo can potentially bail out
```

Relationship:

```text
useMemo
   ↓
stable value reference
   ↓
React.memo
   ↓
possible child bailout
```

---

## 12. useMemo Can Provide Referential Stability

Example:

```jsx
const config = useMemo(() => ({
  theme: "dark",
  pageSize: 20
}), []);
```

Now `config` can retain the same reference between renders.

This can matter when passing it to a memoized child.

But don't stabilize every object or array automatically. There should be a meaningful reason.

---

## 13. Dependency Array as a Contract

Consider:

```jsx
const result = useMemo(() => {
  return price * quantity;
}, [price]);
```

The calculation depends on:

```text
price
quantity
```

but only `price` is listed.

If:

```text
price stays the same
quantity changes
```

the memoized result may be reused even though it should change.

Correct:

```jsx
const total = useMemo(() => {
  return price * quantity;
}, [price, quantity]);
```

Now:

```text
price changes   → recalculate
quantity changes → recalculate
neither changes → reuse
```

The dependency array should represent the reactive values the calculation depends on.

---

## 14. Common Dependency Mistake

This is problematic if `data` can change:

```jsx
const result = useMemo(() => {
  return expensiveCalculation(data);
}, []);
```

Prefer:

```jsx
const result = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

when `data` is a dependency of the calculation.

---

## 15. useMemo and Closures

`useMemo` callbacks are JavaScript closures.

```jsx
function Component({ price }) {
  const total = useMemo(() => {
    return price * 2;
  }, [price]);

  return <div>{total}</div>;
}
```

The callback closes over `price`.

The dependency array communicates:

> The memoized calculation depends on `price`.

This connects React memoization with JavaScript closure concepts.

---

## 16. useMemo Runs During Rendering

The calculation inside `useMemo` is part of rendering when React needs to recalculate it.

Therefore, `useMemo` is **not for side effects**.

Bad:

```jsx
useMemo(() => {
  fetch("/api/users");
}, []);
```

Use the appropriate effect or data-fetching mechanism for side effects.

Good:

```jsx
const sortedUsers = useMemo(() => {
  return [...users].sort(compareUsers);
}, [users]);
```

Think:

```text
inputs → calculated output
```

---

## 17. When useMemo Makes Sense

Strong candidate:

```text
large dataset
+
expensive transformation
+
frequent component renders
+
dependencies often unchanged
```

Example:

```jsx
const processedUsers = useMemo(() => {
  return users
    .filter(...)
    .sort(...)
    .map(...);
}, [users]);
```

Especially when profiling shows that the repeated calculation matters.

---

## 18. When useMemo Is Probably Unnecessary

Avoid automatically memoizing trivial calculations:

```jsx
const fullName = useMemo(() => {
  return firstName + " " + lastName;
}, [firstName, lastName]);
```

The calculation is tiny.

Memoization itself introduces:

- dependency comparison
- additional code
- additional complexity
- retained values

Ask:

> **Does avoiding this calculation justify the cost and complexity?**

---

## 19. useMemo Does Not Stop Component Rendering

Consider:

```jsx
function App() {
  const [count, setCount] = useState(0);

  const result = useMemo(() => {
    return expensiveCalculation();
  }, []);

  console.log("App");

  ...
}
```

When `count` changes:

```text
count changes
   ↓
App renders
   ↓
useMemo checks dependencies
   ↓
dependencies unchanged
   ↓
reuse result
```

The component **still rendered**.

Only the calculation was skipped.

Therefore:

```text
useMemo ≠ prevent component render
```

For component render bailouts, think about `React.memo`.

---

## 20. React.memo vs useMemo vs useCallback

```text
React.memo
    ↓
component render bailout
    ↓
based primarily on props

useMemo
    ↓
calculated value reuse
    ↓
based on dependencies

useCallback
    ↓
function reference stability
    ↓
based on dependencies
```

| Tool | Memoizes | Main purpose |
|---|---|---|
| `React.memo` | Component rendering | Avoid unnecessary child renders |
| `useMemo` | Computed value | Avoid repeating expensive calculations |
| `useCallback` | Function reference | Keep callback reference stable |

---

## 21. Performance Decision Tree

```text
Is something unnecessarily rendering?
          ↓
         YES
          ↓
Why?
          ↓
Unnecessary child render?
          └──→ React.memo may help

Expensive calculation?
          └──→ useMemo may help

Unstable callback?
          └──→ useCallback may help

State causing a large subtree to update?
          └──→ Consider state colocation / architecture
```

Do not blindly add memoization.

---

## 22. Exercise

```jsx
function Dashboard({ users }) {
  const [theme, setTheme] = useState("light");

  const activeUsers = users.filter(
    user => user.active
  );

  const config = {
    theme,
    pageSize: 20
  };

  return (
    <>
      <button onClick={() => setTheme("dark")}>
        Change Theme
      </button>

      <UserList
        users={activeUsers}
        config={config}
      />
    </>
  );
}
```

And:

```jsx
const UserList = React.memo(function UserList({
  users,
  config
}) {
  console.log("UserList rendered");

  return <div>{users.length}</div>;
});
```

Answer:

1. When `theme` changes, does `Dashboard` render?
2. Does `activeUsers` get recalculated?
3. Does `config` get a new reference?
4. What happens to `UserList`?
5. What changes if `activeUsers` is wrapped in `useMemo` with `[users]`?
6. What changes if `config` is wrapped in `useMemo` with `[theme]`?

Remember: if `theme` changes, `config` should change too because `config.theme` depends on `theme`.

---

# Chapter Summary

```text
useMemo(() => calculate(...), [deps])
                    ↓
          First render?
             ↓
         calculate
             ↓
       store value + deps
             ↓
        Later render
             ↓
     compare dependencies
             ↓
       ┌─────┴─────┐
      same       changed
       ↓             ↓
     reuse        calculate
      value         again
```

### Remember

```text
React.memo
    → component render bailout

useMemo
    → calculated value reuse

useCallback
    → function reference stability
```

### Golden Rule

> **`useMemo` is for avoiding unnecessary recalculation or maintaining a useful stable value reference—not for preventing component renders.**

**Next → Chapter 71 — useCallback Internals**
