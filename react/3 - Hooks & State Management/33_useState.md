# Chapter 33 — `useState`

## Chapter Overview

`useState` is the fundamental Hook for giving function components persistent state across renders.

The important mental model is not simply:

```jsx
const [count, setCount] = useState(0);
```

Instead, think:

```text
Component identity
       ↓
Hook position
       ↓
Persistent state
       ↓
Current render snapshot
       ↓
Setter schedules update
       ↓
Future render
```

`useState` connects directly to the concepts already covered:

```text
Component Identity
        ↓
Fiber
        ↓
Hook ordering
        ↓
State
        ↓
Render
        ↓
Reconciliation
        ↓
Commit
```

---

## Deep Dive `[NEW]`

### The Bailout Nuance Almost Everyone Gets Wrong

Section 22 later in this chapter says React "can skip work" when
`setCount(sameValue)` is called with a value that's `Object.is`-equal
to the current state. The precise mechanics matter here, because the
common interview trap is assuming the component never re-runs at all.
What actually happens: React still processes the update — the fiber
is still marked as having pending work, and (outside a small set of
special-cased paths) the component function **still executes once**
to produce a new render output. Only *after* that render's Hook
returns the same value do things change: if it's the very first Hook
in the component and the value is unchanged, and nothing else about
the component's output differs, React can bail out of continuing
reconciliation/commit for that subtree. The safe interview answer is:
"the state update is still requested and the component may still
render once to check, but React can skip the commit if nothing
meaningfully changed" — not "calling setState with the same value does
nothing."

### Why the Setter Function Reference Never Changes

`setCount` looks like it's freshly created every render (it's
destructured from a fresh `useState(0)` call each time), but React
guarantees it's referentially stable across the component's entire
lifetime — the same function object every render. This works because
the setter isn't actually built from anything in the current render's
closure; it's created once, when the Hook's node is first added to the
fiber's Hook list (see Rules of Hooks), and bound directly to *that
Hook's slot* rather than to any render-specific values. That's why
`useCallback`/`useMemo` dependency arrays never need to list a state
setter, and why passing `setCount` down to a deeply nested child is
safe without memoization concerns — it is, by construction, one of the
few values in React guaranteed never to change identity.

# 1. What Does `useState` Actually Give You?

When you write:

```jsx
const [count, setCount] = useState(0);
```

React gives you two things:

```text
count
 ↓
Current render's state value

setCount
 ↓
Function that requests a state update
```

So conceptually:

```text
useState(0)
      ↓
┌───────────────┐
│ current value │
│ update fn     │
└───────────────┘
```

---

# 2. State Belongs to the Component Identity

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <button>{count}</button>;
}
```

The `count` value is not simply a normal local variable stored inside the function.

Remember:

> **The component function runs again on every render.**

If `count` were just a normal variable:

```jsx
function Counter() {
  let count = 0;

  count++;

  return <button>{count}</button>;
}
```

the value would be recreated every time the component function executes.

React needs state to survive renders.

Conceptually:

```text
Counter Fiber
      ↓
Hook state
      ↓
count = 0
```

Next render:

```text
Same Counter identity
      ↓
Same Hook position
      ↓
Existing state
      ↓
count = 1
```

This connects directly to the Rules of Hooks.

---

# 3. Why Doesn't a Normal Variable Work?

Consider:

```jsx
function Counter() {
  let count = 0;

  function increment() {
    count++;
  }

  return (
    <button onClick={increment}>
      {count}
    </button>
  );
}
```

Even though `count++` executes, React doesn't know that the UI needs to be rendered again.

There are two separate problems.

## Problem 1 — Persistence

The function runs again:

```text
Render #1
count = 0

