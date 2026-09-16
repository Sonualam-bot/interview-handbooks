# Chapter 47 — `useSyncExternalStore` `[NEW]`

## 1. Core Mental Model

`useSyncExternalStore` lets a component safely read and subscribe to a value that lives **outside React's own state system** — a browser API, a third-party store (Redux, Zustand), a WebSocket connection's latest message — without risking the component seeing an inconsistent value during concurrent rendering.

```jsx
const isOnline = useSyncExternalStore(
  subscribe,     // how to subscribe to changes
  getSnapshot    // how to read the current value
);
```

Mental model:

```text
External store
      ↓
subscribe(callback)    → store notifies React when it changes
getSnapshot()           → store tells React the current value
      ↓
useSyncExternalStore
      ↓
Component always renders a value consistent with
what the rest of the tree is seeing THIS render
```

> **`useSyncExternalStore` is the official, safe bridge between React's rendering model and state that React doesn't own.**

---

## 2. The Problem It Solves

Plenty of state doesn't live in `useState`/`useReducer` — it lives in a store outside React entirely: `window.navigator.onLine`, a Redux store, a WebSocket's latest payload cached in a module-level variable. The obvious way to read such a value and re-render when it changes is `useState` + `useEffect`:

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function handleChange() {
      setIsOnline(navigator.onLine);
    }
    window.addEventListener("online", handleChange);
    window.addEventListener("offline", handleChange);
    return () => {
      window.removeEventListener("online", handleChange);
      window.removeEventListener("offline", handleChange);
    };
  }, []);

  return isOnline;
}
```

This works for simple cases — but under concurrent rendering, it has a subtle correctness gap: the external value can change *during* a render that's already in progress (see Handbook 2, Concurrent Rendering), and this pattern has no way to guarantee every part of the tree being rendered in that pass sees the *same* snapshot of the external value. Different components could end up rendering against different moments of the external store's history within what should be one consistent render — a bug called **tearing**.

---

## 3. What Tearing Actually Means, Concretely

Imagine two components in the same tree both read `store.getState().theme`, and the store's value changes from `"light"` to `"dark"` partway through a concurrently-interruptible render:

```text
Render starts (low priority)
  → Component A reads theme = "light"
  → [render paused, higher-priority work runs]
  → store updates: theme = "dark"
  → [render resumes]
  → Component B reads theme = "dark"
Commit: A shows "light" styling, B shows "dark" styling — same render, disagreeing.
```

Neither component did anything wrong — each honestly read the store's value at the moment it executed. The bug is structural: nothing coordinated "the value everyone in this render should agree on," because the store lives outside React's own snapshot/versioning model (see Handbook 1/2 — React's own state is inherently snapshot-consistent per render; external stores aren't, by default).

---

## 4. How `useSyncExternalStore` Prevents Tearing

`useSyncExternalStore` gives React a contract it can enforce: `getSnapshot` must return the current value, and React can call it as many times as needed — including re-checking it right before committing — to verify the value hasn't changed since the render started. If React detects the snapshot changed mid-render (via a mismatch when it re-checks), it discards the in-progress render and synchronously re-renders from scratch with the fresh value, rather than letting a stale read commit alongside a fresh one. This is the specific guarantee plain `useState` + `useEffect` cannot offer for *external* data: React can only make this consistency guarantee for state it manages itself, unless a hook explicitly opts external data into the same discipline — which is exactly what `useSyncExternalStore` does.

---

## 5. The Three Arguments

```jsx
useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?);
```

- **`subscribe(callback)`** — a function you provide that registers `callback` with the external store and returns an unsubscribe function. React calls `callback` whenever *you* detect the store changed; React then re-reads `getSnapshot`.
- **`getSnapshot()`** — returns the current value. Must return a value that's `Object.is`-stable when nothing has changed (returning a brand-new object every call — even with identical contents — causes React to think the store changed on every render, producing an infinite re-render loop).
- **`getServerSnapshot()`** (optional) — used during server rendering, since the browser APIs many external stores wrap (like `navigator.onLine`) don't exist on the server at all.

---

## 6. Traced Example: `useOnlineStatus`, Rewritten

```jsx
function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function getSnapshot() {
  return navigator.onLine;
}

function useOnlineStatus() {
  return useSyncExternalStore(subscribe, getSnapshot);
}
```

Compared to the `useState` + `useEffect` version in Section 2, the subscription logic looks similar — the difference is entirely in what React does internally with the value once it has it: enforcing the consistency check described in Section 4, which the manual version has no way to opt into.

---

## 7. `getServerSnapshot` and Hydration

```jsx
function getServerSnapshot() {
  return true; // navigator.onLine doesn't exist on the server; pick a sane default
}

useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
```

Without `getServerSnapshot`, calling `getSnapshot` during server rendering would either throw (if it references `window`/`navigator`, which don't exist server-side) or, if it doesn't throw, risk producing a value that doesn't match what the client's first render produces — a hydration mismatch (see `useId`, which solves a related but distinct server/client consistency problem). `getServerSnapshot` exists specifically to give an explicit, safe value for the server-rendered pass, decoupled from whatever `getSnapshot` does on the client.

---

## 8. Why Library Authors Use This, Even If You Rarely Call It Directly

Most application code never calls `useSyncExternalStore` by hand — it shows up as the implementation detail *inside* libraries like Redux (`react-redux`'s `useSelector`), Zustand, Jotai, and other external-state libraries. Those libraries need exactly the guarantee this Hook provides: many components across a tree reading from one shared, externally-managed store, with a hard requirement that concurrent rendering never lets two components in the same commit disagree about the store's state. `useSyncExternalStore` exists in React's public API specifically so library authors have an officially-supported, correct primitive to build on, instead of every state-management library inventing its own (likely subtly incorrect) way of bridging external state into React's concurrent rendering model.

---

## 9. Common Mistake: Returning a New Object Every Call

```jsx
function getSnapshot() {
  return { online: navigator.onLine };   // ❌ new object every call
}
```

Since React compares snapshots with `Object.is` to decide whether the store "changed," a freshly-allocated object is never `Object.is`-equal to the previous one, even with identical contents — React concludes the store changed on every single check, triggering a re-render loop. The fix is the same discipline as any other reference-equality-sensitive React API (`useMemo` dependencies, `React.memo` props): return a primitive, or return the *same* cached object reference when the underlying data hasn't changed.

---

## 10. `useSyncExternalStore` vs `useState` + `useEffect`, When to Choose Which

| | `useState` + `useEffect` | `useSyncExternalStore` |
|---|---|---|
| Source of truth | Inside this component (or a hoisted local one) | Genuinely external to React |
| Tearing-safe under concurrent rendering | No | Yes |
| Typical use | Most everyday component state | Bridging a real external store/API into React |

If the state is something *you* created and manage entirely within React (even if lifted up or put in Context), plain `useState`/`useReducer` is correct and simpler. Reach for `useSyncExternalStore` specifically when the value's source of truth lives somewhere React doesn't control at all.

---

## 11. Interview Questions

### Q1. What problem does `useSyncExternalStore` solve?

> It safely subscribes a component to state that lives outside React (a browser API, a third-party store) while guaranteeing that value stays consistent across a single render, even under concurrent rendering — something a manual `useState` + `useEffect` subscription can't guarantee.

### Q2. What is "tearing"?

> A bug where different parts of the same render read an external value at different points in time as it changes mid-render, causing them to disagree within what should be one consistent UI update.

### Q3. What are the three arguments to `useSyncExternalStore`?

> `subscribe` (registers a change listener), `getSnapshot` (reads the current value), and an optional `getServerSnapshot` (a safe value to use during server rendering).

### Q4. Why must `getSnapshot` return a stable value when nothing has changed?

> Because React compares snapshots with `Object.is` to detect changes; returning a new object every call makes React think the store changed every time, causing an infinite re-render loop.

### Q5. Why do libraries like Redux use this internally?

> Because they need many components reading one shared external store to never disagree about its state within a single concurrent render — exactly the guarantee `useSyncExternalStore` is built to provide.

---

# Quick Revision

```text
External store changes
      ↓
subscribe's callback fires
      ↓
React re-reads getSnapshot()
      ↓
React re-renders, re-verifying the snapshot hasn't
changed again mid-render before committing
      ↓
No tearing: every consumer in this render agrees
```

- `useSyncExternalStore` bridges external (non-React-owned) state into React safely.
- It exists specifically to prevent tearing under concurrent rendering.
- `getSnapshot` must be `Object.is`-stable when unchanged, or it loops.
- `getServerSnapshot` avoids hydration mismatches for values that don't exist server-side.
- Most app code encounters this indirectly, through state-management libraries built on top of it.

# Final Mental Model

> **`useState` manages state React owns. `useSyncExternalStore` safely reads state React doesn't own, without letting concurrent rendering show two different, disagreeing snapshots of it in the same commit.**

**Chapter 47 — COMPLETE**
