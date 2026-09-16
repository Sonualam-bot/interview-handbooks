# Chapter 31 — Why Hooks?

## Chapter Overview

Hooks are one of the most important parts of modern React.

The goal is not to memorize `useState`, `useEffect`, `useRef`, `useMemo`, and `useCallback` as a list of APIs.

The important question is:

> **What problem were Hooks created to solve?**

Core idea:

```text
Function Components
        ↓
Need state + React capabilities
        ↓
Need reusable stateful logic
        ↓
Hooks
        ↓
Composable React behavior
```

---

## Deep Dive `[NEW]`

### Why Hooks Couldn't Just Be Added as New Class Lifecycle Methods

React could, in principle, have "solved" logic reuse by adding more
composition primitives on top of classes — and it tried exactly that
before Hooks, via Higher-Order Components and render props. Both work,
but both have a structural cost: they add a wrapper *component* to the
tree for every piece of reused logic. Wrap a component in three HOCs
for three pieces of reused behavior (data fetching, auth, theming) and
you get three extra layers in the component tree ("wrapper hell"),
each with its own props collisions to manage (two HOCs both wanting to
inject a prop called `data`) and its own indirection when debugging.
Hooks solve reuse *without* adding to the tree at all — a custom Hook
is just a function call inside the existing component, contributing
zero additional fibers, zero additional nesting, and zero prop-name
collisions. This is the concrete architectural reason Hooks
"composable React behavior" beats HOCs/render props, not just a
stylistic preference.

### Why Reusable Logic Couldn't Just Be a Plain JavaScript Function

Before Hooks, you could already extract plain-function helpers
(`function formatDate(d) {}`), so "reuse logic" wasn't a new problem —
what was missing was reusing logic that needs to *plug into a
component's render cycle*: subscribe on mount, unsubscribe on unmount,
trigger a re-render when the subscribed data changes. A plain function
has no way to do any of that — it has no access to "this component's
state" or "this component's re-render trigger," because those only
exist on a fiber, and a plain function doesn't run with a fiber
context attached. Hooks work specifically because React calls them
*during* a component's render, when React already knows which fiber
is being processed — that's what lets `useState` inside a custom Hook
attach its state to the calling component's fiber, not to some
independent scope. This is the real reason the Rules of Hooks (Chapter
32) exist and the real reason a Hook can't just be called from
anywhere: it needs to run inside a render, with a known fiber, or
there's nothing for it to attach state to.

# 1. Before Hooks

React historically had two major component styles.

### Function Components

```jsx
function Counter() {
  return <button>Count</button>;
}
```

### Class Components

```jsx
class Counter extends React.Component {
  render() {
    return <button>Count</button>;
  }
}
```

Historically, function components were primarily used for presenting UI, while class components were used when state and lifecycle behavior were required.

This created a split:

```text
Function Component
→ Simple UI

Class Component
→ State
→ Lifecycle behavior
→ Stateful logic
```

Hooks changed this model.

---

# 2. The Problem With This Split

Imagine a component needs:

```text
State
+
Lifecycle behavior
+
Reusable logic
```

A class could handle this:

```jsx
class ChatRoom extends React.Component {
  state = {
    connected: false
  };

  componentDidMount() {
    // connect
  }

  componentWillUnmount() {
    // disconnect
  }

  render() {
    // UI
  }
}
```

But another component might need the same connection behavior.

You don't want to duplicate:

```text
connect
subscribe
listen
cleanup
disconnect
```

in every component.

You want:

> **Reusable stateful logic.**

This is one of the major problems Hooks address.

---

# 3. Hooks Let Function Components Use React Features

Hooks allow function components to use React features such as:

```text
State
Effects
Refs
Context
Memoization
Reducers
Transitions
```

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

The model becomes:

```text
Before Hooks

Function Component
→ Primarily presentation

Class Component
→ State
→ Lifecycle behavior
→ Stateful logic
```

After Hooks:

```text
Function Component
→ UI
→ State
→ Effects
→ Refs
→ Context
→ Reusable stateful logic
```

---

# 4. Hooks Are Not Just useState

A common misconception is:

> **Hooks = useState**

No.

`useState` is only one Hook.

Different Hooks provide different capabilities:

```text
useState
→ State

useEffect
→ Synchronization with external systems

useRef
→ Persistent mutable reference

useMemo
→ Memoized calculation

useCallback
→ Memoized function identity

useContext
→ Read context

useReducer
→ State transition logic

useTransition
→ Mark updates as non-urgent

useDeferredValue
→ Defer a value
```