Render #2
count = 0
```

## Problem 2 — React Doesn't Know It Should Render

Changing a normal variable doesn't tell React:

```text
"Hey, the UI needs updating."
```

`useState` solves both:

```text
Persistent value
+
Update mechanism
```

---

# 4. `setState` Does Not Immediately Change the Current Variable

This is one of the most important concepts.

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);

    console.log(count);
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

You might expect:

```text
1
```

But the log is:

```text
0
```

Why?

Because the current render has:

```text
count = 0
```

That render's snapshot doesn't change.

The setter requests a future update:

```text
Current render
count = 0

        ↓ setCount(1)

Future render
count = 1
```

---

# 5. State Is a Snapshot

This connects directly to the render-snapshot concept.

Suppose:

```text
Render #1
count = 0
```

The component executes using:

```text
count = 0
```

Then:

```jsx
setCount(1);
```

doesn't rewrite the JavaScript variable inside that already-running render.

Instead:

```text
setCount(1)
 ↓
schedule state update
 ↓
future render
 ↓
count = 1
```

Therefore:

> **State behaves like a snapshot for each render.**

---

# 6. Multiple Updates and Batching

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return <button>{count}</button>;
}
```

Suppose:

```text
count = 0
```

All three expressions use the same render snapshot:

```text
count + 1
 ↓
0 + 1
 ↓
1
```

Conceptually React receives:

```text
setCount(1)
setCount(1)
setCount(1)
```

The resulting state is:

```text
1
```

not:

```text
3
```

---

# 7. Functional Updates Solve This

Instead:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

React receives state transitions:

```text
previous state
      ↓
+1
      ↓
+1
      ↓
+1
```

Starting from:

```text
0
```

we get:

```text
0 → 1 → 2 → 3
```

Final state:

```text
3
```

---

# 8. Why Functional Updates Are Different

Compare:

```jsx
setCount(count + 1);
```

with:

```jsx
setCount(c => c + 1);
```

### Direct update

Uses the snapshot from the current render:

```text
count = 0
 ↓
setCount(1)
```

### Functional update

Provides React with a function describing how to calculate the next state:

```text
previous state
 ↓
previous + 1
```

This is especially useful when:

> **The next state depends on the previous state.**

---

# 9. Functional Updates Can Be Chained

Consider:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Starting from:

```text
0
```

conceptually:

```text
Update 1:
0 → 1

Update 2:
1 → 2

Update 3:
2 → 3
```

This is why functional updates are the correct pattern for multiple dependent state updates.

---

# 10. `useState` Can Store Any JavaScript Value

It isn't limited to numbers.

### String

```jsx
const [name, setName] = useState("");
```

### Boolean

```jsx
const [isOpen, setIsOpen] = useState(false);
```

### Object

```jsx
const [user, setUser] = useState({
  name: "Sonu",
  age: 27
});
```

### Array

```jsx
const [items, setItems] = useState([]);
```

---

# 11. Objects Are Replaced, Not Automatically Merged

Suppose:

```jsx
const [user, setUser] = useState({
  name: "Sonu",
  age: 27
});
```

This:

```jsx
setUser({
  name: "Rahul"
});
```

does not automatically produce:

```js
{
  name: "Rahul",
  age: 27
}
```

Instead, the state becomes:

```js
{
  name: "Rahul"
}
```

The previous object is replaced.

To preserve existing fields:

```jsx
setUser(prev => ({
  ...prev,
  name: "Rahul"
}));
```

Now:

```text
Previous
{
  name: "Sonu",
  age: 27
}

        ↓

Next
{
  name: "Rahul",
  age: 27
}
```

---

# 12. Arrays Follow the Same Principle

Suppose:

```jsx
const [items, setItems] = useState([]);
```

Don't mutate the existing array:

```jsx
items.push("A");
```

Instead:

```jsx
setItems(prev => [...prev, "A"]);
```

Conceptually:

```text
Old array
     ↓
Create new array
     ↓
Set new state
```

---

# 13. State Should Be Treated as Immutable

React state should generally be treated as immutable.

Bad:

```jsx
user.name = "Rahul";
```

Better:

