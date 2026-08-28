# 25 — React Concurrent Rendering

## Core Definition

Concurrent rendering allows React to make rendering work **interruptible**.

Conceptually:

```text
Start rendering
      ↓
Work for a while
      ↓
Pause / yield
      ↓
Handle more urgent work
      ↓
Resume previous work
```

React may also abandon rendering work if newer work makes it unnecessary.

> **Concurrent rendering does NOT mean React renders on multiple JavaScript threads. It means React can interrupt and coordinate rendering work.**

---

## 1. The Problem Concurrent Rendering Solves

Imagine a large rendering update:

```text
Large update
    ↓
Lots of components
    ↓
Lots of rendering work
```

At the same time, the user types into a search box.

Without interruptible rendering:

```text
Large rendering work
        ↓
JavaScript keeps working
        ↓
User interaction waits
        ↓
UI feels slow
```

With concurrent rendering:

```text
Large rendering work
        ↓
Work for a while
        ↓
Pause / yield
        ↓
Handle more urgent work
        ↓
Resume previous work
```

The main goal is **responsiveness**, not necessarily less total computation.

---

## 2. What "Concurrent" Means

Concurrent does not mean:

```text
Thread 1 → Render A
Thread 2 → Render B
```

Instead:

```text
Work A
  ↓
Work for a while
  ↓
Pause
  ↓
Work B
  ↓
Resume A
```

This is closer to cooperative scheduling than parallel execution.

JavaScript still follows its normal execution model.

---

## 3. Connection to Fiber

Fiber represents rendering work as units of work.

Instead of treating:

```text
Entire application render
```

as one indivisible task, React can work through smaller units:

```text
Fiber A
Fiber B
Fiber C
Fiber D
Fiber E
```

Conceptually:

```text
A
↓
B
↓
C
↓
pause
↓
urgent work
↓
resume D
↓
E
```

Fiber is an important part of the architecture that makes flexible rendering work possible.

---

## 4. Synchronous vs Concurrent Rendering

### Synchronous mental model

```text
Start rendering
      ↓
Keep going
      ↓
Finish rendering
      ↓
Commit
```

### Concurrent mental model

```text
Start rendering
      ↓
Work
      ↓
Yield
      ↓
Other work
      ↓
Resume
      ↓
Finish
      ↓
Commit
```

The important difference:

> **React can interrupt rendering work before it commits it.**

---

## 5. The Render Phase Can Be Interrupted

Recall:

```text
Render
   ↓
Reconciliation
   ↓
Commit
```

With concurrent rendering:

```text
Render
 ↓
Fiber A
 ↓
Fiber B
 ↓
Fiber C
 ↓
PAUSE
```

React can then perform other work.

Later:

```text
Resume
 ↓
Fiber D
 ↓
Fiber E
```

The flexibility primarily applies to rendering work. The commit phase applies the selected completed result to the host environment.

---

## 6. Render Can Be Abandoned

Suppose React begins:

```text
Old UI
   ↓
Update A begins
   ↓
Work in progress
```

Then another update arrives:

```text
Update B
```

React may reassess the existing work.

Conceptually:

```text
Update A
   ↓
Rendering...
   ↓
Interrupted
   ↓
May be abandoned
```

Then:

```text
Update B
   ↓
New rendering work
   ↓
Commit
```

The user never sees an abandoned intermediate tree.

### Important nuance

A new urgent update does **not** automatically mean every existing WIP tree is simply destroyed.

React reassesses the work and may:

- Continue it
- Restart it
- Rebase/reconcile it with newer work
- Abandon it

as appropriate.

---

## 7. Why Abandoning Render Work Is Safe

Render should primarily be a calculation.

```text
Render
 ↓
Calculate UI
```

not:

```text
Render
 ↓
Mutate external systems
```

For example:

```jsx
function Dashboard() {
  sendAnalyticsEvent();

  return <div>Dashboard</div>;
}
```

If React does:

```text
Render
 ↓
sendAnalyticsEvent()
 ↓
Render abandoned
```

the analytics event already happened even though that render never became the UI.

That is a bug.

This connects directly to Strict Mode:

```text
Concurrent rendering
       ↓
Render may be restarted / abandoned
       ↓
Render must be pure
       ↓
Strict Mode helps expose violations
```

