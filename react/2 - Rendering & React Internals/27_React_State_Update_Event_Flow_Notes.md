# 27 — What Happens When React Updates State During an Event?

## Interview Importance

**Priority: High.**

This chapter connects several React concepts that are commonly tested:

- State snapshots
- `setState`
- Re-rendering
- Batching
- Scheduling
- Fiber
- React Element trees
- Reconciliation
- Commit
- DOM updates
- Re-render vs remount

The most important mental model is:

```text
User interaction
      ↓
Event handler
      ↓
setState()
      ↓
State update scheduled
      ↓
Batching / scheduling
      ↓
Render
      ↓
New React Element tree
      ↓
Reconciliation
      ↓
Commit
      ↓
DOM mutation if needed
      ↓
Browser rendering
```

---

## Deep Dive `[NEW]`

### What "setState Is a Request" Actually Means, Mechanically

This chapter later says `setState` is "a request for an update" —
here's the concrete mechanism that makes that true. Calling
`setCount(next)` inside an event handler does three real things,
synchronously, in order: (1) creates an update object carrying either
the new value or an updater function; (2) appends it to that fiber's
pending update queue (a linked list — see Batching chapter); (3) walks
up from that fiber to the root, marking each ancestor's lane bitmask
to say "this subtree has pending work" (see Lanes chapter), then asks
the Scheduler to ensure the root gets rendered. None of this touches
`count` itself — the local `count` binding inside the currently-
executing closure is never mutated, which is the literal reason
`console.log(count)` right after `setCount(...)` still shows the old
value: there is no code path in `setState` that reaches back and
rewrites a variable in a closure that already exists.

### Why the Event Handler Finishing Is What Triggers the Batch to Flush

React waits until the current event handler function returns before
processing the queued updates because it's running inside a batching
boundary (originally `unstable_batchedUpdates`, now the default per
React 18) that wraps the entire native event dispatch. The queue only
flushes once that wrapper's call finishes — exactly when the handler
function returns control back to React's own event dispatch code.
That's why synchronous code after multiple `setState` calls in one
handler always sees pre-update values, but code in a `.then()` or
`setTimeout` inside that same handler runs *outside* the wrapper, so
it may trigger its own separate render depending on React version and
what still falls inside the broader React 18 automatic-batching
boundary.

# 1. Example

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

When the user clicks:

```text
Button
 ↓
Browser event
 ↓
React event handling
 ↓
handleClick()
 ↓
setCount(...)
```

---

# 2. The Browser Event

The user clicks the button.

Conceptually:

```text
User click
   ↓
Browser event
   ↓
React event handling
   ↓
handleClick()
```

React's event system invokes the function supplied to `onClick`.

---

# 3. The Event Handler Executes

Suppose the current render has:

```text
count = 0
```

Then:

```jsx
setCount(count + 1);
```

becomes:

```jsx
setCount(1);
```

The important point is that calling the setter does not rewrite the `count` variable inside the already-running render.

---

# 4. `setState` Does Not Immediately Change the Local Variable

Consider:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    console.log(count);

    setCount(count + 1);

    console.log(count);
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

The output is:

```text
0
0
```

not:

```text
0
1
```

Why?

> **The `count` variable belongs to the current render's snapshot.**

Calling:

```js
setCount(1);
```

requests an update.

It does not mutate the `count` variable inside the already-running render.

---

# 5. State Is a Snapshot

Think:

```text
Render #1

count → 0
```

The event handler created during this render closes over:

```text
count = 0
```

Then:

```text
setCount(1)
```

requests the next state.

The existing render still has:

```text
count = 0
```

A future render receives:

```text
count = 1
```

Mental model:

```text
Render #1
count = 0
     │
     │ setCount(1)
     ↓
Render #2
count = 1
```

---

# 6. React Schedules an Update

After:

```js
setCount(1);
```

React records pending state work.

Conceptually:

```text
setCount(1)
      ↓
State update created
      ↓
Update scheduled
```

React then determines how and when the work should be processed.

---

# 7. Batching Can Happen

If multiple state updates happen during the same event:

```jsx
function handleClick() {
  setCount(count + 1);
  setName("Sonu");
  setLoading(false);
}
```

React can batch compatible updates.

Conceptually:

```text
setCount(...)
setName(...)
setLoading(...)
       ↓
    BATCH
       ↓
Rendering work
```

The purpose is to avoid unnecessary separate render/commit cycles.

---

# 8. Scheduling

React then determines how the work should be scheduled.

Conceptually:

```text
State update
     ↓
Priority / lane
     ↓
Scheduling
```

Ordinary user interactions need to remain responsive.

For transition work:

```jsx
startTransition(() => {
  setResults(results);
});
```

React can treat the update as less urgent.

---

# 9. Fiber Represents the Work

React's Fiber architecture represents rendering work.

Conceptually:

```text
State update
    ↓
Pending work
    ↓
Fiber
```

Remember:

> **Fiber represents/manages rendering work.**

Fiber does not mean a new JavaScript thread is created.

---

# 10. Render Begins

React needs to determine what the next UI should look like.

