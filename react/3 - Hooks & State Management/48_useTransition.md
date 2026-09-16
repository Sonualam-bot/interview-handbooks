# Chapter 48 — `useTransition`

## 1. Core Mental Model

`useTransition` lets React treat certain state updates as **non-urgent transitions**.

```jsx
const [isPending, startTransition] = useTransition();
```

Then:

```jsx
startTransition(() => {
  setResults(newResults);
});
```

Mental model:

```text
Urgent update
→ normal state update

Non-urgent update
→ startTransition(...)
```

> **`useTransition` lets React prioritize urgent interactions over non-urgent rendering work.**

---

## Deep Dive `[NEW]`

### What `startTransition` Actually Does: Tags Updates With a Different Lane

Every state update carries a priority, represented internally as a
lane — a bit in a bitmask (see Handbook 2, Lanes). A normal
`setState` call marks its update with an urgent (synchronous-ish)
lane. Code wrapped in `startTransition(() => setResults(newResults))`
marks that same kind of update with a separate, lower-priority
transition lane instead. That single tag is the entire mechanism:
when both an urgent lane and a transition lane have pending work on
overlapping fibers, the Scheduler processes the urgent lane's render
first, and can interrupt or delay starting the transition lane's
render until the urgent one commits. `startTransition` isn't running
your code differently, deferring it, or scheduling it for "later" in
a timer sense — it's tagging the resulting update with different
metadata that the Scheduler and lanes system already know how to
prioritize.

### Where `isPending` Comes From, Mechanically

