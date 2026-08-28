# Chapter 39 — `React.memo`

## 1. Core Mental Model

`React.memo` is a performance optimization that lets React skip rendering a component when its props are considered unchanged.

```jsx
const UserCard = React.memo(function UserCard({ name }) {
  return <div>{name}</div>;
});
```

Mental model:

```text
Parent renders
      ↓
React.memo child
      ↓
Compare old props vs new props
      ↓
 ┌───────────────┐
same          changed
 ↓                ↓
skip render     render
```

> **`React.memo` memoizes a component's rendering based on its props.**

---

## 2. Why Use It?

Suppose:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>

      <Child name="Sonu" />
    </>
  );
}
```

When `count` changes:

```text
Parent renders
 ↓
Child is reached again
```

If `Child` doesn't depend on `count`, it may be unnecessary to render it again.

With:

```jsx
const Child = React.memo(function Child({ name }) {
  return <div>{name}</div>;
});
```

if:

```text
old name = "Sonu"
new name = "Sonu"
```

the child can skip rendering.

---

## 3. `React.memo` Does NOT Prevent Parent Rendering

Important:

```jsx
const Child = React.memo(ChildComponent);
```

does not stop the parent from rendering.

The parent can still render:

```text
Parent state changes
      ↓
Parent renders
      ↓
React.memo checks Child props
      ↓
Child may skip rendering
```

`React.memo` affects the memoized child.

---

## 4. Props Are Compared

By default, `React.memo` compares props using a shallow comparison based on `Object.is` semantics.

Primitive values are straightforward:

```jsx
<Child count={10} />
```

If:

```text
10 → 10
```

the prop is considered unchanged.

Objects and functions require more care because reference identity matters.

---

## 5. Object Props

Consider:

```jsx
const Child = React.memo(function Child({ user }) {
  return <div>{user.name}</div>;
});
```

Parent:

```jsx
function Parent() {
  const user = {
    name: "Sonu"
  };

  return <Child user={user} />;
}
```

Every render creates a new object:

```text
Render 1 → Object A
Render 2 → Object B
```

Therefore:

```text
Object A !== Object B
```

Even though the contents are identical:

```text
A = { name: "Sonu" }
B = { name: "Sonu" }
```

`React.memo` can therefore treat the prop as changed.

---

## 6. `useMemo` Can Stabilize Object Identity

```jsx
const user = useMemo(() => ({
  name: "Sonu"
}), []);
```

Now the value can retain the same reference:

```text
Render 1 → Object A
Render 2 → Object A
Render 3 → Object A
```

Combined with `React.memo`, this can allow the child to skip unnecessary renders.

---

## 7. Function Props

Functions have the same reference-identity issue.

```jsx
const Child = React.memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});
```

Parent:

```jsx
function Parent() {
  const handleClick = () => {
    console.log("click");
  };

  return <Child onClick={handleClick} />;
}
```

Each parent render creates a new function:

```text
Render 1 → Function A
Render 2 → Function B
```

Therefore:

```text
Function A !== Function B
```

`React.memo` can treat the function prop as changed.

---

## 8. `useCallback` Can Stabilize Function Identity

```jsx
const handleClick = useCallback(() => {
  console.log("click");
}, []);
```

Now:

```text
Render 1 → Function A
Render 2 → Function A
Render 3 → Function A
```

As long as dependencies don't change.

Therefore:

```text
React.memo
+
useCallback
```

can allow a memoized child to skip unnecessary rendering.

---

## 9. `React.memo` Is Not a Guarantee

Important:

> **`React.memo` is a performance optimization, not a guarantee that a component will never render.**

It tells React that when props are equivalent, the previous result may be reused.

Application correctness should never depend on memoization.

---

## 10. Child State Still Causes Rendering

Suppose:

```jsx
const Child = React.memo(function Child() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
});
```

If the child's own state changes:

```text
Child state changes
 ↓
Child renders
```

`React.memo` does not block the component's own state updates.

Mental model:

```text
Parent update
 ↓
React.memo
 ↓
Child may skip

Child state update
 ↓
Child renders
```

---

## 11. Context Can Also Cause Rendering

A memoized component can still re-render when context it consumes changes.

```text
Parent props unchanged
        ↓
React.memo
        ↓
Consumed context changes
        ↓
Child can render
```

So `React.memo` is not a universal "never render this component" switch.

---

## 12. Custom Comparison

You can provide a comparison function:

```jsx
const Child = React.memo(
  ChildComponent,
  (prevProps, nextProps) => {
    return prevProps.id === nextProps.id;
  }
);
```

If the comparison returns:

```text
true
```

React treats the props as equivalent for memoization.

If:

```text
false
```

the child can render.

Be careful: an incorrect comparison can cause the component to miss updates.

---

## 13. The Performance Chain

These three are commonly asked together:

```text
React.memo
→ memoizes component rendering

useMemo
→ memoizes computed value

useCallback
→ memoizes function reference
```

Example:

```jsx
const Child = React.memo(ChildComponent);

function Parent() {
  const data = useMemo(() => createData(), []);

  const handleClick = useCallback(() => {
    doSomething();
  }, []);

  return (
    <Child
      data={data}
      onClick={handleClick}
    />
  );
}
```

The child can potentially skip rendering when:

```text
data reference unchanged
AND
handleClick reference unchanged
```

---

## 14. Don't Use `React.memo` Everywhere

Avoid blindly wrapping every component:

```text
Every component
 ↓
React.memo
 ↓
Everything optimized
```

Memoization has a cost and adds complexity.

Use it when there is a meaningful optimization opportunity, such as:

- expensive child renders
- frequent parent renders
- stable props
- measurable unnecessary work

---

## 15. Interview Questions

### Q1. What does `React.memo` do?

> It memoizes a functional component and can skip rendering it when its props are considered unchanged.

### Q2. Does `React.memo` stop the parent from rendering?

> No. It only affects the memoized child.

### Q3. How are props compared?

> By default, React uses a shallow comparison based on `Object.is` semantics.

### Q4. Why can an object prop break memoization?

> Because a newly created object has a different reference even when its contents are identical.

### Q5. Why use `useCallback` with `React.memo`?

> To keep function props stable so the memoized child can potentially skip rendering.

### Q6. Does `React.memo` prevent state updates inside the child?

> No. The child's own state updates can still cause it to render.

### Q7. Is `React.memo` required for correctness?

> No. It is a performance optimization.

---

# Quick Revision

```text
React.memo
→ memoize component rendering
```

```text
Same props
→ child may skip render
```

```text
Changed props
→ child renders
```

```text
Object/function props
→ reference identity matters
```

```text
useMemo
→ stable computed value
```

```text
useCallback
→ stable function reference
```

```text
Child state changes
→ child still renders
```

```text
Consumed context changes
→ child can still render
```

---

# Final Mental Model

```text
Parent renders
      ↓
React.memo child
      ↓
Compare props
      ↓
 ┌───────────────┐
same          changed
 ↓                ↓
skip            render
```

But:

```text
Child state changes
      ↓
Child renders
```

And:

```text
Consumed context changes
      ↓
Child can render
```

### The three to remember:

> **`React.memo` → component**

> **`useMemo` → value**

> **`useCallback` → function**

**Chapter 39 — COMPLETE**
