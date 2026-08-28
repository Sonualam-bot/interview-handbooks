# 24_React_Strict_Mode_Notes

## Core Definition

**Strict Mode** is a development-only React feature that enables additional checks designed to expose unsafe patterns and assumptions in React applications.

It does **not** mean that React creates two production UIs.

```jsx
<StrictMode>
  <App />
</StrictMode>
```

Core mental model:

```text
StrictMode
     ↓
Development-only checks
     ↓
Exercise rendering / Effects more aggressively
     ↓
Expose unsafe assumptions
     ↓
Developer fixes the underlying code
```

---

## 1. Why Strict Mode Exists

React's modern rendering architecture means developers should not assume that rendering logic executes exactly once.

Rendering work can conceptually be:

```text
Started
  ↓
Paused
  ↓
Resumed
```

or:

```text
Started
  ↓
Abandoned
```

Therefore rendering code should be safe to execute more than once.

Strict Mode helps expose code that violates this assumption.

---

## 2. Render Should Be Pure

A central React rule is:

> **Rendering should be pure.**

Conceptually:

```text
Same inputs
    ↓
Same result
```

Rendering should primarily calculate what the UI should look like. It should not casually mutate the external world.

### Bad

```jsx
function App() {
  localStorage.setItem("something", "value");

  return <h1>Hello</h1>;
}
```

The render itself is performing an external side effect.

---

## 3. Why Impure Rendering Is Dangerous

```jsx
function App() {
  sendAnalyticsEvent();

  return <h1>Hello</h1>;
}
```

If rendering is invoked more than once:

```text
Render
 ↓
sendAnalyticsEvent()

Render again
 ↓
sendAnalyticsEvent()
```

The analytics event could be sent twice.

The problem is not Strict Mode.

The underlying problem is:

> **A side effect was placed inside rendering logic.**

---

## 4. Strict Mode and Double Invocation

In development, Strict Mode may intentionally invoke rendering-related logic more than once.

You may see:

```text
App rendered
App rendered
```

The purpose is to expose:

- Impure rendering
- Unsafe assumptions
- Other rendering-related bugs

Do not interpret this as two separate visible application instances.

---

## 5. Strict Mode Is Development Only

Strict Mode's additional checks are development-only.

```text
Development
    ↓
Strict Mode checks
```

Production:

```text
Production
    ↓
Normal production behavior
```

Avoid saying:

> "Strict Mode makes React render everything twice."

Better:

> **Strict Mode enables additional development-only checks, and React may intentionally invoke rendering-related logic more than once in development to expose impure code and other unsafe patterns.**

---

## 6. Strict Mode Does Not Create Two Visible UIs

If you see:

```text
App rendered
App rendered
```

do not imagine:

```text
DOM #1
+
DOM #2
```

Strict Mode does not mean:

```jsx
<App />
<App />
```

are permanently mounted as two visible instances.

The additional invocation is part of development-time checking.

---

## 7. External Mutation During Render

```jsx
let renderCount = 0;

function App() {
  renderCount++;

  console.log("App rendered", renderCount);

  return <h1>Hello</h1>;
}
```

This is problematic because:

```text
render
   ↓
renderCount++
```

mutates external state.

If React invokes the component again:

```text
First render
    ↓
renderCount = 1

Second render
    ↓
renderCount = 2
```

The issue is:

> **Rendering mutated external state.**

---

## 8. Render vs Side Effect

### Rendering

```text
Calculate UI
```

### Side effect

```text
Perform something outside the UI calculation
```

Examples:

- API requests
- `localStorage` mutations
- Direct DOM manipulation
- Analytics events
- Timer creation
- Subscriptions
- Global variable mutation

These should not casually happen during rendering.

---

## 9. Where Side Effects Belong

React provides Effects for side effects that belong after rendering/commit.

```jsx
useEffect(() => {
  document.title = "Dashboard";
}, []);
```

Conceptually:

```text
Render
   ↓
Commit
   ↓
Effect
```