```jsx
setUser(prev => ({
  ...prev,
  name: "Rahul"
}));
```

Bad:

```jsx
items.push(newItem);
```

Better:

```jsx
setItems(prev => [...prev, newItem]);
```

The principle:

> **Don't mutate existing state objects/arrays directly. Create the next state and pass it to the setter.**

---

# 14. React Uses Identity to Detect Changes

Consider:

```jsx
const user = {
  name: "Sonu"
};
```

If you mutate:

```jsx
user.name = "Rahul";
```

the object reference remains the same.

Whereas:

```jsx
setUser(prev => ({
  ...prev,
  name: "Rahul"
}));
```

creates a new object.

Conceptually:

```text
Old reference
      ↓
{ name: "Sonu" }

New reference
      ↓
{ name: "Rahul" }
```

This makes the state transition explicit.

---

# 15. Lazy Initial State

You can initialize state with a value:

```jsx
const [items, setItems] = useState(createItems());
```

If initialization is expensive, you can pass a function:

```jsx
const [items, setItems] = useState(createItems);
```

The function is used to calculate the initial state.

Conceptually:

```text
Initial mount
 ↓
createItems()
 ↓
initial state
```

This is called **lazy initialization**.

---

# 16. Important Distinction: Function as Initial Value

Suppose you want to store a function itself:

```jsx
const [handler, setHandler] = useState(myFunction);
```

There is an ambiguity because React treats a function passed to `useState` as a lazy initializer.

If you actually want the function itself to be the state value, wrap it:

```jsx
const [handler, setHandler] = useState(() => myFunction);
```

Now the outer function is the initializer, and it returns `myFunction` as the state value.

---

# 17. Lazy Initialization vs Lazy Update

Don't confuse:

```jsx
useState(createItems);
```

with:

```jsx
setItems(createItems);
```

For `useState`, a function passed as the initial argument is treated as an initializer.

For the setter, a function is treated as a functional state updater.

Example:

```jsx
setCount(prev => prev + 1);
```

means:

```text
Use previous state
 ↓
Calculate next state
```

The same function syntax has different meaning depending on where it is passed.

---

# 18. State Updates Are Scheduled

Calling:

```jsx
setCount(1);
```

doesn't mean:

```text
Immediately mutate DOM
```

The broader pipeline is:

```text
setCount(1)
      ↓
Update requested
      ↓
Scheduling / prioritization
      ↓
Render
      ↓
New state snapshot
      ↓
Reconciliation
      ↓
Commit
      ↓
DOM
```

This connects `useState` directly to Handbook 2.

---

# 19. `useState` and Fiber

A useful conceptual model:

```text
Component
     ↓
Fiber
     ↓
Hook #1
     ↓
State
```

For:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
}
```

think:

```text
Counter Fiber
      ↓
Hook state
      ↓
count
```

On the next render:

```text
Same component identity
      ↓
Same Hook position
      ↓
Existing Hook state
      ↓
New snapshot
```

This is why both **component identity** and **Hook ordering** matter.

---

# 20. What Causes a Component to Re-render?

A state update can schedule another render.

For example:

```jsx
setCount(count + 1);
```

Conceptually:

```text
setState
 ↓
React schedules work
 ↓
Component renders again
```

But remember:

> **A re-render does not necessarily mean the DOM is completely rebuilt.**

The new result still goes through reconciliation.

---

# 21. Re-render Does Not Mean Remount

This distinction is critical.

### Re-render

```text
Same component identity
 ↓
Component executes again
 ↓
Hook state preserved
```

### Remount

```text
New component identity
 ↓
Old component removed
 ↓
New component created
 ↓
Fresh Hook state
```

For example, changing a component's identity through a different key can cause the previous instance to be replaced and state to reset.

---

# 22. State Updates Can Bail Out

Suppose:

```jsx
const [count, setCount] = useState(0);

