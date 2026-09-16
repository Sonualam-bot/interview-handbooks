# Handbook 5 — React Performance Engineering

# Chapter 66 — Why React Re-renders

## 1. What Does “Re-render” Mean?

A React re-render means React runs a component function again to calculate what the UI should look like with the latest state, props, or context.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  console.log("Counter rendered");

  return <button>{count}</button>;
}
```

When `setCount()` updates the state, React schedules an update and the component function runs again.

### Important

**Re-render does NOT mean the DOM is completely rebuilt.**

Simplified flow:

```text
setState
   ↓
React schedules an update
   ↓
Component function runs again
   ↓
React reconciles
   ↓
Only necessary DOM changes are committed
```

---

## 2. Main Things That Trigger Re-renders

### 2.1 State Updates

```jsx
const [count, setCount] = useState(0);

setCount(count + 1);
```

A state update schedules rendering work for the component that owns the state.

### 2.2 Parent Re-renders

If a parent renders again, its children normally participate in that render work.

```text
Parent state changes
      ↓
Parent renders
      ↓
Child participates in rendering
```

This does not mean the child necessarily causes a DOM update.

### 2.3 Context Updates

A component consuming context can update when the context value it observes changes.

### 2.4 External Store Updates

Components subscribed to external state systems can update when their subscribed state changes.

Examples:

- Redux
- Zustand
- `useSyncExternalStore`

---

## 3. What Does NOT Automatically Trigger a Re-render?

### Normal Variable Mutation

```js
let count = 0;
count++;
```

React does not track arbitrary JavaScript variables, so this does not schedule a render.

### Ref Mutation

```js
ref.current++;
```

Changing `ref.current` does not itself schedule a re-render.

Refs are useful for values that must persist between renders but whose changes do not need to update the UI immediately.

---

## 4. Parent vs Child State

```jsx
function App() {
  const [count, setCount] = useState(0);

  console.log("App");

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>

      <Child />
    </>
  );
}

function Child() {
  const [value, setValue] = useState(0);

  console.log("Child");

  return (
    <button onClick={() => setValue(value + 1)}>
      {value}
    </button>
  );
}
```

### Initial render

```text
App
Child
```

### Click App's button

```text
App
Child
```

Why?

```text
App state changes
     ↓
App renders
     ↓
Child participates in parent's render
```

### Click Child's button

```text
Child
```

Why?

```text
Child state changes
     ↓
Child renders
```

The child state update does not automatically cause the parent to render.

### Key interview rule

> A child state update does not automatically re-render its parent.

---

## 5. Sibling Components

If:

```text
App
 ├── CounterA
 └── CounterB
```

and `CounterA` updates its own state:

```text
CounterA state changes
      ↓
CounterA renders
```

It does not automatically mean:

```text
CounterB renders
App renders
```

State updates are associated with the component/tree that owns the state.

---

## 6. Why State Colocation Matters

Consider:

```text
App
 ├── Header
 ├── Search
 ├── ProductList
 └── Footer
```

If frequently changing search state lives in `App`:

```text
App state changes
      ↓
App renders
      ↓
Header participates
Search participates
ProductList participates
Footer participates
```

Some of that work may be unnecessary.

If the state can reasonably live closer to the search UI:

```text
App
 ├── Header
 ├── Search
 │    └── owns search state
 ├── ProductList
 └── Footer
```

the update can be more localized.

### Performance principle

> Keep frequently changing state as close as reasonably possible to the components that need it.

This connects directly to **State Colocation — Chapter 60**.

---

## 7. Re-render ≠ DOM Update

When state changes:

```text
State update
   ↓
Render phase
   ↓
React calculates the new UI
   ↓
Reconciliation
   ↓
Commit phase
   ↓
Necessary DOM changes
```

A component function running again does not mean every DOM node is recreated.

React determines what needs to change and commits the necessary changes.

---

## 8. Render Phase vs Commit Phase

### Render Phase

React determines what the UI should look like.

```text
new state/props
      ↓
