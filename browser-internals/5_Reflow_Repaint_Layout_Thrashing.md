# Chapter 5 — Reflow, Repaint & Layout Thrashing

## 1. The Three Rendering Questions

When the DOM or styles change, ask:

1. Does the browser need to recalculate styles?
2. Does it need to recalculate geometry?
3. Does it need to redraw pixels?

Conceptually:

```text
Style Calculation
       ↓
     Layout
       ↓
      Paint
       ↓
   Composite
```

Don't simply say "the page re-renders." Determine which browser work is actually required.

---

# 2. Reflow / Layout

**Reflow** is the traditional term for recalculating layout.

Modern browser terminology generally uses **layout**.

Example:

```js
element.style.width = "500px";
```

Changing width affects geometry and may require:

```text
Style Calculation
       ↓
      Layout
       ↓
      Paint
       ↓
   Composite
```

---

# 3. Why Layout Can Be Expensive

Layout isn't necessarily limited to one element.

Consider:

```text
Container
├── Box A
├── Box B
└── Box C
```

If Box A becomes taller:

```text
Container
├── Box A ↑
├── Box B ↓
└── Box C ↓
```

Other elements may need to move.

Therefore:

> **Changing the geometry of one element can have cascading layout consequences.**

---

# 4. Common Layout-Triggering Changes

Properties that can affect geometry include:

```text
width
height
margin
padding
border
font-size
top
left
```

These are examples, not absolute guarantees for every browser situation.

Use "can trigger layout" rather than "always triggers layout."

---

# 5. Repaint

If something changes visually but doesn't require changing geometry, the browser may need to repaint.

Example:

```js
element.style.backgroundColor = "red";
```

The element's size and position may remain unchanged.

Conceptually:

```text
Style Calculation
       ↓
      Paint
       ↓
   Composite
```

This is commonly called a **repaint**.

A repaint can be cheaper than a layout-triggering change, but its cost depends on how much content must be painted.

---

# 6. Layout vs Repaint

### Width

```js
element.style.width = "500px";
```

Potentially:

```text
Layout → Paint → Composite
```

### Background color

```js
element.style.backgroundColor = "red";
```

Potentially:

```text
Paint → Composite
```

The key question is:

> **Did the change affect geometry?**

---

# 7. Don't Memorize a Fixed Property Chart

Online performance charts are useful, but browser rendering is more sophisticated than:

```text
property → guaranteed pipeline
```

Actual work depends on factors such as:

- Browser engine
- Current rendering state
- Element
- Property
- Layer structure
- Other simultaneous changes

Therefore use:

> "can trigger"

rather than:

> "always triggers."

---

# 8. Forced Synchronous Layout

JavaScript can force the browser to calculate layout immediately.

Example:

```js
element.style.width = "500px";

console.log(element.offsetWidth);
```

The first operation changes geometry.

The second requests current layout information.

The browser may need to perform layout immediately before returning the value.

Conceptually:

```text
WRITE
 ↓
READ
 ↓
Force Layout
```

---

# 9. Why Forced Layout Can Be Expensive

Browsers try to batch rendering work when possible.

But if JavaScript says:

```text
"I changed something.
Now immediately tell me the new layout."
```

the browser may have to make layout up-to-date immediately.

One forced layout may be fine.

Repeated forced layouts can become expensive.

---

# 10. Layout Thrashing

**Layout thrashing** occurs when JavaScript repeatedly alternates between:

- Writes that invalidate layout
- Synchronous reads that require up-to-date layout

Classic pattern:

```text
WRITE
READ
WRITE
READ
WRITE
READ
```

Example:

```js
for (let i = 0; i < 1000; i++) {
  element.style.width = `${i}px`;
  console.log(element.offsetWidth);
}
```

Conceptually:

```text
Write
 ↓
Read → Layout
 ↓
Write
 ↓
Read → Layout
 ↓
Write
 ↓
Read → Layout
```

---

# 11. Common Layout-Related Reads

Examples include:

```js
element.offsetWidth
element.offsetHeight

element.offsetTop
element.offsetLeft

element.clientWidth
element.clientHeight

element.getBoundingClientRect()
```

Also:

```js
getComputedStyle(element)
```

can require style/layout work depending on what is queried and the current document state.

The important point is not that these APIs are "bad."

The problem is using them in patterns that repeatedly force synchronous layout.

---

# 12. Better Pattern: Batch Reads and Writes

### Bad

```js
for (const element of elements) {
  element.style.width = "500px";

  console.log(element.offsetWidth);
}
```

Pattern:

```text
WRITE
 ↓
READ → Layout

WRITE
 ↓
READ → Layout

WRITE
 ↓
READ → Layout
```

### Better

```js
for (const element of elements) {
  element.style.width = "500px";
}

for (const element of elements) {
  console.log(element.offsetWidth);
}
```

Pattern:

```text
WRITE
WRITE
WRITE
 ↓
Layout
 ↓
READ
READ
READ
```

This gives the browser more opportunity to batch rendering work.

---

# 13. Golden Rule

For DOM-heavy JavaScript:

> **Batch your reads and writes.**

Prefer:

```text
READ
READ
READ

WRITE
WRITE
WRITE
```

over repeatedly doing:

```text
READ
WRITE
READ
WRITE
READ
WRITE
```

