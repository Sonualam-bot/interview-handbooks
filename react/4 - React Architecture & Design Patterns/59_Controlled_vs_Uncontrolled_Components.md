# Handbook 4 — React Architecture & Design Patterns

# Chapter 59 — Controlled vs Uncontrolled Components

## 1. Core Idea

The fundamental question is:

> **Who owns the state?**

There are two major approaches:

- **Controlled:** React/application state is the source of truth.
- **Uncontrolled:** The DOM or component's internal state is the source of truth.

The important distinction is **state ownership**, not simply whether a ref or `useState` is used.

---

## Deep Dive `[NEW]`

### Why Switching Modes Specifically Triggers a Warning — The Actual Check

Section 12 says the ownership model should stay stable, but the
underlying reason is mechanical, not stylistic. React's DOM renderer
decides whether a native `<input>` is controlled by checking, at the
point the element is first created, whether its `value` prop is
`undefined` or not — that single yes/no decision is what determines
whether React attaches its own value-syncing behavior to that DOM
node at all. This check effectively happens once, tied to that
input's mount — it's not re-evaluated fresh on every render the way a
prop comparison would be. So when `value` goes from `undefined`
(uncontrolled — React attached no syncing behavior) to an actual
string (controlled — behavior that was never attached), React can
detect the mismatch between "what this input's mode was set up as"
and "what you're now asking it to do," which is exactly the mismatch
the warning reports. It isn't a style linter opinion — it's React
telling you an internal decision it already made can't be silently
changed after the fact.

### Why `defaultValue={initial ?? ""}` Doesn't Fix This

A common but incomplete fix is defaulting the prop so it's "never
undefined":

```jsx
<input value={value ?? ""} onChange={onChange} />
```

This does prevent the specific warning, because `value` is now always
a defined string from the first render onward — the input is
controlled from mount, permanently. But it's a different fix than
"supporting both modes" (Section 11): it commits the component to
being controlled unconditionally. Section 11's actual dual-mode
technique has to detect intent *before* the first render (checking
whether a `value` prop was passed at all, not defaulting it after the
fact), precisely because the mode has to be settled once, correctly,
at mount — there's no supported way to decide provisionally and
correct course later.

## 2. Controlled Components

A component is controlled when React state is the source of truth.

```jsx
function Form() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={e => setName(e.target.value)}
    />
  );
}
```

Data flow:

```text
User types
    ↓
onChange
    ↓
setName()
    ↓
React state changes
    ↓
React renders
    ↓
value={name}
    ↓
Input displays new value
```

React knows the current value.

### Why controlled components are useful

They make it easy to:

- validate while typing
- enable/disable UI based on the value
- format or transform input
- derive other UI from the value
- synchronize multiple components
- apply business rules

Example:

```jsx
const canSubmit = email !== "" && password !== "";
```

---

## 3. Uncontrolled Components

An uncontrolled component does not receive its current value from React state. The value is maintained by the DOM or by the component's own internal state.

For a DOM input:

```jsx
function Form() {
  const inputRef = useRef(null);

  function handleSubmit() {
    console.log(inputRef.current.value);
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>
        Submit
      </button>
    </>
  );
}
```

The DOM owns the current input value.

```text
User types
    ↓
DOM updates its own value
    ↓
React does not need to store every keystroke
    ↓
React can read the value through the ref when needed
```

### Important nuance

Do not define uncontrolled components as simply:

> "Components that use refs."

Refs are a common way to access **uncontrolled DOM state**, but a reusable React component can also be uncontrolled while maintaining its own internal React state.

For example, an uncontrolled `Toggle` can use:

```jsx
const [internalValue, setInternalValue] = useState(defaultValue);
```

The important point is that the **consumer/parent does not own the current value**.

---

## 4. `value` vs `defaultValue`

This distinction is frequently tested in interviews.

### `value`

```jsx
<input value={name} />
```

React controls the current value.

### `defaultValue`

```jsx
<input defaultValue="Sonu" />
```

`defaultValue` supplies the **initial value**. After that, the DOM manages the current value.

Mental model:

```text
value
→ current value controlled by React

defaultValue
→ initial value supplied to the DOM
```

---

## 5. Controlled vs Uncontrolled

