# Chapter 45 — `forwardRef`

## 1. Core Mental Model

`forwardRef` allows a component to receive a ref from its parent and forward that ref to a DOM node or another ref-supporting component.

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  return <input ref={ref} />;
});
```

Usage:

```jsx
function Form() {
  const inputRef = useRef(null);

  return <MyInput ref={inputRef} />;
}
```

Mental model:

```text
Parent
  ↓
ref
  ↓
MyInput
  ↓
<input>
```

> **`forwardRef` forwards a ref through a component boundary.**

---

## 2. The Problem It Solves

Suppose:

```jsx
function MyInput() {
  return <input />;
}
```

Historically, this does not allow a parent to directly pass a ref through the function component:

```jsx
<MyInput ref={inputRef} />
```

`forwardRef` provides the component with the ref:

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input ref={ref} />;
});
```

---

## 3. What Does `forwardRef` Actually Do?

It allows a component to receive a ref from its parent and forward it to a child DOM node or another component that supports refs.

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input ref={ref} />;
});
```

The second argument is the forwarded ref:

```jsx
(props, ref)
```

---

## 4. Why Is `ref` Special?

Historically, React treated `ref` specially.

This:

```jsx
<MyInput ref={inputRef} />
```

was not equivalent to:

```jsx
<MyInput inputRef={inputRef} />
```

With the traditional `forwardRef` model, the component receives the ref through the second argument:

```jsx
forwardRef((props, ref) => {
  // ...
});
```

---

## 5. Typical Use Case

Reusable input component:

```jsx
const TextInput = forwardRef(function TextInput(
  { label },
  ref
) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
});
```

Parent:

```jsx
function Form() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <>
      <TextInput
        ref={inputRef}
        label="Name"
      />

      <button onClick={focusInput}>
        Focus
      </button>
    </>
  );
}
```

Flow:

```text
button click
   ↓
inputRef.current.focus()
   ↓
actual <input>
   ↓
focus
```

---

## 6. `forwardRef` Does NOT Create the Ref

The parent creates the ref:

```jsx
const inputRef = useRef(null);
```

`forwardRef` allows that ref to travel through the component.

Mental model:

```text
useRef
→ creates/holds ref

forwardRef
→ forwards ref through component boundary
```

---

## 7. `forwardRef` Can Forward to Another Component

The target doesn't have to be a DOM node:

```jsx
const InputWrapper = forwardRef((props, ref) => {
  return <AnotherInput ref={ref} />;
});
```

The ref can continue through the hierarchy if the receiving component supports refs.

---

## 8. `forwardRef` + `React.memo`

You may encounter:

```jsx
const MyInput = React.memo(
  forwardRef(function MyInput(props, ref) {
    return <input ref={ref} />;
  })
);
```

These solve different problems:

```text
React.memo
→ memoizes rendering based on props

forwardRef
→ allows ref forwarding
```

---

## 9. `forwardRef` vs `useImperativeHandle`

Don't confuse them.

### `forwardRef`

Answers:

> **How does the ref get from the parent to the child?**

```text
Parent
 ↓
forwardRef
 ↓
Child DOM node
```

### `useImperativeHandle`

Answers:

> **What should the parent be allowed to access through that ref?**

For example:

```text
focus()
clear()
scrollToTop()
```

Mental model:

```text
forwardRef
→ forwards the ref

useImperativeHandle
→ controls what the ref exposes
```

---

## 10. React 19 Note

React 19 changed the ref model: function components can receive `ref` as a prop, reducing the need for `forwardRef` in new code.

For interviews:

> **Know what `forwardRef` does and why it was historically needed, while being aware of the React 19 ref model.**

You will still encounter `forwardRef` in existing React codebases.

---

## 11. When Should You Use It?

Typical situations:

- reusable input components
- custom form controls
- focus management
- scrolling
- measuring DOM elements
- exposing a DOM node through a component abstraction

Example:

```text
Parent
 ↓
Reusable Input component
 ↓
actual <input>
```

The parent may need to focus or measure that input.

---

## 12. When Shouldn't You Use It?

Don't use refs as the normal way to communicate data between components.

Prefer:

```text
props
state
callbacks
context
```

for declarative communication.

Refs are generally useful for:

```text
DOM access
imperative operations
```

---

## 13. Interview Questions

### Q1. What is `forwardRef`?

> `forwardRef` allows a component to receive a ref from its parent and forward it to a DOM node or another ref-supporting component.

### Q2. Why is it useful?

> It allows parent components to access a DOM element hidden behind a reusable component abstraction.

### Q3. Does `forwardRef` create the ref?

> No. The parent usually creates the ref with `useRef`; `forwardRef` forwards it through the component.

### Q4. `forwardRef` vs `useImperativeHandle`?

> `forwardRef` forwards the ref; `useImperativeHandle` controls what the ref exposes.

### Q5. Can refs be used for normal data communication?

> They can, but they should not be the default. Props and state are preferred for declarative communication.

### Q6. Is `forwardRef` still required in React 19?

> Its necessity is reduced because React 19 allows function components to receive `ref` as a prop, but `forwardRef` remains important for understanding existing code and older React patterns.

---

# Quick Revision

```text
forwardRef
→ forwards ref through component
```

```text
useRef
→ creates/holds ref
```

```text
forwardRef
≠
creates ref
```

```text
Parent
 ↓
ref
 ↓
forwardRef component
 ↓
DOM node
```

```text
forwardRef
→ how ref gets there
```

```text
useImperativeHandle
→ what ref exposes
```

```text
Props/state
→ normal declarative communication
```

```text
Refs
→ DOM / imperative operations
```

---

# Final Mental Model

```text
                 Parent
                   │
             inputRef.current
                   │
                   ↓
             forwardRef
                   │
                   ↓
               <input>
```

### Interview line:

> **`forwardRef` lets a parent pass a ref through a component so the ref can ultimately point to a DOM node or another ref-supporting component.**

**Chapter 45 — COMPLETE**
