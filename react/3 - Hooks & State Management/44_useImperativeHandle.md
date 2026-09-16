# Chapter 44 — `useImperativeHandle` `[NEW]`

## 1. Core Mental Model

`useImperativeHandle` customizes what a `ref` points to when a parent attaches a ref to a component. Instead of exposing whatever the component would normally hand back through `forwardRef` (usually a raw DOM node), the component hand-picks a custom object.

```jsx
const Input = forwardRef(function Input(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current.focus();
    },
    clear() {
      inputRef.current.value = "";
    }
  }));

  return <input ref={inputRef} {...props} />;
});
```

Usage:

```jsx
function Form() {
  const inputRef = useRef(null);

  return (
    <>
      <Input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
      <button onClick={() => inputRef.current.clear()}>Clear</button>
    </>
  );
}
```

Mental model:

```text
Parent
  ↓
ref
  ↓
Input (forwardRef)
  ↓
useImperativeHandle
  ↓
Custom object exposed as ref.current
(NOT the raw <input> DOM node)
```

> **`useImperativeHandle` replaces "whatever a ref would normally resolve to" with a hand-picked object the component controls.**

---

## 2. The Problem It Solves

Without `useImperativeHandle`, `forwardRef` exposes whatever the `ref` is attached to inside the component — usually a raw DOM node:

```jsx
const Input = forwardRef((props, ref) => <input ref={ref} {...props} />);
```

Now the parent has the **entire DOM node's API**: `.value`, `.style`, `.remove()`, `.addEventListener()` — everything, unfiltered. That's a much larger surface than most components want to hand out. A parent that reaches in and does `inputRef.current.value = "hacked"` or `inputRef.current.remove()` bypasses the component's own rendering logic entirely — now the real DOM and what React believes it rendered can disagree.

`useImperativeHandle` narrows this: the component decides the complete, explicit list of things a parent is allowed to do through the ref — function by function — instead of exposing its internals wholesale.

---

## 3. Why This Requires `forwardRef` (or React 19's ref-as-prop)

`useImperativeHandle`'s first argument is the `ref` the parent passed in. A plain function component has no built-in way to receive that `ref` — before React 19, `ref` was extracted out of props entirely and only reachable through `forwardRef`'s second argument (see the `forwardRef` chapter for why `ref` is special). So `useImperativeHandle` is *always* paired with something that gives the component access to the incoming `ref` object in the first place:

```jsx
// Pre-React 19
const Input = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({ ... }));
  return <input />;
});

// React 19+ (ref as a normal parameter)
function Input({ ref, ...props }) {
  useImperativeHandle(ref, () => ({ ... }));
  return <input />;
}
```

Either way, `useImperativeHandle` needs a `ref` object to attach its custom handle to — it doesn't create the ref itself, it only decides what the ref's `.current` ends up being.

---

## 4. `useImperativeHandle` Does NOT Create the Ref

A common misreading is that `useImperativeHandle` produces a ref. It doesn't:

```text
useRef() / ref-as-prop
      ↓
Produces the ref object
      ↓
useImperativeHandle(ref, factory)
      ↓
Decides what ref.current becomes
```

If the parent never passes a `ref` at all, `useImperativeHandle` has nothing to attach its custom handle to and does nothing observable.

---

## 5. The Dependency Array

`useImperativeHandle` accepts an optional third argument, a dependency array, exactly like `useMemo`/`useEffect`:

```jsx
useImperativeHandle(ref, () => ({
  focus() {
    inputRef.current.focus();
  }
}), []);
```

Without it, the factory function re-runs on every render, producing a new handle object every time (usually harmless, but wasteful if a parent is comparing identity, e.g. inside a dependency array of its own). With `[]`, the handle object is created once and reused — the same "compute once, reuse until dependencies change" behavior as `useMemo`, applied to the object exposed through the ref.

---

## 6. Why Not Just Expose the DOM Node Directly?

You *can* just forward the raw node (`<input ref={ref} />`) — most components should, since it's simpler and gives the parent the full, familiar DOM API. `useImperativeHandle` is for the narrower case where:

- You want to expose a **curated, intentional API** (`focus()`, `clear()`, `scrollToTop()`) instead of the entire DOM node.
- The "thing" being controlled isn't a single DOM node at all — it might coordinate several internal refs, or trigger internal state changes that a raw DOM handle couldn't express (e.g. `open()`/`close()` on a custom modal that needs to update React state, not just mutate the DOM).
- You want to prevent a parent from reaching into internals you don't want touched directly (`.style`, `.value`, arbitrary DOM mutation).

---