| | Controlled | Uncontrolled |
|---|---|---|
| Source of truth | React/application state | DOM or component internal state |
| Current value | React knows it | DOM/component maintains it |
| Common API | `value` + `onChange` | `defaultValue` + `ref` for DOM inputs |
| Real-time validation | Easy | More manual |
| Dynamic UI | Easy | More manual |
| Code | More state/handlers | Often simpler |
| Direct DOM interaction | Less central | More central |
| Good for | Interactive, data-driven forms | Simple forms / DOM-driven cases |

---

## 6. Controlled Component Data Flow

For a reusable controlled component:

```jsx
<Toggle
  value={isOn}
  onChange={setIsOn}
/>
```

The parent owns the state.

Flow:

```text
Parent state
    ↓
value prop
    ↓
Toggle
    ↓
User clicks
    ↓
Toggle calculates next value
    ↓
onChange(nextValue)
    ↓
Parent setIsOn(nextValue)
    ↓
new value prop
    ↓
Toggle re-renders
```

The child does not own the source of truth.

---

## 7. Uncontrolled Component Data Flow

For an uncontrolled React component:

```text
User interaction
    ↓
Component's internal state
    ↓
setInternalValue(...)
    ↓
Component re-renders
```

For an uncontrolled DOM input:

```text
User interaction
    ↓
DOM
    ↓
DOM maintains value
    ↓
React reads it through ref when needed
```

These are related but should not be confused.

---

## 8. Why Refs Work for Uncontrolled DOM Inputs

```jsx
const inputRef = useRef(null);

<input ref={inputRef} />
```

React gives access to the DOM node through:

```jsx
inputRef.current
```

Therefore:

```jsx
inputRef.current.value
```

reads the value currently held by the DOM.

You are asking the DOM:

> "What value are you currently holding?"

rather than maintaining that value in React state.

---

## 9. Controlled and Uncontrolled Reusable Components

A reusable component can support both modes.

### Controlled API

```jsx
<Toggle
  value={isOn}
  onChange={setIsOn}
/>
```

Parent owns the state.

### Uncontrolled API

```jsx
<Toggle
  defaultValue={false}
/>
```

The component owns the state.

This pattern is common in reusable component libraries.

---

## 10. Detecting the Mode

A common conceptual rule is:

```jsx
const isControlled = value !== undefined;
```

Therefore:

```text
value provided
    ↓
Controlled

value not provided
    ↓
Uncontrolled
```

Do not rely only on the presence of `defaultValue`.

`defaultValue` represents an **initial value**, not ownership of the current value.

---

## 11. Supporting Both Modes

A reusable component can conceptually look like:

```jsx
function Toggle({
  value,
  defaultValue = false,
  onChange
}) {
  const isControlled = value !== undefined;

  const [internalValue, setInternalValue] =
    useState(defaultValue);

  const currentValue =
    isControlled ? value : internalValue;

  function handleToggle() {
    const nextValue = !currentValue;

    if (!isControlled) {
      setInternalValue(nextValue);
    }

    onChange?.(nextValue);
  }

  return (
    <button onClick={handleToggle}>
      {currentValue ? "ON" : "OFF"}
    </button>
  );
}
```

Mental model:

```text
Controlled
Parent state
    ↓
value prop
    ↓
currentValue

Uncontrolled
internalValue
    ↓
currentValue
```

The component can notify the parent through `onChange` in either mode.

---

## 12. Do Not Switch Between Modes

A component should generally remain controlled or uncontrolled for its lifetime.

Bad scenario:

```text
First render:
value === undefined
    ↓
Uncontrolled

Later:
value === true
    ↓
Controlled
```

The ownership model has changed during the component's lifetime.

The opposite is also problematic:

```text
Controlled
    ↓
Uncontrolled
```

### Mental model

At mount, ask:

```text
Who owns my state?

Parent?
    → controlled

Me?
    → uncontrolled
```

Once chosen, keep that ownership model stable.

This avoids ambiguous state ownership and React's controlled/uncontrolled warnings.

---

## 13. Controlled vs State Reducer Pattern

These patterns answer different questions.

### Controlled

> **Who owns the state?**

```text
Parent owns state
```

Example:

```jsx
<Toggle
  value={isOn}
  onChange={setIsOn}
/>
```

### State Reducer

> **Who controls/customizes the state transition?**

```text
Component owns state
Consumer customizes transitions
```

Example:

```jsx
<Toggle
  stateReducer={customReducer}
/>
```

Memory trick:

```text
Controlled
→ Who owns the state?

State Reducer
→ Who controls the state transition logic?
```

