# Chapter 32 — Rules of Hooks

## Chapter Overview

The Rules of Hooks are not arbitrary restrictions.

They exist because React needs to maintain a **stable ordering of Hook calls across renders** so it can correctly associate each Hook call with its corresponding internal state, effect, ref, or other Hook data.

The two fundamental rules are:

1. **Only call Hooks at the top level.**
2. **Only call Hooks from React function components or Custom Hooks.**

Core mental model:

```text
Component
   ↓
Render
   ↓
Hook #1
Hook #2
Hook #3
Hook #4
   ↓
Next render
   ↓
Hook #1
Hook #2
Hook #3
Hook #4
```

Stable order gives React a predictable association:

```text
Hook call
    ↕
Internal Hook state / effect / ref
```

---

## Deep Dive `[NEW]`

### Hooks Are Matched By Position, Not By Anything Else — Here's the Actual Structure

Each fiber keeps its Hook data as a **singly linked list**, one node
per Hook call, built in call order on the first render. `useState`,
`useRef`, `useEffect`, etc. each push one node onto this list on
mount. On every subsequent render, React doesn't look anything up by
name or by argument — it keeps a pointer into this same list and, on
each Hook call, simply advances the pointer to the *next* node and
returns whatever is stored there. There is no map, no key, no
identifier tying a specific `useState(0)` call to its slot other than
"which Hook call is this, in order, during this render." That's the
entire mechanism — and it's also the entire explanation for every rule
in this chapter: any control flow that could make one render call a
different *number or order* of Hooks than the previous render
desynchronizes the pointer walk from the list, and every Hook after
the divergence point silently reads the wrong node.

### Why the Linter Can Catch This But React Itself Can't (At Runtime, Reliably)

React could, in theory, detect "Hook order changed" after the fact by
comparing this render's Hook call count/types against the previous
render's list — and in development it partially does surface warnings
this way. But by the time that mismatch is detected, the damage is
already done: the wrong Hook has already returned the wrong stored
value to your component's code for this render. Static analysis
(`eslint-plugin-react-hooks`) instead prevents the situation before
the code ever runs, by analyzing your source for any Hook call sitting
inside an `if`, loop, or after an early `return` — control-flow shapes
that could plausibly produce a different call sequence on some render.
This is why the lint rule exists as a *separate* line of defense
rather than React just "handling it" — the failure mode is a silent
data corruption, not a crashable error, so catching it before runtime
is far more valuable than catching it after.

# 1. The Two Rules

## Rule 1 — Only Call Hooks at the Top Level

Do not call Hooks inside:

- `if`
- `else`
- loops
- nested functions
- callbacks
- event handlers
- conditional branches

Hooks should be called directly from the component or Custom Hook's top-level execution.

## Rule 2 — Only Call Hooks From React Functions

Hooks should be called from:

- React function components
- Custom Hooks

Not arbitrary JavaScript functions.

---

# 2. Why Does React Care About Hook Order?

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const [name, setName] = useState("Sonu");

  return <div>{count} {name}</div>;
}
```

Conceptually:

```text
Counter Fiber

Hook #1
→ count

Hook #2
→ name
```

On the next render:

```text
Hook #1
→ count

Hook #2
→ name
```

The order remains stable.

---

# 3. React Doesn't Identify Hooks by Variable Name

React does not fundamentally rely on:

```text
count → count state
name  → name state
```

Those are JavaScript variables.

The useful conceptual model is:

```text
Hook #1 → first Hook state
Hook #2 → second Hook state
Hook #3 → third Hook state
```

Therefore:

```text
Stable order
     ↓
Stable association
```

---

# 4. The Problem With Conditional Hooks

Consider:

```jsx
function Counter({ loggedIn }) {
  if (loggedIn) {
    const [user, setUser] = useState(null);
  }

  const [count, setCount] = useState(0);

  return <div>{count}</div>;
}
```

First render:

```text
loggedIn = true

Hook #1 → user
Hook #2 → count
```

Later:

```text
loggedIn = false

