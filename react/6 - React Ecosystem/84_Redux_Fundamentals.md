# Chapter 84 — Redux Fundamentals

## 1. The Problem Redux Solves

Imagine an e-commerce application with a product page, navigation bar, mini-cart, and checkout page. All of them need to agree about the same cart.

If each component maintains its own copy of the cart, those copies can become inconsistent. This is a state ownership and consistency problem.

React offers several ways to share state:

- **Lift state up:** move state to a common parent and pass it down through props.
- **Context:** make a value available to a subtree without passing it through every intermediate component.
- **Redux:** maintain shared state in a store outside the component tree and define an explicit process for changing it.

Redux is not automatically better than local state or Context. It becomes useful when centralized ownership, explicit transitions, predictable updates, and shared access provide enough value to justify the additional architecture.

## 2. What Is Redux?

Redux is a state-management library built around a few core ideas:

1. Shared application state lives in a **store**.
2. An **action** describes an event or intended state change.
3. A **reducer** calculates the next state from the current state and an action.
4. Code sends actions to the store using **dispatch**.
5. The store holds the resulting state and notifies subscribers.

The general flow is:

```text
User interaction
      ↓
    Action
      ↓
   Reducer
      ↓
    Store
      ↓
 UI reads updated state
      ↓
 React renders relevant UI
```

This is called **unidirectional data flow**. The UI expresses an intent, the reducer calculates the transition, and the UI renders from the resulting state.

## 3. State

State is the data that describes the application at a particular moment.

```js
const initialState = {
  items: [
    { id: "p1", name: "Keyboard", price: 3000, quantity: 1 },
  ],
};
```

The state describes what is true now. It does not inherently describe the history of how it became true.

### Store authoritative data; derive what you can

Avoid independently storing values that can be calculated from other state:

```js
// Potentially redundant state
const state = {
  items: [{ id: "p1", price: 3000, quantity: 2 }],
  totalQuantity: 2,
  totalPrice: 6000,
};
```

Instead, store the items and derive totals:

```js
const totalQuantity = state.items.reduce(
  (sum, item) => sum + item.quantity,
  0
);

const totalPrice = state.items.reduce(
  (sum, item) => sum + item.price * item.quantity,
  0
);
```

Storing derived values separately creates a synchronization obligation. It can be justified for performance in some cases, but should be deliberate.

## 4. Actions

An action is a plain JavaScript object describing an event or intended state change.

```js
const action = {
  type: "cart/itemAdded",
  payload: {
    id: "p2",
    name: "Mouse",
    price: 1000,
  },
};
```

- `type` identifies the event.
- `payload` carries information needed to process it.

`payload` is a convention, not a JavaScript requirement. Some actions need no additional data.

Examples:

```js
{ type: "cart/itemRemoved", payload: { id: "p2" } }

{ type: "cart/quantityChanged", payload: { id: "p1", quantity: 3 } }

{ type: "cart/cleared" }
```

Meaningful event names such as `cart/itemAdded` communicate more than generic names such as `UPDATE_DATA`.

An action does not itself update state. Dispatching it asks the store to process it.

## 5. Reducers: State Transition Functions

A reducer receives the current state and an action, then returns the next state.

\[
S_{t+1} = R(S_t, A_t)
\]

Where:

- \(S_t\): current state
- \(A_t\): action
- \(R\): reducer
- \(S_{t+1}\): next state

Example:

```js
const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case "counter/increment":
      return { ...state, count: state.count + 1 };

    case "counter/decrement":
      return { ...state, count: state.count - 1 };

    case "counter/reset":
      return { ...state, count: 0 };

    default:
      return state;
  }
}
```

The `default` branch matters because a reducer can receive actions that it does not handle. It should return its current state for unknown actions.

A reducer is not a UI event handler. An event handler responds to browser interaction; a reducer calculates a state transition independently of a particular component.

## 6. Pure Functions and Determinism

A pure function:

1. Returns the same output for the same inputs.
2. Does not cause observable side effects outside the function.

Reducers should be predictable:

```js
function counterReducer(state, action) {
  if (action.type === "counter/increment") {
    return { count: state.count + 1 };
  }

  return state;
}
```

The same state and action always produce the same result.

Avoid these operations inside reducers:

- Network requests.
- Writing to local storage.
- Direct DOM updates.
- Starting timers.
- Depending on random values or current time to calculate state.

These are side effects. They belong outside reducers; their results can be represented by actions.

Predictable reducers are easier to test, debug, replay, and reason about.

## 7. Immutability

JavaScript objects are reference values. Assigning an object to another variable does not create a copy:

```js
const original = { count: 0 };
const alias = original;

alias.count = 1;

console.log(original.count); // 1
console.log(original === alias); // true
```

