# 29 — `useEffect` Dependency Arrays & Stale Closures

## Interview Importance

**Priority: High.**

This chapter covers:

- Effect dependencies
- State snapshots
- Closures
- Stale closures
- Cleanup before new effect setup
- Function/object/array identity
- Referential equality
- `useCallback`
- Missing dependencies
- Effects that update state
- Derived state anti-patterns

The key mental model:

```text
Every render
    ↓
New state snapshot
    ↓
Functions/effects capture that snapshot
    ↓
Dependencies describe what the effect depends on
    ↓
Relevant dependency changes
    ↓
Cleanup previous effect
    ↓
Run new effect
```

---

## Deep Dive `[NEW]`

### Why React Requires a Manual Dependency Array Instead of Auto-Tracking

Frameworks like Vue or Svelte can auto-detect what a piece of
reactive code "depends on" because they wrap your data in
reactive proxies/signals that record every read. React deliberately
doesn't do this — state and props in React are plain values (a
number, a plain object), not tracked/proxied ones, specifically so
`count` behaves like an ordinary JavaScript variable everywhere else
in your code (no surprising proxy behavior, no restrictions on how you
destructure or pass values around). The cost of that simplicity is
that React has no automatic way to know "this effect read `count`" —
so you declare it yourself in the dependency array. This is a genuine,
acknowledged trade-off in React's design (simplicity/transparency of
values vs. automatic dependency tracking), not an oversight — and it's
exactly why the `eslint-plugin-react-hooks` "exhaustive-deps" rule
exists: since React can't check this for you at runtime, static
analysis of your source code is the closest available substitute.

### The Comparison Is Object.is, Not ===, and That Difference Is Observable

Dependency arrays are compared index-by-index using `Object.is`, not
`===`. They agree for almost every value, but differ in two edge
cases: `Object.is(NaN, NaN)` is `true` (while `NaN === NaN` is
`false`), and `Object.is(0, -0)` is `false` (while `0 === -0` is
`true`). Practically: if a dependency's value is legitimately `NaN`
across renders, `Object.is` correctly treats it as "unchanged" (so the
effect won't re-run every render just because comparing `NaN` the
naive way would always say "different") — precisely why React chose
`Object.is` over `===` for this comparison.

### Traced Example: Why the Object/Array Identity Problem Has No "Automatic" Fix

``` jsx
useEffect(() => {
  setup(options);
}, [options]);   // options = { theme: "dark" }, recreated every render
```

There is no version of this where React could "know" that
`{theme: "dark"}` this render is "the same" as `{theme: "dark"}` last
render without either (a) deep-comparing every dependency on every
render — expensive, and ambiguous for values containing functions or
class instances — or (b) you doing the referential stabilization
yourself (`useMemo`, or lifting the object out of the component).
React chose to require (b) rather than default to (a), trading a small
amount of developer diligence for avoiding a hidden, potentially very
expensive deep-equality check running on every single render of every
component using effects.

# 1. Start With an Example

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(count);
  }, [count]);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

Initially:

```text
count = 0
```

Effect:

```text
console.log(0)
```

After clicking:

```text
count
0 → 1
```

React renders again, and the new effect sees:

```text
count = 1
```

---

# 2. Every Effect Belongs to a Render

The effect function created during a render closes over that render's values.

```text
Render #1
count = 0
   ↓
Effect₁
   ↓
captures 0
```

Then:

```text
Render #2
count = 1
   ↓
Effect₂
   ↓
captures 1
```

This is the same state snapshot model learned earlier.

---

# 3. Every Render Creates New Functions

Consider:

```jsx
function App() {
  const [count, setCount] = useState(0);

  function logCount() {
    console.log(count);
  }

  return <button onClick={logCount}>Click</button>;
}
```

The component executes again on every render.

Therefore, a new function is created:

```text
Render #1
count = 0
 ↓
logCount₁ → closes over 0

Render #2
count = 1
 ↓
logCount₂ → closes over 1
```

This is normal JavaScript closure behavior.

---

# 4. What Does the Dependency Array Do?

Example:

```jsx
useEffect(() => {
  console.log(count);
}, [count]);
```

The dependency array describes the reactive values the effect depends on.

Conceptually:

```text
Previous dependency
count = 0

Current dependency
count = 1

Changed?
YES
 ↓
Effect needs to re-synchronize
```

If the relevant dependency has not changed, React doesn't need to re-synchronize the effect because of that dependency.

---

# 5. Multiple Dependencies

```jsx
useEffect(() => {
  fetchUser(userId, token);
}, [userId, token]);
```

The effect depends on:

```text
userId
token
```

If either relevant dependency changes:

```text
userId changed?
       OR
token changed?
       ↓
YES
       ↓
Effect re-synchronizes
```

If neither changes, there is no dependency change requiring re-synchronization.

---

# 6. Missing Dependencies Can Cause Bugs

Consider:

```jsx
useEffect(() => {
  console.log(count);
}, []);
```

But:

```text
count
```

changes over time.

The effect captures the value from the render in which it was created.

Conceptually:

```text
Render #1
count = 0
 ↓
Effect
 ↓
captures 0
```

Later:

```text
count = 1
count = 2
count = 3
```

the original effect's closure can still contain:

```text
0
```

This is the idea behind a **stale closure**.

---

# 7. What Is a Stale Closure?

A stale closure occurs when a callback retains access to an older value from a previous render when you expected it to see the latest value.

Example:

```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count);
    }, 1000);

    return () => clearInterval(id);
  }, []);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

If the effect was created when:

```text
count = 0
```

the interval callback can continue seeing the old snapshot.

---

# 8. Why Does a Stale Closure Happen?

JavaScript closures capture lexical bindings.

React does not rewrite an old callback's captured render snapshot when state changes.

```text
Render #1
count = 0
 ↓
Callback₁
 ↓
captures 0
```

Then:

```text
Render #2
count = 1
```

doesn't rewrite `Callback₁`.

Instead, Render #2 creates new functions with the new snapshot.

---

# 9. Fixing the Dependency Problem

If the effect needs the latest `count`:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(id);
}, [count]);
```

Now:

```text
count changes
 ↓
Cleanup previous effect
 ↓
New effect setup
 ↓
New interval
 ↓
captures latest count
```

This works, although it means the interval is recreated whenever `count` changes. Whether that is desirable depends on the actual requirement.

---

# 10. Cleanup Before New Setup

When dependencies change:

```text
Previous effect
      ↓
Cleanup
      ↓
New effect setup
```

Example:

```jsx
useEffect(() => {
  const connection = connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

When:

```text
roomId A → B
```

conceptually:

```text
Disconnect A
      ↓
Connect B
```

This prevents multiple active connections.

---

# 11. Dependency Arrays Are More Than "When to Run"

A beginner mental model is:

> "`[count]` means run when count changes."

Useful, but incomplete.

Better:

> **The dependency array describes the reactive values that the effect's synchronization depends on.**

Example:

```jsx
useEffect(() => {
  doSomething(count);
}, [count]);
```

The effect's synchronization depends on `count`.

---

# 12. Dependencies Can Be Props

Dependencies are not limited to state.

```jsx
function User({ userId }) {
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);

  return <Profile />;
}
```

Here:

```text
userId
```

is a prop.

If:

```text
userId A → B
```

the effect needs to synchronize again.

---

# 13. Functions as Dependencies

Consider:

```jsx
function App() {
  const [count, setCount] = useState(0);

  function calculate() {
    return count * 2;
  }

  useEffect(() => {
    console.log(calculate());
  }, [calculate]);

  return <div>{count}</div>;
}
```

Every render creates a new `calculate` function:

```text
Render #1
calculate₁

Render #2
calculate₂

Render #3
calculate₃
```

The function identity changes.

Therefore, an effect that depends on `calculate` can re-run whenever the function identity changes.

---

# 14. Function Identity

In JavaScript:

```js
const a = () => {};
const b = () => {};