## 7. Example: A Custom Video Player

```jsx
const VideoPlayer = forwardRef(function VideoPlayer(props, ref) {
  const videoRef = useRef(null);

  useImperativeHandle(ref, () => ({
    play() {
      videoRef.current.play();
    },
    pause() {
      videoRef.current.pause();
    },
    seek(seconds) {
      videoRef.current.currentTime = seconds;
    }
  }));

  return <video ref={videoRef} src={props.src} />;
});
```

The parent gets `play`, `pause`, and `seek` — nothing else. It cannot directly set `.currentTime`, remove the element, or read internal playback state that wasn't explicitly exposed. The component author decided the entire imperative contract up front.

---

## 8. `useImperativeHandle` Is an Escape Hatch, Not a Data-Flow Mechanism

React's default data flow is props down, events up (see Lifting State Up, Handbook 1). `useImperativeHandle` deliberately steps outside that model — it exists for cases where a parent genuinely needs to *command* a child imperatively (focus an input, play a video, trigger a scroll, open a modal) rather than *describe* what the child's UI should look like via props. Reaching for it to solve an ordinary data-flow problem (passing values, triggering renders) is a sign the component should just use props and state instead — imperative handles should be reserved for actions that don't have a natural declarative expression.

---

## 9. Interaction With `React.memo`

Wrapping a `forwardRef` component in `React.memo` doesn't change anything about `useImperativeHandle` — the ref and the memoization are independent concerns. `React.memo`'s shallow prop comparison still governs whether the component re-renders; `useImperativeHandle`'s dependency array still governs whether the exposed handle object is recreated. The two systems don't interact directly, though a stable (memoized) handle object is what makes it safe for a parent to use `inputRef.current` inside its own `useEffect` dependency array without triggering unnecessary re-synchronization.

---

## 10. Strict Mode Consideration

Like other Hook factory functions, the function passed to `useImperativeHandle` can run more than once in development under Strict Mode's double-invocation behavior (see Handbook 2, Strict Mode). If the factory has a side effect (it shouldn't — it should just build and return an object), that side effect would run twice in development, exposing the same class of bug Strict Mode is designed to catch elsewhere.

---

## 11. When Should You Use It?

- Exposing an imperative API from a reusable UI primitive (input, video player, modal, custom scrollable list) that other teams/consumers will use as a black box.
- Restricting what a ref can do, specifically to prevent uncontrolled DOM mutation from outside the component.
- Coordinating multiple internal refs behind one simple exposed interface.

## 12. When Shouldn't You Use It?

- When props and state can express the same behavior declaratively — always prefer that first.
- As a shortcut to avoid lifting state up. If a parent needs to *know* something about a child's state (not just trigger an action), that's a sign for state/props, not an imperative handle.
- On components you don't intend anyone to hold a ref to at all — don't add it "just in case."

---

## 13. Interview Questions

### Q1. What does `useImperativeHandle` do?

> It lets a component customize the value exposed to a parent's `ref`, instead of exposing the underlying DOM node or component instance directly.

### Q2. Why is `useImperativeHandle` almost always paired with `forwardRef`?

> Because a function component only receives the parent's `ref` object through `forwardRef`'s second argument (or, in React 19, as a normal prop) — `useImperativeHandle` needs that `ref` to decide what its `.current` becomes.

### Q3. Does `useImperativeHandle` create the ref?

> No. It only customizes what an existing `ref` resolves to; the ref itself is created by `useRef` (or passed down) in the parent.

### Q4. Why not just forward the raw DOM node instead?

> Forwarding the raw node exposes its entire native API. `useImperativeHandle` lets the component expose a narrower, intentional set of actions instead.

### Q5. What does the dependency array do?

> Same role as in `useMemo`/`useEffect` — the exposed handle object is only recreated when a listed dependency changes; otherwise the previous handle object is reused.

---

# Quick Revision

```text
ref created (useRef / ref-as-prop)
      ↓
Passed to a forwardRef component
      ↓
useImperativeHandle(ref, factory, deps)
      ↓
ref.current = factory() result
      ↓
Parent calls ref.current.someMethod()
```

- `useImperativeHandle` customizes what a `ref` exposes.
- It requires `forwardRef` (or React 19's ref-as-prop) to receive the ref at all.
- It does not create the ref — only shapes what `.current` becomes.
- Use it to expose a curated imperative API, not to replace props/state data flow.
- The dependency array works exactly like `useMemo`'s.

# Final Mental Model

> **`forwardRef` gets you the ref. `useImperativeHandle` decides what that ref is allowed to do.**

**Chapter 44 — COMPLETE**