This separates:

```text
Calculate UI
```

from:

```text
Perform external side effect
```

---

## 10. Strict Mode and Effects

Strict Mode also helps reveal problems in Effect setup and cleanup.

```jsx
useEffect(() => {
  const connection = connect();

  return () => {
    connection.disconnect();
  };
}, []);
```

In development, Strict Mode can exercise the setup/cleanup behavior more aggressively.

Conceptually:

```text
Setup
 ↓
Cleanup
 ↓
Setup again
```

This can expose:

- Missing cleanup
- Duplicate subscriptions
- Unremoved event listeners
- Leaked connections

---

## 11. Event Listener Example

### Problematic

```jsx
useEffect(() => {
  window.addEventListener("resize", handleResize);
}, []);
```

There is no cleanup.

### Safer

```jsx
useEffect(() => {
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

The cleanup makes the Effect safe to set up and tear down.

---

## 12. Strict Mode as a Stress Test

Useful mental model:

```text
Normal development
       ↓
"Does my code work?"

Strict Mode
       ↓
"Does my code remain correct
 when React exercises it more aggressively?"
```

Strict Mode helps discover assumptions that may break under React's rendering architecture.

---

## 13. Strict Mode Does Not Fix Your Code

Strict Mode does not automatically make unsafe code safe.

Instead:

```text
Strict Mode
    ↓
Expose unsafe assumptions
    ↓
Developer identifies the problem
    ↓
Developer fixes the code
```

---

## 14. Strict Mode and Concurrent Rendering

This connects directly to Fiber and the modern rendering model.

Rendering work can be more flexible:

```text
Render
 ↓
Pause
 ↓
Resume
```

or:

```text
Render
 ↓
Abandon
```

Therefore:

> **Rendering code should be resilient to being invoked more than once.**

Strict Mode helps developers discover code that violates this assumption.

---

## 15. Strict Mode and State

Strict Mode's development checks do not mean React creates two independent visible state instances.

Do not think:

```text
Counter A
Counter B
```

Instead think:

```text
One application
+
Development-time checks
```

---

## 16. Strict Mode and Component Identity

Strict Mode does not mean:

```text
Old component permanently destroyed
        ↓
New component permanently created
```

simply because rendering-related logic was invoked again.

The important focus is:

- Pure rendering
- Safe rendering assumptions
- Correct Effect setup
- Correct Effect cleanup

---

## 17. What Strict Mode Is Trying to Teach

### 1. Keep rendering pure

```text
Render
 ↓
Calculate UI
```

not:

```text
Render
 ↓
Modify external world
```

### 2. Make Effects clean up correctly

```text
Setup
 ↓
Cleanup
```

should be safe to repeat.

### 3. Don't assume rendering happens exactly once

Your component should remain correct if React needs to render it more than once.

---

## 18. Strict Mode vs Fiber vs Scheduler

These concepts are related but are **not sequential lifecycle phases**.

### Fiber

> Architecture for representing and managing rendering work.

### Scheduler

> Helps coordinate when rendering work should happen and how urgent that work is.

### Strict Mode

> Development tool that exposes unsafe assumptions.

Think:

```text
                    React Architecture
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
           Fiber       Scheduler     Strict Mode
             │             │             │
      manages work    coordinates     exposes
                      work timing     unsafe code
```

Do **not** memorize:

```text
StrictMode → Fiber → Scheduler → Rendering
```

as a lifecycle pipeline.

---

## Interview Answer: What Is Strict Mode?

> **Strict Mode is a development-only feature that enables additional checks designed to expose unsafe patterns in React applications. It can intentionally invoke rendering-related logic more than once and exercise Effect setup and cleanup so that impure rendering, missing cleanup, and other bugs become visible during development. These checks don't represent two production UI instances.**

---

## Interview Answer: Why Does My Component Render Twice?

> **If the application is using Strict Mode, React may intentionally invoke rendering-related logic more than once in development to help detect impure rendering and other bugs. This is development-only behavior and should not be interpreted as React rendering two separate copies of the component in production.**

---

## Interview Answer: Why Does Strict Mode Exercise Effects?

> **Strict Mode exercises Effect setup and cleanup more aggressively in development so that missing cleanup, duplicate subscriptions, leaked connections, and similar bugs can be detected early.**

---

## Strict Mode + Rendering Architecture

```text
Fiber
 ↓
