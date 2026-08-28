# Chapter 68 — Memoization

## Handbook
**Handbook 5 — Performance Engineering**

## 1. What Is Memoization?

> **Memoization means remembering a previous result so we don't have to calculate it again when relevant inputs haven't changed.**

```text
Input
 ↓
calculation
 ↓
Result
```

With memoization:

```text
Input
 ↓
Inputs changed?
 ↓
YES → recalculate
NO  → reuse previous result
```

---

## 2. The Three React Memoization Tools

```text
React.memo
→ component rendering

useMemo
→ calculated value

useCallback
→ function reference
```

They solve different problems.

---

## 3. `React.memo`

```jsx
const Child = React.memo(ChildComponent);
```

If the child's props are unchanged, React can skip its render:

```text
Parent renders
 ↓
props unchanged
 ↓
React.memo bailout
 ↓
Child render can be skipped
```

Good candidate:

```text
expensive child
+
frequent parent renders
+
props often unchanged
```

---

## 4. `useMemo`

`useMemo` caches a **value**.

```jsx
const sortedUsers = useMemo(
  () => expensiveSort(users),
  [users]
);
```

Mental model:

```text
users changed?
 ↓
YES → recalculate
NO  → reuse previous result
```

Good candidates:

- expensive calculations
- expensive filtering/sorting
- maintaining referential stability when it has a real benefit

---

## 5. `useCallback`

`useCallback` helps maintain a stable **function reference**.

Without it:

```jsx
const handleClick = () => {
  console.log("click");
};
```

Every render creates a new function.

With:

```jsx
const handleClick = useCallback(() => {
  console.log("click");
}, []);
```

The function reference can remain stable while dependencies remain unchanged.

---

## 6. Referential Equality

Objects and functions are compared by reference.

```js
{} === {} // false
```

Therefore:

```jsx
<Child user={{ name: "Sonu" }} />
```

creates a new object reference every render.

Similarly:

```jsx
<Child onClick={() => doSomething()} />
```

creates a new function reference every render.

This can matter when using `React.memo`.

---

## 7. The Three Tools Together

```jsx
function Parent({ users }) {
  const [count, setCount] = useState(0);

  const sortedUsers = useMemo(
    () => sortUsers(users),
    [users]
  );

  const handleSelect = useCallback((id) => {
    console.log(id);
  }, []);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        {count}
      </button>

      <UserList
        users={sortedUsers}
        onSelect={handleSelect}
      />
    </>
  );
}
```

```jsx
const UserList = React.memo(function UserList({
  users,
  onSelect
}) {
  // ...
});
```

If only `count` changes:

```text
count changes
 ↓
Parent renders
 ↓
useMemo reuses sortedUsers
 ↓
useCallback keeps handleSelect stable
 ↓
UserList receives same references
 ↓
React.memo can bail out
```

---

## 8. Memoization Is Not Free

> **Memoization does not automatically mean faster.**

It has costs:

- memory
- dependency tracking
- comparison
- additional complexity

Don't memoize tiny calculations without a reason.

Bad example:

```jsx
const result = useMemo(() => a + b, [a, b]);
```

for a trivial calculation.

---

## 9. When to Use `useMemo`

Use it when there is a meaningful reason, such as:

```jsx
const result = useMemo(
  () => expensiveCalculation(data),
  [data]
);
```

or:

```jsx
const filtered = useMemo(
  () => expensiveFilter(items, query),
  [items, query]
);
```

---

## 10. When to Use `useCallback`

A common useful case:

```text
Parent
 ↓
memoized Child
 ↓
function prop
```

Without stable reference:

```text
new function
 ↓
Child props changed
 ↓
React.memo can't bail out
```

With `useCallback`, the reference can remain stable.

But don't use `useCallback` everywhere.

---

## 11. `React.memo` Does Not Stop the Parent

If:

```jsx
const Child = React.memo(ChildComponent);
```

the parent can still render.

```text
Parent renders
 ↓
Child receives props
 ↓
memoization check
 ↓
possible bailout
```