a === b; // false
```

Each function expression creates a different function object.

Therefore:

```text
Same code
≠
Same function reference
```

If a function is an effect dependency, React observes its identity.

---

# 15. `useCallback`

React provides:

```jsx
useCallback(...)
```

to memoize a function identity.

Example:

```jsx
const calculate = useCallback(() => {
  return count * 2;
}, [count]);
```

Conceptually:

```text
count unchanged
 ↓
calculate identity can remain stable

count changed
 ↓
new calculate function
```

But do not automatically wrap every function in `useCallback`.

Use it when stable identity actually matters.

---

# 16. Objects and Arrays as Dependencies

The same identity concept applies to objects and arrays.

Example:

```jsx
const options = {
  theme: "dark"
};

useEffect(() => {
  setup(options);
}, [options]);
```

If `options` is recreated on every render:

```text
Render #1
options₁

Render #2
options₂

Render #3
options₃
```

then:

```text
options₁ !== options₂
```

even though their contents look identical.

The effect may therefore re-run.

---

# 17. Referential Equality

For objects:

```js
{} === {} // false
```

For arrays:

```js
[] === [] // false
```

Therefore:

```text
Same contents
≠
Same reference
```

This is extremely important in React.

---

# 18. Primitive Dependencies

Primitive values behave differently:

```js
1 === 1                 // true
"hello" === "hello"     // true
true === true           // true
```

So:

```jsx
useEffect(() => {
  // ...
}, [count]);
```

is straightforward when `count` is a number.

---

# 19. Effects That Update State

Be careful with effects that update state.

Example:

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

This can create:

```text
count changes
 ↓
Effect
 ↓
setCount()
 ↓
count changes
 ↓
Effect
 ↓
setCount()
 ↓
...
```

Potential infinite render/effect loop.

Always examine effects that update state carefully.

---

# 20. Effects Should Not Manufacture Derived State

Bad:

```jsx
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
```

Better:

```jsx
const fullName = firstName + " " + lastName;
```

Why?

Because `fullName` is derived data, not an external synchronization.

The bad approach creates:

```text
State changes
 ↓
Render
 ↓
Effect
 ↓
setState
 ↓
Another render
```

when the value can be calculated directly during render.

---

# 21. Stale Closures Outside Effects

Stale closures are not unique to `useEffect`.

Example:

```jsx
function App() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setTimeout(() => {
      console.log(count);
    }, 1000);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

If the handler was created when:

```text
count = 0
```

the timeout callback captures that render's snapshot.

Even if the user changes the count before the timeout executes, the callback can still see the older value.

---

# 22. Functional Updates Solve a Different Problem

Compare:

```jsx
setCount(count + 1);
```

with:

```jsx
setCount(c => c + 1);
```

The functional form is useful when the next state depends on the previous state.

Example:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

React can process:

```text
0 → 1 → 2 → 3
```

Functional updates do not magically make every closure see the latest state.

They provide the updater with the relevant pending state when React processes the update.

---

# 23. Stale Closure Mental Model

Remember:

```text
Render #1
count = 0
 ↓
Callback₁
 ↓
captures snapshot #1
```

Then:

```text
Render #2
count = 1
 ↓
Callback₂
 ↓
captures snapshot #2
```

Old callback:

```text
Callback₁ → count 0
```

New callback:

```text
Callback₂ → count 1
```

React does not mutate `Callback₁` so that it suddenly sees the new render.

---

# 24. Why ESLint Warns About Missing Dependencies

Consider:

```jsx
useEffect(() => {
  console.log(count);
}, []);
```

The effect reads:

```text
count
```

but the dependency array says:

```text
[]
```

This is suspicious because the effect may be using a stale value.

React Hooks linting rules can identify many such missing dependency situations.

Don't blindly silence the warning without understanding why.

---

# 25. Complete Example

```jsx
function Chat({ roomId }) {
  const [message, setMessage] = useState("");

  useEffect(() => {
    const connection = createConnection(roomId);

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return (
    <input
      value={message}
      onChange={e => setMessage(e.target.value)}
    />
  );
}
```

Initial:

```text
roomId = A
 ↓
Render
 ↓
Commit
 ↓
Connect A
```

