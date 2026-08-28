# Chapter 30 — React Rendering Pipeline: End-to-End Review

## Chapter Overview

This is the final synthesis chapter of **Handbook 2 — Rendering & React Internals**.

The goal is to connect the concepts from the entire handbook into one end-to-end mental model.

The central question is:

> **What exactly happens inside React when something changes and the UI needs to update?**

The complete mental model:

```text
Something changes
      ↓
React receives an update
      ↓
Update gets prioritized / scheduled
      ↓
React performs rendering work
      ↓
Component functions execute
      ↓
New React Element tree
      ↓
Reconciliation
      ↓
Determine required changes
      ↓
Commit
      ↓
DOM / host environment updated
      ↓
Browser renders
```

With concurrent rendering:

```text
Render work
     ↓
May continue
     ↓
May pause
     ↓
May resume
     ↓
May restart
     ↓
May be abandoned
```

**Only committed work becomes the visible UI.**

---

# 1. Start With a User Interaction

Consider:

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

The user clicks the button.

Conceptually:

```text
User
 ↓
Click event
 ↓
React event handling
 ↓
Event handler
 ↓
setCount(...)
```

The setter requests a state update.

---

# 2. State Update

Suppose the current render contains:

```text
count = 0
```

Then:

```jsx
setCount(count + 1);
```

uses the snapshot from that render:

```jsx
setCount(1);
```

The current render does not suddenly become `count = 1`.

Instead:

```text
Update requested
 ↓
Future render
 ↓
count = 1
```

This is the **state snapshot model**.

---

# 3. Batching

Suppose an event performs:

```jsx
setCount(count + 1);
setName("Sonu");
setLoading(false);
```

React can batch compatible updates:

```text
Update A
Update B
Update C
   ↓
Batch
   ↓
Rendering work
```

The purpose is to avoid unnecessary repeated rendering and commit cycles.

> **Batching is about grouping updates.**

It is different from scheduling.

---

# 4. Scheduling

React needs to determine how pending work should be processed.

Conceptually:

```text
State update
      ↓
Priority
      ↓
Scheduling
```

React can prioritize different categories of work.

For example, an urgent interaction may need to be handled before less urgent rendering work.

---

# 5. Lanes

Lanes are an internal React mechanism for tracking pending work and its priority/category.

Conceptually:

```text
Urgent update
      ↓
Higher-priority lane

Transition update
      ↓
Lower-priority lane
```

For interviews:

> **Lanes help React track and prioritize pending rendering work.**

You generally do not need to memorize React's internal lane constants.

---

# 6. Fiber

React Fiber represents rendering work as units that can be processed and managed individually.

Imagine:

```text
       App
        │
 ┌──────┼──────┐
Header  Main  Footer
         │
       List
      / | \
     A  B  C
```

Fiber allows React to represent and manage these pieces of rendering work and their relationships.

Remember:

```text
Fiber
→ Represents / manages rendering work
```

Not:

```text
Fiber
→ A JavaScript thread
```

---

# 7. Current Tree vs Work-In-Progress Tree

React has the conceptual distinction between:

```text
Current Tree
```

and:

```text
Work-In-Progress Tree
```

### Current Tree

Represents the currently committed UI:

```text
Current
 ↓
Visible UI
```

### Work-In-Progress Tree

Represents rendering work React is currently calculating:

```text
WIP
 ↓
Potential next UI
```

Conceptually:

```text
CURRENT TREE
     │
     │ render
     ↓
WIP TREE
```

---

# 8. Why the WIP Tree Matters

Imagine React is rendering:

```text
Dashboard
 ├── Header
 ├── Sidebar
 ├── Charts
 ├── Table
 └── Footer
```

React can work on the WIP tree while the user continues seeing the current committed tree.

If React pauses the render:

```text
WIP
 ↓
Pause
```

the user does not see a half-finished UI.

The current committed UI remains visible.

---

# 9. Concurrent Rendering

Concurrent rendering makes rendering work interruptible.

React may:

```text
Start rendering
      ↓
Pause
      ↓
Handle more urgent work
      ↓
Resume
```

Or:

```text
Start rendering
      ↓
New information arrives
      ↓
Old work is no longer useful
      ↓
Abandon it
```

Important:

> **Concurrent rendering does not mean React is running multiple JavaScript threads executing components simultaneously.**

