# Chapter 85 — Redux Toolkit

## Learning objectives

By the end of this chapter, you should be able to:
- Explain why Redux Toolkit exists and how it improves traditional Redux.
- Understand Immer and immutable state updates.
- Use `createSlice()` and understand generated action creators.
- Configure a Redux store with `configureStore()`.
- Connect React components using `Provider`, `useSelector`, and `useDispatch`.
- Explain memoized selectors with `createSelector()`.
- Implement async workflows with `createAsyncThunk()` and `extraReducers`.
- Understand Redux middleware and thunk middleware.
- Explain RTK Query, endpoints, generated hooks, caching, subscriptions, and tag invalidation.
- Choose between `createAsyncThunk()` and RTK Query.

---

## 1. Why Redux Toolkit exists

Traditional Redux is based on a central store, dispatched actions, reducers, and immutable state transitions. However, writing traditional Redux often involves repetitive code:

1. Defining action type constants.
2. Writing action creator functions.
3. Writing reducer switch cases.
4. Manually creating immutable updates.
5. Configuring the store, middleware, and DevTools.

### Traditional Redux example

```js
const initialState = { count: 0 };

const INCREMENT = "counter/increment";
const DECREMENT = "counter/decrement";
const RESET = "counter/reset";

function increment() {
  return { type: INCREMENT };
}

function decrement() {
  return { type: DECREMENT };
}

function reset() {
  return { type: RESET };
}

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { ...state, count: state.count + 1 };
    case DECREMENT:
      return { ...state, count: state.count - 1 };
    case RESET:
      return { ...state, count: 0 };
    default:
      return state;
  }
}
```

Nested immutable updates can become verbose:

```js
return {
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      preferences: {
        ...state.user.profile.preferences,
        theme: "dark",
      },
    },
  },
};
```

Redux Toolkit (RTK) is the official recommended way to write Redux logic. It does not replace Redux's architecture; it makes Redux code less repetitive and easier to configure.

Important APIs:
- `configureStore()` — creates and configures the Redux store.
- `createSlice()` — groups initial state and reducer logic, and generates action creators.
- Immer — enables mutation-like reducer syntax while producing immutable updates.
- `createSelector()` — creates memoized selectors.
- `createAsyncThunk()` — helps implement asynchronous workflows.
- RTK Query — provides data fetching and caching.

---

## 2. Immer and immutable updates

Redux reducers must not mutate the original state.

Bad mutation:

```js
function badReducer(state, action) {
  if (action.type === "counter/increment") {
    state.count += 1;
    return state;
  }
  return state;
}
```

This modifies the existing object and returns the same reference. Reference-based change detection can fail to recognise the update correctly.

A traditional immutable update creates a new object:

```js
function reducer(state, action) {
  if (action.type === "counter/increment") {
    return {
      ...state,
      count: state.count + 1,
    };
  }
  return state;
}
```

### How Immer helps

Redux Toolkit uses Immer for reducer logic created through `createSlice()` and other Immer-powered APIs.

Immer provides a draft of the state. Reducer code can use mutation-like operations on that draft. Immer tracks the changes and produces a new immutable state.

```js
increment(state) {
  state.count += 1;
}
```

The original state is not mutated. The change is applied to the draft, and Immer produces the next state.

### Nested update example

```js
const oldState = {
  user: {
    name: "Sonu",
    age: 26,
  },
};
```

RTK reducer:

```js
updateAge(state) {
  state.user.age = 27;
}
```

After Immer processes the update:
- `oldState.user.age` remains `26`.
- The next state has `user.age === 27`.
- Changed paths receive new references.
- Unchanged parts can retain their references.

This reference reuse is called **structural sharing**.

Mutation-like syntax is safe in an Immer-managed reducer draft. It is not a general permission to mutate Redux state anywhere in an application.

---

## 3. `createSlice()`

`createSlice()` combines a slice's name, initial state, and reducer functions.

```js
import { createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { count: 0 },
  reducers: {
    increment(state) {
      state.count += 1;
    },
    decrement(state) {
      state.count -= 1;
    },
  },
});
```

It returns an object with important properties:

### `counterSlice.reducer`

The reducer function for this slice:

```js
const reducer = counterSlice.reducer;
```

### `counterSlice.actions`

Action creators are generated from reducer names:

```js
counterSlice.actions.increment();
counterSlice.actions.decrement();
```

For example:

```js
counterSlice.actions.increment();
// { type: "counter/increment" }
```

### `counterSlice.name`

```js
counterSlice.name; // "counter"
```

### Payloads

```js
const userSlice = createSlice({
  name: "user",
  initialState: { name: "Sonu" },
  reducers: {
    updateName(state, action) {
      state.name = action.payload;
    },
  },
});
```

Calling:

```js
userSlice.actions.updateName("Rahul");
```

creates:

```js
{
  type: "user/updateName",
  payload: "Rahul"
}
```

The action creator creates an action object; it does not update state by itself. Dispatch sends the action to the store, and the reducer processes it.

---

## 4. Multiple slices and the root reducer

A Redux application can have multiple slices, each responsible for part of the state tree.

```js
const userSlice = createSlice({
  name: "user",
  initialState: { name: "Sonu" },
  reducers: {
    updateName(state, action) {
      state.name = action.payload;
    },
  },
});

const cartSlice = createSlice({
  name: "cart",
  initialState: { items: [] },
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload);
    },
  },
});
```

Combine them in the store:

```js
const store = configureStore({
  reducer: {
    user: userSlice.reducer,
    cart: cartSlice.reducer,
  },
});
```

The state tree is:

```js
{
  user: { name: "Sonu" },
  cart: { items: [] }
}
```

Conceptually, the root reducer behaves like:

```js
function rootReducer(state, action) {
  return {
    user: userSlice.reducer(state.user, action),
    cart: cartSlice.reducer(state.cart, action),
  };
}
```

When an action is dispatched, each combined slice reducer receives it. A slice that does not handle the action returns its existing state unchanged.

### Avoid duplicated state

If the cart UI needs the user's name, it can select both values from the store. Avoid copying the user's name into the cart slice without a strong reason.

Duplicated values can become inconsistent. Prefer one authoritative source and derive values from it.

---

## 5. `configureStore()`

`configureStore()` creates and configures the Redux store.

```js
import { configureStore } from "@reduxjs/toolkit";

const store = configureStore({
  reducer: {
    user: userSlice.reducer,
    cart: cartSlice.reducer,
  },
});
```

It combines reducers and provides useful defaults, including:
- Redux DevTools integration.
- Thunk middleware.
- Development checks for accidental mutations.
- Development checks for non-serializable values.
- A convenient middleware configuration API.

### Store basics

```js
store.getState();       // Read current state
store.dispatch(action); // Dispatch an action
store.subscribe(fn);    // Subscribe to store updates
```

---

## 6. Connecting Redux to React

Redux Toolkit creates the store. React-Redux connects the store to React components.

### `Provider`

```jsx
import { Provider } from "react-redux";

function App() {
  return (
    <Provider store={store}>
      <UserProfile />
      <Cart />
    </Provider>
  );
}
```

`Provider` makes the Redux store available to descendant components using React-Redux.

### `useSelector`

`useSelector` reads a value from the store and subscribes the component to changes in that selected value.

```jsx
import { useSelector } from "react-redux";

function UserProfile() {
  const name = useSelector((state) => state.user.name);
  return <h1>Hello, {name}</h1>;
}
```

By default, React-Redux compares previous and next selected values using strict reference equality (`===`). If the selected value is unchanged, the component does not need to re-render because of that store update.

### `useDispatch`

```jsx
import { useDispatch, useSelector } from "react-redux";

function UserProfile() {
  const name = useSelector((state) => state.user.name);
  const dispatch = useDispatch();

  function handleNameChange() {
    dispatch(userSlice.actions.updateName("Rahul"));
  }

  return (
    <div>
      <h1>{name}</h1>
      <button onClick={handleNameChange}>Change name</button>
    </div>
  );
}
```

| API | Responsibility |
|---|---|
| `Provider` | Makes the store available to descendant components |
| `useSelector` | Reads and subscribes to selected state |
| `useDispatch` | Provides the dispatch function |
| `dispatch(action)` | Sends an action to the store |

A component selecting `state.user.name` need not re-render when an unrelated cart update leaves that selected value unchanged. A component selecting `state.cart.items` similarly depends on the selected array reference.

---

## 7. Memoized selectors with `createSelector()`

