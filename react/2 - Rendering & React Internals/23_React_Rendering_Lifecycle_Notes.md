# 23_React_Rendering_Lifecycle_Notes

## Core Definition

The React rendering lifecycle describes what happens from an update being requested until the resulting UI is committed and the browser renders the result.

```text
State / Props / Context change
            ↓
       Update scheduled
            ↓
        Render phase
            ↓
      Component executes
            ↓
    React Elements created
            ↓
      Reconciliation
            ↓
     Determine required work
            ↓
       Commit phase
            ↓
       DOM mutations
            ↓
   Browser rendering pipeline
            ↓
           Paint
```

This is a mental model, not a literal internal call stack.

## Deep Dive `[NEW]`

### This Chapter's Diagram Is a Simplification of a Double-Buffered, Queue-Driven Process

Every arrow in the pipeline above corresponds to a concrete mechanism
covered elsewhere in this handbook, and naming them together is what
turns the diagram from a mnemonic into an actual model:

-   "Update scheduled" = an update object appended to a fiber's update
    queue, with ancestors marked as having pending work (see Batching).
-   "Render phase" = the Scheduler repeatedly picks the next unit of
    work by walking `child`/`sibling`/`return` pointers on a
    *work-in-progress* fiber tree built via each fiber's `alternate`,
    yielding back to the browser roughly every 5ms if needed (see
    Fiber, Scheduler).
-   "Reconciliation" = comparing work-in-progress children against
    current-tree children using type/key heuristics, one parent's
    children at a time — never a global tree search (see
    Reconciliation, Diffing).
-   "Commit phase" = a synchronous, non-yielding block
    (before-mutation → mutation → layout sub-passes) that flips the
    current-tree pointer to the finished work-in-progress tree in one
    atomic swap (see Render vs Commit, Fiber).

### Why "Not a Literal Call Stack" Matters

The actual work loop is not nested function calls mirroring the
diagram — it's a `while (workInProgress !== null)` loop in React's
source repeatedly calling `performUnitOfWork`, checking
`shouldYield()` between iterations. There's no call-stack frame for
"Render Phase" that "calls into" Reconciliation which "calls into"
Diffing — those are conceptual groupings of what one flat work loop is
doing at different fibers, not literal nested calls. This matters
practically: it's why rendering can be paused between any two fibers,
but never mid-way through processing a single fiber — the loop only
checks whether to yield at fiber boundaries, not inside the work done
for one fiber.

## 1. Initial Render vs Update Render

### Initial render

React establishes the initial UI:

```text
React
  ↓
App
  ↓
Component tree
  ↓
DOM
```

### Update render

Something changes after the application already exists:

```text
State / Props / Context update
          ↓
        Render
          ↓
   Reconciliation
          ↓
       Commit
```

> **The initial render establishes the UI; subsequent renders determine how the existing UI should change.**

## 2. Component Execution

A React function component is a JavaScript function.

```jsx
function App() {
  return <h1>Hello</h1>;
}
```

React conceptually executes:

```js
App();
```

The function returns a React Element. If the output contains another component, React continues resolving the component tree.

```text
App
 ↓
Profile
 ↓
DOM elements
```

## 3. Render Phase

During the render phase, React performs work to determine what the UI should look like.

This can involve:

- Executing components
- Reading props
- Reading state
- Creating React Elements
- Working through Fiber
- Reconciling the new result with the previous result

> **The render phase calculates.**

## 4. A Render Does Not Guarantee a DOM Update

If a state value changes but the returned UI remains the same:

```text
Component re-rendered       ✅
New React Elements          ✅
Reconciliation              ✅
DOM mutation                ❌
```

> **A render does not guarantee a DOM update.**

## 5. Counter Update

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

Initially:

```text
count = 0
```

When the user clicks:

```text
Click
  ↓
Event handler
  ↓
setCount(count + 1)
  ↓
Update requested
```

The next render sees:

```text
count = 1
```

## 6. Scheduling

The state setter requests an update. React then schedules the rendering work required to process that update.

```text
setState()
    ↓
Update requested
    ↓
Rendering work scheduled
```

The Scheduler helps React coordinate when rendering work should happen.

## 7. Batching