It means React can manage rendering work in an interruptible way.

---

# 10. Render Phase

React executes component functions to calculate the next UI.

Example:

```jsx
function Counter() {
  const [count] = useState(1);

  return (
    <button>
      {count}
    </button>
  );
}
```

The component execution produces a React Element describing the UI:

```js
{
  type: "button",
  props: {
    children: 1
  }
}
```

This work belongs to the **render phase**.

---

# 11. Render Is a Calculation

Think:

```text
Props
+
State
+
Context
 ↓
Component function
 ↓
React Elements
```

The component is calculating:

> **What should the UI look like?**

That is why render should be pure.

---

# 12. Why Render Must Be Pure

React may:

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

Therefore, render should not contain irreversible external side effects.

Bad:

```jsx
function App() {
  sendAnalyticsEvent();

  return <Dashboard />;
}
```

If React renders and then abandons the work:

```text
Render
 ↓
Analytics event
 ↓
Render abandoned
```

the external operation already happened even though the UI was never committed.

---

# 13. Strict Mode

Strict Mode in development intentionally stresses rendering and effect behavior to expose unsafe assumptions.

It can make some behavior appear to run twice.

This helps identify:

```text
Impure rendering
Missing cleanup
Unsafe side effects
```

Remember:

```text
Strict Mode
→ Development tool for finding bugs
```

It does not mean production React randomly renders everything twice.

---

# 14. New React Element Tree

After executing components, React has a new rendered result.

Conceptually:

```text
Previous React Element tree
        +
New React Element tree
        ↓
Reconciliation
```

React now needs to determine:

> **What changed?**

---

# 15. Reconciliation

Reconciliation is the process React uses to determine how the newly rendered element tree relates to the previous tree and what work is required to update the UI.

React considers information such as:

- Element type
- Component identity
- Keys
- Position / structure
- Props
- Children

Conceptually:

```text
Previous tree
      +
New tree
      ↓
Reconciliation
      ↓
Required work
```

---

# 16. Diffing

Diffing is the comparison process used to determine differences between previous and new rendered results.

Example:

```text
Old:
<button>0</button>

New:
<button>1</button>
```

React sees:

```text
button
  ↓
button
```

Same type.

Only the content changed.

Therefore React can preserve the existing button and update the necessary content.

Conceptually:

```text
Keep existing button
        ↓
Update text
```

---

# 17. Keys and Identity

Consider:

```jsx
todos.map(todo => (
  <Todo key={todo.id} todo={todo} />
))
```

Keys help React identify corresponding elements across renders.

Example:

```text
Old:

A key=1
B key=2
C key=3
```

New:

```text
C key=3
A key=1
B key=2
```

React can recognize:

```text
C → same identity
A → same identity
B → same identity
```

even though positions changed.

---

# 18. Why Index Keys Can Cause Bugs

Suppose:

```text
A → index 0
B → index 1
C → index 2
```

Insert X at the beginning:

```text
X → index 0
A → index 1
B → index 2
C → index 3
```

Index-based identity can cause:

```text
old index 0 → new index 0
```

even though:

```text
old item = A
new item = X
```

This can cause state to become associated with the wrong item.

Therefore:

> **Keys should represent stable identity, not merely position.**

---

# 19. Component Identity

React can preserve state when component identity remains compatible.

Example:

```text
<Profile />
```

becomes:

```text
<Profile />
```

with compatible identity.

State can be preserved.

But:

```text
<Profile />
```

becomes:

```text
<AdminProfile />
```

The component type changed.

React treats this as a different component identity.

Conceptually:

```text
Old Profile
 ↓
Removed

New AdminProfile
 ↓
Mounted
```

The old state is not carried over as if it were the same component.

---

# 20. Keys Can Intentionally Reset State

Consider:

```jsx
<UserForm key={userId} />
```

If:

```text
userId = A
```

becomes:

```text
userId = B
```

React sees a new identity.

Conceptually:

```text
Old UserForm
 ↓
Unmount
 ↓
New UserForm
 ↓
Fresh state
```

This is a useful deliberate state-reset technique.

---

# 21. Render Does Not Mean DOM Mutation

A component can render:

```text
Render
 ↓
New React Elements
```

without React necessarily changing the DOM.

Example:

```jsx
return <div>Hello</div>;
```