A selector extracts data from Redux state. A derived selector calculates a value from state.

Example state:

```js
const state = {
  cart: {
    items: [
      { id: 1, name: "Keyboard", price: 2000, quantity: 2 },
      { id: 2, name: "Mouse", price: 1000, quantity: 1 },
    ],
  },
  user: { name: "Sonu" },
};
```

A normal selector:

```js
const selectCartTotal = (state) =>
  state.cart.items.reduce(
    (total, item) => total + item.price * item.quantity,
    0
  );
```

This calculates the total whenever it runs.

### Memoization

Memoization remembers a result and reuses it when its inputs have not changed.

```js
import { createSelector } from "@reduxjs/toolkit";

const selectCartItems = (state) => state.cart.items;

const selectCartTotal = createSelector(
  [selectCartItems],
  (items) =>
    items.reduce(
      (total, item) => total + item.price * item.quantity,
      0
    )
);
```

There are two parts:
1. **Input selectors** extract the inputs.
2. **Result function** calculates the derived result.

If the user name changes but the cart items array retains the same reference, the result function can reuse its cached result.

If the cart items array gets a new reference, the result function recalculates.

Memoization is based on input identity, not a deep comparison of array contents.

### `useSelector` vs `createSelector`

- `useSelector` connects a component to selected Redux state.
- `createSelector` memoizes derived calculations.

They are often used together:

```jsx
function CartSummary() {
  const total = useSelector(selectCartTotal);
  return <h2>Total: ₹{total}</h2>;
}
```

---

## 8. Asynchronous logic with `createAsyncThunk()`

Reducers should be synchronous and pure. They should calculate the next state from the current state and an action, without making network requests or performing other side effects.

A reducer should not be declared `async` and return a Promise. Redux expects a reducer to synchronously return the next state.

### Define an async thunk

```js
import { createAsyncThunk } from "@reduxjs/toolkit";

export const fetchUser = createAsyncThunk(
  "user/fetchUser",
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);

    if (!response.ok) {
      throw new Error("Failed to fetch user");
    }

    return response.json();
  }
);
```

- `"user/fetchUser"` is the action type prefix.
- The async function is the payload creator.
- `userId` is the argument passed when dispatching.
- Returned data becomes the fulfilled action's payload.

### Lifecycle actions

```js
dispatch(fetchUser(123));
```

RTK generates:

| Lifecycle | Action type |
|---|---|
| Request starts | `user/fetchUser/pending` |
| Request succeeds | `user/fetchUser/fulfilled` |
| Request fails | `user/fetchUser/rejected` |

A request starts with `pending` and then ends in either `fulfilled` or `rejected`.

### Handle lifecycle actions with `extraReducers`

```js
const userSlice = createSlice({
  name: "user",
  initialState: {
    data: null,
    loading: false,
    error: null,
  },

  reducers: {
    clearUser(state) {
      state.data = null;
    },
  },

  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});
```

The API request happens inside the thunk. The slice's `extraReducers` handles lifecycle actions and updates state.

### `reducers` vs `extraReducers`

The distinction is about action ownership, not whether an operation is synchronous or asynchronous.

| `reducers` | `extraReducers` |
|---|---|
| Handles actions defined by this slice | Handles actions defined elsewhere |
| Generates action creators | Does not generate action creators |
| Example: `userSlice.actions.clearUser()` | Example: `fetchUser.fulfilled` |
| Commonly used for slice-owned state transitions | Can handle any external Redux action |

A slice can respond to an action created by another slice:

```js
builder.addCase(cartSlice.actions.clearCart, (state) => {
  state.data = null;
});
```

`cartSlice` created the action; `userSlice` listens to it through `extraReducers`.

### One thunk can be handled by multiple slices

Multiple slice reducers can register handlers for the same thunk lifecycle action. The root reducer invokes the combined slice reducers, and each slice independently updates its own state.

### What does dispatch return?

```js
const result = dispatch(fetchUser(123));
```

Dispatching a `createAsyncThunk` returns a Promise that resolves to the final action (fulfilled or rejected).

To get the payload with normal Promise rejection behaviour, use `.unwrap()`:

```js
try {
  const user = await dispatch(fetchUser(123)).unwrap();
  console.log(user);
} catch (error) {
  console.error(error);
}
```

