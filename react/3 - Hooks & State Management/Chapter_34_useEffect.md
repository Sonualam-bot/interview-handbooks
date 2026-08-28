# Chapter 34 — `useEffect`

## 1. Core Mental Model

`useEffect` is primarily a **synchronization mechanism**.

It lets a React component synchronize with something **outside React**, such as:

- WebSockets
- timers
- browser APIs
- event listeners
- subscriptions
- third-party libraries
- external connections

Think:

```text
Render
  ↓
React calculates UI
  ↓
Commit
  ↓
Effect
  ↓
External system synchronized
```

> **`useEffect` synchronizes the component with an external system after React has committed the UI.**

---

## 2. Why Not Put Side Effects During Render?

React rendering should be pure.

React may:

```text
render
pause
resume
restart
abandon
```

Putting side effects directly in render can therefore cause work to happen multiple times or for work that never gets committed.

Bad:

```jsx
function ChatRoom() {
  connectToChat();

  return <div>Chat</div>;
}
```

Better:

```jsx
useEffect(() => {
  const connection = connectToChat();

  return () => {
    connection.disconnect();
  };
}, []);
```

---

## 3. Basic Syntax

```jsx
useEffect(() => {
  // synchronization / side effect
});
```

With dependencies:

```jsx
useEffect(() => {
  // synchronization
}, [count]);
```

The dependency array describes the **reactive values that the effect depends on**.

---

## 4. The Three Common Forms

### No dependency array

```jsx
useEffect(() => {
  // ...
});
```

The effect is eligible to run after every committed render.

### Empty dependency array

```jsx
useEffect(() => {
  // ...
}, []);
```

There are no reactive dependencies declared.

It is associated with the component's mount lifecycle, but don't interpret this as an absolute "runs exactly once forever" guarantee. Development Strict Mode can intentionally perform extra setup/cleanup cycles.

### Dependency array

```jsx
useEffect(() => {
  // ...
}, [count]);
```

The effect re-runs when `count` changes.

---

## 5. Dependencies

```jsx
useEffect(() => {
  connect(roomId);
}, [roomId]);
```

If `roomId` changes:

```text
roomId changes
     ↓
cleanup old effect
     ↓
setup new effect
```

Think:

> **What reactive values affect this synchronization?**

Those are the values the effect needs to account for.

---

## 6. Cleanup

An effect can return a cleanup function:

```jsx
useEffect(() => {
  const connection = createConnection(roomId);

  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

Lifecycle:

```text
Setup
  ↓
External system synchronized
  ↓
Dependency changes
  ↓
Cleanup old synchronization
  ↓
Setup new synchronization
```

When the component is removed:

```text
Cleanup
```

---

## 7. Cleanup Is Not Only for Unmount

Common interview trap:

```jsx
useEffect(() => {
  subscribe(roomId);

  return () => {
    unsubscribe(roomId);
  };
}, [roomId]);
```

Initially:

```text
roomId = A
↓
setup(A)
```

Later:

```text
roomId = B
```

React replaces the old synchronization:

```text
cleanup(A)
    ↓
setup(B)
```

When the component leaves:

```text
cleanup(B)
```

Therefore:

> **Cleanup happens before an effect is replaced and when the component is removed.**

---

## 8. Common Cleanup Patterns

### Event listener

```jsx
useEffect(() => {
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

### Timer

```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

### Subscription

```jsx
useEffect(() => {
  const unsubscribe = subscribe(handler);

  return unsubscribe;
}, []);
```

### WebSocket

```jsx
useEffect(() => {
  socket.connect();

  return () => {
    socket.disconnect();
  };
}, []);
```

General rule:

> **Whatever the effect sets up, the cleanup should undo.**

---

## 9. Effects and Closures

Effects are closely connected to JavaScript closures.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(count);
  }, [count]);

  return ...;
}
```

The effect callback closes over the `count` from the render that created it.

Conceptually:

```text
Render #1
count = 0
 ↓
effect captures 0

Render #2
count = 1
 ↓
new effect captures 1
```

---

## 10. Stale Closures

Example:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(id);
}, []);
```

The effect was created during a particular render, so its callback can retain the value captured by that render.

If the effect doesn't synchronize when `count` changes, the callback can observe an outdated value.

This is a **stale closure** problem.

If the effect genuinely needs to react to `count`:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(id);
}, [count]);
```

Now:

```text
count changes
 ↓
cleanup old interval
 ↓
create new interval
 ↓
new closure sees new count
```

---

## 11. Effects vs Derived Values

Do not use an effect to calculate something that can simply be calculated during render.

Bad:

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

Better:

```jsx
const fullName = `${firstName} ${lastName}`;
```

Why?

```text
firstName + lastName
        ↓
derived value
```

This is not external synchronization.

---

## 12. Effects vs Event Handlers

Ask:

> **What caused this operation?**

If the user clicks:

```jsx
function handleBuy() {
  purchaseProduct();
}
```

the purchase is event-driven.

Don't unnecessarily turn it into:

```jsx
useEffect(() => {
  if (bought) {
    purchaseProduct();
  }
}, [bought]);
```

The click itself caused the operation.

Use an effect when the component needs to synchronize an external system with its current props/state.

---

## 13. Strong Use Case: WebSockets

```jsx
useEffect(() => {
  const socket = new WebSocket(url);

  socket.onmessage = event => {
    setData(JSON.parse(event.data));
  };

  return () => {
    socket.close();
  };
}, [url]);
```