The larger concept is:

> **Hooks provide access to React capabilities from function components.**

---

# 5. The Bigger Problem: Reusing Stateful Logic

Imagine:

```text
ChatRoom
Profile
```

Both need:

```text
Subscribe
 ↓
Listen
 ↓
React to updates
 ↓
Cleanup
```

Without reusable logic, you might duplicate the implementation.

Hooks allow the behavior to be extracted:

```jsx
function useChat(roomId) {
  // connection logic
}
```

Then:

```jsx
function ChatRoom({ roomId }) {
  const chat = useChat(roomId);

  return <ChatUI chat={chat} />;
}
```

Another component can use the same logic:

```jsx
function Sidebar({ roomId }) {
  const chat = useChat(roomId);

  return <ChatStatus chat={chat} />;
}
```

The behavior is reusable.

---

# 6. Important Distinction: Logic vs State

This is a major interview point.

Suppose:

```jsx
function ComponentA() {
  const [count] = useState(0);
}

function ComponentB() {
  const [count] = useState(0);
}
```

These components do **not** share the same state.

Conceptually:

```text
Component A
 ↓
Hook state A

Component B
 ↓
Hook state B
```

Even when both components use the same custom Hook:

```jsx
function ComponentA() {
  const value = useSomething();
}

function ComponentB() {
  const value = useSomething();
}
```

the Hook's **logic is shared**, but each component gets its own Hook state.

Therefore:

> **Custom Hooks reuse stateful logic, not the state itself.**

---

# 7. Custom Hooks

A custom Hook can contain reusable stateful behavior.

Example:

```jsx
function useOnlineStatus() {
  // state
  // effect
  // event subscription

  return isOnline;
}
```

Then:

```jsx
function Profile() {
  const isOnline = useOnlineStatus();

  return (
    <div>
      {isOnline ? "Online" : "Offline"}
    </div>
  );
}
```

And:

```jsx
function Chat() {
  const isOnline = useOnlineStatus();

  return (
    <span>
      {isOnline ? "●" : "○"}
    </span>
  );
}
```

Both components reuse the same logic.

But:

```text
Profile
 ↓
its own Hook state

Chat
 ↓
its own Hook state
```

---

# 8. Hooks Help Prevent Logic Scattering

Consider:

```jsx
class ChatRoom extends React.Component {
  componentDidMount() {
    connect();
    subscribe();
  }

  componentDidUpdate() {
    reconnect();
    updateSubscription();
  }

  componentWillUnmount() {
    disconnect();
    unsubscribe();
  }

  render() {
    // UI
  }
}
```

The behavior for one feature is spread across:

```text
componentDidMount
componentDidUpdate
componentWillUnmount
```

You have to mentally reconstruct the lifecycle to understand the connection behavior.

With Hooks, related behavior can often live together:

```jsx
useEffect(() => {
  const connection = connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

Setup and cleanup are colocated.

---

# 9. Hooks Encourage Feature-Based Organization

Instead of organizing code around lifecycle phases:

```text
Mount
Update
Unmount
```

you can organize it around behavior:

```text
Chat connection
 ↓
useEffect

Window size
 ↓
useWindowSize

Authentication
 ↓
useAuth

Online status
 ↓
useOnlineStatus
```

This changes the way you reason about components.

Instead of asking:

> Which lifecycle method should contain this?

you can ask:

> What behavior does this component need?

---

# 10. Hooks and Closures

Hooks work with normal JavaScript closure behavior.

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(count);
  }, [count]);

  return <button>{count}</button>;
}
```

The effect callback closes over `count` from that render.

Mental model:

```text
Render
 ↓
Lexical Environment
 ↓
Closure
```

Hooks do not eliminate JavaScript closure behavior.

They work with it.

This is why understanding:

```text
Scope
Lexical Environment
Closures
State Snapshots
```

is useful for understanding Hooks.

---

# 11. Every Render Has Its Own Snapshot

Suppose:

```text
Render #1
count = 0
```

Then:

```text
Render #2
count = 1
```

The first render's callbacks do not magically change from:

```text
count = 0
```

to:

```text
count = 1
```

Instead:

```text
Render #1
 ↓
Callback₁
 ↓
captures snapshot #1

Render #2
 ↓
Callback₂
 ↓
captures snapshot #2
```

This is why stale closures are possible.

Hooks do not change this JavaScript rule.

---

# 12. Hooks Are Special Functions

Hooks look like ordinary JavaScript function calls:

