# Chapter 56 — State Reducer Pattern

**Handbook 4 — React Architecture & Design Patterns**

## 1. What is the State Reducer Pattern?

The **State Reducer Pattern** lets a component manage its own state while allowing the consumer to customize how state transitions happen.

> **The component owns the default behavior; the consumer gets an escape hatch to modify state transitions.**

```text
Component default state logic
            ↓
       stateReducer
            ↓
        final state
```

## Deep Dive `[NEW]`

### Why This Pattern Requires `useReducer`, Not `useState`

This is the precondition that makes the entire pattern possible, and
it's easy to take for granted: the escape hatch in Section 4 works
because the *action* — a plain object describing what happened
(`{ type: "toggle" }`) — already exists as a discrete value inside
`Toggle` before any state is computed. If `Toggle` were built with
`useState` and an imperative setter (`setOn(!on)`), there would be no
intermediate "action" object for a consumer reducer to intercept —
just a new state value materializing directly. You cannot bolt a state
reducer escape hatch onto a `useState`-based component without first
restructuring it to describe *what happened* as data (see Handbook 3,
`useReducer`, on why describing events instead of directly computing
next state is a different, more inspectable shape of state
management). The State Reducer pattern is really "expose the
already-existing action/reducer seam that `useReducer` gives you for
free" — it doesn't work by accident that every state reducer example
uses `useReducer` internally.

### Why the Consumer's Function Receives the Action, Not Just a State Diff

`stateReducer(changes, action)` hands the consumer both the computed
result *and* the original action — not just "here's what changed."
This matters because a consumer often needs to make a decision that
depends on *why* the state changed, not just what it changed to:
`preventTurningOff` in Section 5 checks `action.type === "toggle"`
specifically so it only intervenes for that one action, leaving any
other action's default behavior untouched. Without access to the
action, a consumer reducer could only pattern-match on the resulting
state shape — which quickly becomes ambiguous or impossible once two
different actions could plausibly produce the same-looking state
change. Exposing the action preserves the *intent* behind a state
transition, not just its result, which is exactly the same
information asymmetry that motivates action objects in `useReducer`
and Redux generally: the state shows you where you ended up, the
action tells you why.

## 2. Why Do We Need It?

Imagine a reusable Toggle:

```text
Normal:
OFF → ON → OFF → ON

Custom requirement:
OFF → ON → ON → ON
```

Instead of rewriting the component or adding many special-purpose props, expose a reducer that can customize the transition.

## 3. Basic Reducer

```jsx
function toggleReducer(state, action) {
  switch (action.type) {
    case "toggle":
      return { ...state, on: !state.on };

    default:
      return state;
  }
}
```

With:

```jsx
const [state, dispatch] = useReducer(toggleReducer, {
  on: false
});
```

A reducer follows:

```text
current state + action → next state
```

## 4. State Reducer Pattern

The component first calculates its normal state transition and then gives the result to the consumer's reducer.

```jsx
function Toggle({ stateReducer }) {
  const [state, dispatch] = useReducer(
    (state, action) => {
      const changes = toggleReducer(state, action);

      return stateReducer
        ? stateReducer(changes, action)
        : changes;
    },
    { on: false }
  );

  return (
    <button onClick={() => dispatch({ type: "toggle" })}>
      {state.on ? "ON" : "OFF"}
    </button>
  );
}
```

Data flow:

```text
current state
      +
   action
      ↓
default reducer
      ↓
default changes
      ↓
custom stateReducer
      ↓
final state
      ↓
React renders UI
```

## 5. Customizing Behavior

```jsx
function preventTurningOff(state, action) {
  if (action.type === "toggle" && state.on === false) {
    return {
      ...state,
      on: true
    };
  }

  return state;
}
```

Usage:

```jsx
<Toggle stateReducer={preventTurningOff} />
```

The consumer can change the behavior without rewriting `Toggle`.

## 6. The Important Concept

The component owns:

```text
State
Actions
Default reducer
```

The consumer gets:

```text
A hook into state transitions
```

This keeps the component reusable while allowing advanced customization.

## 7. State Reducer vs Controlled Component

### Controlled Component

The consumer owns the actual state:

```jsx
<Toggle
  on={on}
  onChange={setOn}
/>
```

### State Reducer

The component still owns the state:

```jsx
<Toggle
  stateReducer={customReducer}
/>
```

The consumer customizes how transitions happen.

### 🔥 Interview distinction

```text
Controlled
→ Consumer owns state

State Reducer
→ Component owns state
→ Consumer customizes state transitions
```

## 8. Why Is This Powerful?

Complex reusable components may have many transitions:

```text
Dropdown
 ├── open
 ├── close
 ├── select
 ├── highlight
 └── reset
```

Different applications may need slightly different behavior.

Without a general customization mechanism, you may end up with prop explosion:

```jsx
<Component
  preventClose
  keepOpen
  disableSelection
  customSelection
  closeAfterSelect
/>
```

A state reducer provides a general customization escape hatch:

```jsx
<Dropdown stateReducer={customReducer} />
```

## 9. Escape Hatch

A reusable component can provide:

```text
Good default behavior
        +
Customization escape hatch
```

The consumer can effectively say:

> "I like your component, but I need one specific state transition to behave differently."

## 10. Reducers Should Stay Pure