Both variables refer to the same object.

### The mutation bug

```js
function badReducer(state, action) {
  if (action.type === "counter/increment") {
    state.count += 1;
    return state;
  }

  return state;
}
```

This mutates the existing state object and returns the same reference.

That is problematic because Redux and React-Redux use reference equality in their change-detection mechanisms. Consumers may fail to detect a change when the reference is unchanged. It also means code holding the previous state reference can observe it changing unexpectedly.

### Immutable update

```js
function counterReducer(state, action) {
  if (action.type === "counter/increment") {
    return {
      ...state,
      count: state.count + 1,
    };
  }

  return state;
}
```

Now:

```js
const previousState = { count: 0 };

const nextState = counterReducer(previousState, {
  type: "counter/increment",
});

console.log(previousState.count); // 0
console.log(nextState.count);     // 1
console.log(previousState === nextState); // false
```

The previous state remains unchanged.

### Nested state and shallow copies

The spread operator makes a shallow copy:

```js
const state = {
  user: {
    name: "Sonu",
    preferences: {
      theme: "light",
    },
  },
};

const nextState = { ...state };
```

The outer object is new, but `nextState.user` still refers to the same nested object as `state.user`.

To update a nested field immutably, copy every object along the changed path:

```js
const nextState = {
  ...state,
  user: {
    ...state.user,
    preferences: {
      ...state.user.preferences,
      theme: "dark",
    },
  },
};
```

### Structural sharing

Immutability does not mean cloning the entire state tree. Create new references along the path of changed data and reuse unchanged branches.

This is called **structural sharing**. It supports efficient updates and lets reference equality help identify changed portions of state.

### A subtle example: new outer object, no meaningful data change

```js
function reducer(state, action) {
  if (action.type === "cart/itemAdded") {
    return {
      ...state,
      items: state.items,
    };
  }

  return state;
}
```

This creates a new outer object, but it does not add an item.

Assuming `state.items` is an array:

- `previousState === nextState` is `false`.
- `previousState.items === nextState.items` is `true`.
- The array's contents and length are unchanged.

A meaningful immutable addition is:

```js
return {
  ...state,
  items: [...state.items, action.payload],
};
```

This creates a new outer object and a new array containing the previous items plus the new item.

**Important distinction:** shallow copying is not inherently wrong. It is appropriate when only a top-level field changes. The requirement is to create new references along the path of the data being changed.

## 8. The Redux Store

The store owns the current state and coordinates state transitions.

Its three core operations are:

| Operation | Purpose |
|---|---|
| `getState()` | Read the current state |
| `dispatch(action)` | Send an action through the store |
| `subscribe(listener)` | Register a callback for store updates |

An educational simplified store:

```js
function createSimpleStore(reducer, preloadedState) {
  let state = preloadedState;
  const listeners = [];

  function getState() {
    return state;
  }

  function dispatch(action) {
    state = reducer(state, action);

    for (const listener of listeners) {
      listener();
    }

    return action;
  }

  function subscribe(listener) {
    listeners.push(listener);

    return function unsubscribe() {
      const index = listeners.indexOf(listener);

      if (index !== -1) {
        listeners.splice(index, 1);
      }
    };
  }

  dispatch({ type: "@@init" });

  return { getState, dispatch, subscribe };
}
```

This illustrates the core mechanism; it is not a replacement for the official Redux implementation.

Usage:

```js
const store = createSimpleStore(counterReducer, undefined);

console.log(store.getState());
// { count: 0 }

const unsubscribe = store.subscribe(() => {
  console.log("New state:", store.getState());
});

store.dispatch({ type: "counter/increment" });
// New state: { count: 1 }

unsubscribe();
```

The store:

1. Receives an action.
2. Calls the reducer with current state and action.
3. Saves the returned state.
4. Notifies subscribers.
5. Lets subscribers read the latest state.

A listener is not automatically passed the new state; it can call `getState()`.

## 9. Dispatch

`dispatch` is the standard entry point for requesting a state transition:

```js
store.dispatch({
  type: "counter/increment",
});
```

Calling a reducer manually is not equivalent:

```js
const nextState = counterReducer(currentState, action);
```

That only calculates a value. It does not update the store or notify its subscribers.

Not every dispatched action changes state. An unrecognized action can result in the same state being returned.

Also distinguish these events:

- An action is dispatched.
- A reducer calculates a result.
- The store may notify subscribers.
- A particular component may or may not rerender.

They are related, but not identical.

## 10. Connecting Redux to React

Redux can work independently of React. **React-Redux** is the official integration library.

Its key concepts include:

- `Provider`: makes the store available to descendant components.
- `useSelector`: reads a selected value from the store and subscribes to relevant changes.
- `useDispatch`: provides access to the store's dispatch function.

