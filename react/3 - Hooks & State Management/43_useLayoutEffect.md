# Chapter 43 — `useLayoutEffect`

## 1. Core Mental Model

`useLayoutEffect` is similar to `useEffect`, but it runs **synchronously after React commits DOM changes and before the browser paints**.

```jsx
useLayoutEffect(() => {
  // read/measure DOM or make synchronous DOM-related adjustment
}, []);
```

Mental model:

```text
Render
  ↓
Commit DOM
  ↓
useLayoutEffect
  ↓
Browser paints
```

Compare:

```text
useEffect:

Render
  ↓
Commit DOM
  ↓
Browser may paint
  ↓
useEffect
```

> **`useLayoutEffect` is for work that must happen after commit but before paint.**

---

## Deep Dive `[NEW]`

### Why "Before Paint" Requires Blocking, and Why That's the Actual Cost

The browser paints a frame only after the current synchronous block of
JavaScript finishes running — it can't paint mid-script. React exploits
this: by running `useLayoutEffect` synchronously inside the commit
phase's layout sub-pass (see Handbook 2, Render vs Commit), before
that synchronous block of code returns control to the browser, React
guarantees the browser has nothing to paint yet that reflects an
unmeasured/unadjusted DOM state. This is precisely why it can eliminate
flicker (measure a tooltip's size, immediately reposition it, and the
user only ever sees the corrected position) — but also precisely why
anything slow inside it (a network call, a heavy loop) delays the
paint of the *entire* frame, not just this component's corner of it.
There's no partial-paint escape hatch here — blocking is the mechanism,
not a side effect of it.

### Why It Can't Run During Server Rendering