setCount(0);
```

If the next state is the same as the current state according to React's state comparison semantics, React can avoid unnecessary downstream work.

Conceptually:

```text
Current state = 0
Next state = 0
       ↓
No meaningful state change
       ↓
Can bail out
```

The important interview idea is:

> **React can skip work when it determines that the state hasn't meaningfully changed.**

---

# 23. State Updates Are Not Commands to "Change This Variable"

Don't think:

```jsx
setCount(5);
```

means:

> "Change my `count` variable to 5 right now."

Think:

> **"React, please schedule a future render whose state includes 5."**

Then:

```text
Current render
count = 0

setCount(5)

Future render
count = 5
```

This mental model prevents many React bugs.

---

# 24. Socket / Real-Time Example

Suppose a socket sends:

```text
0 → 0 → 1 → 3 → 3 → 3 → 3 → 4
```

and the component does:

```jsx
socket.on("count", value => {
  setCount(value);
});
```

Assuming the initial state is `0`:

```text
Initial render
count = 0

Socket 0
0 → 0
→ no meaningful state change

Socket 1
0 → 1
→ render

Socket 3
1 → 3
→ render

Socket 3
3 → 3
→ no meaningful state change

Socket 3
3 → 3
→ no meaningful state change

Socket 3
3 → 3
→ no meaningful state change

Socket 3
3 → 3
→ no meaningful state change

Socket 4
3 → 4
→ render
```

Therefore:

```text
Initial render = 1
Socket-caused state-change renders = 3

Total meaningful renders = 4
```

This assumes the values are primitive numbers and we're discussing the normal state-update/bailout behavior.

---

# 25. `useState` + `useEffect` Dependencies

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("effect ran", count);
  }, [count]);

  // socket...
}
```

The dependency array means:

> Run the effect after a render when `count` is different from its previous dependency value.

For:

```text
0 → 0 → 1 → 3 → 3 → 4
```

you conceptually get:

```text
Initial render
count = 0
→ effect runs

0 → 0
→ dependency unchanged

0 → 1
→ effect runs

1 → 3
→ effect runs

3 → 3
→ dependency unchanged

3 → 4
→ effect runs
```

So there are:

```text
4 effect executions total
```

including the initial mount.

---

# 26. Primitive Values vs Objects

For primitive state:

```text
0
1
"hello"
true
```

same-value comparisons are straightforward.

But consider:

```jsx
setData({ count: 0 });
setData({ count: 0 });
```

These are different object references:

```text
{count: 0} !== {count: 0}
```

Even though the contents look identical.

This is important for socket/WebSocket data because receiving a new object on every message can create new references even when the underlying data is unchanged.

---

# 27. Multiple Independent State Variables

You can use multiple `useState` calls:

```jsx
function Profile() {
  const [name, setName] = useState("");
  const [age, setAge] = useState(0);
  const [online, setOnline] = useState(false);

  return ...;
}
```

Conceptually:

```text
Hook #1 → name
Hook #2 → age
Hook #3 → online
```

This is another reason Hook order matters.

Each Hook occupies a position in the component's Hook sequence.

---

# 28. Multiple State Variables vs One Object

You could instead write:

```jsx
const [user, setUser] = useState({
  name: "",
  age: 0,
  online: false
});
```

Neither approach is universally correct.

Use separate state when values represent independent pieces of state.

An object can make sense when values form a cohesive state structure.

The important thing is to model state according to how it changes and relates.

---

# 29. Derived State

Avoid storing information that can be calculated from existing state.

For example:

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");

const fullName = `${firstName} ${lastName}`;
```

Don't necessarily create:

```jsx
const [fullName, setFullName] = useState("");
```

because now you have multiple sources of truth:

```text
firstName
lastName
fullName
```

and they can become inconsistent.

Prefer deriving:

```text
fullName
=
firstName + lastName
```

when it doesn't need independent state.

---

# 30. State Is Local to a Component Instance

If:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <button>{count}</button>;
}
```

is rendered twice:

