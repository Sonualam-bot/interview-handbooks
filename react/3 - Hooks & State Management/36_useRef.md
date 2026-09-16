# Chapter 36 — `useRef`

## 1. Core Mental Model

```jsx
const ref = useRef(initialValue);
```

React gives you a persistent mutable object:

```js
{
  current: initialValue
}
```

The key idea:

> **A ref stores a value that persists across renders without causing a re-render when it changes.**

---

## Deep Dive `[NEW]`

### Why Mutating `.current` Never Re-renders — The Actual Mechanism, Not Just the Rule

`useRef` gets a node in the exact same per-fiber Hook list that
`useState` and `useEffect` use (see Rules of Hooks) — so persistence
across renders works identically for both. The difference is what
happens when you change the value. Calling a state setter creates an
update object, appends it to the fiber's update queue, and asks the
Scheduler to render (see Handbook 2). Writing `ref.current = x` does
none of that — it's a plain property assignment on a plain mutable
object that happens to be stored in the Hook list. There is no setter
function involved at all for refs, so there is no code path that ever
creates an update or marks the fiber dirty. The re-render doesn't "not
happen because React decided to skip it" — it doesn't happen because
nothing ever asked for one. This is why refs are often described as
"an escape hatch": they use React's persistence mechanism without
opting into React's rendering mechanism.

### Why the Ref Object Itself Never Changes, Even Though `.current` Does

`useRef(initialValue)` returns the *same* `{ current: ... }` object on
every render — only its `.current` property is ever mutated in place.
This is deliberate: because refs are meant to be passed around (to
DOM elements, to child components, into event handlers, into effect
closures) without worrying about staleness, keeping the *container*
object stable means any closure holding a reference to `ref` — even
one captured from a much earlier render — is always looking at the
live, current value the moment it reads `.current`, since it's reading
through a stable pointer rather than a value frozen into that
closure. This is the opposite trade-off from state: state closures
capture a frozen snapshot per render; ref closures capture a stable
window onto whatever the latest value is.

## 2. `useRef` vs `useState`

### State

```jsx
const [count, setCount] = useState(0);
```

```text
setCount(...)
     ↓
React schedules update
     ↓
Component renders
```

### Ref

```jsx
const countRef = useRef(0);

countRef.current++;
```

```text
current changes
     ↓
No React render
```

| | `useState` | `useRef` |
|---|---|---|
| Persists across renders | ✅ | ✅ |
| Changing it causes render | ✅ | ❌ |
| Mutable | Should be treated immutably | ✅ |
| Access | value | `.current` |

---

## 3. Why Does `useRef` Persist?

React gives the component the same ref object across renders.

Conceptually:

```text
Render 1 → ref object A
Render 2 → ref object A
Render 3 → ref object A
```

Therefore:

```jsx
ref.current = 10;
```

and a later render can still access:

```jsx
ref.current
```

as `10`.

---

## 4. Why Doesn't Changing `.current` Re-render?

Because:

```jsx
ref.current = 10;
```

is simply a mutation of the ref object.

There is no state update:

```text
setState
 ↓
schedule render
```

So:

```text
ref.current = value
→ no automatic render
```

---

## 5. Classic Use Case — DOM References

```jsx
function SearchBox() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>
        Focus
      </button>
    </>
  );
}
```

Conceptually:

```text
<input>
   ↓
ref
   ↓
inputRef.current
```

The ref gives you access to the DOM node so you can use imperative APIs such as:

```jsx
inputRef.current.focus();
```

---

## 6. Ref Assignment and Commit

The DOM node becomes associated with the ref as part of React's commit work.

Conceptually:

```text
Render
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM node connected to ref
```

This is why refs are useful for interacting with the actual committed DOM.

---

## 7. Imperative APIs

React is primarily declarative:

```jsx
<input value={name} />
```

Sometimes you need imperative operations:

```jsx
input.focus();
video.play();
element.scrollIntoView();
```

A ref provides the bridge:

```text
React declarative UI
        ↓
       ref
        ↓
Imperative DOM API
```

---

## 8. `useRef` for Previous Values

A common pattern:

```jsx
function Counter({ count }) {
  const previousCount = useRef();

  useEffect(() => {
    previousCount.current = count;
  }, [count]);

  return <div>{count}</div>;
}
```

The ref persists between renders without triggering another render when it is updated.

This makes refs useful for storing information that future renders or callbacks may need.

---

## 9. `useRef` for Timer IDs

```jsx
const timerRef = useRef(null);
```

Then:

