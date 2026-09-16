# Chapter 35 — `useEffect` Dependency Arrays & Cleanup Deep Dive

## 1. Dependency Array — Core Idea

```jsx
useEffect(() => {
  // effect
}, [count]);
```

`count` is a dependency of the effect.

Think:

```text
Effect depends on count
        ↓
count changes
        ↓
effect needs to synchronize again
```

---

## Deep Dive `[NEW]`

### Why Object.is, Not ===, and Why That's Observable

Section 6 says dependency comparison is "based on `Object.is`
semantics" — here's why that specific choice, and where it differs
from `===` in practice. `Object.is(NaN, NaN)` is `true`, while
`NaN === NaN` is `false`; `Object.is(0, -0)` is `false`, while
`0 === -0` is `true`. Practically, this means if a dependency's value
is legitimately `NaN` on two consecutive renders, `Object.is` correctly
treats it as unchanged (so the effect doesn't re-run every render just
because a naive `===` check would always report `NaN` as different
from itself) — which is exactly why React picked `Object.is` for this
comparison instead of the more familiar `===`.

### Why React Requires a Manual Array Instead of Tracking Dependencies Automatically

Frameworks that use reactive proxies/signals (Vue, Svelte, MobX) can
auto-detect what a piece of code "depends on" because reading a
tracked value is intercepted and recorded. React deliberately keeps
state and props as plain, untracked values — no proxy wrapping, no
special read-interception — specifically so a value like `count`
behaves exactly like an ordinary JavaScript number everywhere in your
code, with no surprising behavior when destructured, passed around, or
logged. The cost is that React has no way to automatically know "this
effect read `count`" — you declare it yourself. This is a genuine,
acknowledged design trade-off (transparent, ordinary values vs.
automatic tracking), and it's exactly why the
`eslint-plugin-react-hooks` "exhaustive-deps" rule exists: since React
can't verify this for you at runtime, static analysis of your source
is the closest available substitute.

## 2. Initial Render

```jsx
useEffect(() => {
  console.log("effect");
}, [count]);
```

If:

```text
count = 0
```

on the initial mount:

```text
Render
 ↓
Commit
 ↓
Effect runs
```

The effect runs for its initial setup even though there is no previous dependency value to compare against.

---

## 3. Dependency Doesn't Change

Suppose:

```text
Previous count = 0
Current count  = 0
```

Then:

```text
count unchanged
      ↓
effect doesn't need to re-run
```

Example:

```text
0 → 0
```

No new effect synchronization is required.

---

## 4. Dependency Changes

Suppose:

```text
Previous count = 0
Current count  = 1
```

Then:

```text
count changed
      ↓
cleanup previous effect
      ↓
run new effect
```

Mental model:

```text
Old effect
    ↓
cleanup
    ↓
New effect
```

---

## 5. Why Cleanup Matters

Example:

```jsx
useEffect(() => {
  const socket = connect(roomId);

  return () => {
    socket.disconnect();
  };
}, [roomId]);
```

Initially:

```text
roomId = "general"
↓
connect("general")
```

Then:

```text
roomId = "sports"
```

React needs:

```text
disconnect("general")
        ↓
connect("sports")
```

Otherwise multiple connections could remain active.

---

## 6. Dependency Comparison

Conceptually:

```text
Previous dependency
        ↓
Current dependency
        ↓
Same?
 /   \
Yes   No
 ↓     ↓
Skip  Re-run
```

For primitive values:

```text
0 → 0   same
0 → 1   different
```

React compares dependency values using its dependency comparison semantics, which are based on `Object.is`.

---

## 7. Objects as Dependencies

Consider:

```jsx
useEffect(() => {
  // ...
}, [user]);
```

If a new object is created on every render:

```jsx
const user = {
  name: "Sonu"
};
```

then:

```text
Render 1
user → Object A

Render 2
user → Object B
```

Even though both contain:

```text
{ name: "Sonu" }
```

they are different references.

Therefore the dependency can be considered changed.