```jsx
useState(...)
useEffect(...)
useRef(...)
```

But React treats Hook calls specially.

React needs to associate:

```text
This useState
      ↓
with
      ↓
This component's state
```

and preserve that state across renders.

For example:

```jsx
function App() {
  const [count, setCount] = useState(0);

  const ref = useRef(null);

  useEffect(() => {
    // ...
  });

  return <div />;
}
```

Conceptually:

```text
Hook #1
→ useState

Hook #2
→ useRef

Hook #3
→ useEffect
```

On the next render, React expects the same ordering:

```text
Hook #1
→ useState

Hook #2
→ useRef

Hook #3
→ useEffect
```

This is why Hooks have strict rules.

Chapter 32 will explain those rules in detail.

---

# 13. Why Conditional Hooks Are Dangerous

Consider:

```jsx
function App({ loggedIn }) {
  if (loggedIn) {
    useState(0);
  }

  useEffect(() => {
    // ...
  });
}
```

When:

```text
loggedIn = true
```

the sequence could be:

```text
Hook #1 → useState
Hook #2 → useEffect
```

But when:

```text
loggedIn = false
```

the sequence becomes:

```text
Hook #1 → useEffect
```

The Hook ordering has changed.

React can no longer reliably associate each Hook call with the state it previously represented.

Therefore:

> **Hooks must be called in a consistent order across renders.**

The exact Rules of Hooks are covered in Chapter 32.

---

# 14. Hooks and Component Identity

From Handbook 2:

```text
Component Identity
       ↓
Fiber
       ↓
State
```

Hooks fit into the same model:

```text
Component identity
       ↓
Fiber
       ↓
Hook state
```

If the component remains the same:

```jsx
<Counter />
```

React can preserve its Hook state.

If the identity changes:

```jsx
<Counter />
```

becomes:

```jsx
<AdminCounter />
```

the old component is removed and the new component gets fresh state.

This connects Hooks directly to:

```text
Identity
Keys
Fiber
State preservation
Remounting
```

---

# 15. Hooks Don't Replace State Management

Another misconception is:

> **"Now that React has Hooks, we don't need Redux or other state-management solutions."**

Not necessarily.

Hooks provide access to React capabilities.

Applications may still need:

```text
Context
Redux
Zustand
External stores
Server-state libraries
Caching libraries
```

Hooks can be used with these systems.

For example:

```jsx
const user = useSelector(state => state.user);
```

Here, a Hook provides the interface to a state-management system.

---

# 16. Local State vs Shared State

Hooks make local state straightforward:

```jsx
const [isOpen, setIsOpen] = useState(false);
```

But if several parts of an application need the same state:

```text
Header
Sidebar
Dashboard
Modal
```

you need to think about architecture:

```text
Lift state up
Context
External state manager
Server-state library
```

Therefore:

> **Hooks make stateful behavior accessible and composable; they do not automatically solve application-wide state architecture.**

---

# 17. Hooks and Composition

Hooks can be composed.

Example:

```jsx
function useUserProfile(userId) {
  const user = useUser(userId);
  const permissions = usePermissions(userId);
  const isOnline = useOnlineStatus(userId);

  return {
    user,
    permissions,
    isOnline
  };
}
```

Then:

```jsx
function Profile({ userId }) {
  const profile = useUserProfile(userId);

  return <ProfileUI {...profile} />;
}
```

Conceptually:

```text
useUser
usePermissions
useOnlineStatus
       ↓
useUserProfile
       ↓
Profile
```

This is composition of stateful logic.

---

# 18. Why This Is Better Than Copy-Pasting Logic

Without reusable Hooks:

```text
Component A
 ├── API logic
 ├── subscription
 └── cleanup

Component B
 ├── API logic
 ├── subscription
 └── cleanup

Component C
 ├── API logic
 ├── subscription
 └── cleanup
```

With a custom Hook:

```text
             useSomething()
              /    |                 ↓     ↓     ↓
            A      B      C
```

The implementation can live in one place.

---

# 19. Hooks Do Not Mean "No Classes"

Hooks provide a modern way to build function components, but class components still matter when working with existing codebases.

You may encounter:

```text
Class components
Lifecycle methods
Legacy patterns
Error boundaries
```

You do not need to become a class-component expert.

But understanding the problems Hooks addressed explains why modern React looks the way it does.

---

# 20. The Three Big Problems Hooks Address

For interviews, remember these three.

## 1. Stateful Logic in Function Components

```text
Function component
+
State
+
Effects
+
Refs
+
Context
```

