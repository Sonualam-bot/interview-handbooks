# Chapter 49 — `useDeferredValue`

## 1. Core Mental Model

`useDeferredValue` lets React create a deferred version of an existing value.

```jsx
const deferredValue = useDeferredValue(value);
```

The current value updates normally, while the deferred value can temporarily lag behind when React is busy.

```text
Current value
     ↓
query = "abcd"

Deferred value
     ↓
may temporarily remain "abc"

Eventually
     ↓
deferredQuery = "abcd"
```

> **`useDeferredValue` lets an expensive part of the UI consume a value at a lower priority while urgent UI remains responsive.**

---

## Deep Dive `[NEW]`

### The Actual Two-Render Mechanism Behind "Temporarily Lags Behind"

When `value` changes, React doesn't hold `useDeferredValue`'s return
value back through some kind of delay timer — it does two distinct
renders. First, an urgent re-render happens immediately with the
deferred value still equal to its *previous* value (so anything urgent
depending on `value` itself updates instantly, and the expensive part
of the tree consuming the deferred value doesn't have to redo its
expensive work yet). Then React schedules a second render, tagged with
a lower-priority transition lane (the same lane mechanism
`useTransition` uses — see that chapter), where the deferred value
finally catches up to match `value`. This is why "may temporarily lag
behind" isn't a vague description — it's exactly two renders, one
urgent and one transition-priority, both real, both committed
separately, with the gap between them being whatever time the
Scheduler needs before it gets to the lower-priority lane.

### Why This Is `useTransition`'s Mechanism, Applied to a Value Instead of a Callback

`useTransition` lets *you* choose which `setState` calls get the
lower-priority lane. `useDeferredValue` is the same underlying
lanes/scheduling machinery, but React manages the transition on your
behalf around one specific value rather than around a state update you
write yourself — useful specifically when the value comes from a
parent (a prop) and you have no `setState` call of your own to wrap in
`startTransition`. This is the precise answer to "when do I choose
which": if you own the state update, wrap it in `startTransition`; if
you only own the value being consumed, defer the value.

## 2. The Problem It Solves

Suppose:

```jsx
const [query, setQuery] = useState("");
```

The user types:

```text
a → ab → abc → abcd
```

The input needs to update immediately, but a large results list may be expensive to render.

Instead of making both parts update at exactly the same speed:

```jsx
const deferredQuery = useDeferredValue(query);
```

Now:

```text
Input
→ query

Results
→ deferredQuery
```

---

## 3. Basic API

```jsx
const deferredQuery = useDeferredValue(query);
```

Example:

```jsx
function Search() {
  const [query, setQuery] = useState("");

  const deferredQuery = useDeferredValue(query);

  return (
    <>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
      />

      <SearchResults query={deferredQuery} />
    </>
  );
}
```

---

## 4. The Key Idea

`useDeferredValue` does **not** mean:

> "Wait exactly X milliseconds."

Instead:

> **React can defer updating consumers of the value so more urgent work can happen first.**

The deferred value may temporarily be stale.

---

## 5. Original Value Is Not Delayed

This is important.

```jsx
const deferredQuery = useDeferredValue(query);
```

does not delay:

```jsx
query
```

The original value updates normally.

Only the deferred version can lag.

```text
query
→ current value

deferredQuery
→ potentially stale/deferred value
```

---

## 6. Why Is This Useful?

Imagine:

```text
query
 ↓
10,000 results
 ↓
expensive rendering
```

You want:

```text
Typing
 ↓
input remains responsive
```

while:

```text
Results
 ↓
update when React gets an opportunity
```

So:

```text
query
→ urgent/current value

deferredQuery
→ lower-priority consumption
```

---

## 7. `useDeferredValue` Does NOT Make JavaScript Faster

Just like `useTransition`:

```text
useDeferredValue
≠
make expensive JavaScript faster
```

It helps React manage rendering priority.

If you have an expensive calculation, `useDeferredValue` does not make that calculation itself faster.

---

## 8. It Doesn't Create Another Thread

```text
useDeferredValue
≠
background JavaScript thread
```

React is still operating within the normal JavaScript execution model.

The benefit comes from React's ability to prioritize rendering work.

---

## 9. `useDeferredValue` vs `useTransition`

This is the most important comparison.

### `useTransition`

You control the **state update**:

```jsx
startTransition(() => {
  setResults(results);
});
```

Meaning:

> "This state update is non-urgent."

### `useDeferredValue`

You already have a value:

```jsx
const deferredQuery = useDeferredValue(query);
```

Meaning:

> "Consumers of this value can use a deferred version."

Mental model:

```text
useTransition
→ defer an UPDATE

useDeferredValue
→ defer a VALUE
```

---

## 10. When Should You Choose Which?

If you control the state setter:

```jsx
setResults(...)
```

use:

```jsx
startTransition(...)
```

If you already have a value:

```jsx
query
```

and want a deferred version:

```jsx
useDeferredValue(query)
```

---

## 11. `useDeferredValue` + `React.memo`

You may see:

```jsx
const deferredQuery = useDeferredValue(query);

return <Results query={deferredQuery} />;
```

If `Results` is expensive, it can also be memoized:

```jsx
const Results = React.memo(function Results({ query }) {
  // expensive rendering
});
```

Conceptually:

```text
query changes
 ↓
deferredQuery may remain unchanged temporarily
 ↓
Results receives same query
 ↓
React.memo can skip unnecessary rendering
```

These optimizations solve different problems.

---

## 12. Deferred Values Can Be Stale Temporarily

Suppose:

```text
query = "react"
deferredQuery = "rea"
```

This can happen while React is handling higher-priority work.

Therefore, consumers should be able to handle:

```text
current value !== deferred value
```

---

## 13. Showing Stale/Pending UI

You can detect when the deferred value is behind:

```jsx
const deferredQuery = useDeferredValue(query);

const isStale = query !== deferredQuery;
```

Then:

```jsx
<div style={{ opacity: isStale ? 0.5 : 1 }}>
  <Results query={deferredQuery} />
</div>
```

Mental model:

```text
query !== deferredQuery
        ↓
deferred UI is behind
        ↓
show subtle stale/loading indication
```

---

## 14. Common Pattern

```jsx
function SearchPage() {
  const [query, setQuery] = useState("");

  const deferredQuery = useDeferredValue(query);

  return (
    <>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
      />

      <SlowResults query={deferredQuery} />
    </>
  );
}
```

The input uses:

```text
query
```

The expensive component uses:

```text
deferredQuery
```

This separates the urgent interaction from expensive rendering.

---

## 15. `useDeferredValue` vs Debouncing

### Debouncing

```text
User types
 ↓
wait 300ms
 ↓
execute
```

Debouncing is **time-based**.

### `useDeferredValue`

```text
Value changes
 ↓
React prioritizes urgent work
 ↓
deferred rendering happens when possible
```

It is about **React rendering priority**, not a fixed time delay.

Mental model:

```text
Debounce
→ wait for time

useDeferredValue
→ defer rendering priority
```

---

## 16. It Doesn't Automatically Reduce Network Requests

Using:

```jsx
const deferredQuery = useDeferredValue(query);
```

does not automatically turn your application into a debounced API search system.

For network request control, explicit strategies such as:

```text
debouncing
caching
request cancellation
data-fetching libraries
```

may be appropriate.

Do not confuse deferred rendering with guaranteed request throttling/debouncing.

---

## 17. Interview Questions

### Q1. What does `useDeferredValue` do?

> It lets a value's consumers receive a deferred version of the value so urgent updates can remain responsive.

### Q2. Can the deferred value temporarily differ from the current value?

> Yes. The deferred value can lag behind the current value while React handles higher-priority work.

### Q3. Does it delay the original state update?

> No. The original value updates normally. Only the deferred version can lag.

### Q4. `useTransition` vs `useDeferredValue`?

> `useTransition` marks a state update as non-urgent; `useDeferredValue` creates a deferred version of an existing value.

### Q5. Is it the same as debouncing?

> No. Debouncing is time-based; `useDeferredValue` is about React rendering priority.

### Q6. Does it make expensive JavaScript calculations faster?

> No. It helps React prioritize rendering work.

---

# Quick Revision

```text
useDeferredValue(value)
→ creates deferred version of value
```

```text
Current value
→ updates normally
```

```text
Deferred value
→ can temporarily lag
```

```text
useDeferredValue
→ rendering priority
```

```text
useDeferredValue
≠
fixed delay
```

```text
useDeferredValue
≠
debounce
```

```text
useDeferredValue
≠
new JS thread
```

```text
useTransition
→ defer an update
```

```text
useDeferredValue
→ defer a value
```

---

# Final Mental Model

```text
User types
    ↓
query = "react"
    ↓
URGENT
    ↓
Input updates immediately

useDeferredValue(query)
    ↓
deferredQuery
    ↓
NON-URGENT rendering
    ↓
Results can temporarily show
older value
    ↓
React catches up
    ↓
deferredQuery = "react"
```

### Interview line

> **`useDeferredValue` lets React keep an expensive part of the UI responsive by allowing that part to consume a deferred version of a value that may temporarily lag behind the current value.**

**Chapter 49 — COMPLETE**
