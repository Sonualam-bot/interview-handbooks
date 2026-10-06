# Handbook 5 — React Performance Engineering
# Chapter 79 — Profiling React Apps

## 1. Why Profiling Matters

When an app feels slow, don't immediately add `React.memo`, `useMemo`, or `useCallback`.

First determine:
- What is actually slow?
- Which component is responsible?
- How often does it render?
- How expensive is each render?
- What caused the render?

```text
Observe
   ↓
Measure
   ↓
Identify bottleneck
   ↓
Understand cause
   ↓
Optimize
   ↓
Measure again
```

---

## 2. React DevTools Profiler

**React DevTools Profiler** is the primary tool for investigating React rendering performance.

It helps inspect:
- Which components rendered
- How often they rendered
- Rendering cost
- Which interactions caused rendering

```text
User interaction
      ↓
Record
      ↓
React renders
      ↓
Profiler captures information
      ↓
Inspect component performance
```

---

## 3. What to Look For

Look for:
- Frequently rendering components
- Expensive renders
- Unnecessary renders
- Large subtrees being rendered repeatedly

**Important:** the component with the highest render count is not automatically the biggest performance problem.

---

## 4. Render vs Commit

```text
Render  → React calculates what should change
Commit  → React applies necessary DOM changes
```

A component rendering does **not** necessarily mean the DOM changed.

---

## 5. Example

```jsx
function Dashboard() {
  const [theme, setTheme] = useState("light");

  return (
    <>
      <button onClick={() => setTheme("dark")}>
        Change Theme
      </button>

      <ExpensiveChart />
      <UserList />
      <Sidebar />
    </>
  );
}
```

A theme update can cause the `Dashboard` subtree to render.

Don't immediately memoize every child. First determine which component actually consumes meaningful rendering time.

---

## 6. Console Logging

For simple investigation:

```jsx
function UserList() {
  console.log("UserList rendered");

  return <div>Users</div>;
}
```

This helps answer:
- Does it render?
- When does it render?
- How often?

You can also inspect props:

```js
console.log({
  users,
  config,
  onSelect
});
```

This can reveal changing object, array, or function references.

---

## 7. Find What Changed

```jsx
<UserList
  users={users}
  config={config}
  onSelect={onSelect}
/>
```

Suppose:

```text
users     → same reference
config    → new reference
onSelect  → new reference
```

Then:

> The parent rendered, and `config` and `onSelect` received new references, so a memoized child cannot bail out based on props.

The goal is to identify **what changed**, not just observe that a render happened.

---

## 8. StrictMode

Development `StrictMode` can cause additional render-related behavior.

Therefore:

> Duplicate console logs in development do not automatically mean the production application renders twice.

Distinguish development behavior from production behavior.

---

## 9. Render Count Isn't Everything

```text
Component A
20 renders × 1 ms = 20 ms

Component B
2 renders × 50 ms = 100 ms
```

Component B has the larger **measured rendering cost** in this example.

But neither component is automatically memoized.

React does not decide that B is expensive and memoize it automatically.

> Optimize based on measured cost and user impact, not render count alone.

---

## 10. Expensive Calculation vs Expensive Render

```jsx
function Dashboard({ users }) {
  const activeUsers = users.filter(user => user.active);

  return <UserList users={activeUsers} />;
}
```

The cost could be the filtering calculation, or it could be inside a child such as:

```jsx
<LargeChart data={data} />
```

Profiling helps identify the actual cost.

---

## 11. Common Causes of Unnecessary Rendering

### State too high in the tree

State placed in a parent can cause a larger subtree to render.

### Unstable object props

```jsx
<Child config={{ theme: "dark" }} />
```

A new object is created on every render.

### Unstable function props

```jsx
<Child onClick={() => doSomething()} />
```

A new function reference is created on every render.

### Unstable arrays

```jsx
<Child items={[1, 2, 3]} />
```

A new array is created on every render.

### Other causes

- Broad context updates
- External store subscriptions

---

## 12. State Colocation

Consider:

```text
App
 ├── Header
 ├── Dashboard
 ├── Sidebar
 └── SearchBox
```