`React.memo` only provides a bailout opportunity for the child.

---

## 12. `React.memo` Does Not Mean "Never Renders"

A memoized component can still update because of:

```text
its own state
context it consumes
other relevant subscriptions
```

Therefore:

```text
React.memo
≠
component can never render
```

It means React can skip rendering when the memoization conditions allow it.

---

## 13. `useMemo` Does Not Memoize the Component

This:

```jsx
const result = useMemo(
  () => expensiveCalculation(data),
  [data]
);
```

memoizes:

```text
result
```

It does not memoize the component.

Whereas:

```jsx
const Child = React.memo(ChildComponent);
```

memoizes the component's rendering opportunity.

---

## 14. `useCallback` vs `useMemo`

You can mentally understand:

```jsx
useCallback(fn, deps)
```

as roughly:

```jsx
useMemo(() => fn, deps)
```

Conceptually:

```text
useMemo
→ memoize a computed value

useCallback
→ memoize a function reference
```

---

## 15. Memoization and Immutable Data

Memoization works well when data follows predictable reference semantics.

Example:

```jsx
const nextUser = {
  ...user,
  name: "New Name"
};
```

The reference changes when the object meaningfully changes.

This makes reference-based comparisons useful.

---

## 16. Don't Use Memoization to Hide Bad Architecture

Avoid:

```text
React.memo everywhere
useMemo everywhere
useCallback everywhere
```

Sometimes the better solution is architectural:

```text
move state closer to where it is used
split components
split contexts
separate server/client state
reduce unnecessary dependencies
```

> **Memoization should complement good architecture, not replace it.**

---

## 17. Performance Optimization Hierarchy

```text
Good component/state architecture
        ↓
Avoid unnecessary work
        ↓
Measure with Profiler
        ↓
Identify actual bottleneck
        ↓
Targeted memoization
```

Not:

```text
Problem
 ↓
useMemo
 ↓
useCallback
 ↓
React.memo
 ↓
hope
```

> **Measure first, optimize second.**

---

# Interview Questions

### Q1. What is memoization?

> Memoization is caching a previous result so repeated work can be avoided when relevant inputs haven't changed.

### Q2. Difference between `React.memo`, `useMemo`, and `useCallback`?

> `React.memo` can skip a component render when props are suitable for a bailout, `useMemo` memoizes a calculated value, and `useCallback` memoizes a function reference.

### Q3. Does `React.memo` prevent a component from ever rendering?

> No. It provides a bailout when props haven't meaningfully changed, but the component can still update because of its own state, context, or other relevant updates.

### Q4. Why can an inline object defeat `React.memo`?

```jsx
<Child user={{ name: "Sonu" }} />
```

> Because a new object reference is created on every render, even if the contents are identical.

### Q5. Should you use `useMemo` for every calculation?

> No. Memoization has overhead and adds complexity. Use it when the calculation is sufficiently expensive or referential stability provides a meaningful benefit.

### Q6. Why use `useCallback`?

> Mainly to preserve a function reference when referential stability matters, commonly when passing callbacks to memoized children.

---

# Quick Revision

```text
Memoization
→ remember previous result
→ avoid repeated work
```

```text
React.memo
→ component rendering
```

```text
useMemo
→ calculated value
```

```text
useCallback
→ function reference
```

```text
New object
→ new reference
```

```text
New function
→ new reference
```

```text
Memoization
≠
always faster
```

```text
Measure
→ identify bottleneck
→ then memoize
```

---

# Final Mental Model

```text
                    MEMOIZATION
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
    React.memo       useMemo       useCallback
          ↓              ↓              ↓
     Component         Value        Function
      rendering       result        reference
```

### Interview line

> **"Memoization is an optimization technique where we reuse previous results when inputs haven't changed. In React, `React.memo` can skip unnecessary component renders, `useMemo` caches calculated values, and `useCallback` preserves function references. I use them based on measured performance needs rather than applying them everywhere."**

**Chapter 68 — COMPLETE**
