# Chapter 69 — `React.memo` in Depth

## Handbook

**Handbook 5 — Performance Engineering**

---

## 1. What Is `React.memo`?

`React.memo` lets React skip rendering a component when its props are considered unchanged.

```jsx
const UserCard = React.memo(function UserCard({ user }) {
  return <h2>{user.name}</h2>;
});
```

Mental model:

```text
Parent renders
      ↓
React checks Child props
      ↓
Props unchanged?
   ┌──┴──┐
  YES    NO
   ↓      ↓
bailout  render
```

> **`React.memo` = component-level render bailout.**

---

## 2. Why Use It?

Suppose a parent updates frequently:

```text
Parent state changes
 ↓
Parent renders
 ↓
ExpensiveChild can render again
```

If `ExpensiveChild` doesn't depend on that changing state, memoization may avoid unnecessary work.

```jsx
const ExpensiveChild = React.memo(function ExpensiveChild() {
  // expensive rendering
});
```

---

## 3. Default Prop Comparison

`React.memo` uses a **shallow comparison** of props by default.

For primitives:

```jsx
<Child count={10} />
```

React can compare the value.

For objects:

```js
oldUser === newUser
```

The reference matters.

React does not deeply compare:

```js
oldUser.name === newUser.name
```

---

## 4. Object Reference Problem

Example:

```jsx
function Parent() {
  const user = {
    name: "Sonu"
  };

  return <Child user={user} />;
}
```

Every parent render creates a new object:

```text
render 1 → user A
render 2 → user B
```

Therefore:

```js
userA !== userB
```

Even if both contain:

```text
{ name: "Sonu" }
```

A memoized child can therefore render again.

---

## 5. Array Reference Problem

Same issue with arrays:

```jsx
function Parent() {
  const items = [1, 2, 3];

  return <List items={items} />;
}
```

Every render creates a new array:

```text
render 1 → array A
render 2 → array B
```

Therefore:

```js
arrayA !== arrayB
```

A memoized child sees the prop as changed.

---

## 6. Function Reference Problem

Example:

```jsx
function Parent() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <Child onClick={handleClick} />;
}
```

Every render creates a new function reference:

```text
render 1 → function A
render 2 → function B
```

Therefore:

```js
functionA !== functionB
```

This can prevent a `React.memo` bailout.

---

## 7. `React.memo` + `useCallback`

A stable callback can help:

```jsx
function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  return <Child onClick={handleClick} />;
}
```

Now the reference can remain stable while dependencies remain unchanged.

```text
Parent renders
 ↓
same function reference
 ↓
Child props can remain unchanged
 ↓
React.memo can bail out
```

Don't use `useCallback` everywhere. It is useful when referential stability actually matters.

---

## 8. `React.memo` + `useMemo`

The same principle applies to objects/arrays.

```jsx
const user = useMemo(() => ({
  name: "Sonu"
}), []);
```

Now the reference can remain stable between renders.

If the child is memoized:

```jsx
const Child = React.memo(ChildComponent);
```

React can potentially bail out when the other props are also unchanged.

Again:

> **Don't use `useMemo` merely to make every value stable. Have a performance reason.**

---

## 9. Custom Comparison Function

`React.memo` can receive a comparison function:

```jsx
const Child = React.memo(
  ChildComponent,
  arePropsEqual
);
```

Example:

```jsx
const Child = React.memo(
  function Child({ user }) {
    return <h1>{user.name}</h1>;
  },
  (prevProps, nextProps) => {
    return prevProps.user.id === nextProps.user.id;
  }
);
```

Mental model:

```text
arePropsEqual()
      ↓
true  → bailout possible
false → render
```

---

## 10. Custom Comparison Can Be Dangerous

A comparison must correctly account for every prop that can affect rendering.

Example:

```text
old:
{id: 1, name: "Sonu"}

new:
{id: 1, name: "Rahul"}
```

If the comparison only checks:

```js
prev.user.id === next.user.id
```

it returns `true`.

React may skip the render even though `name` changed.

> **A bad comparison can create stale UI or incorrect behavior.**

---

## 11. `React.memo` Is a Bailout, Not a Guarantee

Don't say:

> "`React.memo` prevents re-renders."

Better:

> **"`React.memo` allows React to bail out of rendering a component when its props are considered unchanged."**

This is more accurate.

---

## 12. `React.memo` Does Not Stop State Updates

A memoized component can still update its own state:

```jsx
const Child = React.memo(function Child() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      {count}
    </button>
  );
});
```

When its own state changes:

```text
Child state changes
 ↓
Child renders
```

---

## 13. `React.memo` Does Not Block Context Updates