component functions execute
      ↓
React creates the next element tree
      ↓
reconciliation
```

### Commit Phase

React applies the required changes to the host environment, such as the DOM.

```text
reconciled result
      ↓
commit
      ↓
DOM mutations / relevant effects
```

### Interview phrasing

> Rendering is the calculation of the next UI. Committing is when React applies the necessary changes to the DOM.

---

## 9. Why Does React Re-render a Child When the Parent Changes?

When a parent renders:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>

      <Child />
    </>
  );
}
```

React evaluates:

```jsx
<Child />
```

again as part of the parent's output.

Without an optimization such as `React.memo`, the child normally participates in that render work.

This does **not** mean the child necessarily causes a DOM change.

---

## 10. React.memo and Render Bailouts

`React.memo` can allow React to skip rendering a child when its props have not changed according to the memo comparison.

```jsx
const Child = React.memo(function Child() {
  console.log("Child");

  return <div>Hello</div>;
});
```

Conceptually:

```text
Parent renders
      ↓
React reaches memoized Child
      ↓
Compare previous props vs next props
      ↓
Same?
   ┌──┴──┐
  YES    NO
   ↓      ↓
 skip   render
```

### Important

`React.memo` does **not** memoize an expensive calculation.

It memoizes the component's rendering decision based on props.

Think:

```text
React.memo
    ↓
Compare props
    ↓
No meaningful prop change → possible bailout
Prop change               → render component
```

---

## 11. React.memo Does Not Simply Do `prevProps === nextProps`

A common oversimplification is:

```js
prevProps === nextProps
```

That is not the default comparison you should describe in an interview.

Conceptually, React.memo performs a shallow comparison of props, using `Object.is` for individual prop values.

For example:

```jsx
<Child name="Sonu" age={25} />
```

Conceptually:

```js
Object.is(prevProps.name, nextProps.name);
Object.is(prevProps.age, nextProps.age);
```

If the prop values are equal, the memoized component can bail out.

---

## 12. Referential Equality Matters

Primitives:

```js
Object.is("Sonu", "Sonu"); // true
Object.is(25, 25);         // true
```

Objects and arrays are compared by reference:

```js
const a = { name: "Sonu" };
const b = { name: "Sonu" };

Object.is(a, b); // false
```

So:

```jsx
<Child user={{ name: "Sonu" }} />
```

creates a new object when that JSX expression executes.

A memoized child may therefore see:

```text
previous user object
        ≠
new user object
```

and render again.

This becomes especially important with:

- `useMemo`
- `useCallback`
- referential equality
- `React.memo`

---

## 13. Custom React.memo Comparison

You can provide a custom comparison function:

```jsx
const Child = React.memo(
  ChildComponent,
  arePropsEqual
);
```

Example:

```js
function arePropsEqual(prevProps, nextProps) {
  return prevProps.user.id === nextProps.user.id;
}
```

Conceptually:

```text
prevProps + nextProps
        ↓
arePropsEqual()
        ↓
true  → skip render
false → render
```

Custom comparisons must be correct. If the comparator incorrectly reports equality, the component may fail to update when expected.

---

## 14. Same-State Updates and Bailouts

If an update results in the same state value, React can avoid unnecessary work.

For example:

```js
setCount(5);
```

when the current state is already `5` can be treated as no meaningful state change.

React uses `Object.is`-style equality semantics for state comparisons.

Referential identity matters here too:

```js
setUser(existingUser);
```

is different from:

```js
setUser({ ...existingUser });
```

The second creates a new object reference.

---

## 15. Re-render vs Remount `[NEW]`

These two are easy to conflate, and the distinction matters for
everything above.

### Re-render

The component function executes again while the component **retains
its identity and state** — this is everything Sections 1–14 describe.

```text
same component identity
      ↓
render again
      ↓
state preserved
```

### Remount

React discards the old component instance entirely and creates a new
one from scratch — different identity, not the same fiber continuing.