when those reads require layout.

---

# 14. Layout Thrashing During Animation

At approximately 60 FPS, the browser has about:

```text
1000 / 60 ≈ 16.7ms
```

per frame.

If JavaScript repeatedly forces expensive layout work, it can consume too much of the available frame budget.

Possible result:

```text
Dropped frames
      ↓
Janky animation
      ↓
Poor user experience
```

---

# 15. `transform` and Animation

For movement, compare:

### Potentially expensive

```js
element.style.left = `${x}px`;
```

Changing `left` can affect layout.

### Often better for animation

```js
element.style.transform = `translateX(${x}px)`;
```

Transforms generally don't change document layout and can often be handled efficiently by the compositor.

Therefore:

> **Use transforms for animations when the desired visual effect can be achieved without changing layout.**

---

# 16. `requestAnimationFrame`

For visual updates, use:

```js
requestAnimationFrame(() => {
  // visual update
});
```

`requestAnimationFrame` schedules work for a point appropriate for the browser's next rendering opportunity.

Example:

```js
requestAnimationFrame(() => {
  element.style.transform = `translateX(${x}px)`;
});
```

It is useful for coordinating JavaScript-driven visual updates with the browser's rendering lifecycle.

---

# 17. Why Not Use `setTimeout` for Animation Timing?

You might see:

```js
setTimeout(() => {
  element.style.transform = "...";
}, 16);
```

This does not synchronize the update with the browser's rendering cycle in the same way.

For frame-synchronized visual updates:

```js
requestAnimationFrame(update);
```

is the appropriate browser API.

---

# 18. React Does Not Make Browser Rendering Disappear

React optimizes application updates and DOM operations, but the browser still has to render the resulting changes.

For example:

```jsx
<div style={{ width }} />
```

may eventually cause browser work such as:

```text
Style
 ↓
Layout
 ↓
Paint
 ↓
Composite
```

Therefore:

> **React performance and browser rendering performance are related but distinct problems.**

---

# 19. Re-render ≠ Reflow

This is a major React interview distinction.

### React re-render

A component function executes again:

```jsx
function App() {
  console.log("render");
  return <div>Hello</div>;
}
```

### Browser layout/reflow

The browser recalculates physical geometry.

These are different concepts.

You can have:

```text
React re-render
      ↓
No meaningful DOM change
      ↓
No layout
```

And browser layout can happen because of DOM/style changes without talking about it as a React component re-render.

---

# 20. Common Interview Traps

### "Every DOM change causes reflow."

**Too broad.**

A DOM change may result in:

- Style recalculation
- Layout
- Paint
- Composite

depending on what changed.

Reason about what the change invalidates.

---

### "Repaint is always expensive."

**No.**

A repaint can be small and relatively cheap, or it can involve a large amount of visual content.

---

### "`transform` means zero rendering work."

**No.**

Transform can avoid layout and often avoid repeated painting, but the browser still needs to composite the relevant content.

---

### "More compositing layers are always better."

**No.**

Layers can help some animations, but they consume memory and have management costs.

---

# 21. Practical Optimization Strategy

When investigating a performance issue, ask:

### Step 1 — Did geometry change?

If yes:

```text
Potential Layout
```

### Step 2 — Does visual content need redrawing?

If yes:

```text
Potential Paint
```

### Step 3 — Can the change be handled efficiently by the compositor?

Examples:

```text
transform
opacity
```

may be compositor-friendly.

### Step 4 — Are we forcing synchronous layout?

Look for:

```text
write
read layout
write
read layout
```

---

# 22. Core Mental Model

```text
             DOM / CSS change
                    ↓
             Style Calculation
                    ↓
             Does geometry change?
                ↙        ↘
              YES         NO
               ↓           ↓
             Layout      maybe Paint
               ↓           ↓
             Paint       Composite
                \          /
                 \        /
                  Composite
                      ↓
                    Pixels
```

This is a simplified mental model, but useful for reasoning about performance.

---

# 23. Interview Cheat Sheet

### Reflow

> Traditional term for recalculating layout/geometry after a change.

### Repaint

> Redrawing visual content after a change that requires new pixels but doesn't necessarily require layout.

### Layout thrashing

> Repeatedly alternating between layout-invalidating writes and synchronous layout reads, causing repeated layout calculations.

### Forced synchronous layout

> When JavaScript requests layout information while the browser has pending changes, potentially forcing layout to happen immediately.

### Common layout reads

```js
offsetWidth
offsetHeight
offsetTop
offsetLeft
clientWidth
clientHeight
getBoundingClientRect()
```

### Animation-friendly properties

```css
transform
opacity
```

They are often compositor-friendly and generally don't require document layout changes.

### Optimization

```text
Batch DOM reads
+
Batch DOM writes
+
Avoid forced synchronous layout
+
Prefer transform/opacity for suitable animations
+
Use requestAnimationFrame for frame-synchronized visual updates
```

---

# 24. Final Mental Model

Don't memorize:

> "`width` = reflow, `color` = repaint, `transform` = GPU."

Instead reason:

```text
What changed?
      ↓
Did geometry change?
      ↓
Does anything need repainting?
      ↓
Can the compositor handle it efficiently?
      ↓
Am I forcing the browser to do this synchronously?
```

That reasoning is much stronger than memorizing property charts.