A reducer should generally be a pure function:

```jsx
function reducer(state, action) {
  return newState;
}
```

Avoid side effects inside reducers:

```jsx
function reducer(state, action) {
  localStorage.setItem(...);
  fetch(...);

  return state;
}
```

The reducer's responsibility is:

```text
state + action → next state
```

## 11. Common Use Cases

Useful for reusable components with complex state transitions:

- Dropdowns
- Selects
- Comboboxes
- Autocomplete
- Modals
- Menus
- Tabs
- Accordions
- Form components

## 12. State Reducer vs Custom Hook

A custom hook primarily provides:

> **Reusable stateful logic.**

A state reducer provides:

> **Customizable state transitions.**

They can be combined:

```jsx
function useToggle({ reducer = toggleReducer }) {
  const [state, dispatch] = useReducer(reducer, {
    on: false
  });

  return { state, dispatch };
}
```

## 13. State Reducer vs Render Props

```text
Render Props
→ customize rendering

State Reducer
→ customize state transitions
```

Render prop:

```jsx
<DataProvider>
  {(data) => <UI data={data} />}
</DataProvider>
```

State reducer:

```jsx
<Toggle stateReducer={customReducer} />
```

## 14. State Reducer vs HOC

```text
HOC
→ enhance/wrap a component

State Reducer
→ customize internal state transitions
```

HOC:

```jsx
const Enhanced = withAuth(Component);
```

State reducer:

```jsx
<Component stateReducer={customReducer} />
```

## 15. Interview Answer

> **A State Reducer Pattern allows a reusable component to own its internal state while giving consumers a reducer hook that can customize how state transitions occur. The component provides the default behavior, and the consumer can modify specific transitions without rewriting the component. It is particularly useful for complex reusable components such as dropdowns, selects, and autocomplete components.**

## 16. Core Data Flow

Memorize:

```text
User interaction
      ↓
     Action
      ↓
Default reducer
      ↓
State changes
      ↓
Custom stateReducer
      ↓
Final state
      ↓
React renders UI
```

More precisely:

```text
                 ┌──────────────────┐
                 │  Default Reducer │
                 └────────┬─────────┘
                          ↓
State + Action ─────→ Default State
                          ↓
                   Custom Reducer
                          ↓
                     Final State
                          ↓
                       Render
```

# 🧪 Exercise — Build a Customizable Toggle

## Goal

Implement:

```jsx
<Toggle stateReducer={customReducer} />
```

Default behavior:

```text
OFF → ON → OFF → ON
```

Custom behavior:

```text
OFF → ON → ON → ON
```

Once ON, it must never turn OFF.

## Step 1 — Build the default reducer

Create:

```jsx
function toggleReducer(state, action) {
  // implement this
}
```

Handle:

```js
{ type: "toggle" }
```

Expected:

```text
false → true
true → false
```

## Step 2 — Build the Toggle component

Use `useReducer()`.

The component should:

- keep `on` in internal state
- dispatch `{ type: "toggle" }` on click
- render `ON` or `OFF`

Do not add custom behavior yet.

## Step 3 — Add `stateReducer`

Change the API to:

```jsx
<Toggle stateReducer={customReducer} />
```

The component should:

1. Run the default reducer.
2. Receive the resulting state.
3. Pass that state and action to `stateReducer`.
4. Use the returned value as the final state.

## Step 4 — Create the custom reducer

Create:

```jsx
function preventTurningOff(state, action) {
  // implement this
}
```

Requirement:

```text
OFF + toggle → ON
ON + toggle  → ON
```

For actions it does not customize:

```jsx
return state;
```

## Step 5 — Trace the data flow

For:

```text
Initial state = OFF
User clicks
```

Explain:

```text
1. onClick fires
2. dispatch({ type: "toggle" })
3. default toggleReducer runs
4. default state becomes ON
5. custom stateReducer receives ON + toggle
6. custom reducer decides the final state
7. React stores the final state
8. component renders ON
```

Click again:

```text
Current state = ON
User clicks
```

Trace:

```text
1. dispatch toggle
2. default reducer → OFF
3. custom reducer receives OFF
4. custom reducer changes it back to ON
5. final state = ON
6. React renders ON
```

### 🔥 The key question

> **At what point does the consumer's reducer get control over the state transition?**

Answer:

> **After the component's default reducer calculates the state change, the custom state reducer gets the opportunity to modify that result before it becomes the final state.**

## Step 6 — Extension Challenge

Add:

```text
reset
```

Action:

```js
{ type: "reset" }
```

Default behavior:

```text
ON → OFF
OFF → OFF
```

Then decide whether `preventTurningOff` should also block `reset`.

Implement your answer in the custom reducer.

This forces you to reason about **actions and state transitions**, rather than simply copying the pattern.

# 🎯 Interview Priority

### 🔥🔥🔥 Know this

- State Reducer Pattern definition
- Default reducer + custom reducer
- State transition data flow
- Escape hatch concept
- State Reducer vs Controlled Component
- Why it prevents prop explosion
- Real-world use cases
- Reducer purity

### Lower priority

- Complex reducer composition
- Advanced state-machine implementations
- Historical library-specific implementations

## One-line Memory Trick

> **State Reducer = "You manage the state; I give you a chance to change how the state transitions."**