```text
old component
      ↓
removed (cleanup runs)
      ↓
new component
      ↓
fresh state
```

A remount happens when an element's type changes at the same position,
or when its `key` changes (see Handbook 1, Keys & Identity) — not from
an ordinary prop or state update.

> **A re-render never destroys the component. Only a remount does — and a remount is a different event from anything "causing a re-render" in this chapter.**

---

## 16. Don't Memoize Everything `[NEW]`

Everything in this chapter about `React.memo`, referential equality,
and bailouts exists to explain *when* memoization helps — not to argue
you should reach for it by default:

```text
Every component  → React.memo
Every function    → useCallback
Every calculation → useMemo
```

is not a good default. Memoization has its own, real costs:

- the comparison itself (walking props, checking `Object.is` per key)
- memory (holding onto the previous props/value/deps to compare against)
- code complexity (dependency arrays that must stay correct, or the
  memoization silently misses updates or never invalidates)

Reach for memoization when you've identified an actual, measured
re-render cost worth avoiding — not preemptively on every component,
which adds real overhead of its own for no guaranteed benefit.

---

## 17. Complete Mental Model

For interview questions:

```text
Something changes
      ↓
React receives/schedules an update
      ↓
Affected component/tree participates in render work
      ↓
Component functions may execute again
      ↓
React reconciles the new result
      ↓
Necessary changes are committed
```

Then ask:

### What changed?

- state?
- props?
- context?
- external store?

### Where is that state owned?

- component?
- parent?
- context?
- external store?

### What tree participates?

- only the component?
- parent subtree?
- context consumers?
- subscribed components?

### Can React bail out?

- same state value
- `React.memo`
- stable references
- other reconciliation optimizations

---

## 18. Interview Traps

### Trap 1

> “setState directly updates the DOM.”

Better:

> “A state update schedules React to render with the new state; React then reconciles and commits the necessary DOM changes.”

### Trap 2

> “Every re-render means the DOM is recreated.”

False.

A re-render means React calculates the next UI. Only necessary host changes are committed.

### Trap 3

> “When a child updates, the parent re-renders.”

False.

A child can update its own state without automatically re-rendering its parent.

### Trap 4

> “React.memo compares the entire props object with `===`.”

Too simplistic.

Default memoization uses shallow comparison of individual prop values with `Object.is` semantics.

### Trap 5

> “React.memo prevents re-renders.”

Too broad.

It can allow React to bail out of rendering a memoized component when props are equal. It does not prevent every possible render.

---

## 19. Exercise — Predict the Render Log

```jsx
function App() {
  const [count, setCount] = useState(0);

  console.log("App");

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>

      <Child />
    </>
  );
}

function Child() {
  const [value, setValue] = useState(0);

  console.log("Child");

  return (
    <button onClick={() => setValue(value + 1)}>
      {value}
    </button>
  );
}
```

### Step 1
Write the console output on the initial render.

### Step 2
Click the **App button once**. Write the output.

### Step 3
Click the **Child button once**. Write the output.

### Step 4
Change the child to:

```jsx
const Child = React.memo(function Child() {
  const [value, setValue] = useState(0);

  console.log("Child");

  return (
    <button onClick={() => setValue(value + 1)}>
      {value}
    </button>
  );
});
```

Predict the output again.

### Step 5
Explain exactly why the output changes.

Trace:

```text
Which state changed?
       ↓
Which component owns it?
       ↓
Which component renders?
       ↓
Does the child receive changed props?
       ↓
Can React.memo bail out?
```

---

## 20. Chapter Summary

Remember these five rules:

```text
1. State update → schedules rendering work.

2. Parent render → child may participate in render work.

3. Child state update → does not automatically render parent.

4. Re-render ≠ DOM update.

5. React.memo → prop comparison + possible bailout.
```

### Core Mental Model

> **React re-renders components to calculate the next UI; reconciliation determines what changed, and the commit phase applies the necessary changes.**

---

**Next: Chapter 67 — Re-render Debugging**