Rendering work can be managed flexibly
 ↓
Rendering may need to be restarted/re-executed
 ↓
Render should be pure
 ↓
Strict Mode helps expose violations
```

This is why Strict Mode naturally follows chapters on Fiber, Scheduler, and the rendering lifecycle.

---

## Common Mistakes

### ❌ "Strict Mode renders everything twice."

Better:

> Strict Mode enables additional development-only checks, and React may intentionally invoke rendering-related logic more than once.

### ❌ "Strict Mode creates two component instances."

Better:

> The additional behavior is for development-time checking, not two permanently visible application instances.

### ❌ "Strict Mode causes a performance problem in production."

Better:

> The additional Strict Mode checks are development-only.

### ❌ "The problem with `renderCount++` is that React rendered twice."

Better:

> The problem is that rendering mutated external state, making the render impure.

### ❌ "Effects don't need cleanup because Strict Mode will clean them."

Better:

> Strict Mode exposes missing cleanup; the developer must implement the cleanup.

### ❌ "Strict Mode → Fiber → Scheduler → Render."

Better:

> Fiber, Scheduler, and Strict Mode are related parts of the React development/rendering model, but they are not sequential lifecycle phases.

---

## Flashcards

**Q:** What is Strict Mode?

**A:** A development-only feature that enables additional checks to expose unsafe patterns and assumptions.

**Q:** Why might a component appear to render twice in development?

**A:** Strict Mode may intentionally invoke rendering-related logic more than once to expose impure rendering and other bugs.

**Q:** Does Strict Mode create two visible application instances?

**A:** No.

**Q:** Does Strict Mode change production behavior in the same way?

**A:** No. Its additional checks are development-only.

**Q:** Why should rendering be pure?

**A:** React's rendering work may be executed more than once, paused, resumed, or abandoned, so rendering should not depend on one-time execution or mutate external state.

**Q:** Why is `renderCount++` problematic inside rendering?

**A:** It mutates external state during rendering.

**Q:** What problems can Strict Mode expose in Effects?

**A:** Missing cleanup, duplicate subscriptions, unremoved event listeners, and leaked connections.

**Q:** What is the purpose of Effect cleanup?

**A:** To undo external setup performed by the Effect so setup/teardown can safely be repeated.

**Q:** What is Fiber?

**A:** React's architecture/data structure for representing and managing rendering work.

**Q:** What is the Scheduler?

**A:** It helps coordinate when rendering work should happen and its relative urgency.

**Q:** Is Strict Mode a rendering phase?

**A:** No. It is a development-time checking mechanism.

---

## 30-Second Revision

```text
StrictMode
     ↓
Development-only checks
     ↓
Exercise rendering / Effects more aggressively
     ↓
Expose unsafe assumptions
```

### Rendering

```text
Render
 ↓
Should be pure
 ↓
Calculate UI
```

### Effects

```text
Setup
 ↓
Cleanup
 ↓
Safe to repeat
```

### Core reason

```text
React rendering work
        ↓
May be invoked more than once
        ↓
Render must be resilient
        ↓
Strict Mode helps expose violations
```

### Don't confuse

```text
Fiber
→ Architecture for rendering work

Scheduler
→ Coordinates rendering work

Strict Mode
→ Development checks
```

---

## Final Mental Model

> **Strict Mode is not about making React "strict" in production. It is a development tool that intentionally makes certain rendering and Effect assumptions more demanding so that unsafe code is exposed early.**

The most important rule:

> **Do not make rendering depend on running exactly once. Keep render pure and make Effect setup/cleanup safe to repeat.**