```jsx
timerRef.current = setInterval(() => {
  // ...
}, 1000);
```

Cleanup:

```jsx
clearInterval(timerRef.current);
```

Why use a ref?

The timer ID needs to persist between renders, but changing it doesn't need to update the UI.

---

## 10. Refs and Closures

Refs can help callbacks access a mutable value:

```jsx
const latestValue = useRef(value);

latestValue.current = value;
```

A later callback can read:

```jsx
latestValue.current
```

This can be useful with asynchronous code, subscriptions, and event handlers.

However, don't automatically use refs to hide dependency problems. First understand why a closure is stale.

---

## 11. Don't Use Ref When UI Depends on the Value

Bad:

```jsx
const count = useRef(0);

function increment() {
  count.current++;
}
```

and:

```jsx
return <h1>{count.current}</h1>;
```

Changing:

```jsx
count.current++;
```

does not tell React to render.

If the value affects the UI:

```jsx
const [count, setCount] = useState(0);
```

Use state.

---

## 12. State vs Ref Mental Model

Ask:

> **Does changing this value need to update the UI?**

### Yes

```text
useState
```

### No

```text
useRef
```

Examples:

| Requirement | Use |
|---|---|
| Counter displayed on screen | `useState` |
| Input DOM element | `useRef` |
| Timer ID | `useRef` |
| Previous value | `useRef` |
| Loading indicator shown in UI | `useState` |
| WebSocket instance | often `useRef` |
| DOM node | `useRef` |

---

## 13. Ref vs Normal Variable

Normal variable:

```jsx
function App() {
  let count = 0;
}
```

It is recreated on each render:

```text
Render 1
count = 0

Render 2
count = 0
```

A ref persists:

```jsx
const countRef = useRef(0);
```

Conceptually:

```text
Render 1
ref.current = 0

ref.current = 5

Render 2
ref.current = 5
```

Therefore:

```text
Normal variable
→ doesn't persist

Ref
→ persists

State
→ persists + triggers render
```

---

## 14. Ref vs State vs Normal Variable

| Property | Normal variable | `useRef` | `useState` |
|---|---:|---:|---:|
| Persists between renders | ❌ | ✅ | ✅ |
| Changing value causes render | ❌ | ❌ | ✅ |
| Mutable | ✅ | ✅ | Should be treated immutably |
| Intended for UI state | ❌ | ❌ | ✅ |
| Useful for DOM references | ❌ | ✅ | ❌ |

---

## 15. Don't Arbitrarily Mutate Refs During Render

Avoid using refs as a way to perform arbitrary side effects during rendering:

```jsx
function App() {
  ref.current++;

  return ...;
}
```

This makes rendering impure and can conflict with React's rendering model.

Use refs primarily for:

- DOM references
- persistent mutable values
- imperative APIs
- values needed by callbacks

Update them at appropriate points such as event handlers or effects.

---

## 16. `useRef` Is Not Only for DOM

`useRef` can store any mutable value:

```jsx
const timerId = useRef(null);
const socket = useRef(null);
const previousValue = useRef(null);
const renderCount = useRef(0);
```

The DOM is simply the most famous use case.

---

## 17. Interview Questions

### Q1. What is `useRef`?

> `useRef` returns a persistent mutable object whose `.current` value survives renders without causing a re-render when changed.

### Q2. Difference between `useRef` and `useState`?

> Both persist between renders, but updating state schedules a render while changing `ref.current` does not.

### Q3. Why use refs for DOM elements?

> They provide access to the underlying DOM node so imperative APIs such as `focus()` or `scrollIntoView()` can be used.

### Q4. Can refs store non-DOM values?

> Yes. Timer IDs, WebSocket instances, previous values, and other mutable values can be stored in refs.

### Q5. Why not use a normal variable?

> Normal variables are recreated when the component renders, while a ref object persists across renders.

---

# Quick Revision

```text
Normal variable
→ recreated every render
```

```text
useRef
→ same object across renders
→ .current is mutable
→ changing it doesn't render
```

```text
useState
→ state persists
→ setter schedules render
```

```text
If value affects UI
→ useState
```

```text
If value persists but doesn't need UI update
→ useRef
```

```text
DOM / timer / socket / previous value
→ common useRef cases
```

---

# Final Mental Model

> **If the value affects what the user sees → state.**
>
> **If the value needs to persist across renders but doesn't need to trigger a UI update → ref.**

```text
Normal variable
→ recreated

useRef
→ persists
→ mutable
→ no render

useState
→ persists
→ setter schedules render
```

**Chapter 36 — COMPLETE**