If multiple updates can be batched:

```jsx
setCount(c => c + 1);
setName("Sonu");
setLoading(false);
```

React can group them:

```text
setCount
setName
setLoading
     ↓
Batch
     ↓
Rendering work
```

Batching is not itself a Render or Commit phase.

## 8. Fiber's Role

Fiber is not a separate lifecycle phase.

Fiber is React's internal architecture/data structure for representing and managing rendering work as units of work.

```text
Scheduler
    ↓
React performs work
    ↓
Fiber architecture
    ↓
Render
```

> **Fiber represents the work.**

## 9. Render Creates the Next React Element Tree

With:

```text
count = 1
```

the output:

```jsx
return (
  <div>
    <h1>{count}</h1>
    <button>Increment</button>
  </div>
);
```

conceptually produces:

```js
{
  type: "div",
  props: {
    children: [
      {
        type: "h1",
        props: {
          children: 1
        }
      },
      {
        type: "button",
        props: {
          children: "Increment"
        }
      }
    ]
  }
}
```

The previous result contained `children: 0` for the `h1`.

React now has an old and new result to reconcile.

## 10. Reconciliation

Reconciliation is the broader process through which React determines how the new result relates to the existing tree and what work is required.

```text
Previous tree
      ↕
New tree
      ↓
Reconciliation
      ↓
Determine required work
```

React uses reconciliation heuristics and diffing logic to determine what should be preserved and what needs to change.

## 11. Diffing

For:

```text
Previous:

<button>0</button>

New:

<button>1</button>
```

The element type is still `button`, so React can preserve the existing button instance.

The content changed:

```text
0 → 1
```

Therefore React determines that the button's displayed content needs updating.

> **Reconciliation is the broader process; diffing is the comparison/heuristic logic used to determine necessary changes.**

## 12. Fiber During Rendering

Conceptually:

```text
Current Fiber
button → 0

        ↓

Work-in-Progress Fiber
button → 1
```

Fiber gives React an internal representation through which it can manage rendering work.

## 13. Commit Phase

Once React has determined the work to apply, it enters the commit phase.

For React DOM, the commit work includes applying the required DOM mutations.

```text
Render
   ↓
Reconciliation
   ↓
Commit
   ↓
Apply DOM mutations
```

> **DOM mutation is part of the commit work.**

## 14. Browser Rendering

After React updates the DOM, the browser handles its own rendering pipeline.

```text
React
  ↓
DOM mutation
  ↓
Browser rendering pipeline
  ↓
Layout / Paint / Compositing as needed
  ↓
Pixels
```

React does not directly paint pixels.

## 15. Component Render ≠ DOM Update ≠ Browser Paint

```text
Component execution
        ≠
DOM mutation
        ≠
Browser paint
```

A component can execute again without causing a DOM mutation, and a DOM mutation is distinct from the browser's painting work.

## 16. Complete Counter Lifecycle

### Initial render

```text
Counter executes
        ↓
React Elements created
        ↓
Fiber work
        ↓
Initial reconciliation
        ↓
Commit
        ↓
DOM created
        ↓
Browser renders UI
```

### User clicks

```text
Click
  ↓
Event handler
  ↓
setCount(1)
```

### Update render

```text
Update requested
      ↓
Scheduling
      ↓
Counter executes again
      ↓
count = 1
      ↓
New React Elements
      ↓
Reconciliation
      ↓
Diff
      ↓
Determine button text changed
      ↓
Commit
      ↓
DOM mutation
      ↓
Browser rendering
```

## 17. Where Batching Fits

```text
setCount
setName
setLoading
     ↓
Batch
     ↓
Schedule/render work
     ↓
Component renders
     ↓
Reconciliation
     ↓
Commit
```

Batching groups updates; it is not itself a render or commit phase.

## 18. Where Scheduler Fits

```text
State Update
     ↓
Scheduling
     ↓
Rendering work
```

The Scheduler helps coordinate when rendering work should happen.

## 19. Fiber + Scheduler + Rendering

```text
Fiber
→ What work exists?

Scheduler
→ When should work happen?

Render
→ What should the next UI be?

Reconciliation
→ What work is necessary?

Commit
→ Apply the result
```

