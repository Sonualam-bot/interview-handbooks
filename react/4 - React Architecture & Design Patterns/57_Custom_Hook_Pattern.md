# Chapter 57 — Custom Hook Pattern

**Handbook 4 — React Architecture & Design Patterns**

---

## 1. What is a Custom Hook?

A **Custom Hook** is a JavaScript function whose name starts with `use` and which can call other React Hooks.

Its purpose is to:

> **Extract and reuse stateful React logic without reusing the UI itself.**

Example:

```jsx
function useToggle(initialValue = false) {
  const [on, setOn] = useState(initialValue);

  function toggle() {
    setOn(value => !value);
  }

  return { on, toggle };
}
```

Multiple components can reuse the same logic with completely different UI.

---

## Deep Dive `[NEW]`

### Why Custom Hooks Solve HOCs' Oldest Problem for Free

Chapter 54 traced "wrapper hell" to a literal structural cost: every
HOC application creates a genuinely new fiber in the tree. A custom
Hook creates none. When `Checkout` calls `useCheckout()`, any
`useState`/`useReducer`/`useEffect` calls inside `useCheckout` add
their nodes to **`Checkout`'s own fiber's Hook list** — not to some
separate fiber for `useCheckout` (see Handbook 3, Custom Hooks: a
custom Hook is not a runtime "thing," just a function whose Hook calls
land wherever it's called from). Compose ten custom Hooks inside one
component and you still have exactly one fiber, one component in
DevTools, one render call — compare this to composing ten HOCs, which
produces ten nested fibers. This is the concrete, mechanical reason
custom Hooks displaced HOCs and render props as the default reuse
mechanism once Hooks existed: the same "extract and reuse stateful
logic" goal, with the wrapper-hell cost structurally impossible to
incur, because there's no wrapping component in the picture at all.

### The Trade-off This Creates: No Extra Rendering Control

The flip side is that a custom Hook cannot intercept or conditionally
skip rendering the way an HOC can (an HOC can choose to return
`<Spinner />` instead of rendering the wrapped component at all — see
Chapter 54's typical use cases). A custom Hook only returns *values* —
it participates in the calling component's single render, it cannot
short-circuit that render or substitute a different output on the
component's behalf. This is precisely why patterns like
loading/error/auth gating, which used to be implemented as HOCs
(`withAuth(Component)` redirecting instead of rendering), are
typically rewritten today as "a custom Hook that returns state, plus
the component itself decides what to render based on it" — the
rendering decision moves back into the component, because the Hook
mechanism has no equivalent of an HOC's ability to intercept the
render output.

## 2. Core Idea

A Custom Hook extracts things such as:

```text
State
Effects
Refs
Memoization
Event logic
Other Hooks
```

But **not the component's UI**.

```text
Component A ──┐
Component B ──┼──→ Custom Hook → reusable logic
Component C ──┘
```

The components decide how to render.

---

## 3. Why Do We Need Custom Hooks?

Suppose several components need the browser's current window width.

Without a Custom Hook, the same state + effect logic might be duplicated.

Instead:

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return width;
}
```

Now:

```jsx
function Sidebar() {
  const width = useWindowWidth();

  return ...;
}
```

and:

```jsx
function Header() {
  const width = useWindowWidth();

  return ...;
}
```

The behavior is reused without sharing the UI.

---

## 4. Custom Hooks Share Logic, Not State

This is **extremely important**.

```jsx
function useCounter() {
  const [count, setCount] = useState(0);

  return {
    count,
    increment: () => setCount(c => c + 1)
  };
}
```

If two components call it:

```jsx
function A() {
  const counter = useCounter();
}

function B() {
  const counter = useCounter();
}
```

They do **not** share the same `count`.

Each invocation gets its own state:

```text
Component A
    ↓
useCounter()
    ↓
State A

Component B
    ↓
useCounter()
    ↓
State B
```

Therefore:

> **Custom Hooks share stateful logic, not state itself.**

For shared state, use something such as:

- Context
- external state store
- lifted state
- another shared-state mechanism

---

## 5. Custom Hook Naming

A Custom Hook starts with:

```text
use
```

Examples:

```jsx
useToggle()
useFetch()
useWindowWidth()
useDebounce()
useOnlineStatus()
useLocalStorage()
usePrevious()
```

The naming convention tells React and developers that the function may use Hooks.

---

## 6. A Custom Hook Can Use Other Hooks

Example:

```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    document.title = title;
  }, [title]);
}
```

Or:

```jsx
function useUser() {
  const context = useContext(UserContext);

  return context;
}
```

A Custom Hook is essentially a way to **compose Hooks into a reusable abstraction**.

---

## 7. Custom Hook API Design

A good Custom Hook exposes a clear API:

```jsx
const {
  data,
  loading,
  error,
  refetch
} = useFetch(url);
```

Instead of exposing implementation details.

Mental model:

```text
Internal implementation
        ↓
      Hook
        ↓
  Small public API
        ↓
    Component