Example:

```jsx
const Child = React.memo(function Child() {
  const theme = useContext(ThemeContext);

  return <div>{theme}</div>;
});
```

If the consumed context changes:

```text
context changes
 ↓
Child can update
```

Therefore:

```text
React.memo
≠
no updates ever
```

---

## 14. Memoization Can Fail Because of Unstable Props

This:

```jsx
<ExpensiveChild
  data={newObject}
  onClick={() => doSomething()}
/>
```

creates new references every render:

```text
new object
new function
 ↓
props changed
 ↓
React.memo bailout fails
 ↓
Child renders
```

Memoization is useful only when its inputs are suitable for comparison.

---

## 15. When Is `React.memo` Useful?

Classic scenario:

```text
Parent renders frequently
        +
Child is expensive
        +
Child props often don't change
```

Example:

```text
Dashboard
 ↓
search state changes frequently
 ↓
LargeChart
 ↓
chart data unchanged
```

`React.memo` may prevent unnecessary `LargeChart` rendering.

---

## 16. When Is It Probably Not Useful?

Avoid blindly memoizing:

```text
tiny component
+
cheap render
+
rare parent renders
+
props always changing
```

If a component is trivial, memoization may add complexity without meaningful performance benefit.

---

## 17. Sometimes Architecture Is the Better Optimization

Instead of:

```text
React.memo everywhere
```

sometimes change the architecture:

```text
Large Parent
 ↓
state changes frequently
 ↓
huge subtree
```

Better:

```text
Move state closer to where it is used
 ↓
only affected subtree updates
```

Other useful architectural approaches:

- split components
- split contexts
- separate server/client state
- reduce unnecessary dependencies

---

## 18. Practical Interview Scenario

Question:

> A parent component re-renders every time the user types in a search box. It also renders a large chart that doesn't depend on the search text. What would you do?

Strong answer:

> First I'd profile the interaction and confirm that the chart is actually expensive. If its props remain stable, I could wrap it with `React.memo` so it can bail out when the parent renders. I'd also check whether the parent creates new object or function props on every render. If necessary, I'd stabilize those references with `useMemo` or `useCallback`. Then I'd profile again to verify the improvement.

---

# Interview Questions

### Q1. What does `React.memo` do?

> It allows React to skip rendering a component when its props are considered unchanged.

### Q2. How does `React.memo` compare props?

> By default, it performs a shallow comparison. Primitives are compared by value while objects, arrays, and functions are compared by reference.

### Q3. Why can this defeat memoization?

```jsx
<Child data={{ id: 1 }} />
```

> Because a new object reference is created every render.

### Q4. How can you stabilize a function prop?

> Use `useCallback` when referential stability provides a meaningful benefit.

### Q5. How can you stabilize an object/array prop?

> Potentially use `useMemo`, but only when the stable reference is actually useful.

### Q6. Does `React.memo` prevent state updates?

> No. A component's own state can still cause it to render.

### Q7. Does `React.memo` prevent context updates?

> No. Consumed context changes can still cause the component to update.

### Q8. Should every component use `React.memo`?

> No. It has overhead and is most useful when preventing meaningful expensive work.

---

# Quick Revision

```text
React.memo
→ component-level bailout
```

```text
Default comparison
→ shallow comparison
```

```text
Primitive
→ value comparison
```

```text
Object / Array / Function
→ reference comparison
```

```text
New object
→ memo bailout can fail
```

```text
New function
→ memo bailout can fail
```

```text
useCallback
→ stable function reference
```

```text
useMemo
→ stable calculated value/reference
```

```text
React.memo
≠
never renders
```

```text
React.memo
≠
state/context blocked
```

```text
Best use case
→ expensive child
→ frequent parent renders
→ props often unchanged
```

```text
Measure first
→ memoize second
```

---

# Final Mental Model

```text
                  Parent renders
                       ↓
                 Child is reached
                       ↓
                 React.memo?
                       ↓
                Compare props
                       ↓
             ┌─────────┴─────────┐
             ↓                   ↓
        Props same           Props changed
             ↓                   ↓
          Bailout              Render
```

With reference-sensitive props:

```text
Object / Array / Function
          ↓
   reference changed?
      ┌───┴───┐
     YES      NO
      ↓        ↓
   render    bailout
```

### Interview line

> **"`React.memo` is a performance optimization that lets React bail out of rendering a component when its props haven't changed according to the comparison. Because objects, arrays, and functions are reference-based, unstable references can defeat the optimization, which is why `useMemo` or `useCallback` can sometimes be used alongside it."**

**Chapter 69 — COMPLETE ✅**