If the next render produces the same result:

```jsx
return <div>Hello</div>;
```

React can determine:

```text
No meaningful host change
```

Therefore:

> **Render ≠ DOM update**

---

# 22. Commit Phase

Once React has determined the changes that need to be applied, it enters the commit phase.

Conceptually:

```text
Render
 ↓
Reconciliation
 ↓
Commit
```

The commit phase applies the selected changes to the host environment.

For React DOM:

```text
Host environment
 ↓
Browser DOM
```

---

# 23. Commit vs Render

### Render

Rendering work can potentially:

```text
Pause
Resume
Restart
Abandon
```

### Commit

The completed result is applied during commit.

The critical distinction:

> **Render is interruptible; commit is the point where the finished result is applied.**

This is why users don't see a half-finished React tree.

---

# 24. DOM Mutation

Suppose:

```text
Old:
<button>0</button>

New:
<button>1</button>
```

React can preserve the existing DOM node and update its text.

Conceptually:

```text
Existing DOM node
       ↓
Update text
       ↓
<button>1</button>
```

React does not need to destroy and recreate the entire button.

---

# 25. Browser Rendering

React updates the DOM.

Then the browser handles its own rendering pipeline:

```text
React
 ↓
DOM mutation
 ↓
Browser
 ↓
Layout / Paint / Compositing
 ↓
Pixels
```

Remember:

> **React does not paint the screen. The browser does.**

---

# 26. Effects

After the committed UI is established, React can run passive effects.

Example:

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

Useful mental model:

```text
Render
 ↓
Commit
 ↓
DOM
 ↓
Effect
```

Effects synchronize React with external systems.

---

# 27. Why Effects Aren't in Render

Bad:

```jsx
function App() {
  sendAnalyticsEvent();

  return <Dashboard />;
}
```

If rendering is restarted:

```text
Render
 ↓
Analytics
 ↓
Pause
 ↓
Restart
 ↓
Analytics
```

the external operation could happen multiple times.

Instead:

```jsx
useEffect(() => {
  sendAnalyticsEvent();
}, []);
```

separates external synchronization from pure rendering.

---

# 28. Dependency Arrays

Consider:

```jsx
useEffect(() => {
  console.log(count);
}, [count]);
```

The effect depends on:

```text
count
```

When the relevant dependency changes, the effect needs to be re-synchronized.

Conceptually:

```text
Dependency changes
       ↓
Cleanup previous effect when necessary
       ↓
New effect setup
```

This connects directly to stale closures.

---

# 29. Stale Closures

Every render has its own state snapshot.

```text
Render #1
count = 0
 ↓
Effect₁ captures 0
```

Then:

```text
Render #2
count = 1
 ↓
Effect₂ captures 1
```

The old callback does not magically receive the new value.

Therefore:

> **A callback can hold a stale value from an older render.**

This connects React's state snapshot model directly to JavaScript closures.

---

# 30. Complete Example

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Effect:", count);

    return () => {
      console.log("Cleanup:", count);
    };
  }, [count]);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

Initial render:

```text
count = 0
 ↓
Render
 ↓
Commit
 ↓
Effect setup
 ↓
Effect sees 0
```

User clicks:

```text
setCount(1)
 ↓
Render
 ↓
New snapshot: count = 1
 ↓
Reconciliation
 ↓
Commit
 ↓
Cleanup previous effect
 ↓
New effect setup
 ↓
Effect sees 1
```

This example connects:

```text
State
+
Snapshot
+
Render
+
Reconciliation
+
Commit
+
Effects
+
Cleanup
```

---

# 31. Hydration

The server can produce HTML:

```html
<button>0</button>
```

The browser receives already-generated HTML.

React then needs to connect its component logic to that existing DOM.

This is **hydration**.

Conceptually:

```text
Server
 ↓
HTML
 ↓
Browser
 ↓
Existing DOM
 ↓
React hydration
 ↓
Interactive React application
```

Hydration is different from starting with an empty page and constructing the DOM entirely from scratch.

---

# 32. Why Hydration Exists

Without hydration:

```text
Server HTML
 ↓
Browser displays it
 ↓
React throws it away
 ↓
Build everything again
```

That would waste the server-rendered HTML.

Instead:

```text
Server HTML
 ↓
Browser can display it
 ↓
React hydrates
 ↓
Interactive application
```

