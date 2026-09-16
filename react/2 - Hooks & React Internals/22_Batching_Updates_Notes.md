# 22_Batching_Updates_Notes

## Core Definition

**Batching** means React groups multiple state updates together so they can be processed during the same rendering update rather than causing a separate render for every individual update.

```text
setState()
setState()
setState()
   ↓
Batch updates
   ↓
Render
   ↓
Commit
```

> **Batching groups updates together; it does not mean parallel execution.**

## Deep Dive `[NEW]`

### The Actual Data Structure Behind "Batching"

Each fiber that owns state (via `useState`/`useReducer`) has its own
**update queue** — conceptually a small linked list of pending
updates for that fiber. Calling `setCount(x)` doesn't recompute
anything immediately; it appends an update object (`{ action: x }`,
or `{ action: fn }` for functional updates) to that fiber's queue and
marks the fiber's ancestors as having pending work, then asks the
Scheduler to ensure a render happens. When React later processes that
fiber during render, it walks the entire queue front-to-back,
applying each update in order to compute the final state — which is
exactly why `setCount(c => c+1)` called three times produces three
sequential applications (0→1→2→3), while `setCount(count+1)` called
three times just enqueues the same "become 1" instruction three
times: they're not being "overwritten," each is independently
computing `count+1` from the same stale `count` closure.

### Why React 18 Needed an Architecture Change for "Automatic" Batching

Before React 18, batching only happened inside React-owned event
handlers (click, change, etc.) — because those were the only places
React had already wrapped in `unstable_batchedUpdates()`, a function
that set an internal flag telling the update queue "don't flush yet,
more updates might come in this tick." Code running inside a
`setTimeout`, a native `addEventListener`, or a resolved `Promise`
wasn't inside that wrapped call, so each `setState` there flushed and
re-rendered individually. React 18's automatic batching isn't a
smarter heuristic — it moved the batching boundary from "inside
React's own event handler wrapper" to "for the whole duration of a
microtask/macrotask, regardless of who started it," using the same
underlying update-queue mechanism, just triggered more consistently.
That's why the change shipped as an architectural update (tied to
`createRoot`), not a tuning knob — it changed *where* the batching
boundary is drawn, not how batching itself works.

## 1. Why Batching Is Useful

Without batching:

```text
setState() → Render → Commit
setState() → Render → Commit
setState() → Render → Commit
```

With batching:

```text
setState()
setState()
setState()
   ↓
Batch
   ↓
Render
   ↓
Commit
```

Grouping updates can reduce unnecessary rendering work.

## 2. The Classic Multiple `setState` Example

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return <h1>{count}</h1>;
}
```

If the current render has:

```text
count = 0
```

then all three expressions use that same snapshot:

```text
setCount(0 + 1)
setCount(0 + 1)
setCount(0 + 1)
```

Conceptually:

```text
setCount(1)
setCount(1)
setCount(1)
```

Therefore:

```text
0 → 1
```

not:

```text
0 → 3
```

## 3. State Snapshots

Each render has its own state snapshot.

If a render has:

```text
count = 0
```

then:

```jsx
setCount(count + 1);
```

uses `count = 0` from that render.

It does not mean "read whatever the newest state will be after every previous update."

## 4. Functional Updates

When the next state depends on the previous state:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Conceptually:

```text
Initial: 0

Update 1: 0 → 1
Update 2: 1 → 2
Update 3: 2 → 3
```

Final:

```text
count = 3
```

> **Use the functional updater when the next state depends on the previous state.**

## 5. Direct vs Functional Updates

### Direct

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

If `count = 0`:

```text
setCount(1)
setCount(1)
setCount(1)
```

Result:

```text
1
```

### Functional

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Result:

```text
0 → 1 → 2 → 3
```

## 6. Batching Does Not Mean Parallel Execution

Batching does not mean:

```text
Thread 1 → Update A
Thread 2 → Update B
Thread 3 → Update C
```

Instead:

```text
Update A
Update B
Update C
    ↓
Grouped together
    ↓
Processed together
```

> **Batching is grouping work, not parallelizing work.**

## 7. React 18 and Automatic Batching

React 18 expanded automatic batching to more update contexts when using the modern root API.

Modern React can batch updates from contexts including:

- React event handlers
- Promises
- `setTimeout`
- Native event handlers
- Other asynchronous callbacks

For example:

```jsx
fetchData().then(() => {
  setCount(c => c + 1);
  setName("Sonu");
});
```

Conceptually:

```text
Promise callback
      ↓
setCount
setName
      ↓
Batch
      ↓
Render
      ↓
Commit
```

> **React 18 expanded automatic batching so updates from more asynchronous contexts are batched by default.**

## 8. Batching Is Not One Giant Global Batch

Batching does not mean React waits for every state update in the entire application.

Think:

```text
Related updates
      ↓