Lifecycle:

```text
Component
   ↓
Effect
   ↓
Create socket
   ↓
Messages arrive
   ↓
setData(...)
   ↓
Component re-renders
```

If `url` changes:

```text
cleanup old socket
      ↓
create new socket
```

If the component leaves:

```text
close socket
```

---

## 14. Strict Mode and Effects

In development, Strict Mode can intentionally perform:

```text
setup
 ↓
cleanup
 ↓
setup
```

This exposes effects that aren't correctly implemented.

Suspicious:

```jsx
useEffect(() => {
  socket.connect();
}, []);
```

Better:

```jsx
useEffect(() => {
  socket.connect();

  return () => {
    socket.disconnect();
  };
}, []);
```

Mental model:

> **Strict Mode checks whether setup/cleanup logic is resilient to being started and stopped.**

---

## 15. Effect Loops

Common interview trap:

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Cycle:

```text
count changes
 ↓
effect runs
 ↓
setCount
 ↓
count changes
 ↓
effect runs
 ↓
setCount
 ↓
...
```

This can create an infinite update loop.

Don't use effects simply to react to state changes unless there is a genuine synchronization reason.

---

## 16. Missing Dependencies

Suppose:

```jsx
useEffect(() => {
  connect(roomId);
}, []);
```

but `roomId` can change.

Then:

```text
Initial:
roomId = A
 ↓
connect(A)

Later:
roomId = B
 ↓
effect doesn't rerun
```

The external system can now be out of sync with the component.

If `roomId` genuinely affects the synchronization:

```jsx
useEffect(() => {
  connect(roomId);
}, [roomId]);
```

---

## 17. Socket + State Example

Suppose a socket sends:

```text
0 → 0 → 1 → 3 → 3 → 3 → 4
```

and:

```jsx
socket.on("count", value => {
  setCount(value);
});
```

If `count` starts at `0`:

```text
0 → 0
→ no meaningful state change

0 → 1
→ render

1 → 3
→ render

3 → 3
→ no meaningful state change

3 → 3
→ no meaningful state change

3 → 3
→ no meaningful state change

3 → 4
→ render
```

Therefore:

```text
Initial render = 1
Socket-caused state-change renders = 3

Total = 4 meaningful renders
```

If:

```jsx
useEffect(() => {
  console.log("effect");
}, [count]);
```

the effect executes:

```text
Initial count = 0 → effect
0 → 0 → no effect
0 → 1 → effect
1 → 3 → effect
3 → 3 → no effect
3 → 3 → no effect
3 → 3 → no effect
3 → 4 → effect
```

So:

```text
4 effect executions total
```

including the initial one.

---

## 18. Interview Questions

### Q1. What is `useEffect`?

> `useEffect` lets a component synchronize with an external system after React commits the UI. It can establish a synchronization and optionally return cleanup logic.

### Q2. Why shouldn't side effects happen during render?

> React rendering should be pure because rendering can be repeated, interrupted, restarted, or abandoned.

### Q3. When does cleanup run?

> Before an effect is replaced because its dependencies changed, and when the component is removed.

### Q4. Is cleanup only for unmount?

> No. It also runs before the previous effect is replaced.

### Q5. What causes an effect with `[count]` to run again?

> A change in `count` between the relevant renders.

### Q6. What is a stale closure?

> An effect or callback can retain values from the render that created its closure, causing it to observe outdated state or props.

### Q7. Why can an effect cause an infinite loop?

> If the effect updates state that is itself one of the effect's dependencies, the update can cause the effect to run again indefinitely.

---

## 19. Effect Decision Framework

Before writing:

```jsx
useEffect(...)
```

ask:

### 1. Am I synchronizing with something outside React?

If no, you may not need an effect.

### 2. Is this simply a value I can calculate during render?

If yes:

```text
derive it
```

### 3. Was this caused directly by a user event?

If yes:

```text
event handler
```

may be the correct place.

### 4. Does an external system need to be started, stopped, subscribed to, or updated when props/state change?

If yes:

```text
useEffect
```

is a strong candidate.

---

## 20. Final Mental Model

Don't think:

> "`useEffect` means run this after render."

Think:

> **"`useEffect` synchronizes the component with an external system after commit."**

Core lifecycle:

```text
Render
 ↓
Commit
 ↓
Effect setup
 ↓
External system synchronized
```

When dependencies change:

```text
New render
 ↓
Commit
 ↓
Cleanup old synchronization
 ↓
Setup new synchronization
```

When the component leaves:

```text
Cleanup
```

---

# Quick Revision

```text
useEffect
→ synchronize with external systems
```

```text
Render
→ pure UI calculation
```

```text
Commit
→ DOM changes
```

```text
Effect
→ external synchronization
```

```text
Dependency changes
→ cleanup old
→ setup new
```

```text
Unmount
→ cleanup
```

```text
Strict Mode
→ can expose missing cleanup
```

```text
Derived value
→ calculate during render
```

```text
User event
→ event handler
```

```text
Socket / subscription / timer / external API
→ strong use case for effect
```

---

# Chapter 34 — Core Takeaway

> **`useEffect` is a synchronization mechanism, not a generic "run some code after render" mechanism. Use it when React needs to coordinate with something outside React. The dependency array describes the reactive values that synchronization depends on, and cleanup undoes the previous synchronization when dependencies change or the component is removed.**

**Chapter 34 — COMPLETE**
