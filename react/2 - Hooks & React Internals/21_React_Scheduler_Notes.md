# 21_React_Scheduler_Notes

## Core Definition

The Scheduler is part of React's rendering architecture that helps coordinate **when rendering work should be performed** and how urgent different work is relative to other work.

> **Fiber represents the work; Scheduler coordinates when the work should happen.**

## Deep Dive `[NEW]`

### The Actual Problem: JavaScript Has No Preemption

This is the piece conceptual explanations of the Scheduler usually
skip, and it's the one that explains why a Scheduler has to exist at
all: JavaScript is single-threaded and *run-to-completion* — once a
function starts executing, nothing else (a click handler, a paint,
another script) can run until it returns. There is no way for the
browser, or React, to forcibly pause a running JS function midway and
resume it later; a long-running function simply blocks everything
else until it finishes. So "interruptible rendering" cannot mean
actual OS-level preemption — it has to mean React's rendering code
voluntarily stops itself, hands control back to the browser, and asks
to be called again later. That voluntary stop-and-resume is the
Scheduler's entire job.

### How Yielding Actually Works

React's Scheduler processes fiber tree work in small chunks and,
after each chunk, checks "have I been running for about 5ms without
yielding?" (`shouldYield()`). If so, instead of continuing
synchronously, it schedules a callback to resume — historically via
`setTimeout(0)`, and in modern implementations via a `MessageChannel`'s
`postMessage`, because a posted-message callback runs as a new
macrotask *after* the browser has had a chance to process pending
input and paint, but sooner than a `setTimeout` (which browsers
throttle and delay more aggressively). This is the concrete mechanism
behind "pause / yield / resume" — not magic interruption, cooperative
scheduling: do a bit of work, ask "should I stop?", and if yes,
explicitly return control via a browser API built for exactly this
purpose.

### Why This Means Scheduling Can Never Guarantee Timing

Because yielding is cooperative and dependent on the browser's own
task queue, React can request "run this soon" but can never guarantee
exactly when — if the main thread is busy with something else (a long
synchronous script, a heavy layout), the scheduled continuation waits
in line like any other macrotask. That's why the Scheduler is about
*priority and coordination*, not real-time guarantees — it's built
entirely on top of ordinary JS event-loop primitives, not any special
access to the browser's internals.

## Why React Needs Scheduling

Not all updates have the same urgency.

```text
User typing
    ↓
Highly user-facing
    ↓
Should feel responsive

Large dashboard update
    ↓
Potentially expensive
    ↓
Can be less urgent
```

React needs to reason about what work should happen first and what work can wait.

## Fiber vs Scheduler

```text
Fiber
  ↓
What work exists?
  ↓
Units of work

Scheduler
  ↓
When should work happen?
```

Fiber represents rendering work as units that React can manage. The Scheduler helps coordinate when that work should be performed.

## Different Updates Can Have Different Urgency

Conceptually:

```text
Work A → lower urgency
Work B → higher urgency
Work C → lower urgency
```

React can prioritize more urgent work before less urgent work.

This is a conceptual model; React's actual scheduling and priority implementation is more sophisticated than a simple queue.

## Interrupting Lower-Priority Work

Conceptually:

```text
Fiber A
   ↓
Fiber B
   ↓
Pause / deprioritize lower-priority work
   ↓
Handle more urgent work
   ↓
Continue lower-priority work later
```

Fiber makes rendering work representable as units that React can manage. Scheduling helps coordinate that work.

## Scheduler Does Not Manipulate the DOM

```text
Scheduler
    ↓
Rendering Work
    ↓
Render
    ↓
Reconciliation
    ↓
Commit
    ↓
DOM
```

The Scheduler coordinates rendering work; the commit phase applies DOM mutations.

## Scheduling vs Rendering

```text
setState()
    ↓
Update scheduled
    ↓
React determines when work should run
    ↓
Render
    ↓
Reconciliation
    ↓
Commit
```

> **Scheduling determines when React gets to the work; rendering performs the work.**

## Why Scheduling Matters

A large application may be performing expensive rendering while the user is typing, clicking, scrolling, or opening menus.

Treating every update with identical urgency could make expensive work interfere with responsiveness.

Scheduling gives React a way to reason about urgency.

## Browser Responsiveness

JavaScript runs in the browser's execution environment alongside other important work.

A large amount of uninterrupted JavaScript work can make the browser less responsive:

```text
User interaction
       ↓
JavaScript is busy
       ↓
Response delayed
       ↓
UI feels slow
```

React's scheduling architecture helps coordinate rendering work so more urgent user-facing work can be prioritized appropriately.

## Fiber + Scheduler