```jsx
<Counter />
<Counter />
```

you get two independent state values.

Conceptually:

```text
Counter instance A
 ↓
count A

Counter instance B
 ↓
count B
```

Updating one does not automatically update the other.

---

# 31. State Preservation Depends on Identity

This connects directly to Handbook 2:

```text
Same identity
+
Same Hook position
 ↓
State can be preserved
```

Whereas:

```text
Different identity
 ↓
New component instance
 ↓
Fresh Hook state
```

Keys can deliberately change identity.

---

# 32. `useState` and the Render Pipeline

Putting everything together:

```text
User clicks
     ↓
setCount(...)
     ↓
Update scheduled
     ↓
React may batch / prioritize
     ↓
Render phase
     ↓
Component executes again
     ↓
useState returns updated snapshot
     ↓
New React Element tree
     ↓
Reconciliation
     ↓
Commit
     ↓
DOM update
     ↓
Browser rendering
```

This is the complete mental model.

---

# 33. Interview Question — What Does `useState` Do?

### Strong Answer

> **"`useState` lets a function component associate persistent state with its React component identity. It returns the current state value for the current render and a setter that schedules a state update. When React processes that update, the component renders again with a new state snapshot."**

---

# 34. Interview Question — Why Doesn't `setState` Immediately Update the Variable?

### Strong Answer

> **"Because each render has its own state snapshot. Calling the setter schedules an update for a future render; it doesn't mutate the state variable captured by the current render."**

---

# 35. Interview Question — Why Use Functional Updates?

### Strong Answer

> **"Functional updates are useful when the next state depends on the previous state. React passes the latest state to the updater, allowing multiple updates to be applied sequentially rather than all reading the same render snapshot."**

Example:

```jsx
setCount(prev => prev + 1);
```

---

# 36. Interview Question — Does `useState` Merge Objects?

No.

```jsx
setUser({
  name: "Rahul"
});
```

replaces the previous object.

To preserve existing fields:

```jsx
setUser(prev => ({
  ...prev,
  name: "Rahul"
}));
```

---

# 37. Interview Question — Why Should State Be Immutable?

### Strong Answer

> **"React state should be treated as immutable so that state transitions are explicit and new object/array identities can represent changes. Direct mutation can make changes harder for React and developers to reason about and can prevent expected updates when references remain unchanged."**

---

# 38. Interview Question — What Is Lazy Initialization?

Instead of:

```jsx
useState(createExpensiveValue());
```

use:

```jsx
useState(createExpensiveValue);
```

when appropriate.

The function is used to calculate the initial state.

Conceptually:

```text
Initial mount
 ↓
initializer()
 ↓
initial state
```

---

# 39. Final Mental Model

`useState` is not simply:

```text
"Give me a variable."
```

It is:

```text
Component identity
        ↓
Hook position
        ↓
Persistent state
        ↓
Current render snapshot
        ↓
Setter schedules update
        ↓
Future render
```

And the update pipeline is:

```text
setState
 ↓
Schedule
 ↓
Render
 ↓
New state snapshot
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM
```

---

# 40. Quick Revision

```text
useState(initial)
       ↓
[value, setter]
```

```text
value
 ↓
Current render's snapshot
```

```text
setter
 ↓
Schedules state update
```

```text
setCount(count + 1)
 ↓
Uses current render snapshot
```

```text
setCount(prev => prev + 1)
 ↓
Uses previous/latest state during update processing
```

```text
Object state
 ↓
Replaced, not automatically merged
```

```text
State mutation
 ❌
Direct mutation

State update
 ✅
Create next value + setter
```

---

# Chapter 33 — Core Takeaway

> **`useState` gives function components persistent state across renders. The state is associated with the component's identity and Hook position, while the value exposed to the component is a snapshot for the current render. Calling the setter schedules an update; it does not mutate the current render's snapshot. Functional updates are used when the next state depends on the previous state.**

**Chapter 33 — COMPLETE**