### Provider

```jsx
import { Provider } from "react-redux";
import { store } from "./store";

function Root() {
  return (
    <Provider store={store}>
      <App />
    </Provider>
  );
}
```

### Reading state with `useSelector`

```jsx
import { useSelector } from "react-redux";

function CartCount() {
  const count = useSelector((state) =>
    state.cart.items.reduce(
      (total, item) => total + item.quantity,
      0
    )
  );

  return <span>Cart ({count})</span>;
}
```

React-Redux compares the selected result using reference equality by default. If the selected result changes, the component can rerender. If it remains equal, that update alone does not require a rerender.

Selectors that create new arrays or objects on every evaluation can cause unnecessary rerenders unless designed or memoized appropriately.

### Dispatching from a component

```jsx
import { useDispatch } from "react-redux";

function AddToCartButton({ product }) {
  const dispatch = useDispatch();

  function handleClick() {
    dispatch({
      type: "cart/itemAdded",
      payload: product,
    });
  }

  return (
    <button onClick={handleClick}>
      Add to cart
    </button>
  );
}
```

The component expresses intent. The reducer owns the transition rules.

## 11. Reducer Composition

A large application can divide state-transition logic into smaller reducers by domain.

```js
import { combineReducers } from "redux";

function cartReducer(state = { items: [] }, action) {
  switch (action.type) {
    case "cart/cleared":
      return { items: [] };

    default:
      return state;
  }
}

function preferencesReducer(
  state = { theme: "light" },
  action
) {
  switch (action.type) {
    case "preferences/themeChanged":
      return { ...state, theme: action.payload };

    default:
      return state;
  }
}

const rootReducer = combineReducers({
  cart: cartReducer,
  preferences: preferencesReducer,
});
```

The root state becomes:

```js
{
  cart: {
    items: []
  },
  preferences: {
    theme: "light"
  }
}
```

The root reducer calls each child reducer with its own state slice and the action.

For a `preferences/themeChanged` action:

- `cartReducer` doesn't recognize it and returns its current state.
- `preferencesReducer` handles it and returns a new preferences state.
- The root reducer assembles the resulting slices.

Each reducer receives actions and decides whether they affect its slice. It does not need a separate registration for every action type.

## 12. Redux versus React `useReducer`

Both use the reducer pattern, but they address different state-ownership needs.

| Feature | React `useReducer` | Redux |
|---|---|---|
| State ownership | A React component | An external store |
| Typical sharing | Component and descendants, via props or Context | Components across the application |
| Update mechanism | Hook's `dispatch(action)` | Store's `dispatch(action)` |
| Reducer pattern | Yes | Yes |
| External-store subscriptions | Not a general-purpose Redux-style store | Built into the store model |
| Ecosystem | React | Redux Toolkit, React-Redux, middleware, DevTools |

Use `useReducer` for complex local state transitions, such as a multi-step wizard.

Use Redux when shared state has many consumers, transition logic benefits from a central home, and the tooling and ecosystem justify the additional architecture.

A complex reducer alone is not a reason to introduce Redux.

## 13. Asynchronous Operations and Side Effects

Reducers should not perform network requests because requests involve external effects, latency, failures, and results that are not known at dispatch time.

A request workflow can be represented as:

```text
Request begins
    ↓
requestStarted action
    ↓
Loading state

Request succeeds
    ↓
requestSucceeded action
    ↓
Data state

Request fails
    ↓
requestFailed action
    ↓
Error state
```

The asynchronous operation happens outside the reducer. Its result is represented by an action, and the reducer updates state in response.

Redux middleware can coordinate side effects. Thunk middleware, for example, allows dispatching functions that perform asynchronous work.

Modern Redux Toolkit supports `createAsyncThunk` and includes RTK Query for common server-data fetching and caching needs.

**Principle:** reducers calculate state transitions; effects perform work involving the outside world and report results through actions.

Redux does not automatically solve server-state concerns such as cache freshness, request deduplication, refetching, and invalidation. Libraries such as TanStack Query specialize in these problems.

## 14. Strengths and Trade-offs

### Strengths

- Explicit, recognizable state transitions.
- Centralized ownership for shared client state.
- Reducers can be tested as ordinary functions.
- Actions and state transitions can be inspected with suitable tooling.
- UI components can be separated from shared transition rules.
- State logic can be organized by domain.

### Costs

- More concepts: stores, actions, reducers, selectors, middleware, and integration.
- Unnecessary architecture for simple local state.
- State shape still requires careful design.
- Team conventions need to be understood and maintained.
- Generic Redux state does not automatically provide a complete server-data cache.

Redux provides structure, not magic. Its value depends on whether that structure is worth its complexity.