---

# 33. Hydration Requires Consistency

Suppose the server produces:

```text
Hello
```

but the client initially expects:

```text
Goodbye
```

React can encounter a hydration mismatch.

Conceptually:

```text
Server result
    ↓
Hello

Client result
    ↓
Goodbye

Mismatch
```

Therefore, server and client need compatible initial output.

---

# 34. Server Components

Modern React can separate:

```text
Server Components
```

from:

```text
Client Components
```

Conceptually:

### Server Component

Can render on the server and does not need to ship its component logic to the browser in the same way client components do.

### Client Component

Needed for client-side interactivity such as:

```text
useState
useEffect
event handlers
browser APIs
```

The exact architecture depends on the framework/runtime, especially frameworks such as Next.js.

For this handbook, understand the conceptual boundary rather than framework-specific implementation details.

---

# 35. Streaming Rendering

Server rendering does not necessarily have to wait for the entire application before sending HTML.

Conceptually:

```text
Server
 ↓
Render part
 ↓
Send part
 ↓
Render more
 ↓
Send more
```

This is streaming rendering.

It can allow the browser to receive useful content progressively instead of waiting for the entire page to finish rendering on the server.

---

# 36. Full React Update Pipeline

Combine everything:

```text
                 USER INTERACTION
                       ↓
                    EVENT
                       ↓
                  setState()
                       ↓
                UPDATE CREATED
                       ↓
                  BATCHING
                       ↓
              PRIORITY / LANES
                       ↓
                  SCHEDULING
                       ↓
                     FIBER
                       ↓
             WORK-IN-PROGRESS TREE
                       ↓
                    RENDER
                       ↓
              COMPONENT EXECUTION
                       ↓
              REACT ELEMENT TREE
                       ↓
                RECONCILIATION
                       ↓
                    DIFFING
                       ↓
                REQUIRED CHANGES
                       ↓
                    COMMIT
                       ↓
                  DOM MUTATION
                       ↓
                    BROWSER
                       ↓
             LAYOUT / PAINT / COMPOSITE
                       ↓
                     PIXELS
                       ↓
                    EFFECTS
```

Surrounding the render phase:

```text
Concurrent Rendering
        ↓
Can pause
Can resume
Can restart
Can abandon
```

While:

```text
Commit
 ↓
Applies the finished result
```

---

# 37. The Five Most Important Distinctions

## 1. Render vs Commit

```text
Render
→ Calculate

Commit
→ Apply
```

## 2. React Element vs DOM

```text
React Element
→ Description of desired UI

DOM
→ Browser's representation of the UI
```

## 3. Re-render vs Remount

```text
Re-render
→ Same identity
→ State can be preserved

Remount
→ New identity
→ State resets
```

## 4. State Update vs DOM Update

```text
setState()
→ Requests / schedules rendering work

Not:
setState()
→ Immediately mutate DOM
```

## 5. Render vs Effect

```text
Render
→ Calculate UI

Effect
→ Synchronize external systems
```

---

# 38. Fiber vs Scheduler vs Lanes

| Concept | Mental Model |
|---|---|
| **Fiber** | Represents and manages rendering work |
| **Scheduler** | Coordinates when work should happen |
| **Lanes** | Represent priority/category of pending work |
| **Reconciliation** | Determines how new and previous trees relate |
| **Commit** | Applies the finished result |

Do not collapse these concepts into one.

---

# 39. Batching vs Scheduling

### Batching

> **Which updates can be processed together?**

```text
Update A
Update B
Update C
 ↓
Batch
```

### Scheduling

> **When/how should pending work be processed?**

```text
Pending work
 ↓
Priority
 ↓
Scheduling
```

They are related but different.

---

# 40. Concurrent Rendering vs Parallelism

Concurrent rendering means React can:

```text
Pause work
Resume work
Restart work
Abandon work
Prioritize work
```

It does not mean:

```text
Multiple JavaScript threads
running React components simultaneously
```

---

# 41. Interview Answer: How Does React Update the UI?

> **When a state or prop update occurs, React schedules rendering work. It may batch updates and prioritize them using its internal scheduling and lane mechanisms. During the render phase, React executes components to produce a new React Element tree. React then reconciles that result with the previous tree and determines the required changes. Once the result is ready, React commits those changes to the host environment, such as the DOM. The browser then performs its own rendering work to display the result. In concurrent rendering, the render phase can be interrupted or restarted, but only committed work becomes visible.**

