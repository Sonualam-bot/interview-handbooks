# Chapter 67 — Re-render Debugging

## Handbook
**Handbook 5 — Performance Engineering**

## 1. Core Principle

A re-render is not automatically a performance problem.

Don't immediately ask:

> "How do I stop this re-render?"

Ask:

> **"Why is it rendering, and is the render actually expensive?"**

```text
Normal:
state change
 ↓
component renders
 ↓
small amount of work
 ↓
no meaningful DOM change
```

```text
Performance problem:
frequent updates
 ↓
expensive component
 ↓
large subtree
 ↓
unnecessary work
 ↓
poor responsiveness
```

## 2. Main Debugging Question

When a component renders unexpectedly, ask:

```text
WHY?
```

Main possibilities:

- State changed
- Props changed
- Parent rendered
- Context changed
- External store/subscription changed

Then ask:

> **Was the update actually necessary?**

## 3. React DevTools

React DevTools helps investigate:

- component tree
- props
- state
- context
- profiling information

Mental model:

```text
React DevTools
 ↓
Find component
 ↓
Inspect props/state/context
 ↓
Profile rendering
 ↓
Find expensive work
```

## 4. React Profiler

The Profiler helps investigate slow React interactions.

It helps answer:

```text
Which components rendered?
How often?
How long did rendering take?
What happened during an interaction?
```

## 5. Find What Is Rendering

Don't immediately add:

```jsx
React.memo(...)
```

First determine:

> **Which component is rendering?**

Different render patterns require different investigation.

## 6. Identify What Changed

Check:

```text
State?
Props?
Context?
Parent?
External subscription?
```

Mental model:

```text
Component rendered
       ↓
What changed?
 ┌─────┼─────┬───────┐
State Props Context External
```

## 7. Object Reference Problems

Example:

```jsx
function Parent() {
  const user = {
    name: "Sonu"
  };

  return <Child user={user} />;
}
```

Every render creates a new object:

```text
render 1 → user A
render 2 → user B
```

Therefore:

```js
userA !== userB
```

Even if their contents are identical.

This matters when using `React.memo`, because reference identity affects prop comparison.

## 8. Function Reference Problems

Example:

```jsx
function Parent() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <Child onClick={handleClick} />;
}
```

Every render creates another function reference:

```text
render 1 → function A
render 2 → function B
```

Therefore:

```js
functionA !== functionB
```

This can cause a memoized child to render again.

## 9. `useCallback` Can Help

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

This can keep the function reference stable.

But don't use `useCallback` everywhere.

Ask:

```text
Is the function passed to a memoized child?
        ↓
Does the child render expensively?
        ↓
Is the changing reference causing meaningful work?
        ↓
If yes → consider useCallback
```

## 10. `React.memo` as an Optimization

```jsx
const ExpensiveChild = React.memo(function ExpensiveChild() {
  // expensive rendering
});
```

If its props remain unchanged:

```text
Parent renders
 ↓
Child props unchanged
 ↓
memo bailout
 ↓
Child render can be skipped
```

Profile first when possible.

## 11. Context Debugging

A large context can make rendering behavior harder to reason about:

```jsx
<AppContext.Provider
  value={{
    user,
    theme,
    notifications,
    filters
  }}
>
```

If the relevant context value changes, consumers can update.

Meaningful boundaries can sometimes be created with separate contexts:

```text
AuthContext
ThemeContext
NotificationContext
FilterContext
```

## 12. Parent Re-renders

A child can render because its parent renders:

```text
Parent
 ↓
Child
```

When parent state changes:

```text
Parent state changes
 ↓
Parent renders
 ↓
Child is encountered during rendering
```

Before optimizing the child, ask:

> **Why is the parent rendering so frequently?**

The real problem may be one level higher.

## 13. Render Count ≠ Performance Problem

For example:

```text
50 renders × 0.05ms = 2.5ms
```

may be fine.

But:

```text
50 renders × 20ms = 1000ms
```

is much more concerning.

> **Optimize measured expensive work, not render counts alone.**

## 14. Expensive Calculations

If every render runs:

```jsx
const result = expensiveCalculation(data);
```

and the calculation is genuinely expensive, investigate whether `useMemo` is appropriate:

```jsx
const result = useMemo(
  () => expensiveCalculation(data),
  [data]
);
```

Use it when there is a meaningful performance reason.

## 15. Large Lists

A large list can create expensive rendering work:

```text
10,000 items
 ↓
parent renders
 ↓
large list rendering
```