```text
                 Fiber
                   │
          "What work exists?"
                   ↓
            Units of work
                   │
                   ↓
               Scheduler
                   │
          "When should it run?"
                   ↓
              Render Work
                   ↓
            Reconciliation
                   ↓
                 Commit
```

## Scheduler and Concurrent Rendering

Conceptually:

```text
Lower-priority update
        ↓
Rendering starts
        ↓
Higher-priority interaction arrives
        ↓
React prioritizes urgent work
        ↓
Lower-priority work can continue later
```

This is part of the architecture that supports more flexible concurrent rendering.

## Scheduling vs Batching

These are related but different.

### Scheduling

> **Determines when rendering work should be performed and which work should receive priority.**

### Batching

> **Groups multiple state updates so React can process them together rather than performing a separate render/commit cycle for each update.**

Example:

```jsx
setCount(1);
setName("Sonu");
setLoading(false);
```

Conceptually:

```text
update
update
update
  ↓
Batch
  ↓
Render
  ↓
Commit
```

Scheduling asks:

```text
When should the work happen?
```

Batching asks:

```text
Which updates can be processed together?
```

## Scheduling vs Lanes

For now:

```text
Scheduler
    ↓
Scheduling mechanism

Lanes
    ↓
React's mechanism for representing/prioritizing update work
```

The exact lane model comes later.

## Scheduler Does Not Mean Multithreading

Scheduling does not mean:

```text
Thread 1 → urgent work
Thread 2 → normal work
Thread 3 → background work
```

React is still operating within JavaScript's execution environment.

Scheduling organizes and coordinates **when work gets performed**.

## Full Pipeline

```text
User Interaction
       ↓
State Update
       ↓
Update Scheduled
       ↓
Scheduler
       ↓
Fiber Work
       ↓
Render Phase
       ↓
Reconciliation
       ↓
Diffing
       ↓
Commit Phase
       ↓
DOM
```

This is a conceptual model; the actual implementation is more nuanced, especially around scheduling and priorities.

## Interview Answer

**Q: What is React Scheduler?**

> **The Scheduler is part of React's rendering architecture that helps coordinate when rendering work should be performed. It allows React to reason about the urgency of different work and supports prioritizing more important updates relative to less urgent rendering work.**

Then connect it to Fiber:

> **Fiber provides the units of work that React can manage, while scheduling determines when those units should be processed.**

## Interview Nuggets

- Fiber represents rendering work as units.
- Scheduler coordinates when rendering work should happen.
- Not all updates have equal urgency.
- More urgent user-facing work can be prioritized over less urgent work.
- Scheduler does not directly manipulate the DOM.
- Render performs rendering work; commit applies completed work.
- Scheduling is different from batching.
- Scheduling is different from lanes.
- Scheduling does not mean multithreading.
- Fiber and scheduling together support more flexible rendering.

## Common Mistakes

❌ Fiber schedules every update itself.

✅ Fiber provides units of work; scheduling coordinates when work should happen.

❌ Scheduler directly updates the DOM.

✅ Scheduler coordinates rendering work; commit applies DOM mutations.

❌ Scheduling means React uses multiple JavaScript threads.

✅ Scheduling organizes work within JavaScript's execution environment.

❌ Scheduling and batching are the same.

✅ Scheduling concerns when/priority; batching concerns grouping updates.

❌ React's scheduler is simply a normal priority queue.

✅ A queue is only a conceptual analogy; React's actual scheduling model is more sophisticated.

## Flashcards

**Q:** What is the Scheduler's main responsibility?

**A:** Coordinating when rendering work should happen and helping React reason about the urgency of work.

**Q:** What does Fiber provide?

**A:** Internal units of rendering work that React can manage.

**Q:** Does the Scheduler directly update the DOM?

**A:** No. DOM mutations occur during the commit phase.

**Q:** What is the difference between scheduling and batching?

**A:** Scheduling concerns when and how urgently work should happen; batching concerns grouping multiple state updates for processing together.

**Q:** Does scheduling mean multithreading?

**A:** No.

**Q:** Why is scheduling useful?

**A:** It helps React prioritize user-facing work and coordinate expensive rendering so the UI can remain responsive.

## 30-Second Revision

```text
Fiber
  ↓
What work exists?

Scheduler
  ↓
When should work happen?
Which work is more urgent?
What can wait?

Render
  ↓
Calculate UI / rendering work

Reconciliation
  ↓
Determine changes

Commit
  ↓
Apply completed changes

DOM
```

Remember:

> **Fiber represents the work.**

> **Scheduler coordinates when the work should happen.**

> **Render calculates.**

> **Commit applies.**