## 15. Choosing the Right State Owner

| Feature | Likely starting point | Reason |
|---|---|---|
| Tooltip open/closed | Local `useState` | One small UI concern |
| Checkout wizard step | `useReducer` or local state | Related transitions in one flow |
| Theme used across the app | Context or shared store | Shared preference |
| Cart used by distant components | Redux Toolkit may fit | Shared client state |
| Product listing from an API | TanStack Query or RTK Query | Remote data and cache lifecycle |
| Search filters and pagination | URL/query parameters | Shareable and restorable view |

These are starting points, not absolute rules. Requirements such as persistence, offline support, data ownership, and coordination needs affect the decision.

## 16. Common Mistakes

1. **Putting everything into Redux:** local UI state often belongs in the component.
2. **Mutating state in a traditional reducer:** return immutable updates.
3. **Performing side effects inside reducers:** keep effects outside and report results through actions.
4. **Storing redundant state:** derive values where practical.
5. **Assuming every dispatch causes every component to rerender:** selected-value equality matters.
6. **Confusing an action with an update:** dispatching an action does not guarantee that state changes.
7. **Treating Redux as a server-state cache by default:** remote data has separate lifecycle concerns.

## 17. Interview Questions

### Q1. Why does Redux use reducers instead of allowing components to update the store directly?

Reducers centralize transition rules and make changes explicit and testable. Components express intent through actions rather than implementing shared mutation logic independently.

### Q2. What is the difference between an action and a reducer?

An action describes an event. A reducer interprets the current state and action to calculate the next state.

### Q3. Why is immutability important?

It preserves previous state values and supports reference-based change detection, predictable updates, debugging, and efficient UI subscriptions.

### Q4. What happens if a reducer receives an unknown action?

It should return the current state unchanged. In a combined reducer setup, each slice reducer may receive actions relevant to other slices.

### Q5. Does dispatching an action always cause a component to rerender?

No. Subscribers can be notified, but a component's selected value may remain unchanged.

### Q6. Why shouldn't a reducer make an API request?

A request is a side effect with timing, failure, and external dependencies. Reducers should remain deterministic state-transition functions.

### Q7. When would you choose `useReducer` instead of Redux?

When state and its transitions belong to one component or a small local flow, and a global store's sharing and tooling benefits are unnecessary.

### Q8. Why might you use TanStack Query alongside Redux?

Redux can manage shared client-owned state, while TanStack Query specializes in remote-data caching, freshness, refetching, and invalidation.

## 18. Practice Exercises

### Exercise 1 — Write a reducer

Given:

```js
const initialState = {
  count: 0,
};
```

Implement a reducer handling:

- `counter/increment`
- `counter/decrement`
- `counter/reset`
- Unknown actions

Requirements:

- Do not mutate existing state.
- Return the existing state for unknown actions.
- Keep the reducer deterministic.

### Exercise 2 — Model a shopping cart

Start with:

```js
const initialState = {
  items: [],
};
```

Design actions and reducer transitions for:

- Adding an item.
- Increasing an existing item's quantity.
- Removing an item.
- Clearing the cart.

Decide what should happen when the same product is added twice. Either design can be valid if it is deliberate and consistent.

### Exercise 3 — Find the state bug

```js
function reducer(state, action) {
  switch (action.type) {
    case "cart/itemAdded":
      state.items.push(action.payload);
      return state;

    default:
      return state;
  }
}
```

Explain both the mutation problem and why returning the same reference matters.

### Exercise 4 — Choose the right state owner

For each feature, explain whether you would start with local state, `useReducer`, Context, Redux, or a server-state tool:

- A modal's open state.
- A multi-step checkout flow.
- A cart shared by the header, product pages, and checkout.
- A product listing fetched from an API.
- A search filter that should survive refresh and be shareable.

The reasoning matters more than choosing a particular library.

---

## Final Mental Model

Redux is built around explicit, predictable state transitions:

```text
Action
  ↓
Reducer
  ↓
Next State
  ↓
Store
  ↓
Subscribers
  ↓
UI updates where needed
```

The deepest takeaway is not that Redux has a store, actions, and reducers. It is that **explicit state transitions make shared application behavior easier to reason about**.

When you understand centralized ownership, deterministic transitions, immutability, structural sharing, and one-way data flow, Redux becomes an application of those ideas rather than a collection of APIs to memorize.

## What's Next?

**Chapter 85 — Redux Toolkit**

We'll explore:

- Why Redux Toolkit became the recommended approach.
- `configureStore` and `createSlice`.
- How Immer enables mutation-like reducer syntax while preserving immutable updates.
- Selectors and derived state.
- Async logic with `createAsyncThunk`.
- How modern Redux applications are structured.
