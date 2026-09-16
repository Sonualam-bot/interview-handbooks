# Chapter 6 — Browser Events, Rendering & Performance

## 1. Browser Work: JavaScript vs Rendering

From a frontend-performance perspective, think about two broad categories of work.

### JavaScript / application work

```text
Run JavaScript
Handle events
Process data
Update DOM
```

### Rendering work

```text
Style
 ↓
Layout
 ↓
Paint
 ↓
Composite
```

These interact.

Example:

```js
button.addEventListener("click", () => {
  element.style.width = "500px";
});
```

Conceptually:

```text
User Event
   ↓
JavaScript
   ↓
DOM / Style change
   ↓
Browser rendering
   ↓
Pixels
```

---

## 2. The Main Thread

A simplified mental model is:

```text
                 Main Thread
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
 JavaScript       DOM work      Rendering
       │
       ↓
   Event handling
```

Modern browsers use multiple processes and threads, so this is simplified.

For frontend performance, the key point is:

> **Long-running JavaScript on the main thread can prevent the browser from responding and rendering smoothly.**

---

## 3. JavaScript Can Delay Rendering

Consider:

```js
while (true) {
  // huge amount of synchronous work
}
```

The browser's main thread is occupied.

Possible effects:

```text
UI freezes
Clicks don't respond
Animations stop
Scrolling becomes janky
```

Long-running synchronous JavaScript can therefore prevent smooth rendering and input handling.

---

## 4. The Event Loop and Rendering

You already know the JavaScript event loop.

A simplified browser mental model is:

```text
Task
 ↓
JavaScript
 ↓
Microtasks
 ↓
Browser gets rendering opportunity
 ↓
requestAnimationFrame callbacks
 ↓
Style
 ↓
Layout
 ↓
Paint
 ↓
Composite
```

This is a simplified model, not a guarantee that every browser performs every stage in exactly this order for every event-loop turn.

The important idea is:

> JavaScript execution and microtasks can delay the browser from reaching rendering work.

---

## 5. The 16.67ms Frame Budget

At approximately **60 FPS**:

```text
1 second / 60 frames
≈ 16.67ms per frame
```

A frame has to accommodate work such as:

```text
JavaScript
Style calculation
Layout
Paint
Composite
```

If JavaScript takes:

```text
30ms
```

you have exceeded the approximate 60 FPS frame budget.

The browser may miss a frame.

---

## 6. Dropped Frames and Jank

Suppose the browser wants to produce:

```text
Frame 1
Frame 2
Frame 3
Frame 4
```

but JavaScript blocks the main thread:

```text
Frame 1
   ↓
30ms JavaScript
   ↓
Frame 2 missed
   ↓
Frame 3
```

The user experiences:

> **Jank / dropped frames**

Animations and scrolling can feel uneven rather than smooth.

---

## 7. Long Tasks

Example:

```js
button.addEventListener("click", () => {
  for (let i = 0; i < 1_000_000_000; i++) {
    // expensive work
  }
});
```

The click handler executes synchronously.

While it runs, the main thread is busy.

Possible consequences:

- Input feels delayed
- Animation freezes
- Scrolling becomes less responsive
- Rendering gets delayed

---

## 8. `requestAnimationFrame`

The browser provides:

```js
requestAnimationFrame(callback);
```

It schedules a callback for an appropriate point before the browser's next repaint opportunity.

Example:

```js
function animate() {
  element.style.transform = `translateX(${x}px)`;

  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

Conceptually:

```text
Frame
 ↓
requestAnimationFrame callback
 ↓
Rendering
 ↓
Frame
 ↓
requestAnimationFrame callback
 ↓