The component executes again:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button>
      {count}
    </button>
  );
}
```

This time React provides:

```text
count = 1
```

The component therefore produces a new React Element:

```js
{
  type: "button",
  props: {
    children: 1
  }
}
```

---

# 11. A New React Element Tree Is Created

Before the update:

```js
{
  type: "button",
  props: {
    children: 0
  }
}
```

After:

```js
{
  type: "button",
  props: {
    children: 1
  }
}
```

React now has:

```text
Previous result
      ↓
<button>0</button>

New result
      ↓
<button>1</button>
```

---

# 12. Reconciliation

React compares the new result with the previous result.

Conceptually:

```text
Old Element
    ↓
<button>0</button>

New Element
    ↓
<button>1</button>
```

React determines:

```text
Same element type
        ↓
Same button
        ↓
Only content changed
```

So the required work is small.

Reconciliation considers things such as:

- Element type
- Component identity
- Keys
- Props
- Tree structure / position

---

# 13. Commit

Once React determines the work that needs to be applied:

```text
Render
 ↓
Reconciliation
 ↓
Commit
```

The commit phase applies the selected changes to the host environment.

For a browser:

```text
DOM
```

So:

```text
<button>0</button>
```

becomes:

```text
<button>1</button>
```

---

# 14. Browser Rendering

After DOM changes, the browser performs its own rendering work.

Conceptually:

```text
React
 ↓
DOM mutation
 ↓
Browser rendering pipeline
 ↓
Pixels
```

Important:

> **React does not paint the pixels. The browser does.**

---

# 15. Complete State Update Pipeline

```text
User clicks
      ↓
Browser event
      ↓
React event handler
      ↓
setState()
      ↓
State update
      ↓
Batching / scheduling
      ↓
Fiber rendering work
      ↓
Component executes again
      ↓
New React Element tree
      ↓
Reconciliation
      ↓
Determine required changes
      ↓
Commit
      ↓
DOM mutation if needed
      ↓
Browser rendering
      ↓
Updated pixels
```

---

# 16. What Happens to the Old React Element Tree?

The previous result is used as the basis for comparison.

React determines:

```text
What changed?
```

It does not simply mean:

```text
Throw away everything
+
Create everything again
```

Instead:

```text
Old tree
   +
New tree
   ↓
Reconciliation
   ↓
Determine required work
```

---

# 17. What Happens to the Old DOM?

The old DOM is not automatically destroyed because a new React Element tree was created.

Example:

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
same button
     ↓
update text
```

Not:

```text
destroy button
     ↓
create new button
```

---

# 18. Component Function Can Run Again

When state changes:

```text
Counter()
```

may execute again.

That does **not** mean:

```text
Old DOM destroyed
+
New DOM created
```

Instead:

```text
Component executes again
        ↓
New React Elements
        ↓
Reconciliation
        ↓
Only necessary host changes
        ↓
Commit
```

---

# 19. State Is Preserved Across Renders

Suppose:

```text
Render #1
count = 0
```

Then:

```text
setCount(1)
```

React renders again:

```text
Render #2
count = 1
```

The state isn't recreated from the initial value:

```jsx
useState(0)
```

on every render.

React remembers state associated with the component's identity.

That's why:

```jsx
useState(0)
```

doesn't reset to `0` on every render.

---

# 20. Re-render vs Remount

This distinction is extremely important.

### Re-render

```text
Same component identity
       ↓
Render again
       ↓
State preserved
```

### Remount

```text
Component identity changes
       ↓
Old instance removed
       ↓
New instance created
       ↓
State starts fresh
```

Keys and component type can affect identity.

---

# 21. What If State Isn't Used in the UI?

Consider:

```jsx
function App() {
  const [count, setCount] = useState(0);

  console.log("render");

  return (
    <button onClick={() => setCount(count + 1)}>
      Click
    </button>
  );
}
```

The state changes:

```text
0 → 1 → 2 → 3
```

The component still re-renders because its state changed.

But if the state doesn't affect the returned UI:

```text
count
 ↓
No visible UI difference
```

there may be no meaningful DOM mutation.

The render still occurred.

---

# 22. Render Does Not Equal DOM Update

A state update can cause rendering work without causing a meaningful DOM mutation.

```text
State update
 ↓
Render
```

does not guarantee:

```text
DOM mutation
```

React may determine:

```text
New result
=
Existing result
```

and therefore do little or no DOM work.

Remember:

```text
Render
≠
DOM mutation
```

---

# 23. Render Does Not Equal Browser Paint

Similarly:

```text
DOM mutation
```

is not the same thing as:

```text
Browser paint
```

Conceptually:

```text
React
 ↓
DOM mutation
 ↓
Browser
 ↓
Layout / paint / compositing
 ↓
Pixels
```

The browser controls its rendering pipeline.

---

# 24. Concurrent Rendering Changes the Middle

Basic model:

```text
setState
 ↓
Render
 ↓
Reconciliation
 ↓
Commit
```

With concurrent rendering:

```text
setState
 ↓
Scheduling
 ↓
Render
 ↓
Can pause / resume / abandon
 ↓
Reconciliation
 ↓
Commit
```

