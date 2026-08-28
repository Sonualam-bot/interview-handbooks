# Chapter 40 — `useReducer`

## 1. Core Mental Model

`useReducer` is an alternative to `useState` for managing state through a **reducer function**.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

Mental model:

```text
User interaction
      ↓
dispatch(action)
      ↓
reducer(state, action)
      ↓
new state
      ↓
React re-renders
```

> **`useReducer` manages state by describing actions and centralizing how those actions transform state.**

---

## 2. Basic Example

```jsx
function reducer(state, action) {
  if (action.type === "increment") {
    return { count: state.count + 1 };
  }

  return state;
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0
  });

  return (
    <button onClick={() => dispatch({ type: "increment" })}>
      {state.count}
    </button>
  );
}
```

Flow:

```text
dispatch({ type: "increment" })
        ↓
reducer(state, action)
        ↓
{ count: 1 }
        ↓
React renders
```

---

## 3. What Is a Reducer?

A reducer is a function:

```jsx
function reducer(state, action) {
  // calculate next state
}
```

Conceptually:

```js
nextState = reducer(currentState, action);
```

A reducer should be **pure**:

```text
same state + same action
        ↓
same result
```

Do not perform side effects inside it.

---

## 4. What Is an Action?

An action describes **what happened**.

Simple:

```jsx
dispatch({
  type: "increment"
});
```

With data:

```jsx
dispatch({
  type: "setName",
  payload: "Sonu"
});
```

Think:

> **Action = event description**

The action usually describes what happened rather than directly assigning the final state.

---

## 5. `dispatch` vs State Setter

With `useState`:

```jsx
setCount(count + 1);
```

You directly request the new state.

With `useReducer`:

```jsx
dispatch({
  type: "increment"
});
```

You describe the event and let the reducer determine the next state.

Mental model:

```text
useState
→ directly update state

useReducer
→ dispatch what happened
→ reducer determines next state
```

---

## 6. Why Use `useReducer`?

`useState` is excellent for simple state:

```jsx
const [count, setCount] = useState(0);
```

But complex state can have many related transitions:

```text
loading
data
error
selected item
pagination
filters
```

Instead of spreading transition logic across many setters, a reducer can centralize it:

```text
Action
 ↓
Reducer
 ↓
State transition
```

---

## 7. Example — Request State

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "FETCH_START":
      return {
        ...state,
        loading: true,
        error: null
      };

    case "FETCH_SUCCESS":
      return {
        ...state,
        loading: false,
        data: action.payload
      };

    case "FETCH_ERROR":
      return {
        ...state,
        loading: false,
        error: action.payload
      };

    default:
      return state;
  }
}
```

State transitions become explicit:

```text
FETCH_START
     ↓
loading = true

FETCH_SUCCESS
     ↓
loading = false
data = result

FETCH_ERROR
     ↓
loading = false
error = error
```

---

## 8. Reducers Must Be Pure

Bad:

```jsx
function reducer(state, action) {
  fetch("/api/users");

  return state;
}
```

Side effects don't belong in reducers.

Also avoid mutating state:

```jsx
function reducer(state, action) {
  state.count++;
  return state;
}
```

Prefer:

```jsx
function reducer(state, action) {
  return {
    ...state,
    count: state.count + 1
  };
}
```

Mental model:

```text
Reducer
→ calculate next state

Effect
→ synchronize with external systems
```

---

## 9. Reducer State Should Be Treated Immutably

Don't:

```jsx
state.count++;
return state;
```

Prefer:

```jsx
return {
  ...state,
  count: state.count + 1
};
```

The new state object represents the new state transition and avoids mutating the previous state.

---

## 10. `useReducer` vs `useState`

| | `useState` | `useReducer` |
|---|---|---|
| Simple state | Excellent | Works |
| Complex transitions | Can become messy | Excellent |
| State logic centralized | Usually less centralized | Yes |
| Updates | Setter | `dispatch(action)` |
| Transition logic | Often in handlers | Reducer |
| Related state transitions | Sometimes | Strong use case |

Don't think:

> "`useReducer` is always better."

Choose based on the complexity of the state transitions.

---

## 11. When Should You Use `useReducer`?

Good candidates:

### Complex state

```text
loading
data
error
filters
pagination
```

### Many related transitions

```text
FETCH_START
FETCH_SUCCESS
FETCH_ERROR
RESET
```

### Transitions depend heavily on previous state

### You want state transition logic centralized

For simple state:

```jsx
const [count, setCount] = useState(0);
```

is often easier.

---

## 12. `dispatch` Doesn't Immediately Mutate State

Conceptually:

```jsx
dispatch(action);
```

requests a state update.

It does not synchronously mutate the current render's `state` variable.

Mental model:

```text
dispatch
 ↓
schedule update
 ↓
render
 ↓
reducer calculates next state
 ↓
commit
```

---

## 13. State Snapshot Still Applies

Suppose:

```jsx
dispatch({ type: "increment" });
console.log(state.count);
```

The current render's `state` still represents its existing snapshot.

Example:

```text
Current render
state.count = 0

dispatch(increment)
      ↓
current render still sees 0

Next render
state.count = 1
```

This is the same snapshot concept learned with `useState`.

---

## 14. Multiple Dispatches

You can dispatch multiple actions:

```jsx
dispatch({ type: "increment" });
dispatch({ type: "increment" });
```

React can batch state updates.

The important idea is:

```text
dispatch
→ request state transition

reducer
→ determines next state
```

The reducer should not depend on mutable external state to make transitions work correctly.

---

## 15. Action Design

Good action:

```jsx
dispatch({
  type: "ADD_TODO",
  payload: {
    id: 1,
    text: "Study React"
  }
});
```

The action describes what happened.

Reducer:

```jsx
case "ADD_TODO":
  return {
    ...state,
    todos: [...state.todos, action.payload]
  };
```

Mental model:

```text
Action
→ event

Reducer
→ state transition
```

---

## 16. Interview Questions

### Q1. What is `useReducer`?

> `useReducer` is a state management hook where state transitions are handled by a reducer function based on dispatched actions.

### Q2. When would you use it instead of `useState`?

> When state transitions are complex, multiple pieces of state are related, or centralizing transition logic makes the component easier to reason about.

### Q3. What is a reducer?

> A pure function that takes the current state and an action and returns the next state.

### Q4. What is an action?

> An object describing an event or state transition request, usually containing a `type` and optionally a payload.

### Q5. Can reducers perform side effects?

> No. Reducers should be pure.

### Q6. Does `dispatch` immediately mutate the current state?

> No. It schedules a state update; the current render continues to see its existing state snapshot.

---

# Quick Revision

```text
useReducer
→ state + dispatch
```

```text
dispatch(action)
→ request state transition
```

```text
reducer(state, action)
→ returns next state
```

```text
Action
→ describes what happened
```

```text
Reducer
→ pure state transition logic
```

```text
Reducer
≠
side effects
```

```text
Complex related state
→ strong useReducer candidate
```

```text
Simple state
→ useState is often simpler
```

---

# Final Mental Model

```text
User interaction
      ↓
dispatch(action)
      ↓
React schedules update
      ↓
reducer(currentState, action)
      ↓
new state
      ↓
render
      ↓
commit
```

> **`useState` is often about directly updating a value. `useReducer` is about describing events and centralizing how those events transform state.**

**Chapter 40 — COMPLETE**