---

# 42. Interview Answer: What Is Fiber?

> **Fiber is React's internal architecture for representing and managing rendering work as units that can be processed independently. It enables React to track relationships in the component tree and supports features such as scheduling and interruptible concurrent rendering.**

---

# 43. Interview Answer: What Is Reconciliation?

> **Reconciliation is the process React uses to determine how the newly rendered element tree relates to the previous tree and what work is required to update the UI. React uses element type, keys, structure, and other information to preserve identity where possible and determine necessary changes.**

---

# 44. Interview Answer: What Is Concurrent Rendering?

> **Concurrent rendering allows React to make rendering work interruptible. React can pause, resume, restart, or abandon rendering work when higher-priority updates arrive. The important distinction is that only committed work becomes visible to the user; the render phase can be interrupted, while the commit applies the completed result.**

---

# 45. Interview Answer: Why Must Render Be Pure?

> **Because React may execute rendering work multiple times, restart it, or abandon it before commit. If render contained irreversible side effects, those effects could happen multiple times or happen for work that never becomes visible.**

---

# 46. Interview Answer: What Happens When `setState` Is Called?

> **Calling the setter schedules a state update rather than synchronously changing the state variable in the current render. React processes the update, potentially batches and prioritizes it, renders the component again with a new state snapshot, reconciles the new result with the previous one, and commits any necessary host changes.**

---

# 47. Final Revision Sheet

## React Rendering

```text
State / Props Update
 ↓
Schedule
 ↓
Render
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM
 ↓
Browser
```

## Concurrent Rendering

```text
Render
 ↓
Pause
 ↓
Resume / Restart / Abandon
 ↓
Commit
```

## Identity

```text
Same type + compatible identity
→ Preserve state

Different identity
→ Remount
→ Reset state
```

## Effects

```text
Render
 ↓
Commit
 ↓
Effect
```

## State Snapshot

```text
Render #1
count = 0

Render #2
count = 1
```

Each render gets its own snapshot.

---

# 48. Ultimate Mental Model

If an interviewer asks what happens when a user clicks a button that updates state, reconstruct React from this:

```text
                 ┌─────────────────────┐
                 │   STATE / PROPS     │
                 └──────────┬──────────┘
                            ↓
                    UPDATE REQUEST
                            ↓
                    BATCH / PRIORITY
                            ↓
                       SCHEDULER
                            ↓
                         FIBER
                            ↓
                   ┌────────────────┐
                   │ RENDER PHASE   │
                   │                │
                   │ Component()    │
                   │      ↓         │
                   │ React Elements │
                   └───────┬────────┘
                           ↓
                    RECONCILIATION
                           ↓
                       DIFFING
                           ↓
                  REQUIRED CHANGES
                           ↓
                   ┌───────────────┐
                   │ COMMIT PHASE  │
                   └───────┬───────┘
                           ↓
                      DOM UPDATE
                           ↓
                       BROWSER
                           ↓
                     PAINT / PIXELS
                           ↓
                        EFFECTS
```

Surrounding the render phase:

```text
Concurrent Rendering
        ↓
Can pause
Can resume
Can restart
Can abandon
```

While:

```text
Commit
↓
Applies the finished result
```

---

# 49. Handbook 2 — Complete

The entire React Rendering & Internals mental model:

```text
Click
 ↓
Event Handler
 ↓
setState
 ↓
Batch / Priority
 ↓
Schedule
 ↓
Fiber
 ↓
Render
 ↓
React Elements
 ↓
Reconciliation
 ↓
Diff
 ↓
Commit
 ↓
DOM
 ↓
Browser
 ↓
Pixels
 ↓
Effects
```

The five rules to retain:

```text
1. Render calculates.
2. Reconciliation determines what changed.
3. Commit applies the finished result.
4. Browser renders the DOM into pixels.
5. Effects synchronize React with external systems.
```

The most important concurrent-rendering rule:

> **Rendering work can be paused, resumed, restarted, or abandoned. Only committed work becomes visible.**

---

# Handbook 2 Complete

**Chapters 16–30: COMPLETE.**

This chapter is the final synthesis of the handbook. You should now be able to explain the React update pipeline without treating React as a black box.
