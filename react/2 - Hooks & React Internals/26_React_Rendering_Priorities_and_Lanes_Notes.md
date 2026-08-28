# 26 — React Rendering Priorities & Lanes

## Interview Importance

**Priority: Medium-Low for normal frontend interviews.**

Understand the concept, but don't spend time memorizing React's internal lane constants or source-level implementation details.

For most frontend interviews, know:

```text
Update
 ↓
Priority
 ↓
React schedules work
 ↓
Concurrent rendering can prioritize/interupt work
```

The important API-level concept is usually:

```jsx
startTransition(...)
```

---

## 1. Why React Needs Priorities

Not every update is equally urgent.

```text
User typing
    ↓
Very responsive interaction

Huge list/results update
    ↓
Potentially expensive work
```

React should be able to prioritize the interaction so expensive rendering doesn't make the UI feel unresponsive.

---

## 2. What Are Lanes?

**Lanes are an internal React mechanism used to represent priority and pending rendering work.**

Useful mental model:

> **Lanes are buckets that help React track different categories/priorities of pending work.**

```text
Lane A → urgent work
Lane B → transition work
Lane C → lower-priority work
```

A lane is **not** a thread.

---

## 3. Lanes + Concurrent Rendering

Chapter 25 taught that concurrent rendering allows React to interrupt rendering work.

Lanes provide information about the priority of that work.

```text
Update
 ↓
Lane / priority
 ↓
Pending work
 ↓
Scheduling
 ↓
Concurrent rendering
```

---

## 4. Example: Search Input

```jsx
function Search() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);

  function handleChange(e) {
    setQuery(e.target.value);

    startTransition(() => {
      setResults(search(e.target.value));
    });
  }

  return (
    <>
      <input
        value={query}
        onChange={e => handleChange(e)}
      />

      <Results results={results} />
    </>
  );
}
```

Conceptually:

```text
setQuery()
   ↓
Urgent interaction

setResults()
   ↓
Transition / non-urgent work
```

The goal is to keep typing responsive even if rendering the results is expensive.

---

## 5. Why Input Is More Urgent

If the user types:

```text
S
```

the input should reflect:

```text
S
```

quickly.

If React spends too much time rendering a huge result list before updating the input, the application can feel laggy.

So conceptually:

```text
Input update
    ↓
Higher urgency
```

while:

```text
Huge result update
    ↓
Lower urgency
```

---

## 6. `startTransition`

`startTransition` marks updates as **non-urgent transitions**.

```jsx
startTransition(() => {
  setResults(results);
});
```

This tells React:

> **Treat this state update as transition work that should not block more urgent interactions.**

It does **not** mean:

```text
Run on another thread
```

and it does **not** simply mean:

```text
Run later with setTimeout
```

---

## 7. Lanes vs Scheduler

### Lanes

Conceptually answer:

> **What priority/category does this pending work belong to?**

### Scheduler

Conceptually answers:

> **When should React work on it?**

```text
Lanes
 ↓
Describe pending work / priority

Scheduler
 ↓
Coordinates when work happens
```

---

## 8. Lanes vs Fiber

### Fiber

```text
Represents/manages rendering work
```

### Lane

```text
Represents priority/category of rendering work
```

Mental model:

```text
Fiber
   +
Lane
   ↓
"What work exists?"
+
"How urgent is it?"
```

---

## 9. Lanes vs Reconciliation

Lanes do **not** determine which DOM nodes changed.

```text
Lanes
→ Priority / pending work

Reconciliation
→ Determine how the new tree relates to the existing tree and what needs to change

Commit
→ Apply the selected changes
```

---

## 10. Lanes vs Batching

### Batching

> Groups compatible state updates so React can process them together.

### Lanes

> Track the priority/category of pending work.

Conceptually:

```text
Update A → Lane 1
Update B → Lane 1
Update C → Lane 3
```

A and B may be compatible for processing together, while C can remain pending.

---

## 11. Lanes and Interruptibility

Suppose React is working on:

```text
Lane 3
 ↓
Expensive rendering
```

Then a more urgent update arrives:

```text
Lane 1
 ↓
User interaction
```

Conceptually:

```text
Lane 3 rendering
      ↓
Pause / yield
      ↓
Process Lane 1
      ↓
Return to Lane 3
```

This is:

```text
Lanes
+
Concurrent Rendering
```

working together.

---

## 12. Multiple Pending Updates

React may have multiple pending updates:

```text
Update A → Lane A
Update B → Lane B
Update C → Lane C
```

The lane system allows React to track these categories and select appropriate work.

---

## 13. Lanes Are More Than High vs Low

Don't reduce React's internal system to:

```text
HIGH
LOW
```

React has a more granular lane system.

For normal frontend interview preparation, you do **not** need to memorize every internal lane constant.

The important concept is:

> **Lanes allow React to track multiple pending updates with different priorities/categories.**

---

## 14. Lanes Use Bitmasks Internally

At a deeper implementation level, React's lane system uses **bitmasks**.

Conceptually:

```text
00000001
00000010
00000100
00001000
```

Each bit can represent a lane/category.

Multiple lanes can be represented together:

```text
00000101
```