Batch
      ↓
Render
```

not:

```text
Every update everywhere
      ↓
One giant global batch
```

## 9. `flushSync`

React provides `flushSync` from `react-dom` when code explicitly needs an update to be flushed synchronously.

```jsx
import { flushSync } from "react-dom";

flushSync(() => {
  setCount(1);
});
```

This is an escape hatch and should not be the normal way of updating state.

## 10. Batching vs Scheduling

### Batching

> **Which updates can be processed together?**

### Scheduling

> **When should work happen, and which work is more urgent?**

Shortcut:

```text
Batching
   ↓
GROUPING

Scheduling
   ↓
TIMING + PRIORITY
```

## 11. Batching vs Reconciliation

### Batching

Groups state updates:

```text
Update
Update
Update
 ↓
Batch
```

### Reconciliation

Determines how the new UI tree relates to the previous tree:

```text
Old Tree
    ↕
New Tree
    ↓
Reconciliation
```

Therefore:

```text
Batching
    ↓
reduces separate rendering work

Reconciliation
    ↓
determines what changed
```

## 12. Batching vs Commit

```text
Batching
    ↓
Group updates

Render
    ↓
Calculate new UI

Reconciliation
    ↓
Determine changes

Commit
    ↓
Apply changes
```

## 13. Connection to Previous Chapters

```text
Multiple State Updates
          ↓
       Batching
          ↓
   Group related work
          ↓
      Scheduling
          ↓
 Determine when work happens
          ↓
        Fiber
          ↓
     Units of work
          ↓
     Render Phase
          ↓
    Reconciliation
          ↓
        Commit
          ↓
         DOM
```

This is a conceptual model rather than a literal internal call sequence.

## Interview Answer

**Q: What is batching in React?**

> **Batching is the process of grouping multiple state updates so React can process them together instead of performing a separate rendering update for every individual state update. This can reduce unnecessary rendering work.**

## Interview: Why Does This Produce `1`?

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

> **Because each render has its own state snapshot. If `count` is `0` in the current render, all three expressions calculate `0 + 1`, so they all request the state value `1`. Batching groups the updates, but it does not make the `count` variable change inside that render.**

## Interview: Why Does the Functional Form Produce `3`?

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

> **The functional updater receives the previous state for each update in the sequence, allowing the updates to be applied as `0 → 1 → 2 → 3`. This is useful when the next state depends on the previous state.**

## Interview Nuggets

- Batching groups multiple state updates.
- Batching can reduce unnecessary rendering work.
- Batching does not mean parallel execution.
- Each render has its own state snapshot.
- Direct updates can all calculate from the same snapshot.
- Functional updaters receive the previous state for each update.
- Use functional updates when the next state depends on the previous state.
- React 18 expanded automatic batching to more asynchronous contexts.
- Scheduling and batching are different.
- Reconciliation and batching are different.
- `flushSync` is an escape hatch for explicitly flushing updates synchronously.

## Common Mistakes

❌ Three `setCount(count + 1)` calls always produce `3`.

✅ If they use the same render's snapshot, they can all request the same next value.

❌ Batching means React executes updates in parallel.

✅ Batching groups updates for processing together.

❌ Functional updates immediately change the `count` variable in the current render.

✅ They provide React with state transformations that can be applied using the latest previous state.

❌ Batching and scheduling are the same.

✅ Batching concerns grouping; scheduling concerns timing and priority.

## Flashcards

**Q:** What is batching?

**A:** Grouping multiple state updates so React can process them together.

**Q:** Does batching mean parallel execution?

**A:** No.

**Q:** Why do three direct `setCount(count + 1)` calls often produce `1` from an initial `0`?

**A:** Because all three use the same `count` snapshot from the current render.

**Q:** Why does the functional form produce `3`?

**A:** Each functional updater can receive the previous result in the update sequence.

**Q:** When should you use a functional state updater?

**A:** When the next state depends on the previous state.

**Q:** What changed with React 18 automatic batching?

**A:** React expanded automatic batching to more update contexts, including asynchronous callbacks, when using the modern root API.

**Q:** What is the difference between batching and scheduling?

**A:** Batching groups updates; scheduling coordinates when work should happen and its relative urgency.

## 30-Second Revision

```text
Every render
    ↓
Has its own state snapshot

Direct update:

setCount(count + 1)
setCount(count + 1)
setCount(count + 1)

count = 0
    ↓
1
1
1
    ↓
Result = 1


Functional update:

setCount(c => c + 1)
setCount(c => c + 1)
setCount(c => c + 1)

0 → 1 → 2 → 3
```

Remember:

> **Batching = grouping updates.**

> **Scheduling = timing + priority.**

> **Direct state updates use the current render's snapshot.**

> **Functional updates are useful when the next state depends on the previous state.**