`isPending` is itself just an ordinary piece of state, internally
flipped to `true` the instant `startTransition`'s callback is invoked
and flipped back to `false` once the transition's render has
committed — it's tracked by React specifically so you have a way to
show a stale/loading UI *while* a transition-lane render is still in
flight, without needing to manage that boolean yourself. This is why
`isPending` updates immediately (it's an urgent-lane update itself)
even though the transition it's describing is deliberately non-urgent
— the pending flag and the transitioning content are updated through
two different lanes on purpose, so the "loading" indicator never feels
sluggish even when the underlying update does.

## 2. The Problem It Solves

Imagine a search box:

```text
User types
 ↓
Input state changes
 ↓
Huge list needs to update
 ↓
UI becomes sluggish
```

Not every update has the same urgency.

```text
Input update
→ urgent

Large results update
→ less urgent
```

`useTransition` lets React distinguish between them.

---

## 3. Basic API

```jsx
const [isPending, startTransition] = useTransition();
```

Then:

```jsx
startTransition(() => {
  setResults(newResults);
});
```

The state update inside `startTransition` is marked as a transition.

---

## 4. Search Example

```jsx
function Search() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);

  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    const value = e.target.value;

    setQuery(value);

    startTransition(() => {
      setResults(search(value));
    });
  }

  return (
    <>
      <input
        value={query}
        onChange={handleChange}
      />

      {isPending && <p>Updating results...</p>}

      <Results results={results} />
    </>
  );
}
```

Now React knows:

```text
query update
→ urgent

results update
→ transition / non-urgent
```

---

## 5. What Does "Non-Urgent" Mean?

It does **not** mean:

> "Run this after exactly 5 seconds."

It means:

> **React can prioritize other more urgent work over this update.**

For example:

```text
User types
 ↓
urgent input update
 ↓
React keeps input responsive
 ↓
transition work continues
```

---

## 6. `useTransition` and Concurrent Rendering

Previously:

```text
Render work
 ↓
can be interrupted
 ↓
urgent work arrives
 ↓
React can prioritize urgent work
 ↓
resume or abandon previous render
```

`useTransition` provides a way to mark an update as lower priority.

Conceptually:

```text
startTransition()
       ↓
mark update as non-urgent
       ↓
Scheduler can prioritize urgent work
       ↓
Concurrent rendering can keep UI responsive
```

---

## 7. `isPending`

The first value is:

```jsx
const [isPending, startTransition] = useTransition();
```

`isPending` tells you that the transition is currently pending.

Example:

```jsx
{isPending && <Spinner />}
```

Mental model:

```text
Transition starts
 ↓
isPending = true
 ↓
transition finishes
 ↓
isPending = false
```

---

## 8. `startTransition`

The second value is:

```jsx
startTransition
```

Use it to mark state updates as transitions:

```jsx
startTransition(() => {
  setSomething(value);
});
```

The important point:

> **The React state update is placed inside the transition callback.**

---

## 9. What `useTransition` Does NOT Do

### It does not make JavaScript faster.

If you have:

```js
for (let i = 0; i < 10_000_000_000; i++) {
  // expensive JS
}
```

`useTransition` doesn't make that calculation faster.

It helps React schedule **rendering work** so urgent interactions can remain responsive.

---

## 10. It Doesn't Create a Background Thread

Important:

```text
useTransition
≠
new JavaScript thread
```

React still operates within the normal JavaScript execution model.

The benefit comes from React being able to:

```text
prioritize
pause
resume
abandon
schedule
```

render work.

---

## 11. `useTransition` Is About State Updates

Don't think:

```jsx
startTransition(() => {
  expensiveCalculation();
});
```

will magically make that calculation non-blocking.

The purpose is to mark React state updates:

```jsx
startTransition(() => {
  setResults(results);
});
```

---

## 12. Urgent vs Transition Update

Example:

```jsx
setQuery(value);

startTransition(() => {
  setResults(search(value));
});
```

Think:

```text
setQuery()
    ↓
urgent update
    ↓
keep input responsive


setResults()
    ↓
transition update
    ↓
lower-priority rendering
```

This distinction is the heart of `useTransition`.

---

## 13. `useTransition` vs `setTimeout`

Don't confuse:

```jsx
setTimeout(...)
```

with:

```jsx
startTransition(...)
```

They solve different problems.

```text
setTimeout
→ timing mechanism / delays JavaScript execution

startTransition
→ tells React an update is non-urgent
```

This is a common interview distinction.

---

## 14. `useTransition` vs `useDeferredValue`

### `useTransition`

You control the **state update**:

```jsx
startTransition(() => {
  setResults(results);
});
```

### `useDeferredValue`

You already have a value and want a deferred version:

```jsx
const deferredQuery = useDeferredValue(query);
```

Mental model:

```text
useTransition
→ "This state update is non-urgent."

useDeferredValue
→ "This value can lag behind."
```

---

## 15. When Should You Use It?

Good candidates:

- expensive UI updates
- filtering large lists
- rendering complex results
- tab/page transitions
- keeping user input responsive while another part of the UI updates

Example:

```text
Typing
→ urgent

Rendering huge filtered list
→ transition
```

---

## 16. Interview Questions

### Q1. What does `useTransition` do?

> It lets you mark state updates as non-urgent transitions so React can prioritize more urgent updates and keep the UI responsive.

### Q2. What does `isPending` represent?

> It indicates that a transition is currently pending.

### Q3. Does `useTransition` create another thread?

> No. It uses React's scheduling and concurrent rendering capabilities; it doesn't create a separate JavaScript thread.

### Q4. Does it make expensive JavaScript calculations faster?

> No. It helps React prioritize rendering work; it doesn't make arbitrary JavaScript execution faster.

### Q5. `useTransition` vs `setTimeout`?

> `setTimeout` delays JavaScript execution, while `startTransition` marks React state updates as non-urgent.

### Q6. `useTransition` vs `useDeferredValue`?

> `useTransition` lets you mark a state update as non-urgent; `useDeferredValue` creates a deferred version of an existing value.

---

# Quick Revision

```text
useTransition()
→ [isPending, startTransition]
```

```text
startTransition(...)
→ mark state update as non-urgent
```

```text
isPending
→ transition currently pending
```

```text
Transition
→ lower-priority rendering
```

```text
Urgent update
→ prioritized over transition work
```

```text
useTransition
≠
new JS thread
```

```text
useTransition
≠
make JS calculations faster
```

```text
setTimeout
→ delay execution

startTransition
→ React scheduling priority
```

```text
useTransition
→ defer an update

useDeferredValue
→ defer a value
```

---

# Final Mental Model

```text
User types
    ↓
setQuery()
    ↓
URGENT
    ↓
Keep input responsive


startTransition()
    ↓
setResults()
    ↓
NON-URGENT
    ↓
React schedules rendering
    ↓
Can yield to urgent work
    ↓
Complete transition
```

### Interview line:

> **`useTransition` lets React treat certain state updates as non-urgent, allowing urgent interactions to take priority during concurrent rendering.**

**Chapter 48 — COMPLETE**
