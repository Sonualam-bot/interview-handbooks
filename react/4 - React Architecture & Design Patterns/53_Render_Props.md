# Chapter 53 — Render Props

**Handbook 4 — React Architecture & Design Patterns**

---

## 1. What is a Render Prop?

A **render prop** is a React pattern where a component receives a **function as a prop** and calls that function to decide what UI should be rendered.

The key idea:

> **The component owns reusable logic; the consumer owns the UI.**

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div onMouseMove={(e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    }}>
      {render(position)}
    </div>
  );
}
```

Usage:

```jsx
<MouseTracker
  render={({ x, y }) => (
    <p>
      Mouse: {x}, {y}
    </p>
  )}
/>
```

`MouseTracker` owns the mouse-tracking logic, while the consumer decides how to display the data.

---

## Deep Dive `[NEW]`

### Why This Doesn't Add an Extra Component to the Tree

Calling `render(position)` inside `MouseTracker`'s own render output is
ordinary JavaScript function invocation — it happens *during*
`MouseTracker`'s render, and whatever it returns becomes part of
`MouseTracker`'s own React Element tree directly. No new fiber is
created for "the render prop" itself; there's no wrapper component in
between `MouseTracker` and whatever `render` returns. This is the
structural reason render props don't suffer from HOCs' "wrapper hell"
(Chapter 54) — a render prop is just a value (a function) passed
through props, evaluated inline, not a separate component boundary
that shows up in the tree or in React DevTools as its own node.

### The Real Cost: a New Function Identity on Every Parent Render

```jsx
<MouseTracker render={(pos) => <p>{pos.x}, {pos.y}</p>} />
```

Every time the component holding this JSX re-renders, `(pos) => ...`
is a fresh arrow function — a new function object, every time, exactly
like any other inline function prop (see Handbook 3, `useCallback`).
If `MouseTracker` were wrapped in `React.memo`, that memoization would
be defeated on every parent re-render purely because `render`'s
identity always changes, even though the *logic* inside it hasn't.
This is the concrete, measurable version of "render props can hurt
performance if not used carefully" — it isn't about the pattern being
slow in the abstract, it's the same reference-identity cost every
inline function prop has, just easy to overlook because the function
is doing the interesting work of describing UI rather than looking
like "just a callback."

## 2. Why use Render Props?

Suppose multiple components need the same behavior:

- mouse tracking
- fetching data
- subscriptions
- form state
- animation state
- drag/drop behavior

Instead of duplicating the logic, a component can own the behavior and expose the relevant state through a function.

Conceptually:

```text
Reusable component
       |
       | state / behavior
       ↓
render(data)
       |
       ↓
Consumer-controlled UI
```

This gives you **logic reuse + UI flexibility**.

---

## 3. Basic Example

```jsx
function DataProvider({ render }) {
  const [data, setData] = useState(null);

  // imagine data fetching here

  return render(data);
}
```

Consumer:

```jsx
<DataProvider
  render={(data) => (
    <div>
      {data ? data.name : "Loading..."}
    </div>
  )}
/>
```

The provider does not need to know what the UI looks like.

---

## 4. The Render Prop Does Not Have to be Called `render`

The pattern is about the **function**, not the prop name.

All of these are possible:

```jsx
<Component render={(data) => ...} />
```

```jsx
<Component children={(data) => ...} />
```

```jsx
<Component content={(data) => ...} />
```

For example, using `children`:

```jsx
function MouseTracker({ children }) {
  const position = { x: 100, y: 200 };

  return children(position);
}
```

Usage:

```jsx
<MouseTracker>
  {(position) => <p>{position.x}, {position.y}</p>}
</MouseTracker>
```

This is often called a **function-as-children** pattern, and it is effectively a render prop.

---

## 5. Render Props vs Normal Props

Normal prop:

```jsx
<Button label="Save" />
```

The prop provides **data/configuration**.

Render prop:

```jsx
<DataProvider
  render={(data) => <UserCard user={data} />}
/>
```

The prop provides a **function that controls rendering**.

### Interview distinction

```text
Normal prop
→ "Here is some data/configuration."

Render prop
→ "Here is a function that tells me what UI to render."
```

---

## 6. Render Props vs Composition

### Composition

```jsx
<Card>
  <Header />
  <Body />