## 2. Reuse of Stateful Logic

```text
Custom Hook
 ↓
Reusable behavior
```

## 3. Better Organization of Related Logic

Instead of scattering behavior across:

```text
Mount
Update
Unmount
```

related behavior can often be colocated:

```text
Setup
+
Cleanup
```

around one feature.

---

# 21. What Hooks Do NOT Solve

Hooks do not automatically solve:

```text
Global state architecture
Server-state caching
Application architecture
Performance problems
Every form problem
Every asynchronous problem
```

Hooks are a mechanism for accessing and composing React behavior.

---

# 22. Core Mental Model

Don't think:

> **"Hooks are magic React functions."**

Think:

```text
Function Component
       ↓
React renders it
       ↓
Hooks are called in a stable order
       ↓
React associates Hook calls with component state
       ↓
Hook values persist across renders
```

And:

```text
Custom Hook
       ↓
Reusable stateful logic
       ↓
Used by multiple components
       ↓
Each component gets its own Hook state
```

---

# 23. Interview Question — Why Were Hooks Introduced?

### Strong Answer

> **Hooks were introduced to allow function components to use React features such as state, effects, refs, and context, while also making it easier to reuse and compose stateful logic. They also allow related behavior to be colocated instead of scattering it across lifecycle methods.**

---

# 24. Interview Question — What Problem Do Custom Hooks Solve?

### Strong Answer

> **Custom Hooks allow developers to extract and reuse stateful React logic across components. The logic is shared, but each component that calls the Hook gets its own Hook state.**

---

# 25. Interview Question — Do Two Components Calling the Same Custom Hook Share State?

**No.**

```jsx
function A() {
  const count = useCounter();
}

function B() {
  const count = useCounter();
}
```

Conceptually:

```text
A
 ↓
useCounter
 ↓
State A

B
 ↓
useCounter
 ↓
State B
```

The Hook implementation is shared.

The state is not automatically shared.

---

# 26. Interview Question — Why Can't Hooks Be Called Conditionally?

### Strong Answer

> **React relies on the stable order of Hook calls across renders to associate each Hook call with the correct internal state. Conditional or reordered Hook calls can change that order and cause React to associate state with the wrong Hook.**

---

# 27. Interview Question — Are Hooks Just Functions?

A nuanced answer:

> **Hooks are JavaScript functions from the developer's perspective, but React treats calls to Hooks specially. React relies on their stable call order and associates their state and other information with the component being rendered.**

---

# 28. Hooks + Everything We've Learned

### Handbook 1

```text
Components
 ↓
Props
 ↓
State
 ↓
Events
 ↓
Forms
 ↓
Composition
```

### Handbook 2

```text
State Update
 ↓
Render
 ↓
React Elements
 ↓
Reconciliation
 ↓
Fiber
 ↓
Commit
```

### Handbook 3

```text
Hooks
 ↓
Access React capabilities
 ↓
Reusable stateful logic
 ↓
Composition
```

Complete picture:

```text
User interaction
      ↓
Hook state update
      ↓
Scheduling
      ↓
Render
      ↓
Hooks execute in stable order
      ↓
New React Elements
      ↓
Reconciliation
      ↓
Commit
      ↓
DOM
```

---

# 29. Final Mental Model

> **Hooks are React's mechanism for allowing function components to use and compose React features such as state, effects, refs, context, and memoization. Their larger value is not simply replacing class syntax; Hooks make stateful logic reusable and allow related behavior to be colocated and composed. Each component gets its own Hook state, and React relies on stable Hook call order to associate each Hook call with the correct internal state.**

Remember:

```text
Hooks
 ↓
React capabilities in function components
```

```text
Custom Hooks
 ↓
Reusable stateful logic
```

```text
Stable Hook order
 ↓
Correct Hook ↔ state association
```

### One-sentence interview answer

> **"Hooks let function components use React's stateful and lifecycle-related capabilities while making stateful logic easier to reuse, compose, and colocate."**

---

# Quick Revision

If you have 30 seconds before an interview:

```text
Why Hooks?
    ↓
Function components needed state + React capabilities
    ↓
Classes had state/lifecycle behavior
    ↓
Stateful logic reuse was awkward
    ↓
Hooks brought these capabilities to functions
    ↓
Custom Hooks enable reusable stateful logic
    ↓
Each component gets its own Hook state
    ↓
Hooks must execute in stable order
```

---

# Chapter 31 — Complete

**Handbook 3: Chapter 31 — Why Hooks?**

Status: **COMPLETE**