Hook #1 → count
```

The sequence changed.

React can no longer maintain the same Hook-to-state relationship.

---

# 5. What Could Go Wrong?

Imagine React conceptually has:

```text
Hook #1 → user state
Hook #2 → count state
```

Then the next render does:

```text
Hook #1 → count state
```

The position that previously represented `user` is now being used by `count`.

The underlying association becomes inconsistent.

This is why conditional Hooks are forbidden.

---

# 6. Another Example

Bad:

```jsx
function App({ show }) {
  const [count, setCount] = useState(0);

  if (show) {
    const [name, setName] = useState("");
  }

  const ref = useRef(null);

  return <div />;
}
```

When `show = true`:

```text
Hook #1 → useState(count)
Hook #2 → useState(name)
Hook #3 → useRef
```

When `show = false`:

```text
Hook #1 → useState(count)
Hook #2 → useRef
```

The second Hook changed from `useState(name)` to `useRef`.

That is the fundamental problem.

---

# 7. The Correct Approach

Instead of conditionally calling:

```jsx
if (show) {
  useEffect(...);
}
```

call the Hook unconditionally:

```jsx
useEffect(() => {
  if (show) {
    // do something
  }
}, [show]);
```

The Hook order remains stable.

The **logic inside** the Hook can be conditional.

The Hook call itself should not be conditional.

---

# 8. Conditional Logic vs Conditional Hook

### ❌ Wrong

```jsx
if (isLoggedIn) {
  useEffect(() => {
    // ...
  });
}
```

### ✅ Correct

```jsx
useEffect(() => {
  if (isLoggedIn) {
    // ...
  }
}, [isLoggedIn]);
```

Remember:

> **Don't conditionally call the Hook. Put the condition inside the Hook.**

---

# 9. Hooks Inside Loops

Invalid:

```jsx
function App() {
  for (let i = 0; i < 3; i++) {
    useState(0);
  }

  return <div />;
}
```

Why?

Because the number and ordering of Hook calls could change.

For example:

```text
Render 1
3 iterations
 ↓
Hook #1
Hook #2
Hook #3
```

Later:

```text
Render 2
2 iterations
 ↓
Hook #1
Hook #2
```

The Hook sequence changed.

---

# 10. Hooks Inside Nested Functions

Bad:

```jsx
function App() {
  function initialize() {
    useState(0);
  }

  initialize();

  return <div />;
}
```

The Hook is not being called directly from the component's top-level execution.

Nested functions can change when and how Hook calls execute.

---

# 11. Hooks Inside Event Handlers

Do not do:

```jsx
function App() {
  function handleClick() {
    const [count, setCount] = useState(0);
  }

  return <button onClick={handleClick}>Click</button>;
}
```

The event handler runs later as a response to an event. It is not part of the component's normal Hook execution sequence.

The correct pattern is:

```jsx
function App() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return <button onClick={handleClick}>+</button>;
}
```

The setter is fine inside the event handler. A new Hook call is not.

---

# 12. Hooks Inside Callbacks

Invalid:

```jsx
const callback = () => {
  useState(0);
};
```

The same principle applies:

> **Hooks must participate in the component's predictable Hook call sequence.**

---

# 13. Hooks From Regular JavaScript Functions

Consider:

```jsx
function calculateSomething() {
  const [value, setValue] = useState(0);
}
```

and:

```jsx
calculateSomething();
```

This is not a React component or Custom Hook.

If the function represents reusable React stateful logic, make it a Custom Hook:

```jsx
function useSomething() {
  const [value, setValue] = useState(0);

  return value;
}
```

Then:

```jsx
function App() {
  const value = useSomething();

  return <div>{value}</div>;
}
```

---

# 14. Why the `use` Prefix Matters

Custom Hooks conventionally begin with:

```text
use
```

Examples:

```text
useAuth
useOnlineStatus
useWindowSize
useFetch
useDebounce
```

This communicates that the function contains Hook logic and follows the Rules of Hooks.

It also allows linting tools to identify Hook usage patterns.

---

# 15. React Function Components

A function component is a valid place to call Hooks:

```jsx
function Profile() {
  const [name, setName] = useState("Sonu");

  return <div>{name}</div>;
}
```

Conceptually:

```text
React
 ↓