The user does not see partially completed render work.

The currently committed UI remains visible until React commits a completed result.

---

# 25. Strict Mode Connection

Strict Mode helps expose assumptions such as:

```text
"My render only runs once."
```

Concurrent rendering gives React architectural reasons why rendering may need to be restarted or abandoned.

Therefore:

```text
Concurrent rendering
        ↓
Render may be interrupted
        ↓
Render may be restarted
        ↓
Render may be abandoned
        ↓
Render should be pure
```

Strict Mode helps developers discover violations of this principle.

---

# 26. `setState` Is a Request for an Update

Useful interview phrase:

> **Calling a state setter schedules an update; it doesn't synchronously mutate the state variable in the current render.**

Example:

```jsx
setCount(count + 1);

console.log(count);
```

If the current render contains:

```text
count = 0
```

the log still sees:

```text
0
```

The next render receives the updated state.

---

# 27. Functional Updates

When the next state depends on the previous state:

```jsx
setCount(c => c + 1);
```

React can process the updater against the current pending state.

For multiple updates:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

the updates can be processed sequentially:

```text
0
 ↓ +1
1
 ↓ +1
2
 ↓ +1
3
```

Functional updates are especially useful when the next update depends on the previous state.

---

# 28. The Snapshot Model

Suppose:

```text
Render #1
count = 0
```

Every function created during that render closes over that snapshot:

```text
handleClick
    ↓
count = 0
```

Then:

```text
setCount(1)
```

requests the next state.

It does not rewrite the existing snapshot:

```text
Render #1
count = 0
```

Instead:

```text
Render #2
count = 1
```

gets a new snapshot.

---

# 29. Important Example

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    console.log("A:", count);

    setCount(count + 1);

    console.log("B:", count);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

After one click:

```text
A → 0
B → 0
Next render → 1
```

Why?

Because both logs run inside the same render's snapshot.

The setter schedules the next state; it doesn't mutate the current snapshot.

---

# 30. Interview Questions

## Does `setState()` immediately update the state variable?

**No.**

> The setter schedules a state update. The state variable belongs to the current render's snapshot, so it doesn't change within that already-running render. A subsequent render receives the updated state.

---

## Does a state update always cause a DOM update?

**No.**

A state update schedules rendering work. React may determine that:

```text
No DOM changes
```

or:

```text
Only minimal DOM changes
```

are required.

Therefore:

```text
State update
≠
Guaranteed DOM mutation
```

---

## Does a component re-render mean it remounts?

**No.**

### Re-render:

```text
Same identity
 ↓
Execute again
 ↓
State preserved
```

### Remount:

```text
Old identity removed
 ↓
New identity created
 ↓
State reset
```

---

# 31. Common Mistakes

### ❌ `setState()` directly changes the state variable.

✅ It schedules an update; the current render's snapshot remains unchanged.

### ❌ Every re-render recreates the DOM.

✅ React creates a new React Element result and reconciles it with the existing tree.

### ❌ Every render causes DOM mutation.

✅ React may determine that no DOM change is necessary.

### ❌ Re-render means remount.

✅ Re-render preserves component identity and state.

### ❌ React immediately paints after `setState()`.

✅ React schedules and processes rendering work, commits DOM changes if needed, and the browser then performs its rendering work.

### ❌ Fiber is the same thing as reconciliation.

✅ Fiber represents/manages rendering work; reconciliation determines how the new result relates to the previous tree.

---

# 32. Final Interview Mental Model

When asked:

> **"What happens when you call setState?"**

Think:

```text
setState()
    ↓
Update scheduled
    ↓
Batch / determine priority
    ↓
React performs rendering work
    ↓
Component executes again
    ↓
New React Element tree
    ↓
Reconciliation
    ↓
Determine required changes
    ↓
Commit
    ↓
DOM mutation if needed
    ↓
Browser renders
```

And remember:

```text
Current render
    ↓
State snapshot remains fixed

Next render
    ↓
New state snapshot
```

---

# 33. 30-Second Revision

```text
User clicks
 ↓
Event handler
 ↓
setState()
 ↓
State update scheduled
 ↓
Batching / scheduling
 ↓
Render
 ↓
New React Element tree
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM mutation if needed
 ↓
Browser rendering
```

State snapshot:

```text
Render #1
count = 0
     ↓
setCount(1)
     ↓
Render #2
count = 1
```

Remember:

```text
setState()
→ schedules an update

Current render
→ snapshot doesn't change

Re-render
→ same component identity, state preserved

Remount
→ new identity, state reset

Render
≠
DOM mutation

DOM mutation
≠
Browser paint
```

---

# Final Mental Model

> **When a React state setter is called, React schedules an update rather than synchronously changing the state variable in the current render. The current render keeps its state snapshot. React then processes the update, executes the component again with the new state, creates a new React Element tree, reconciles it against the previous result, commits the necessary host changes, and the browser performs its own rendering work.**

The key example:

```text
A → 0
B → 0
Next render → 1
```

That is the state snapshot model in its simplest form.