```

The component should receive what it actually needs.

---

## 8. Custom Hooks Can Accept Parameters

```jsx
function useToggle(initialValue = false) {
  const [on, setOn] = useState(initialValue);

  const toggle = () => {
    setOn(value => !value);
  };

  return { on, toggle };
}
```

Usage:

```jsx
const toggle = useToggle(true);
```

Parameters make hooks reusable across different situations.

---

## 9. Custom Hooks Can Return Anything

A Custom Hook can return:

### A value

```jsx
return width;
```

### An object

```jsx
return {
  width,
  isMobile
};
```

### An array

```jsx
return [value, setValue];
```

### Functions

```jsx
return {
  open,
  close,
  toggle
};
```

There is no required return type. Design the API around how consumers naturally use the hook.

---

## 10. Custom Hooks vs Utility Functions

A normal utility:

```js
function formatDate(date) {
  return ...;
}
```

doesn't use React state or lifecycle behavior.

A Custom Hook:

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(...);

  useEffect(...);

  return width;
}
```

uses React Hooks.

```text
Utility function
→ reusable general-purpose computation

Custom Hook
→ reusable React stateful behavior
```

---

## 11. Custom Hooks vs HOCs

### HOC

```jsx
const Enhanced = withAuth(Component);
```

The component is wrapped.

### Custom Hook

```jsx
function Profile() {
  const user = useUser();

  return <h1>{user.name}</h1>;
}
```

The component directly consumes reusable logic.

Custom Hooks generally avoid:

- wrapper components
- wrapper nesting
- injected props
- HOC naming complexity

This is one reason Hooks became an important architectural improvement.

---

## 12. Custom Hooks vs Render Props

### Render Props

```jsx
<MouseTracker>
  {({ x, y }) => (
    <Cursor x={x} y={y} />
  )}
</MouseTracker>
```

The component exposes state through a rendering function.

### Custom Hook

```jsx
function Cursor() {
  const { x, y } = useMousePosition();

  return <CursorUI x={x} y={y} />;
}
```

Mental model:

```text
Render Props
→ reuse logic through a component + function

Custom Hook
→ reuse logic directly inside a component
```

---

## 13. Custom Hooks vs Context

### Custom Hook

Provides reusable logic:

```jsx
const user = useUser();
```

### Context

Provides shared data to a subtree:

```jsx
const user = useContext(UserContext);
```

They can be combined:

```jsx
function useUser() {
  return useContext(UserContext);
}
```

Mental model:

```text
Context
→ mechanism for sharing values

Custom Hook
→ abstraction for consuming/combining React logic
```

---

## 14. Rules of Hooks Still Apply

Custom Hooks do **not** bypass the Rules of Hooks.

❌ Don't:

```jsx
function useData(condition) {
  if (condition) {
    useEffect(...);
  }
}
```

Hooks must still be called:

- at the top level
- consistently between renders
- from React components or Custom Hooks

The fact that the code lives inside a Custom Hook doesn't change the Rules of Hooks.

---

## 15. A Good Custom Hook Has One Clear Responsibility

Avoid a giant hook:

```jsx
function useEverything() {
  // authentication
  // fetching
  // analytics
  // window resize
  // form validation
  // notifications
}
```

Prefer cohesive hooks:

```jsx
useAuth()
useFetch()
useWindowSize()
useFormValidation()
useNotifications()
```

Principle:

> **A Custom Hook should encapsulate a cohesive piece of behavior.**

---

## 16. Custom Hooks Can Compose Other Custom Hooks

```jsx
function useUserProfile() {
  const user = useUser();
  const { data, loading } = useFetch(`/users/${user.id}`);

  return {
    user,
    data,
    loading
  };
}
```

Mental model:

```text
useUserProfile
      ↓
 ┌────┴─────┐
 ↓          ↓
useUser   useFetch
```

You can build higher-level abstractions from smaller hooks.

This is **composition of behavior**.

---

## 17. Example: Online/Offline Status

A useful example is reacting to browser connectivity events:

```jsx
function useOnlineStatus() {
  const [online, setOnline] = useState(true);

  useEffect(() => {
    function handleOnline() {
      setOnline(true);
    }

    function handleOffline() {
      setOnline(false);
    }

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return online;
}
```

Component:

```jsx
function Status() {
  const online = useOnlineStatus();

  return <p>{online ? "Online" : "Offline"}</p>;
}
```

### Important data flow

`online` and `offline` are **browser events**.

The browser detects a connectivity change and fires the corresponding event.

```text
Network connectivity changes
          ↓
Browser fires "online"/"offline"
          ↓
handleOnline / handleOffline
          ↓
setOnline(...)
          ↓
Hook state updates
          ↓
Component re-renders
          ↓
UI updates
```

We do **not** manually trigger the browser event and we are not polling.

### Important distinction

`navigator.onLine` / the `online` and `offline` events represent the browser's general network connectivity state.

They do **not** guarantee that your particular backend/API is reachable.

For example:

```text
Wi-Fi connected
      ↓
Browser: online
      ↓
Your API server: down
```

The browser can still report `online`.

---

## 18. Custom Hook Data Flow

For `useOnlineStatus()`:

```text
Browser detects network change
          ↓
offline event
          ↓
handleOffline()
          ↓
setOnline(false)
          ↓
React schedules state update
          ↓
Component renders again
          ↓
useOnlineStatus() returns false
          ↓
"Offline" appears in UI
```

When connectivity returns:

```text
online event
     ↓
handleOnline()
     ↓
setOnline(true)
     ↓
React re-renders
     ↓
"Online" appears
```

This is a good example of a Custom Hook encapsulating an **external browser API** and exposing a simple React-friendly value.

---

## 19. The Biggest Architectural Benefit

Custom Hooks separate:

```text
WHAT THE COMPONENT DISPLAYS
```

from:

```text
HOW THE COMPONENT OBTAINS/MANAGES THE BEHAVIOR
```

Example:

```jsx
function Checkout() {
  const {
    submit,
    loading,
    error
  } = useCheckout();

  return (
    <button onClick={submit}>
      {loading ? "Processing..." : "Pay"}
    </button>
  );
}
```

The component focuses on UI.

`useCheckout()` focuses on checkout behavior.

This creates a clean separation of concerns.

---

## 20. Interview Answer

> **A Custom Hook is a JavaScript function whose name starts with `use` and that can compose React Hooks to encapsulate reusable stateful logic. It allows components to share behavior without sharing their UI or state. Each invocation normally gets its own hook state unless the hook explicitly connects to a shared mechanism such as Context or an external store.**

---

## 21. Key Takeaways

1. **Custom Hook = reusable React logic.**
2. It is a function whose name starts with `use`.
3. It can call other Hooks.
4. It shares **logic, not state**.
5. Every invocation normally gets independent state.
6. It should expose a clean public API.
7. Custom Hooks can compose other Custom Hooks.
8. Rules of Hooks still apply.
9. They often replace many HOC and Render Prop use cases.
10. They separate **behavior from UI**.
11. Browser events can be encapsulated cleanly inside Custom Hooks.
12. `online` / `offline` are browser connectivity events, not API health checks.

### One-line memory trick

> **Custom Hook = "Extract the React logic, leave the UI to the component."**

---

# 🧪 Exercise — Build `useOnlineStatus()`

Implement the hook yourself rather than copying the example.

## Step 1 — Create the hook

Create:

```jsx
function useOnlineStatus() {
  // ...
}
```

Start with:

```jsx
const [online, setOnline] = useState(true);
```

---

## Step 2 — Subscribe to browser events

Inside `useEffect`, create:

```jsx
function handleOnline() {
  // ...
}

function handleOffline() {
  // ...
}
```

Attach them to:

```jsx
window.addEventListener("online", handleOnline);
window.addEventListener("offline", handleOffline);
```

---

## Step 3 — Update the hook state

Make:

```text
online event
→ setOnline(true)

offline event
→ setOnline(false)
```

---

## Step 4 — Clean up

Return a cleanup function:

```jsx
return () => {
  window.removeEventListener("online", handleOnline);
  window.removeEventListener("offline", handleOffline);
};
```

Remember:

> **Every event listener added by an Effect should generally be removed when the Effect is cleaned up.**

---

## Step 5 — Consume the hook

Create:

```jsx
function Status() {
  const online = useOnlineStatus();

  return (
    <p>
      {online ? "Online" : "Offline"}
    </p>
  );
}
```

---

## Step 6 — Test the data flow

Don't just verify the UI.

Explain the complete flow.

### Going offline

```text
1. User disconnects Wi-Fi / loses network
2. Browser detects connectivity loss
3. Browser fires "offline"
4. handleOffline() runs
5. setOnline(false)
6. Hook state changes
7. Component re-renders
8. useOnlineStatus() returns false
9. UI displays "Offline"
```

### Coming back online

```text
1. Network connectivity returns
2. Browser fires "online"
3. handleOnline() runs
4. setOnline(true)
5. Hook state changes
6. Component re-renders
7. useOnlineStatus() returns true
8. UI displays "Online"
```

### 🔥 Check yourself

You should be able to answer:

> **Who fires the `online` and `offline` events?**

**Answer:** The browser.

> **Does our React code manually trigger them?**

**Answer:** No. We subscribe to browser-provided events.

> **Does `useOnlineStatus()` share one `online` state across every component using it?**

**Answer:** No. Each invocation has its own hook state, although each instance can respond to the same browser events.

> **Does `online === true` guarantee our backend is reachable?**

**Answer:** No. It represents the browser's general connectivity status, not backend health.

---

# 🎯 Interview Priority

### 🔥🔥🔥 Know this

- What a Custom Hook is
- Why it starts with `use`
- Custom Hooks share logic, not state
- Rules of Hooks
- Hook API design
- Custom Hook composition
- Custom Hook vs HOC
- Custom Hook vs Render Props
- Custom Hook vs Context
- External browser APIs/events inside Hooks

### Lower priority

- Extremely elaborate hook abstractions
- Library-specific hook implementations
- Over-engineered hook composition

---

## One-line Memory Trick

> **Custom Hook = "Extract the React logic, leave the UI to the component."**