The whole premise of `useLayoutEffect` — "before the browser paints" —
presupposes a browser that's about to paint something. During
server-side rendering, there is no DOM, no paint, and no browser at
all; React is producing an HTML string. There's nothing for
"synchronously before paint" to mean in that context, which is exactly
why React warns when `useLayoutEffect` runs during SSR and recommends
`useEffect` (which simply doesn't run at all on the server) or
conditionally skipping the effect until hydration on the client.

## 2. Why Does `useLayoutEffect` Exist?

Sometimes you need to:

1. Render something
2. Measure the resulting DOM
3. Make an adjustment
4. Let the user see the final result

Example:

```jsx
const ref = useRef(null);

useLayoutEffect(() => {
  const height = ref.current.getBoundingClientRect().height;
  console.log(height);
}, []);
```

The DOM has already been committed, so it can be measured.

---

## 3. Classic Use Case — Measuring DOM

```jsx
function Box() {
  const boxRef = useRef(null);

  useLayoutEffect(() => {
    const rect = boxRef.current.getBoundingClientRect();

    console.log(rect.width);
    console.log(rect.height);
  }, []);

  return (
    <div ref={boxRef}>
      Hello
    </div>
  );
}
```

Flow:

```text
Render
 ↓
<div> committed to DOM
 ↓
useLayoutEffect
 ↓
measure DOM
 ↓
Browser paint
```

---

## 4. `useEffect` vs `useLayoutEffect`

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Runs after commit | ✅ | ✅ |
| Runs before browser paint | Generally ❌ | ✅ |
| Can measure committed DOM | ✅ | ✅ |
| Can block paint | Generally ❌ | ✅ |
| Default choice | ✅ | ❌ |
| Useful for layout measurement | Sometimes | ✅ |

Important rule:

> **Prefer `useEffect` unless you specifically need work to happen before the browser paints.**

---

## 5. Why Can `useLayoutEffect` Be Dangerous?

It runs synchronously and can block painting.

For example:

```jsx
useLayoutEffect(() => {
  // expensive work
}, []);
```

Flow:

```text
Render
 ↓
Commit
 ↓
useLayoutEffect
 ↓
expensive work
 ↓
Browser paint
```

If the work is expensive, the user experiences delayed painting.

Therefore:

> **Don't use `useLayoutEffect` just because it exists.**

---

## 6. Preventing Visual Flicker

One common reason to use it is preventing the user from seeing an intermediate DOM state.

Example:

```text
Initial DOM
   ↓
Measure
   ↓
Adjust position
   ↓
Paint final result
```

With a normal effect:

```text
Initial DOM
   ↓
Paint
   ↓
useEffect
   ↓
Adjust
   ↓
Another paint
```

The user might briefly see the wrong position.

`useLayoutEffect` can perform the adjustment before that paint.

---

## 7. Classic Example — Tooltip Positioning

Imagine a tooltip:

```text
Render tooltip
      ↓
Measure tooltip
      ↓
Calculate correct position
      ↓
Position tooltip
      ↓
Browser paints
```

This is a classic layout-effect scenario.

Goal:

```text
Avoid:
wrong position briefly visible
        ↓
correct position
```

---

## 8. Dependencies and Cleanup

Just like `useEffect`:

```jsx
useLayoutEffect(() => {
  // ...
}, [value]);
```

If:

```text
value changes
```

the layout effect runs again.

It can also have cleanup:

```jsx
useLayoutEffect(() => {
  // setup

  return () => {
    // cleanup
  };
}, []);
```

So the dependency and cleanup model is similar to `useEffect`.

---

## 9. Both Run After Commit

Important:

> **`useLayoutEffect` does not run during render.**

The DOM must be committed first.

Conceptually:

```text
Render
 ↓
Commit
 ↓
DOM exists
 ↓
useLayoutEffect
```

This is why it can safely measure the DOM.

---

## 10. `useLayoutEffect` + `useRef`

These commonly work together:

```jsx
const ref = useRef(null);

useLayoutEffect(() => {
  const rect = ref.current.getBoundingClientRect();
}, []);
```

Mental model:

```text
useRef
→ access DOM node

useLayoutEffect
→ synchronously measure/use it before paint
```

---

## 11. Don't Use It for Data Fetching

Generally avoid:

```jsx
useLayoutEffect(() => {
  fetch("/api/users");
}, []);
```

Data fetching doesn't normally need to block browser painting.

Prefer:

```jsx
useEffect(() => {
  fetch("/api/users");
}, []);
```

Use `useLayoutEffect` for work that actually needs to happen before paint.

---

## 12. Server Rendering Consideration

`useLayoutEffect` is designed around browser DOM/layout work.

During server rendering, there is no browser layout to measure.

Therefore, code using `useLayoutEffect` in server-rendered environments can require special handling or produce warnings.

Interview takeaway:

> **`useLayoutEffect` is primarily a browser-side DOM/layout synchronization tool.**

---

## 13. Interview Questions

### Q1. Difference between `useEffect` and `useLayoutEffect`?

> Both run after React commits DOM changes, but `useLayoutEffect` runs synchronously before the browser paints, while `useEffect` is generally deferred until after paint.

### Q2. When should you use `useLayoutEffect`?

> When you need to measure or synchronously adjust the DOM before the browser paints, such as positioning a tooltip or preventing visual flicker.

### Q3. Why not use it everywhere?

> Because it can block browser painting and hurt performance.

### Q4. Can you access DOM elements inside it?

> Yes. It runs after the DOM has been committed, so refs can point to the relevant DOM nodes.

### Q5. Should API fetching normally use `useLayoutEffect`?

> No. `useEffect` is normally appropriate because data fetching doesn't generally need to block painting.

---

# Quick Revision

```text
useEffect
→ default for side effects
→ generally after paint
```

```text
useLayoutEffect
→ after DOM commit
→ before paint
→ synchronous
```

```text
useLayoutEffect
→ DOM measurement
→ positioning
→ prevent visual flicker
```

```text
useLayoutEffect
→ can block paint
→ don't overuse
```

```text
useRef + useLayoutEffect
→ common DOM measurement combination
```

---

# Final Mental Model

```text
Render
  ↓
Commit
  ↓
DOM updated
  ↓
useLayoutEffect
  ↓
Browser paint
```

### Interview line:

> **`useEffect` is the default for side effects. `useLayoutEffect` is for DOM/layout work that must happen synchronously after commit but before paint.**

**Chapter 43 — COMPLETE**