## 20. Rendering Lifecycle

For an update:

```text
State / Props / Context change
        ↓
Schedule
        ↓
Render
        ↓
Reconcile
        ↓
Commit
```

Effects will be added later when studying their relationship with the commit phase.

## 21. Render vs Paint

React render:

```text
React calculates the next UI
```

Browser paint:

```text
Browser turns the resulting visual state into pixels
```

Therefore:

> **React re-render ≠ browser repaint.**

Conceptually:

```text
React Render
      ↓
React Commit / DOM mutations
      ↓
Browser rendering pipeline
      ↓
Paint as needed
```

## Interview Question: State Update Lifecycle

**Q: What happens when a user clicks a button that updates state?**

> The click event invokes the event handler, which calls the state setter. React schedules the resulting rendering work. During the render phase, the component executes again with the updated state and produces the next React Element tree. React reconciles that result with the existing tree and determines the necessary changes. During the commit phase, React applies the required DOM mutations. The browser then handles its rendering pipeline and updates the pixels as necessary.

## Interview Nuggets

- Initial rendering establishes the initial UI.
- Subsequent renders determine how existing UI should change.
- A component can render without causing a DOM mutation.
- Render phase calculates.
- Reconciliation determines how the new result relates to the existing tree.
- Diffing is part of reconciliation logic/heuristics.
- Fiber is an architecture/data structure, not a lifecycle phase.
- Scheduler coordinates when rendering work should happen.
- Batching groups updates; it is not a separate Render/Commit phase.
- Commit applies the required host/DOM mutations.
- DOM mutation is part of commit work.
- Browser rendering happens after the DOM has been updated.
- Component execution, DOM mutation, and browser paint are different concepts.

## Common Mistakes

❌ Every render causes a DOM update.

✅ A component can render while producing no DOM changes.

❌ Fiber is a separate phase after scheduling.

✅ Fiber is React's internal architecture/data structure for representing rendering work.

❌ Batching is a Render phase.

✅ Batching groups updates before the resulting rendering work.

❌ Reconciliation and diffing are completely separate lifecycle phases.

✅ Reconciliation is the broader process; diffing represents the comparison/heuristics used to determine necessary work.

❌ Commit happens and then DOM mutations happen as a separate phase.

✅ React applies the required DOM mutations during the commit phase.

❌ React paints the screen.

✅ React updates the DOM; the browser handles visual rendering and painting.

❌ React render equals browser paint.

✅ React render and browser painting are different operations.

## Flashcards

**Q:** What is the render phase?

**A:** The phase where React performs work to determine what the next UI should look like.

**Q:** Does every render cause a DOM update?

**A:** No.

**Q:** What is reconciliation?

**A:** The broader process of determining how the new UI result relates to the existing tree and what work is required.

**Q:** What is diffing?

**A:** The comparison/heuristic logic used during reconciliation to determine necessary changes.

**Q:** Is Fiber a lifecycle phase?

**A:** No. Fiber is React's internal architecture/data structure for representing and managing rendering work.

**Q:** What happens during commit?

**A:** React applies the required host changes, including DOM mutations for React DOM.

**Q:** Who paints the pixels?

**A:** The browser, not React.

**Q:** Is a component render the same as a browser repaint?

**A:** No.

**Q:** What does the Scheduler do?

**A:** It helps coordinate when rendering work should happen and how urgent that work is relative to other work.

**Q:** What does batching do?

**A:** It groups multiple state updates so they can be processed together.

## 30-Second Revision

```text
User Interaction
      ↓
Event Handler
      ↓
setState()
      ↓
Update Requested
      ↓
Batching (if applicable)
      ↓
Scheduling
      ↓
Fiber-based Rendering Work
      ↓
Render Phase
      ├── Component executes
      └── New React Elements
      ↓
Reconciliation
      └── Determine required work
      ↓
Commit Phase
      └── Apply DOM mutations
      ↓
Browser Rendering
      ↓
Paint
```

Remember:

> **Render calculates.**

> **Reconciliation determines what work is needed.**

> **Commit applies the result.**

> **The browser handles the pixels.**

And the important correction:

> **DOM mutations happen during the commit phase; they are not a separate phase after commit.**