---

## 8. Render vs Commit

Concurrent rendering makes this distinction especially important.

```text
Render
 ↓
Possible work
 ↓
Can be interrupted
 ↓
Can be abandoned
 ↓
Eventually selected work
 ↓
Commit
 ↓
DOM mutations
```

Therefore:

> **Render does not mean the user will definitely see that result.**

Only committed work becomes the visible UI.

---

## 9. Current Tree vs Work-in-Progress Tree

### Current Tree

```text
Currently committed UI
```

This represents what the user is currently seeing.

### Work-in-Progress Tree

```text
Potential next UI being prepared
```

Conceptually:

```text
Current Tree
     ↓
VISIBLE UI

WIP Tree
     ↓
Being prepared
```

Concurrent rendering can work on the WIP tree while the Current Tree continues representing the visible UI.

---

## 10. What the User Sees During an Interruption

Suppose:

```text
Current Tree
      ↓
Visible UI
```

and:

```text
WIP Tree
      ↓
Rendering...
      ↓
Pause
```

The user continues seeing the currently committed UI.

The WIP tree is not partially painted into the browser.

---

## 11. If Rendering Work Is Abandoned

Suppose:

```text
Current Tree
     ↓
WIP Tree A
     ↓
Rendering...
```

React later decides A is no longer useful:

```text
WIP Tree A
     ↓
ABANDONED
```

The user continues seeing the current committed tree until some completed work is committed.

```text
Current UI
   ↓
New rendering work
   ↓
Commit
   ↓
New UI
```

---

## 12. Concurrent Rendering Does Not Mean Concurrent DOM Mutation

Do not say:

> "React concurrently updates the DOM."

Instead:

```text
Concurrent rendering
        ↓
Rendering work can be interrupted

Commit
        ↓
Applies the selected completed changes
```

The concurrent flexibility primarily concerns rendering work.

---

## 13. Example: Large List

Imagine:

```jsx
function App() {
  return <HugeList />;
}
```

React conceptually works through many items:

```text
Item 1
Item 2
Item 3
...
Item 500
...
```

With interruptible rendering:

```text
Render items
   ↓
Work for a while
   ↓
Yield
   ↓
Handle urgent work
   ↓
Resume list rendering
```

The goal is to keep the application responsive.

---

## 14. User Interaction Example

```jsx
function Search() {
  const [query, setQuery] = useState("");

  return (
    <>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
      />

      <HugeSearchResults query={query} />
    </>
  );
}
```

When the user types:

```text
S
```

the state becomes:

```text
query = "S"
```

Rendering a huge result list could be expensive.

Concurrent rendering gives React more flexibility to manage that rendering work without treating every operation as an uninterruptible block.

---

## 15. Concurrent Rendering Does Not Automatically Make Code Faster

Concurrent rendering does not necessarily reduce:

```text
Total CPU work
```

Its major benefit can be:

```text
Better responsiveness
+
Better scheduling of rendering work
```

Conceptually:

```text
Same total work
+
Better scheduling
=
Better responsiveness
```

---

## 16. Responsiveness vs Total Work

### Performance

Often asks:

> How much total work did we perform?

### Responsiveness

Asks:

> Can the application respond quickly to the user while that work is happening?

Concurrent rendering primarily gives React more tools to improve responsiveness.

---

## 17. Scheduler Connection

The Scheduler helps coordinate when work should happen.

Conceptually:

```text
Multiple pieces of work
        ↓
Scheduler
        ↓
Prioritize / coordinate
        ↓
Fiber work
        ↓
Render
        ↓
Yield when appropriate
```

Mental model:

```text
Fiber
→ Represents/manages units of rendering work

Scheduler
→ Coordinates when work should happen
```

---

## 18. Concurrent Rendering and Priority

Imagine:

```text
Work A → Expensive list rendering
Work B → User interaction
```

If B is more urgent:

```text
Work A
   ↓
Pause / yield
   ↓
Work B
   ↓
Finish B
   ↓
Resume / reassess A
```

This is where:

```text
Scheduler
+
Fiber
+
Concurrent rendering
```

work together.

---

## 19. `startTransition`

React provides APIs that allow developers to mark certain updates as **non-urgent transitions**.

Example:

```jsx
import { startTransition } from "react";

function handleChange(e) {
  setQuery(e.target.value);

  startTransition(() => {
    setSearchResults(
      getResults(e.target.value)
    );
  });
}
```

Conceptually:

```text
query update
    ↓
urgent

search results update
    ↓
transition / less urgent
```

This allows React to prioritize keeping the interaction responsive while treating the expensive update as interruptible work.

---

## 20. What `startTransition` Does NOT Mean

It does not mean:

> "Run this code on another thread."

It does not simply mean:

> "Run this later with setTimeout."

Instead:

> **`startTransition` tells React that the update is a transition and can be treated as non-urgent rendering work.**

The JavaScript callback still executes according to normal JavaScript execution rules.

---

## 21. Urgent vs Transition Update

```jsx
function handleChange(e) {
  setInput(e.target.value);

  startTransition(() => {
    setResults(search(e.target.value));
  });
}
```

Conceptually:

```text
Typing
 ↓
setInput()
 ↓
Urgent UI update
```

and:

```text
startTransition()
 ↓
Results update
 ↓
Non-urgent rendering work
```

The goal is to prioritize the interaction.

---

## 22. Concurrent Rendering Is About React Rendering Work

Do not say:

> "React makes my JavaScript asynchronous."

JavaScript functions still execute according to normal JavaScript execution rules.

The difference is that React has more flexibility in how it organizes and schedules rendering work.

```text
JavaScript execution
    ↓
Normal JS execution rules
```

while:

```text
React rendering work
    ↓
Can be scheduled / interrupted / resumed
```

---

## 23. No Multiple JavaScript Threads

Concurrent:

```text
≠
```

Multithreaded.

Think:

```text
Single JS execution environment
        ↓
React organizes work
        ↓
Work A
        ↓
Yield
        ↓
Work B
        ↓
Resume A
```

Not:

```text
CPU Thread 1 → A
CPU Thread 2 → B
```

---

## 24. No Partial UI

Suppose React is rendering:

```text
Header
Dashboard
Footer
```

and has only completed:

```text
Header
Dashboard
```

React does not want to show the partially completed result as the final UI.

Instead:

```text
Current committed tree
        ↓
Remains visible
        ↓
WIP tree
        ↓
Can be worked on privately
        ↓
Commit when ready
```

The user should see a consistent committed UI.

---

## 25. Current Tree vs Work-in-Progress Tree

### Current Tree

```text
Currently committed UI
```

### Work-in-Progress Tree

```text
Potential next UI being prepared
```

Concurrent rendering can work on:

```text
WIP
```

while:

```text
Current Tree
```

continues representing the visible UI.

---

## 26. Connection to the Rendering Lifecycle

The Chapter 23 model was:

```text
State update
 ↓
Scheduling
 ↓
Render
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM
```

Concurrent rendering adds flexibility inside rendering:

```text
State Update
      ↓
Scheduling
      ↓
Fiber
      ↓
Concurrent Render
      ↓
 ┌───────────────┐
 │ Work / Yield  │
 │ / Resume      │
 │ / Abandon     │
 └───────────────┘
      ↓
Reconciliation
      ↓
Commit
      ↓
DOM
```

The key idea:

> **Render work can be interrupted before commit.**

---

# 27. Most Important Rules

If you remember only one thing:

> **Concurrent rendering makes the render phase interruptible; it does not mean React renders on multiple JavaScript threads.**

Second:

> **Only committed work becomes the UI the user sees.**

Third:

> **Render must be pure because rendering work may be paused, restarted, or abandoned.**

---

# Interview Questions

## What is Concurrent Rendering?

> **Concurrent rendering is a React architecture that allows rendering work to be interruptible. React can work on a render, pause or yield to more urgent work, resume it later, or abandon it if a newer update makes it unnecessary. This improves responsiveness without requiring multiple JavaScript threads. The render phase can be interrupted, while the commit phase applies the selected result to the DOM.**

## Does Concurrent Rendering mean React uses multiple threads?

No.

> **Concurrent rendering refers to React's ability to interrupt and coordinate rendering work, not parallel execution across multiple JavaScript threads.**

## Does React show half-finished work?

Generally, no.

React can work on the Work-in-Progress tree while keeping the Current Tree as the visible committed UI.