Conceptually:

```text
Lane 1
+
Lane 3
```

This lets React efficiently check, combine, select, and compare sets of pending work.

For normal frontend interviews, know the concept; don't memorize actual internal constants.

---

## 15. Lanes Are Not CPU Priority

Do not interpret:

```text
Lane 1
```

as:

```text
CPU priority 1
```

A lane represents **React rendering-work priority/category**.

It does not tell the JavaScript engine to execute an instruction with higher CPU priority.

---

## 16. Current Tree + WIP Tree + Lanes

Recall:

```text
Current Tree
 ↓
Visible committed UI
```

and:

```text
Work-in-Progress Tree
 ↓
Potential next UI
```

Lanes help React determine which pending work should be processed.

```text
Current Tree
     ↓
Visible UI

Pending lanes
     ↓
Select / prioritize work
     ↓
WIP Tree
     ↓
Render
```

---

## 17. Complete Mental Model

```text
                    STATE UPDATE
                          ↓
                        LANE
                          ↓
                Priority / pending work
                          ↓
                      SCHEDULING
                          ↓
                        FIBER
                          ↓
                CONCURRENT RENDERING
                          ↓
             ┌────────────┴────────────┐
             ↓                         ↓
          Continue                  Yield
                                        ↓
                                More urgent work
                                        ↓
                                   Render it
                                        ↓
                                Resume / reassess
                                        ↓
                                 RECONCILIATION
                                        ↓
                                     COMMIT
                                        ↓
                                       DOM
```

---

## 18. Interview-Level Distinctions

| Concept | What to remember |
|---|---|
| **Fiber** | Represents/manages units of rendering work |
| **Lane** | Represents priority/category of pending work |
| **Scheduler** | Coordinates when work should happen |
| **Concurrent Rendering** | Makes rendering work interruptible |
| **Reconciliation** | Determines what changed |
| **Commit** | Applies the selected result |
| **Batching** | Groups compatible updates |
| **`startTransition`** | Marks an update as non-urgent transition work |

---

## 19. Interview Answer — What Are Lanes?

> **Lanes are an internal React mechanism for representing the priority and pending state of rendering work. Different updates can be associated with different lanes, allowing React to track multiple pending updates and determine which work should be processed with higher priority. Lanes work together with scheduling and concurrent rendering so that more urgent work can take precedence over less urgent rendering work.**

---

## 20. Interview Answer — Lanes vs Scheduler

> **Lanes represent the priority and pending state of React work, while the Scheduler coordinates when that work should be performed. They work together: lanes provide information about the work's priority, and scheduling uses that information to coordinate rendering.**

---

## 21. Interview Answer — Lanes vs Fiber

> **Fiber represents React's rendering work as units that can be processed and managed, while lanes represent the priority or category of that work. Fiber represents what work React is managing; lanes provide information about the priority of pending work.**

---

## 22. Common Mistakes

### ❌ Lanes are threads.

✅ Lanes represent React work priority/categories.

### ❌ Lanes directly update the DOM.

✅ Commit applies DOM mutations.

### ❌ Lanes decide what DOM nodes changed.

✅ Reconciliation/diffing determines necessary changes.

### ❌ Scheduler and lanes are the same.

✅ Lanes represent priority/pending work; Scheduler coordinates when work happens.

### ❌ Fiber and lanes are the same.

✅ Fiber represents work; lanes represent priority/category.

### ❌ Lane priority means CPU thread priority.

✅ It is React's rendering-work priority, not JavaScript thread priority.

### ❌ You need to memorize every lane constant.

✅ Understand the conceptual model; source-level constants are unnecessary for normal frontend interviews.

### ❌ `startTransition` runs code on another thread.

✅ It marks an update as a non-urgent transition.

---

## 23. What You Actually Need to Know for Interviews

### Must Know

```text
Concurrent rendering
        ↓
React can interrupt rendering work

Priority
        ↓
Not all updates are equally urgent

startTransition
        ↓
Marks an update as non-urgent

Lanes
        ↓
Internal mechanism for tracking priority/pending work
```

### Nice to Know

```text
Lanes use a bitmask-based model internally.
```

### Don't Waste Time Memorizing

```text
Exact lane constants
Internal bit values
Source-code implementation details
Every lane category
```

Unless the interviewer specifically goes into React internals.

---

## 24. 30-Second Revision

```text
Update
 ↓
Lane
 ↓
Priority / pending work
 ↓
Scheduling
 ↓
Fiber
 ↓
Concurrent rendering
 ↓
Render
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM
```

Remember:

```text
Fiber
→ What rendering work exists?

Lane
→ How should React categorize/prioritize it?

Scheduler
→ When should work happen?

Concurrent rendering
→ Can rendering work be interrupted?

Reconciliation
→ What changed?

Commit
→ Apply the result.
```

---

# Final Mental Model

> **Lanes are an internal React mechanism for tracking pending rendering work and its priority. They work together with scheduling and concurrent rendering so React can prioritize more urgent updates over less urgent work. For normal frontend interviews, understand lanes conceptually and focus more heavily on concurrent rendering, `startTransition`, render purity, reconciliation, and the render/commit distinction.**
