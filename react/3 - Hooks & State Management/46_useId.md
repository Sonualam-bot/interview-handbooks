# Chapter 46 — `useId` `[NEW]`

## 1. Core Mental Model

`useId` generates a unique, stable identifier string for a component instance, safe to use across a single render and, critically, safe across server-rendering and client hydration.

```jsx
function LabeledInput() {
  const id = useId();

  return (
    <>
      <label htmlFor={id}>Name</label>
      <input id={id} />
    </>
  );
}
```

Mental model:

```text
Component instance
      ↓
useId()
      ↓
Stable, unique string
      ↓
Same value on server render AND client hydration
```

> **`useId` exists to produce IDs that are unique per component instance and identical between server-rendered HTML and the client's hydration pass.**

---

## 2. The Problem It Solves

Accessible HTML frequently needs to link two elements by ID:

```jsx
<label htmlFor="name">Name</label>
<input id="name" />
```

A hardcoded string works for one instance of a component, but breaks the moment the component is rendered twice on the same page — both instances would produce `id="name"`, and `htmlFor="name"` would ambiguously match two inputs, which is invalid HTML and breaks accessibility tooling (screen readers rely on this pairing being unambiguous).

You need a *different* id per instance, generated automatically. That sounds like a job for a counter or `Math.random()` — and that's exactly where it goes wrong in a server-rendered app.

---

## 3. Why You Can't Just Use `Math.random()`

```jsx
function LabeledInput() {
  const id = useMemo(() => Math.random().toString(36), []);
  ...
}
```

This produces a *different* random value on the server (when the HTML is first generated) than it does on the client (when React hydrates that HTML and re-renders to attach event handlers). React expects the server-rendered markup and the client's first render to produce matching output; when they don't, React logs a hydration mismatch warning and, in the worst case, throws away the server-rendered DOM and rebuilds it on the client — losing the performance benefit server rendering was supposed to provide.

---

## 4. Why Not a Module-Level Counter?

```jsx
let counter = 0;
function useCounterId() {
  return useMemo(() => counter++, []);
}
```

This has the same fatal flaw as `Math.random()`, for a subtler reason: the *order* components render in on the server and the order they render in during client hydration are not guaranteed to be identical in every scenario (streaming SSR, Suspense boundaries resolving out of order, concurrent rendering interleaving work). A shared mutable counter assumes a strict, matching sequential order on both sides — an assumption `useId` is specifically designed not to depend on.

---

## 5. How `useId` Actually Achieves Server/Client Consistency

`useId` doesn't generate randomness or rely on call-order counting at all. Instead, it derives an identifier from the component's **position in the component tree** — effectively a path describing "which branch, at which depth, in which fiber's Hook list" this particular `useId` call occupies. Because the tree structure produced by rendering the same component tree is deterministic given the same input (props, route, initial data), the *same tree position* is reached on both the server and the client, producing the same ID string on both sides — without either side needing to coordinate through shared mutable state or matching call timing. This is precisely why `useId` returns something like `:r0:` or `:r1:` rather than a plain incrementing number — the format encodes tree-position information, not a simple counter.

---

## 6. Basic Example, Traced

```jsx
function Form() {
  return (
    <>
      <LabeledInput />
      <LabeledInput />
    </>
  );
}
```

Each `LabeledInput` instance is a separate fiber, at a different position in the tree. Each instance's `useId()` call resolves to a different, but *for that instance, consistent* string:

```text
LabeledInput #1 → useId() → ":r0:"
LabeledInput #2 → useId() → ":r1:"
```

Both the server-rendered HTML and the client's hydration pass reach `LabeledInput #1` at the same tree position, so both compute `":r0:"` for it — no mismatch.

---

## 7. `useId` Is Not for List Keys

A tempting but incorrect use:

```jsx
{items.map(item => (
  <li key={useId()}>{item.text}</li>   // ❌
))}
```

Two problems: Hooks cannot be called inside a callback passed to `.map()` in the first place (see Rules of Hooks — this violates "only call Hooks at the top level" the moment the list has more than one item, since a Hook can't be called inside a loop body like this at all), and even if it were allowed, `useId` is meant to be stable for accessibility attributes on a *fixed* piece of UI, not as a substitute for a data-derived key that must track a specific list item's identity across reorders (see Lists & Keys, Handbook 1). Use the item's own stable ID (`item.id`) for keys; use `useId` only for DOM-attribute linking within a component instance.

---

## 8. One `useId` Call Can Produce Multiple Related IDs

If a component needs several related but distinct IDs (e.g. a form section with multiple labeled fields), call `useId` once and derive suffixes from it, rather than calling it multiple times:

```jsx
function AddressFields() {
  const id = useId();

  return (
    <>
      <label htmlFor={`${id}-street`}>Street</label>
      <input id={`${id}-street`} />

      <label htmlFor={`${id}-city`}>City</label>
      <input id={`${id}-city`} />
    </>
  );
}
```

This guarantees the two derived IDs are related and unique together, using one stable base rather than relying on two separate `useId()` calls (which would also work, but is less common practice — deriving suffixes keeps the component's identifiers visibly grouped).

---

## 9. `useId` and Component Reuse

Each component *instance* gets its own `useId()` result, exactly like `useState` gives each instance its own state (see Handbook 1, Components: definition vs instance). Rendering `<LabeledInput />` five times produces five distinct IDs, one per fiber — there's no sharing, no global registry to manage, and no risk of accidental collision between instances of the same component used repeatedly on one page.

---

## 10. `useId` Values Are Not Meant to Be Stable Across Different Page Loads or Versions

The exact string `useId` returns is an implementation detail that can change between React versions or between different renders of a differently-shaped tree — it is guaranteed to be *consistent between server and client for one particular render*, not a permanent, portable identifier you should persist, store, or rely on having a specific format. Don't use it as a database key or a value you save and compare across sessions.

---

## 11. Interview Questions

### Q1. What problem does `useId` solve?

> It generates unique IDs for accessibility attributes (like `htmlFor`/`id` pairs) that stay consistent between server-rendered HTML and the client's hydration render, preventing hydration mismatches.

### Q2. Why can't you just use `Math.random()` for this?

> Because the server and client would independently generate different random values, causing the server-rendered markup and the client's initial render to disagree — a hydration mismatch.

### Q3. Why can't a simple incrementing counter work either?

> Because it relies on components rendering in exactly the same sequential order on the server and during client hydration, which isn't guaranteed once streaming SSR, Suspense, or concurrent rendering are involved.

### Q4. How does `useId` avoid needing shared/mutable state to stay consistent?

> It derives the ID from the component's position within the tree structure, which is deterministic given the same component tree — so the server and client independently arrive at the same value without coordinating.

### Q5. Should `useId` be used for list keys?

> No. It's for stable DOM-attribute linking within one component instance, not for tracking a list item's identity across reorders — use the data's own stable ID for keys.

---

# Quick Revision

```text
useId()
   ↓
Derived from component's tree position
   ↓
Same tree position on server and client
   ↓
Same ID string on both
   ↓
No hydration mismatch
```

- `useId` solves accessibility ID uniqueness across component instances.
- Its real purpose is server/client hydration consistency — `Math.random()` and counters both fail at this.
- It's derived from tree position, not randomness or call-order counting.
- Never use it for list keys.
- One call can be reused with derived suffixes for multiple related IDs.

# Final Mental Model

> **`useId` isn't primarily about "give me a unique string" — plenty of ways do that. It's about giving me a unique string that the server and the client agree on without talking to each other.**

**Chapter 46 — COMPLETE**