Only committed work is reflected in the DOM.

## Does Concurrent Rendering mean React is faster?

Not necessarily.

It can improve responsiveness even if total computation does not decrease.

```text
Same work
+
Better scheduling
=
Better responsiveness
```

---

# Fiber vs Scheduler vs Reconciliation vs Concurrent Rendering vs Commit

| Concept | Main responsibility |
|---|---|
| **Fiber** | Represents/manages units of rendering work |
| **Scheduler** | Coordinates when work should happen |
| **Reconciliation** | Determines how the new tree relates to the existing tree and what needs to change |
| **Concurrent Rendering** | Allows rendering work to be interrupted, resumed, or abandoned |
| **Commit** | Applies the selected completed result to the host environment |

This distinction is worth memorizing.

---

# Common Mistakes

### ❌ Concurrent rendering means multiple JavaScript threads.

✅ It means React can interrupt and coordinate rendering work.

### ❌ Concurrent rendering means the DOM is updated concurrently.

✅ The main flexibility is in rendering work; the selected result is applied during commit.

### ❌ Every urgent update automatically destroys the old WIP tree.

✅ React reassesses existing work and may continue, restart, rebase, or abandon it as appropriate.

### ❌ React shows partially completed rendering work.

✅ The currently committed tree remains the visible UI until work is committed.

### ❌ Concurrent rendering automatically reduces total computation.

✅ It primarily improves responsiveness and scheduling flexibility.

### ❌ `startTransition` runs code on another thread.

✅ It marks an update as a non-urgent transition.

### ❌ Concurrent rendering makes JavaScript asynchronous.

✅ JavaScript execution still follows normal JavaScript rules; React rendering work becomes more interruptible and schedulable.

### ❌ Fiber decides what UI changes are necessary.

✅ Reconciliation determines how the new result relates to the existing tree; Fiber represents/manages rendering work.

---

# Flashcards

**Q:** What is concurrent rendering?

**A:** A React architecture that allows rendering work to be interrupted, resumed, or abandoned.

**Q:** Does concurrent rendering use multiple JavaScript threads?

**A:** No.

**Q:** What is Fiber's role?

**A:** Fiber represents and manages rendering work as units of work.

**Q:** What is the Scheduler's role?

**A:** It helps coordinate when rendering work should happen and its relative urgency.

**Q:** Can render work be interrupted?

**A:** Yes.

**Q:** Can render work be abandoned?

**A:** Yes, if newer work makes it unnecessary.

**Q:** Can the user see an abandoned render?

**A:** No. Only committed work becomes the visible UI.

**Q:** What does the user see while a WIP render is paused?

**A:** The currently committed UI represented by the Current Tree.

**Q:** Does concurrent rendering automatically reduce CPU work?

**A:** No. Its major benefit is improved responsiveness and scheduling flexibility.

**Q:** What does `startTransition` do?

**A:** It marks an update as a non-urgent transition so React can treat its rendering work with lower urgency.

**Q:** Why must render be pure?

**A:** Because React may execute, restart, pause, or abandon rendering work.

**Q:** What is the relationship between Current and WIP trees?

**A:** Current represents the committed visible UI; WIP represents a possible next UI being prepared.

---

# 30-Second Revision

```text
State Update
      ↓
Scheduling
      ↓
Fiber
      ↓
Render
      ↓
Can:
  ├── Continue
  ├── Yield / Pause
  ├── Resume
  └── Abandon
      ↓
Reconciliation
      ↓
Commit
      ↓
DOM
      ↓
Browser
```

Meanwhile:

```text
Current Tree
     ↓
Visible UI

WIP Tree
     ↓
Potential next UI
```

And:

```text
Fiber
→ What rendering work exists?

Scheduler
→ When should work happen?

Concurrent Rendering
→ Can rendering work be interrupted?

Reconciliation
→ What needs to change?

Commit
→ Apply the selected result.
```

---

# Final Mental Model

> **Concurrent rendering is React's ability to treat rendering as interruptible work rather than one indivisible operation. React can work on a Work-in-Progress tree, yield to more urgent work, resume or reassess previous work, and abandon it if necessary. The user continues seeing the Current committed tree until React commits a completed result.**

> **Only committed work becomes the UI. Render work can be interrupted or abandoned.**