Possible strategies:

- virtualization
- pagination
- memoization
- better state boundaries
- list-specific optimizations

## 16. Debugging Workflow

### Step 1 — Reproduce

Make the problem happen consistently.

### Step 2 — Profile

Use React DevTools Profiler:

```text
What rendered?
How long?
How often?
```

### Step 3 — Find the expensive component

Focus on the actual expensive subtree.

### Step 4 — Identify the trigger

```text
State?
Props?
Parent?
Context?
External store?
```

### Step 5 — Inspect references

Look for:

```text
new object
new array
new function
```

being passed unnecessarily.

### Step 6 — Optimize

Possible tools:

```text
React.memo
useMemo
useCallback
better state placement
context splitting
code splitting
virtualization
```

### Step 7 — Profile again

Verify that the optimization actually helped.

```text
Before
 ↓
optimize
 ↓
After
 ↓
measure
```

## 17. Don't Optimize Blindly

Bad:

```text
Performance issue
 ↓
React.memo everything
 ↓
useCallback everything
 ↓
useMemo everything
```

Good:

```text
Performance issue
 ↓
measure
 ↓
find bottleneck
 ↓
understand cause
 ↓
targeted optimization
 ↓
measure again
```

> **Measure first, optimize second.**

## 18. Practical Example

```jsx
function Dashboard() {
  const [search, setSearch] = useState("");

  const user = {
    name: "Sonu"
  };

  return (
    <>
      <Search value={search} onChange={setSearch} />
      <ExpensiveTable user={user} />
    </>
  );
}
```

Every time the user types:

```text
search changes
 ↓
Dashboard renders
 ↓
new user object created
 ↓
ExpensiveTable receives new reference
 ↓
ExpensiveTable renders
```

If `ExpensiveTable` is expensive, this could be unnecessary.

You could investigate whether a stable `user` reference combined with `React.memo` provides a meaningful improvement.

But **measure first**.

# Interview Questions

### Q1. How would you debug unnecessary React re-renders?

> I'd reproduce the issue, use React DevTools Profiler to identify which components are rendering and how expensive they are, determine whether the trigger is state, props, context, parent rendering, or an external subscription, then apply a targeted optimization and profile again.

### Q2. Would you use `React.memo` immediately?

> No. I'd first measure and identify whether the component is actually expensive and why it is rendering. Memoization adds its own comparison and complexity costs.

### Q3. What are common causes of unnecessary child renders?

> Parent renders, changing object/array references, changing function references, context updates, or unnecessary state placement.

### Q4. Why can this cause problems?

```jsx
<Child user={{ name: "Sonu" }} />
```

> A new object is created on every parent render, so its reference changes even when the data is logically identical. That can defeat memoization.

### Q5. How would you investigate a slow React page?

> I'd use the React Profiler to identify expensive renders and interactions, determine the component/subtree responsible, identify what triggers it, optimize the bottleneck, and profile again to verify the improvement.

### Q6. Is a component rendering many times automatically a performance bug?

> No. The cost of each render matters. Many cheap renders may be fine, while fewer expensive renders can be problematic.

# Quick Revision

```text
Don't ask:
"How do I stop this render?"

Ask:
"Why is it rendering and is it expensive?"
```

```text
Debugging
→ reproduce
→ profile
→ find expensive component
→ identify trigger
→ optimize
→ profile again
```

```text
Common triggers
→ state
→ props
→ parent
→ context
→ external store
```

```text
Common reference problems
→ objects
→ arrays
→ functions
```

```text
React.memo
→ possible bailout
```

```text
useCallback
→ stable function reference
```

```text
useMemo
→ cache expensive calculation / stabilize value
```

```text
Measure first
→ optimize second
```

# Final Mental Model

```text
             Performance Problem
                     ↓
                 Reproduce
                     ↓
             React Profiler
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
     What rendered?       How expensive?
          ↓                     ↓
          └──────────┬──────────┘
                     ↓
               Why rendered?
                     ↓
        ┌──────┬──────┬──────┬──────┐
        ↓      ↓      ↓      ↓      ↓
      State  Props  Parent Context Store
                     ↓
             Identify bottleneck
                     ↓
             Targeted optimization
                     ↓
               Profile again
```

### Interview line

> **"I wouldn't optimize a re-render just because it happens. I'd use the React Profiler to determine what is rendering, why it is rendering, and whether the work is actually expensive. Then I'd apply a targeted optimization and measure again."**

**Chapter 67 — COMPLETE ✅**