Profile()
 ↓
Hooks
```

---

# 16. Custom Hooks

A Custom Hook is another valid place:

```jsx
function useUser() {
  const [user, setUser] = useState(null);

  return user;
}
```

Then:

```jsx
function Profile() {
  const user = useUser();

  return <div>{user?.name}</div>;
}
```

Conceptually:

```text
React
 ↓
Profile
 ↓
useUser
 ↓
useState
```

A Custom Hook does not create a separate component. It extracts reusable Hook logic.

---

# 17. Custom Hooks Still Follow the Rules

A Custom Hook cannot bypass the Rules of Hooks.

Still wrong:

```jsx
function useSomething(condition) {
  if (condition) {
    useState(0);
  }
}
```

Instead:

```jsx
function useSomething(condition) {
  const [value, setValue] = useState(0);

  if (condition) {
    // conditional logic
  }

  return value;
}
```

The Hook itself remains unconditional.

---

# 18. Think of Hooks as an Ordered Sequence

A useful mental model:

```text
Component render

Hook #1
Hook #2
Hook #3
Hook #4
```

React expects:

```text
Render 1:
1 → 2 → 3 → 4

Render 2:
1 → 2 → 3 → 4

Render 3:
1 → 2 → 3 → 4
```

Not:

```text
Render 1:
1 → 2 → 3 → 4

Render 2:
1 → 3 → 4

Render 3:
1 → 2 → 4
```

The sequence must remain stable.

---

# 19. Hooks and Fiber

From Handbook 2:

```text
Component
    ↓
Fiber
    ↓
Hook state
```

Hooks are associated with the component's rendering state.

Therefore:

```text
Fiber identity
      +
Stable Hook ordering
      ↓
Correct Hook state association
```

The Rules of Hooks are therefore not arbitrary syntax restrictions.

---

# 20. Hooks and Re-rendering

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

Initial render:

```text
Counter
 ↓
Hook #1 → count = 0
```

Click:

```text
setCount(1)
 ↓
React schedules update
 ↓
Counter renders again
```

Next render:

```text
Counter
 ↓
Hook #1 → count = 1
```

Because Hook #1 remains the same Hook position, React can associate it with the same state.

---

# 21. What If Hook Order Changes?

Suppose:

```jsx
function Counter({ condition }) {
  if (condition) {
    useState(0);
  }

  useState(10);

  return <div />;
}
```

When `condition = true`:

```text
Hook #1 → first state
Hook #2 → second state
```

When `condition = false`:

```text
Hook #1 → second state
```

The second state has effectively moved from Hook #2 to Hook #1.

That's why React rejects this pattern.

---

# 22. The Linter Helps You

React's ecosystem provides ESLint rules for the Rules of Hooks.

A development setup can warn about mistakes such as:

```jsx
if (condition) {
  useState(0);
}
```

or:

```jsx
function handleClick() {
  useEffect(...);
}
```

The linter is useful because these mistakes can otherwise be difficult to reason about manually.

But:

> **The linter enforces the rule; it is not the reason the rule exists.**

The underlying reason is stable Hook ordering.

---

# 23. Interview Trap — Why Can't Hooks Be Conditional?

Weak answer:

> "Because React says you can't."

Strong answer:

> **"React relies on the stable order of Hook calls across renders to associate each Hook with its corresponding internal state. A conditional Hook can change that ordering between renders, causing React to associate a Hook with the wrong state."**

---

# 24. Interview Trap — Hook in a Regular Helper

Question:

> Can I call a Hook inside a regular helper function?

Answer:

> **Not if that helper is simply an arbitrary JavaScript function. Hooks should be called from React function components or Custom Hooks, following the Rules of Hooks.**

If the helper contains reusable Hook logic, turn it into a Custom Hook:

```jsx
function useSomething() {
  const [value, setValue] = useState(0);

  return value;
}
```

---

# 25. Why Can't Hooks Be Called in Event Handlers?

The execution timeline is:

```text
Render
 ↓