`createAsyncThunk` generates lifecycle actions. It does not automatically decide how or where the application stores the returned data; reducers handle that.

---

## 9. Redux middleware

Middleware sits in the Redux dispatch pipeline. It can inspect actions, perform side effects, dispatch additional actions, transform or stop actions, and pass actions along.

```text
Component
   |
   v
dispatch(action)
   |
   v
Middleware 1
   |
   v
Middleware 2
   |
   v
Root reducer
   |
   v
New state
   |
   v
Subscribers
```

### Middleware structure

```js
const loggerMiddleware = (store) => (next) => (action) => {
  console.log("Dispatching:", action);

  const result = next(action);

  console.log("Updated state:", store.getState());

  return result;
};
```

| Parameter | Meaning |
|---|---|
| `store` | Provides access to `getState()` and `dispatch()` |
| `next` | Passes the action to the next middleware in the chain |
| `action` | The current action |

`next(action)` forwards the action. If middleware doesn't forward it, it won't reach the reducer through that middleware chain.

### Register custom middleware

```js
const store = configureStore({
  reducer: {
    user: userSlice.reducer,
    cart: cartSlice.reducer,
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(loggerMiddleware),
});
```

Use `getDefaultMiddleware()` so the default middleware is retained.

### Thunk middleware

Ordinary Redux dispatch expects action objects. Thunk middleware also allows functions to be dispatched.

Conceptual implementation:

```js
const thunkMiddleware = ({ dispatch, getState }) =>
  (next) =>
  (action) => {
    if (typeof action === "function") {
      return action(dispatch, getState);
    }

    return next(action);
  };
```

- If the action is a function, thunk middleware executes it and supplies `dispatch` and `getState`.
- If the action is a normal object, it forwards it using `next(action)`.

This is the foundation that allows thunk functions and `createAsyncThunk` workflows to execute.

### RTK Query middleware

RTK Query's reducer and middleware have different responsibilities:

- Its reducer stores API cache and related state.
- Its middleware coordinates query lifecycles, subscriptions, invalidation, polling and other operational behaviour.

Both are registered in the normal RTK Query store setup.

---

## 10. RTK Query

`createAsyncThunk` provides an asynchronous workflow, but the application still has to manage much of the server-state infrastructure: caching, freshness, refetching, deduplication and invalidation.

RTK Query is Redux Toolkit's data-fetching and caching solution.

It is useful for common server-data operations such as fetching products, profiles and order history, and updating or deleting server data.

### Query vs mutation

| Query | Mutation |
|---|---|
| Reads server data | Changes server data |
| Usually GET | Usually POST, PUT, PATCH or DELETE |
| Caches responses | Can invalidate cached data |
| Example: `getProducts` | Example: `deleteProduct` |

The distinction is about the operation's purpose, not a strict HTTP-method restriction.

### Define an API service

```js
import {
  createApi,
  fetchBaseQuery,
} from "@reduxjs/toolkit/query/react";

export const productsApi = createApi({
  reducerPath: "productsApi",

  baseQuery: fetchBaseQuery({
    baseUrl: "/api",
  }),

  endpoints: (builder) => ({
    getProducts: builder.query({
      query: () => "/products",
    }),

    deleteProduct: builder.mutation({
      query: (id) => ({
        url: `/products/${id}`,
        method: "DELETE",
      }),
    }),
  }),
});
```

Important pieces:
- `createApi()` defines an API service.
- `reducerPath` identifies the API reducer's state location.
- `fetchBaseQuery()` provides a lightweight wrapper around `fetch`.
- `endpoints` defines API operations.
- `builder.query()` defines a read-oriented endpoint.
- `builder.mutation()` defines a write-oriented endpoint.

### Generated React hooks

RTK Query generates hooks from endpoint names:

```js
productsApi.useGetProductsQuery();
productsApi.useDeleteProductMutation();
```

Query hook:

```jsx
function ProductList() {
  const {
    data,
    isLoading,
    isError,
    error,
  } = productsApi.useGetProductsQuery();

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>{error.message}</p>;

  return data.map((product) => (
    <p key={product.id}>{product.name}</p>
  ));
}
```

Mutation hook:

```jsx
function ProductActions({ id }) {
  const [deleteProduct, { isLoading }] =
    productsApi.useDeleteProductMutation();

  return (
    <button
      disabled={isLoading}
      onClick={() => deleteProduct(id)}
    >
      Delete
    </button>
  );
}
```