Important:

```js
{} !== {}
```

---

## 8. Functions as Dependencies

Functions behave similarly:

```jsx
function Component() {
  const handleClick = () => {
    console.log("click");
  };

  useEffect(() => {
    // ...
  }, [handleClick]);
}
```

A new function is created on each render:

```text
Render 1
handleClick → Function A

Render 2
handleClick → Function B
```

So the dependency can change every render.

This is one reason `useCallback` exists and will become important later.

---

## 9. Don't Remove Dependencies Just to Stop Effects

Suppose:

```jsx
useEffect(() => {
  connect(roomId);
}, []);
```

but the effect genuinely depends on `roomId`.

Removing `roomId` doesn't solve the underlying problem.

You can end up with:

```text
React state:
roomId = B

External system:
still connected to A
```

The two systems are now out of sync.

If the synchronization depends on `roomId`:

```jsx
useEffect(() => {
  connect(roomId);
}, [roomId]);
```

---

## 10. Complete Effect Lifecycle

Initial:

```text
Render
 ↓
Commit
 ↓
Effect setup
 ↓
External system synchronized
```

Dependency changes:

```text
Render
 ↓
Commit
 ↓
Cleanup previous effect
 ↓
Setup new effect
```

Component removed:

```text
Cleanup
```

The key mental model is:

> **When the synchronization inputs change, stop the old synchronization and establish the new one.**

---

## 11. Strict Mode

Development Strict Mode can intentionally expose bad cleanup:

```text
setup
 ↓
cleanup
 ↓
setup
```

Bad:

```jsx
useEffect(() => {
  subscribe();
}, []);
```

Better:

```jsx
useEffect(() => {
  subscribe();

  return () => {
    unsubscribe();
  };
}, []);
```

The purpose is to expose effects that aren't resilient to being started and stopped.

---

## 12. Common Mistakes

### Using effects for derived data

Avoid:

```jsx
useEffect(() => {
  setFullName(firstName + lastName);
}, [firstName, lastName]);
```

Prefer:

```jsx
const fullName = firstName + lastName;
```

---

### Forgetting cleanup

Bad:

```jsx
useEffect(() => {
  window.addEventListener("resize", handler);
}, []);
```

Better:

```jsx
useEffect(() => {
  window.addEventListener("resize", handler);

  return () => {
    window.removeEventListener("resize", handler);
  };
}, []);
```

---

### Hiding dependencies

Don't remove dependencies simply because an effect runs more often than expected.

Instead ask:

> **Why is this dependency changing?**

Then fix the underlying cause.

---

## 13. Interview Questions

### Q1. What happens when a dependency changes?

> React re-synchronizes the effect: the previous effect's cleanup runs, and the new effect setup runs with the latest values.

### Q2. Does cleanup only run on unmount?

> No. It also runs before the previous effect is replaced.

### Q3. What determines whether a dependency changed?

> React compares the current dependency with its previous value using `Object.is` semantics.

### Q4. Why can an object dependency cause an effect to run repeatedly?

> Because a newly created object has a new reference even if its contents are identical.

### Q5. Why can a function dependency cause an effect to run repeatedly?

> A function declared during rendering is recreated on each render, producing a new function reference.

---

# Quick Revision

```text
Dependency array
→ describes reactive values the effect depends on
```

```text
Initial mount
→ effect setup runs
```

```text
Dependency unchanged
→ no new synchronization
```

```text
Dependency changed
→ cleanup old
→ setup new
```

```text
Component removed
→ cleanup
```

```text
Object/function dependency
→ reference identity matters
```

```text
Strict Mode
→ can expose missing cleanup
```

---

# Final Mental Model

```text
             dependency changes
                    ↓
Render → Commit → cleanup old effect
                    ↓
              setup new effect
```

The reason for this entire process:

> **An effect represents synchronization between React and an external system. When its inputs change, React needs to stop the old synchronization and establish the new one.**

**Chapter 35 — COMPLETE**