---

## 14. When to Prefer Controlled

Controlled components are especially useful when:

- validation happens while typing
- UI depends on the current value
- multiple components need the value
- formatting/transformation is required
- React needs to apply business rules
- the current value is part of application state

Example:

```text
Search input
    ↓
query state
    ↓
API request
    ↓
results
```

---

## 15. When Uncontrolled Can Be Useful

Uncontrolled components can be useful when:

- the form is simple
- values are only needed at submission
- real-time React state is unnecessary
- direct DOM interaction is convenient
- you want to avoid React state for every keystroke

Example:

```text
Simple form
    ↓
User fills fields
    ↓
Submit
    ↓
Read values
```

Do not interpret this as:

> "Uncontrolled is always faster."

Performance depends on what rendering and other work happens when state changes.

---

## 16. Interview Trap

Weak answer:

> "Controlled components use state, while uncontrolled components use refs."

Better answer:

> **Controlled and uncontrolled components differ primarily in who owns the source of truth. In a controlled component, React/application state owns the current value and passes it through props. In an uncontrolled component, the DOM or component's internal state maintains the value, with refs commonly used to read uncontrolled DOM values when needed.**

---

# 🧪 Exercise — Controlled vs Uncontrolled Name Input

Build the same component in two ways.

## Part A — Controlled

Create:

```jsx
function ControlledInput() {
  // ...
}
```

Requirements:

- React state stores the name
- input uses `value`
- input uses `onChange`
- display the current name below the input

Expected flow:

```text
User types
    ↓
onChange
    ↓
setName()
    ↓
React state
    ↓
re-render
    ↓
value + displayed text update
```

Check yourself:

> Where is the current name stored?

**React state.**

---

## Part B — Uncontrolled

Create:

```jsx
function UncontrolledInput() {
  // ...
}
```

Use:

```jsx
const inputRef = useRef(null);
```

and:

```jsx
<input ref={inputRef} />
```

Add:

```jsx
<button onClick={handleSubmit}>
  Read Name
</button>
```

Then:

```jsx
console.log(inputRef.current.value);
```

Check yourself:

> Where is the current name stored while the user types?

**The DOM input.**

---

## Part C — Trace Both Data Flows

### Controlled

```text
User types
    ↓
DOM event
    ↓
onChange
    ↓
setName
    ↓
React state
    ↓
React render
    ↓
value prop
    ↓
DOM
```

### Uncontrolled DOM input

```text
User types
    ↓
DOM
    ↓
DOM updates its own value
    ↓
React does not need to update state
    ↓
Later: ref.current.value
    ↓
React reads DOM value
```

---

## Part D — API Design Challenge

Design:

```jsx
<Toggle />
```

that supports:

### Controlled

```jsx
<Toggle
  value={isOn}
  onChange={setIsOn}
/>
```

### Uncontrolled

```jsx
<Toggle
  defaultValue={false}
/>
```

Think through:

1. How do you determine which mode is being used?
2. Where does the state live in each mode?
3. What happens when the user toggles?
4. How do you prevent switching between modes?

### Expected conceptual answer

```text
value provided
    ↓
controlled
    ↓
parent owns state

value absent
    ↓
uncontrolled
    ↓
component owns internal state
```

And:

```text
Choose ownership model
        ↓
Keep it stable
        ↓
Do not switch controlled ↔ uncontrolled
```

---

# 🎯 Interview Checklist

### 🔥 Must Know

- [ ] Definition of controlled component
- [ ] Definition of uncontrolled component
- [ ] **Source of truth / state ownership**
- [ ] `value` vs `defaultValue`
- [ ] `onChange` + React state
- [ ] `ref` + DOM value
- [ ] Controlled data flow
- [ ] Uncontrolled data flow
- [ ] When to use each
- [ ] Controlled/uncontrolled reusable component APIs
- [ ] Why switching modes is problematic
- [ ] Controlled vs State Reducer

### Lower Priority

- Micro-optimizing controlled inputs
- Complex form-library internals
- Historical details

---

# 🧠 Final Mental Model

```text
                    WHO OWNS STATE?
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
          Parent                 Component/DOM
             ↓                       ↓
       CONTROLLED               UNCONTROLLED
```

The one-line interview memory trick:

> **Controlled = React/application state owns the current value. Uncontrolled = the DOM or component internally owns the current value.**