</Card>
```

The consumer supplies React elements.

### Render prop

```jsx
<DataProvider
  render={(data) => <UserCard data={data} />}
/>
```

The consumer supplies a function that receives data/state and produces UI.

So:

```text
Composition
→ supply UI structure

Render prop
→ supply UI as a function of state/data
```

---

## 7. Render Props vs Compound Components

### Compound Components

Used when several components work together:

```jsx
<Tabs>
  <Tabs.List />
  <Tabs.Panel />
</Tabs>
```

The parent coordinates shared behavior/state.

### Render Props

Used when a component exposes behavior/state to a consumer:

```jsx
<MouseTracker
  render={(position) => <Cursor position={position} />}
/>
```

A simple mental model:

```text
Compound Components
→ flexible component structure

Render Props
→ flexible rendering based on exposed state
```

---

## 8. Render Props vs HOCs

These are both older React patterns for **logic reuse**.

### HOC

```jsx
const EnhancedComponent = withMouseTracking(Component);
```

The HOC wraps a component and injects behavior/data.

### Render Prop

```jsx
<MouseTracker
  render={(position) => <Component position={position} />}
/>
```

The consumer explicitly decides how to use the provided data.

### Main difference

```text
HOC
→ wraps your component

Render prop
→ calls your function
```

---

## 9. Common Historical Use Cases

Render props were commonly used for:

- mouse tracking
- data fetching
- subscriptions
- animation
- forms
- drag and drop
- reusable stateful behavior

Example:

```jsx
<MouseTracker>
  {({ x, y }) => (
    <div>
      Mouse is at {x}, {y}
    </div>
  )}
</MouseTracker>
```

The same tracking logic could power completely different UIs.

---

## 10. The Main Drawback

Render props can lead to deeply nested functions.

```jsx
<A>
  {(a) => (
    <B>
      {(b) => (
        <C>
          {(c) => (
            <UI a={a} b={b} c={c} />
          )}
        </C>
      )}
    </B>
  )}
</A>
```

This can become difficult to read.

Another consideration is function identity: creating new inline functions during rendering can matter in certain performance-sensitive situations, particularly when passed to memoized children.

Don't overstate this point, though. **Using an inline render function is not automatically a performance problem.**

---

## 11. Modern React: Custom Hooks Often Replace Render Props

Today, many render-prop use cases are better expressed with **custom hooks**.

Render prop:

```jsx
<MouseTracker>
  {({ x, y }) => <Cursor x={x} y={y} />}
</MouseTracker>
```

Custom hook:

```jsx
function Cursor() {
  const { x, y } = useMousePosition();

  return <div>{x}, {y}</div>;
}
```

The hook separates the reusable logic from the UI without requiring an extra wrapper component or nested render function.

### Important

Render props are **not obsolete**.

They are still useful when:

- the API naturally revolves around rendering
- you need to expose behavior directly through a component
- you are working with an existing library/API that uses the pattern

But for many new React designs, **custom hooks are simpler**.

---

## 12. Interview Example

If asked:

> "What is a render prop?"

A strong answer:

> **A render prop is a React pattern where a component accepts a function as a prop and calls it with its internal state or behavior. The component owns the reusable logic, while the consumer controls the resulting UI. Historically it was commonly used for things like mouse tracking and data fetching, although custom hooks often provide a simpler approach in modern React.**

---

## 13. Quick Comparison

| Pattern | Main Idea |
|---|---|
| Composition | Combine components to build UI |
| Compound Components | Related components coordinate around shared state |
| Render Props | Function receives state/behavior and returns UI |
| HOC | Wrapper component adds reusable behavior |
| Custom Hook | Reuse stateful logic directly |

---

## 14. Key Takeaways

1. **Render prop = function prop used to determine UI.**
2. The reusable component owns **logic/state**.
3. The consumer owns **rendering**.
4. The prop can be called `render`, `children`, or something else.
5. `children` as a function is effectively a render prop.
6. Render props were historically important for reusable stateful behavior.
7. They can become verbose when heavily nested.
8. Custom hooks often provide a cleaner modern alternative.

### One-line memory trick

> **Render Props = "You give me a function, I'll give it my state, and you decide what I render."**