If only `SearchBox` needs the search query state, keeping it in `App` may cause unnecessary work elsewhere.

Prefer keeping state close to where it is needed when appropriate.

> State colocation can reduce the amount of the tree affected by an update.

---

## 13. React.memo After Profiling

If profiling shows:

```text
Parent renders frequently
        ↓
Child receives same props
        ↓
Child performs expensive rendering
```

Then:

```jsx
const Child = React.memo(function Child({ data }) {
  return <ExpensiveUI data={data} />;
});
```

If props are equal, React can potentially skip the child render.

### Correct sequence

```text
Profile
   ↓
Identify expensive child
   ↓
Check prop stability
   ↓
React.memo
   ↓
Profile again
```

---

## 14. useMemo After Profiling

If an expensive calculation runs during unrelated renders:

```jsx
const filteredUsers = expensiveFilter(users);
```

Use:

```jsx
const filteredUsers = useMemo(
  () => expensiveFilter(users),
  [users]
);
```

when the calculation is worth avoiding.

---

## 15. useCallback After Profiling

Consider:

```jsx
const Child = React.memo(ChildComponent);

function Parent() {
  const handleSelect = item => {
    // ...
  };

  return <Child onSelect={handleSelect} />;
}
```

Every parent render creates a new function reference:

```text
Parent renders
      ↓
New handleSelect reference
      ↓
Child prop changed
      ↓
React.memo bailout fails
```

If profiling shows this matters:

```jsx
const handleSelect = useCallback((item) => {
  // ...
}, []);
```

Again: **profile first**.

---

## 16. Don't Optimize Every Render

A component rendering frequently is not automatically a problem.

```text
Simple component
+
1 ms render
+
rare update
```

There may be no meaningful performance issue.

Avoid blindly adding:
- `React.memo`
- `useMemo`
- `useCallback`

Optimization adds complexity and maintenance cost.

---

## 17. Performance Optimization Loop

```text
Observe
  ↓
Measure
  ↓
Find Cause
  ↓
Optimize
  ↓
Measure Again
```

Don't stop at:

> "I added `useMemo`, so it's optimized."

Ask:

> Did the optimization actually reduce meaningful work?

---

## 18. Interview Scenario

**Question:** A child component is rendering every time its parent renders. How would you debug it?

**Strong answer:**

1. Use React DevTools Profiler or logging.
2. Identify when the parent renders.
3. Inspect the child's props.
4. Check which prop references changed.
5. Check state, context, and store subscriptions.
6. Determine whether the child's render is actually expensive.
7. Consider state colocation.
8. Consider `React.memo` if appropriate.
9. Stabilize object/function/array references only when necessary.
10. Profile again after the change.

---

## 19. What Profiling Should Tell You

```text
Which component?
      ↓
When does it render?
      ↓
How often?
      ↓
How expensive?
      ↓
What changed?
      ↓
Why did it render?
      ↓
What optimization could help?
```

Profiling turns:

> "React feels slow."

into:

> "This component renders because this prop reference changes, and its render costs X."

That gives you an actionable optimization target.

---

## 20. Golden Rules

1. **Measure before optimizing.**
2. A render does not necessarily mean a DOM update.
3. Use React DevTools Profiler for real investigation.
4. Console logging is useful for simple render debugging.
5. StrictMode can affect development observations.
6. Render count alone does not determine performance cost.
7. Find the actual expensive work.
8. Check state placement.
9. Check unstable object, array, and function references.
10. Use `React.memo` when it addresses an identified rendering problem.
11. Use `useMemo` when an identified calculation is worth reusing.
12. Use `useCallback` when function identity affects an identified optimization.
13. Profile again after optimization.
14. The goal is **less unnecessary work**, not fewer renders at any cost.

---

## Quick Interview Answer

> I would first profile the application using React DevTools to identify which components are rendering, how often they render, and where the actual time is being spent. Then I would inspect state placement, props, references, context, and expensive calculations to determine the cause. Only after identifying the bottleneck would I apply techniques such as state colocation, `React.memo`, `useMemo`, or `useCallback`, and then profile again to verify the improvement.