Query hooks subscribe to query cache entries and initiate fetching when needed. They do not necessarily make a new request on every component render.

Mutation hooks return a trigger function. The component decides when to execute the mutation.

### Shared cache and subscriptions

If `ProductList` and `FeaturedProducts` both use:

```js
useGetProductsQuery();
```

they request the same endpoint with the same arguments. RTK Query identifies the same query cache key and can share the cache entry and deduplicate requests.

When the shared cache updates, both subscribed components receive the updated data.

### Cache invalidation

Cache invalidation marks cached data as potentially stale so RTK Query can refresh or remove affected cache entries.

Tags are labels associated with query cache entries. They are not freshness measurements or deep comparisons of data.

```js
getProducts: builder.query({
  query: () => "/products",
  providesTags: ["Products"],
}),

deleteProduct: builder.mutation({
  query: (id) => ({
    url: `/products/${id}`,
    method: "DELETE",
  }),
  invalidatesTags: ["Products"],
}),
```

Sequence:
1. `getProducts` fetches products.
2. The response is cached.
3. The query provides the `Products` tag.
4. `deleteProduct` sends the DELETE request.
5. When the mutation succeeds, it invalidates the `Products` tag.
6. RTK Query finds affected cache entries.
7. Active subscriptions are refetched; inactive invalidated entries can be removed.

No browser refresh is needed when the current application initiates the mutation.

If another administrator changes server data from another browser, this client is not automatically notified. Polling, WebSockets, Server-Sent Events, refetch-on-focus or reconnection may be needed.

### Specific tags and list tags

```js
getProducts: builder.query({
  query: () => "/products",

  providesTags: (result) =>
    result
      ? [
          ...result.map(({ id }) => ({
            type: "Products",
            id,
          })),
          { type: "Products", id: "LIST" },
        ]
      : [{ type: "Products", id: "LIST" }],
}),
```

Delete mutation:

```js
deleteProduct: builder.mutation({
  query: (id) => ({
    url: `/products/${id}`,
    method: "DELETE",
  }),

  invalidatesTags: (result, error, id) => [
    { type: "Products", id },
    { type: "Products", id: "LIST" },
  ],
}),
```

The specific tag identifies an individual product. The `LIST` tag identifies the product-list query.

Invalidating `{ type: "Products", id: 10 }` matches queries providing that specific tag. It does not automatically match `{ type: "Products", id: "LIST" }`. To invalidate both the detail query and list query, invalidate both tags.

RTK Query's default invalidation behaviour is delayed: if requests are still pending, invalidations can be queued until pending queries and mutations settle.

### Register RTK Query in the store

```js
const store = configureStore({
  reducer: {
    [productsApi.reducerPath]: productsApi.reducer,
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(productsApi.middleware),
});
```

Both are needed:

| Registration | Responsibility |
|---|---|
| `productsApi.reducer` | Stores RTK Query cache and related state |
| `productsApi.middleware` | Coordinates query lifecycles, subscriptions, invalidation, polling and other behaviour |

---

## 11. `createAsyncThunk` vs RTK Query

Both can make API requests, but they solve different problems.

| Aspect | `createAsyncThunk` | RTK Query |
|---|---|---|
| Primary purpose | Custom asynchronous workflows | Server-state fetching and management |
| API requests | Implement in payload creator | Define endpoints |
| Loading/error state | Usually manage in slice reducers | Generated query/mutation state |
| Caching | Implement manually if needed | Built in |
| Cache invalidation | Implement manually if needed | Tag-based |
| Refetching | Implement manually if needed | Built in |
| Good fit | Custom async business workflows | Common server-data operations |

### When `createAsyncThunk` is useful

A multi-step checkout workflow might involve:
1. Validating checkout information.
2. Submitting payment.
3. Processing the response.
4. Updating application state.
5. Handling custom failure or compensation logic.

This kind of custom asynchronous workflow can be a good fit for `createAsyncThunk`.

### When RTK Query is useful

Typical server-data operations include fetching products, user profiles and order history, and updating product details or deleting cart items.

RTK Query is designed to manage the associated cache and request lifecycle.

This is not an absolute division: `createAsyncThunk` can fetch data, and RTK Query supports mutations. Choose based on the problem.