User types:

```text
message changes
 ↓
Render
 ↓
Commit
```

But:

```text
roomId unchanged
```

so the connection doesn't need to be recreated.

If:

```text
roomId A → B
```

then conceptually:

```text
Render
 ↓
Commit
 ↓
Cleanup A
 ↓
Setup B
```

---

# 26. Interview Question — What Is a Stale Closure?

Strong answer:

> **A stale closure happens when a callback retains values from an older render's lexical scope and later executes expecting those values to be current. In React, each render has its own state snapshot, so callbacks created during that render capture that snapshot.**

---

# 27. Interview Question — Why Can an Effect See an Old State Value?

Strong answer:

> **Because the effect callback belongs to a particular render and closes over that render's state snapshot. If the effect isn't re-synchronized when the relevant state changes, its callback can continue using the older value.**

---

# 28. Interview Question — Why Are Functions Sometimes Problematic Dependencies?

Strong answer:

> **Because functions created inside a component are recreated on each render, so their reference identity can change. If that function is an effect dependency, the effect can re-run whenever the function identity changes.**

---

# 29. Interview Question — Why Shouldn't You Use `useEffect` for Derived State?

Strong answer:

> **Because derived data can usually be calculated directly during render. Using an effect to set derived state adds an unnecessary render cycle and creates additional synchronization complexity.**

---

# 30. Common Mistakes

### ❌ Dependency arrays are just optimization hints.

✅ They describe the reactive values an effect depends on.

### ❌ JavaScript updates old closures when state changes.

✅ Each render has its own lexical environment/snapshot.

### ❌ Objects with identical contents are equal dependencies.

✅ Dependency comparison uses identity semantics; a newly created object is a different reference.

### ❌ Every function dependency is automatically bad.

✅ Function dependencies are fine when their identity and effect behavior are intentional.

### ❌ Add `useCallback` to every function.

✅ Memoize function identity only when there is a meaningful reason.

### ❌ `useEffect` should calculate derived state.

✅ Calculate derived values during render whenever possible.

### ❌ Missing dependencies are harmless.

✅ They can cause stale values and incorrect synchronization.

---

# 31. What You Actually Need to Know for Interviews

### Must Know

```text
Every render
→ New state snapshot

Functions / effects
→ Capture that render's snapshot

Dependency array
→ Describes reactive values the effect depends on

Dependency changes
→ Effect re-synchronizes

Cleanup
→ Previous synchronization is undone when necessary

Stale closure
→ Callback uses an older render's captured value

Function/object dependency
→ Identity matters
```

### Nice to Know

```text
useCallback
→ Can stabilize function identity
```

### Don't Waste Time Memorizing

```text
Internal hook implementation
React source-code dependency algorithms
Internal Fiber fields for effects
```

Focus on:

```text
Snapshot
Closure
Dependencies
Cleanup
Identity
```

---

# 32. 30-Second Revision

```text
Every render
    ↓
New snapshot
    ↓
Functions/effects capture that snapshot
```

For effects:

```text
Effect
 ↓
Reads reactive values
 ↓
Dependencies describe those values
 ↓
Relevant dependency changes
 ↓
Cleanup previous effect
 ↓
Run new effect
```

Remember:

```text
Missing dependency
→ Potential stale closure

Changed dependency
→ Effect re-synchronizes

New object/function reference
→ Dependency identity may change

Derived data
→ Usually calculate during render
```

---

# Final Mental Model

> **Every React render creates its own snapshot. Functions and effects created during that render close over that snapshot. The dependency array tells React which reactive values the effect's synchronization depends on. When those dependencies change, React cleans up the previous effect when necessary and establishes the new synchronization. Missing dependencies can cause stale closures, while unnecessary object/function dependencies can cause effects to re-run more often than intended.**

Core model:

```text
Render #1
count = 0
 ↓
Effect₁ captures 0

count changes
 ↓
Render #2
count = 1
 ↓
Effect₂ captures 1
```

Interview rule:

> **Don't think of dependencies only as "when should this function run?" Think: "what values does this synchronization depend on?"**