Component execution
 ↓
Event handler registered
 ↓
Render completes
 ↓
User clicks later
 ↓
Event handler executes
```

Hook calls need to belong to the predictable component Hook sequence.

An event handler is an arbitrary later callback.

Therefore this is valid:

```jsx
function App() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return <button onClick={handleClick}>+</button>;
}
```

But calling another Hook inside `handleClick` is invalid.

---

# 26. The Rules Are About Call Order

This distinction is critical.

This is valid:

```jsx
useEffect(() => {
  if (user) {
    connect(user);
  }
}, [user]);
```

The Hook itself is always called.

The condition only controls what the effect does.

Likewise:

```jsx
const [count, setCount] = useState(0);

if (count > 10) {
  console.log("Large");
}
```

The condition is fine because it is not controlling whether `useState` is called.

---

# 27. Complete Example

Consider:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // fetch user
  }, [userId]);

  const inputRef = useRef(null);

  return (
    <div>
      {loading ? "Loading..." : user?.name}
    </div>
  );
}
```

Conceptually:

```text
UserProfile Fiber
      ↓
Hook #1
→ user state

Hook #2
→ loading state

Hook #3
→ effect

Hook #4
→ ref
```

Every render should preserve this sequence.

---

# 28. Custom Hook Example

```jsx
function useUser(userId) {
  const [user, setUser] = useState(null);

  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // fetch user
  }, [userId]);

  return {
    user,
    loading
  };
}
```

Then:

```jsx
function UserProfile({ userId }) {
  const { user, loading } = useUser(userId);

  return (
    <div>
      {loading ? "Loading..." : user?.name}
    </div>
  );
}
```

Conceptually:

```text
UserProfile Fiber
       ↓
useUser
       ↓
Hook #1 → user
Hook #2 → loading
Hook #3 → effect
```

The Custom Hook composes Hook logic without creating a new component.

---

# 29. Final Mental Model

The Rules of Hooks exist because React needs **stable Hook ordering across renders**.

```text
Component
 ↓
Render
 ↓
Hook #1
Hook #2
Hook #3
Hook #4
 ↓
Next render
 ↓
Hook #1
Hook #2
Hook #3
Hook #4
```

That stable ordering allows React to preserve the association between:

```text
Hook call
     ↕
Internal Hook state / effect / ref
```

Therefore:

```text
Top-level calls
       +
Stable order
       ↓
Correct Hook association
```

---

# 30. Interview Summary

## Rule 1

> **Only call Hooks at the top level.**

Not inside:

```text
if
loops
nested functions
callbacks
event handlers
```

## Rule 2

> **Only call Hooks from React function components or Custom Hooks.**

---

# 31. The Most Important "Why"

> **Hooks rely on their call order to associate each Hook call with the correct internal state across renders. Conditional, loop-based, or nested Hook calls can change that order, breaking the association.**

---

# Quick Revision

```text
Why can't Hooks be conditional?
        ↓
Hook order can change
        ↓
React loses stable Hook ↔ state association
```

```text
Why top-level?
        ↓
Stable execution order
        ↓
Predictable Hook state
```

```text
Why Custom Hooks?
        ↓
Reusable Hook logic
        ↓
Still follows the same rules
```

```text
Component
   ↓
Custom Hook
   ↓
useState / useEffect / useRef
```

### One-sentence interview answer

> **"The Rules of Hooks exist because React relies on a stable order of Hook calls across renders to correctly associate each Hook with its internal state and other data."**

---

# Chapter 32 — Complete

**Handbook 3: Chapter 32 — Rules of Hooks**

Status: **COMPLETE**

Next: **Chapter 33 — `useState`**