---

## 12. Redux Toolkit best practices

### Don't put everything into Redux

| State | Typical location |
|---|---|
| Input field value | Local component state |
| Modal open/close | Local component state |
| Shared authentication data | Shared state solution |
| Product catalogue | RTK Query |
| Cart shared across components | Redux Toolkit or server-backed design |
| Search filters that belong in URL | URL state |
| Complex form state | React Hook Form |

Put state where it naturally belongs. Don't globalize state without a reason.

### Avoid duplicating server state

If RTK Query already manages products, avoid copying the same response into another Redux slice without a strong reason. Two sources of truth can become inconsistent.

### Keep reducers pure

Do not perform network requests, storage operations, timers or other side effects inside reducers. Use appropriate middleware, thunks or external mechanisms.

### Prefer serializable state and actions

Prefer ordinary serializable values such as strings, numbers, arrays and plain objects. Avoid storing functions, Promises, DOM elements, class instances, `Map` or `Set` unless there is a specific, deliberate reason and relevant tooling is configured accordingly.

### Use selectors

Create reusable selectors for state access and derived calculations. Use memoized selectors when derived calculations are expensive or stable references matter.

### Choose the right async tool

Use RTK Query for server-state fetching and caching. Use `createAsyncThunk` when implementing custom asynchronous workflows that need explicit control over their lifecycle.

---

## 13. Complete application architecture

An e-commerce application might organize its state and data flow as follows:

```text
React Components
  |
  |-- useSelector / useDispatch
  |-- RTK Query hooks
  |
  v
Redux Store (configureStore)
  |
  |-- User slice
  |-- Cart slice
  |-- Preferences slice
  |-- RTK Query API reducer
  |
  v
Middleware
  |
  |-- Thunk middleware
  |-- RTK Query middleware
  |-- Custom middleware
  |
  v
Backend APIs
  |
  v
Database and business logic
```

| Requirement | Redux Toolkit API |
|---|---|
| Create the store | `configureStore` |
| Define slice state and actions | `createSlice` |
| Immutable reducer updates | Immer |
| Read state in React | `useSelector` |
| Dispatch actions | `useDispatch` |
| Memoize derived calculations | `createSelector` |
| Custom async workflows | `createAsyncThunk` |
| API fetching and caching | RTK Query |
| Middleware configuration | `configureStore` |

---

## 14. Interview questions and revision prompts

1. Why does Redux Toolkit exist? What problems does it solve compared with traditional Redux?
2. How does Immer allow mutation-like syntax while preserving immutability?
3. What does `createSlice()` generate?
4. How does `configureStore()` combine multiple slice reducers?
5. What is the difference between `useSelector` and `useDispatch`?
6. How does React-Redux decide whether a component needs to re-render after a store update?
7. What is memoization? How does `createSelector()` use input selector results?
8. Why should reducers be synchronous and pure?
9. What lifecycle actions does `createAsyncThunk()` generate?
10. What is the difference between `reducers` and `extraReducers`?
11. Can multiple slices handle the same action? How?
12. What does `dispatch(createAsyncThunk(...))` return? What does `.unwrap()` do?
13. What is Redux middleware? Explain `store`, `next`, and `action`.
14. How does thunk middleware allow functions to be dispatched?
15. What is the difference between RTK Query queries and mutations?
16. How does RTK Query share cache entries between components?
17. Explain `providesTags` and `invalidatesTags`.
18. What is the difference between a specific entity tag and a list tag?
19. Why does RTK Query need both a reducer and middleware?
20. When would you choose `createAsyncThunk` over RTK Query?

---

## 15. Final mental model

- **Redux** provides a predictable, centralized state architecture.
- **Redux Toolkit** makes that architecture easier and safer to implement.
- **Immer** simplifies immutable updates.
- **createSlice** defines slice state transitions and generates action creators.
- **configureStore** assembles the application store and configures defaults.
- **React-Redux** connects React components to the store.
- **createSelector** memoizes derived calculations.
- **createAsyncThunk** manages custom asynchronous workflows through lifecycle actions.
- **Middleware** extends the dispatch pipeline.
- **RTK Query** manages server-data fetching, caching, subscriptions and invalidation.

The most important principle: understand the problem each API solves, rather than memorising its syntax.