Rendering
```

This makes it useful for JavaScript-driven animations.

---

## 9. `requestAnimationFrame` Is Not a Timer

Do not think:

```text
requestAnimationFrame = every 16ms
```

Instead:

> **`requestAnimationFrame` schedules a callback before the browser's next repaint opportunity.**

The browser controls when that opportunity occurs.

This makes it better suited to visual updates than manually trying to synchronize animation with:

```js
setTimeout(update, 16);
```

---

## 10. `requestAnimationFrame` + `transform`

A common animation pattern is:

```js
function animate(timestamp) {
  const x = calculatePosition(timestamp);

  element.style.transform = `translateX(${x}px)`;

  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

This combines:

```text
requestAnimationFrame
        +
transform
        ↓
Frame-synchronized,
compositor-friendly animation
```

---

## 11. Input Events

Common input events include:

```text
click
mousemove
pointermove
scroll
keydown
touch events
```

Some events can occur very frequently.

For example:

```js
window.addEventListener("pointermove", handler);
```

A user moving the pointer can generate many events.

If every event performs expensive work:

```text
pointermove
 ↓
expensive JS
 ↓
pointermove
 ↓
expensive JS
 ↓
pointermove
 ↓
expensive JS
```

the main thread can become overloaded.

---

## 12. High-Frequency Events

Examples:

```text
mousemove
pointermove
scroll
touchmove
```

You generally don't want every event to perform heavy work.

Example:

```js
window.addEventListener("scroll", () => {
  // expensive calculations
});
```

If this handler is expensive, it can hurt responsiveness.

---

## 13. Throttling

**Throttling** means limiting how frequently a function can execute.

Conceptually:

```text
Events:
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓

Handler:
↓     ↓     ↓
```

instead of:

```text
Event → Handler
Event → Handler
Event → Handler
Event → Handler
```

Throttling is useful when you need periodic processing during continuous events.

---

## 14. Debouncing

**Debouncing** means waiting until events stop occurring for a specified period before executing the function.

Classic example:

```text
User typing:

H
He
Hel
Hell
Hello
```

Instead of making an API request for every character:

```text
typing
typing
typing
typing
STOP
 ↓
wait
 ↓
API request
```

---

## 15. Throttle vs Debounce

| | Throttle | Debounce |
|---|---|---|
| Goal | Limit execution frequency | Wait for activity to stop |
| Execution | Periodically during continuous events | After a quiet period |
| Common use | Scroll, pointer movement | Search input |
| Mental model | "At most once every X ms" | "Run after they stop" |

Easy memory trick:

```text
Throttle
→ Slow it down.

Debounce
→ Wait until it settles.
```

---

## 16. Passive Event Listeners

For certain input events, the browser may need to determine whether JavaScript intends to call:

```js
event.preventDefault();
```

before proceeding with a default action such as scrolling.

You can indicate that a listener won't cancel the event:

```js
element.addEventListener("touchstart", handler, {
  passive: true
});
```

This tells the browser that the handler will not call `preventDefault()`.

For appropriate scrolling interactions, this can allow the browser to proceed without waiting for JavaScript to potentially cancel the default action.

---

## 17. Why Passive Listeners Matter

Without the passive contract, conceptually:

```text
User scrolls
    ↓
Browser receives event
    ↓
"Will JS cancel this?"
    ↓
Potential wait
    ↓
Scroll
```

With:

```js
{ passive: true }
```

you are saying:

> "This listener will not call `preventDefault()`."

The browser can therefore proceed more confidently.

---

## 18. When NOT to Use `passive: true`

If your event handler needs:

```js
event.preventDefault();
```

you cannot declare that listener passive.

Therefore:

> **Use passive listeners when appropriate, especially when you don't need to cancel the default scrolling behavior.**

---

## 19. Microtasks Can Delay Rendering

Remember:

```js
Promise.resolve().then(() => {
  // microtask
});
```

Microtasks run before the browser gets to a rendering opportunity in the simplified model.

A huge number of microtasks can therefore keep JavaScript work going and delay rendering.

The important lesson:

> **Microtasks are not free just because they are asynchronous.**

They are still JavaScript work.

---

## 20. Promises Do Not Move CPU Work Off the Main Thread

Consider:

```js
Promise.resolve().then(() => {
  // huge computation
});
```

The callback still executes JavaScript on the main thread.

Therefore:

> **Asynchronous scheduling does not automatically mean non-blocking CPU work.**

Compare:

```text
Async I/O
→ can free the main thread while waiting

CPU-heavy JavaScript
→ occupies the main thread while executing
```

---

## 21. Heavy Computation and Web Workers

If genuinely CPU-heavy work needs to happen without blocking the page, a **Web Worker** can be appropriate.

Conceptually:

```text
Main Thread
     │
     │ message
     ↓
Web Worker
     ↓
Heavy computation
     ↓
Result
     │
     ↓ message
Main Thread
```

Workers run JavaScript in a separate worker context rather than the page's main JavaScript execution context.

---

## 22. Practical High-Frequency Event Pattern

Suppose a draggable element receives many pointer events.

A basic approach might be:

```js
element.addEventListener("pointermove", (event) => {
  element.style.left = `${event.clientX}px`;
});
```

A more performance-conscious pattern can be:

```text
Many pointer events
       ↓
Store latest position
       ↓
requestAnimationFrame
       ↓
Update visual position
       ↓
transform
```

This avoids unnecessarily performing expensive visual work for every event.

---

## 23. Example Pattern

```js
let latestX = 0;
let scheduled = false;

element.addEventListener("pointermove", (event) => {
  latestX = event.clientX;

  if (!scheduled) {
    scheduled = true;

    requestAnimationFrame(() => {
      element.style.transform = `translateX(${latestX}px)`;
      scheduled = false;
    });
  }
});
```

Conceptually:

```text
Many pointer events
       ↓
Keep latest value
       ↓
One visual update per frame
       ↓
transform
```

---

## 24. What Causes UI Jank?

When a UI feels slow, ask:

### Is JavaScript taking too long?

```text
Long JS task
```

### Is layout expensive?

```text
Large layout
```

### Is paint expensive?

```text
Large/complex paint
```

### Are we forcing layout repeatedly?

```text
Layout thrashing
```

### Are we processing too many events?

```text
mousemove / scroll / pointermove
```

### Are we doing unnecessary work every frame?

```text
Animation
```

This gives you a systematic way to investigate performance problems.

---

## 25. Frame Budget Mental Model

At 60 Hz:

```text
~16.67ms
```

But don't interpret this as:

> "My JavaScript gets exactly 16.67ms."

The entire frame has competing work:

```text
          ~16.67ms
┌──────────────────────────┐
│ JS │ Style │ Layout │    │
│    │       │        │    │
│       Paint │ Composite │
└──────────────────────────┘
```

Your JavaScript is only one part of the frame.

Therefore:

> **The less work you force into each frame, the more room the browser has to maintain smoothness.**

---

## 26. Interview: Why Does Long JavaScript Cause Jank?

> JavaScript execution on the main thread competes with other main-thread work such as event handling and rendering. If a JavaScript task takes too long, the browser may miss rendering opportunities, causing dropped frames and making the UI feel unresponsive.

---

## 27. Interview: What Is `requestAnimationFrame`?

> `requestAnimationFrame` schedules a callback to run before the browser's next repaint opportunity, making it appropriate for frame-synchronized visual updates and animations.

---

## 28. Interview: Why Not `setTimeout(..., 16)`?

> A timer schedules work based on elapsed time, whereas `requestAnimationFrame` is coordinated with the browser's rendering cycle. Therefore `requestAnimationFrame` is better suited to visual updates.

---

## 29. Interview: Throttle vs Debounce?

> Throttling limits how frequently a function can execute during continuous events. Debouncing delays execution until the event stream has been quiet for a specified period.

---

## 30. Interview: Why Are Passive Event Listeners Useful?

> A passive listener tells the browser that the handler won't call `preventDefault()`. For appropriate touch or scrolling interactions, this can allow the browser to proceed with scrolling without waiting for JavaScript to potentially cancel it.

---

## 31. Interview: Can Promises Block the UI?

> Promise callbacks are scheduled asynchronously but still execute JavaScript on the main thread. A CPU-heavy Promise callback can therefore block the main thread and delay rendering.

---

## 32. Final Mental Model

```text
                 USER INPUT
                     ↓
                  EVENT
                     ↓
              JavaScript Task
                     ↓
                Microtasks
                     ↓
             Rendering Opportunity
                     ↓
          requestAnimationFrame
                     ↓
             Style Calculation
                     ↓
                  Layout
                     ↓
                   Paint
                     ↓
                Composite
                     ↓
                  PIXELS
```

This is a simplified model for reasoning about browser performance.

---

## 33. Performance Problems

### Long JavaScript

```text
Long JS
   ↓
Main-thread blocking
   ↓
Missed frames
   ↓
Jank
```

### Layout thrashing

```text
DOM writes + layout reads
        ↓
Repeated forced layout
        ↓
Expensive frames
        ↓
Jank
```

### Too many input events

```text
Many events
     ↓
Too much JS
     ↓
Main-thread pressure
     ↓
Jank
```

### Heavy computation

```text
Heavy computation
        ↓
Main thread blocked
        ↓
Consider Web Worker
```

---

## 34. Chapter 6 Takeaway

To keep a web application smooth:

```text
Keep main-thread JS short
        +
Avoid unnecessary layout
        +
Batch DOM reads/writes
        +
Use transform/opacity for suitable animations
        +
Use requestAnimationFrame for visual updates
        +
Throttle/debounce high-frequency events appropriately
        +
Use passive listeners where appropriate
        +
Move genuinely CPU-heavy work to Web Workers
```

The central principle is:

> **Keep the main thread available for user input and rendering, and avoid forcing unnecessary work into each frame.**
