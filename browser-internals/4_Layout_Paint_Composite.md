# Chapter 4 — Layout, Paint & Composite

## 1. Layout — "Where does everything go?"

After the browser knows what elements exist and what styles apply, it needs to calculate their **geometry**.

Layout determines things such as:

- Width
- Height
- X/Y position
- Margins
- Padding
- Text positions
- Other geometric relationships

Mental model:

> **Layout answers: "Where and how big is everything?"**

Example:

```css
.box {
  width: 200px;
  height: 100px;
  margin: 20px;
}
```

The browser needs to determine the element's position and dimensions.

---

## 2. Layout Depends on Other Elements

Layout is not always isolated to one element.

Consider:

```text
Parent
├── Child A
├── Child B
└── Child C
```

If Child A becomes much taller:

```text
Parent
├── Child A ↑
├── Child B ↓
└── Child C ↓
```

Other elements may need to move.

Therefore:

> **Changing the geometry of one element can require layout calculations for other elements.**

---

# 3. Reflow / Layout

**Reflow** is the commonly used term for recalculating layout after a change.

Modern browser documentation often uses the term **layout**, but "reflow" is still common in frontend interviews.

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

## 4. What Can Trigger Layout?

Examples include changing properties that affect geometry:

```text
width
height
margin
padding
border
font-size
position
top
left
```

Not every property change has exactly the same cost, but geometry-affecting changes can require layout.

---

# 5. Paint — "What should be drawn?"

After layout, the browser knows:

- Where things are
- How large they are

It now needs to determine what visual content should be drawn.

That's **Paint**.

Paint deals with things such as:

- Text
- Colors
- Backgrounds
- Borders
- Shadows
- Images
- Visual effects

Mental model:

> **Layout = geometry.**
>
> **Paint = visual drawing.**

---

## 6. Example of Paint

Suppose:

```css
button {
  background: blue;
  color: white;
  border-radius: 8px;
  box-shadow: 0 4px 10px gray;
}
```

Layout determines:

```text
x = 100
y = 200
width = 120
height = 50
```

Paint determines the visual drawing:

```text
blue background
white text
rounded corners
shadow
```

---

# 7. Repaint

If something changes visually but doesn't require changing geometry, the browser may need to repaint.

Example:

```js
element.style.backgroundColor = "red";
```

The element's size and position may remain unchanged.

Conceptually:

```text
Style change
     ↓
Paint
     ↓
Composite
```

rather than necessarily recalculating the entire layout.

This is generally cheaper than a layout-triggering change, although the actual cost depends on the page and browser.

---

# 8. Layout vs Paint

### Changing width

```js
element.style.width = "500px";
```

Potentially:

```text
Style
 ↓
Layout
 ↓
Paint
 ↓
Composite
```

### Changing background color

```js
element.style.backgroundColor = "red";
```

Potentially:

```text
Style
 ↓
Paint
 ↓
Composite
```

The second change doesn't alter geometry.

---

# 9. Composite — "How do the layers come together?"

After painting, the browser may have multiple **layers**.

It needs to combine those layers into the final image.

That's **compositing**.

Conceptually:

```text
Layer 1 ──┐
Layer 2 ──┤
Layer 3 ──┤
Layer 4 ──┘
     ↓
Composite
     ↓
Final pixels
```

Think of layers as transparent sheets stacked on top of one another.

---

# 10. Why Do Layers Exist?

Modern webpages can have complex visual structures:

```text
Page
 ├── Background
 ├── Header
 ├── Content
 ├── Fixed navbar
 ├── Modal
 └── Animated element
```

The browser can represent some elements as separate compositing layers.

This can allow certain visual changes to happen without repainting the entire page.

---

# 11. `transform`

This is a major performance concept.

Example:

```css
.box {
  transform: translateX(100px);
}
```

A transform changes the visual position without necessarily changing the element's layout geometry.

This means the browser can often handle the movement during compositing.

Conceptually:

```text
Layout
  ↓
Paint
  ↓
Layer
  ↓
transform
  ↓
Composite
```

This can be significantly cheaper for animations than repeatedly changing layout properties.

---

# 12. `left` vs `transform`

### Using `left`

```js
element.style.left = `${x}px`;
```

Changing `left` can affect layout.

Potentially:

```text
Style
 ↓
Layout
 ↓
Paint
 ↓
Composite
```

for repeated animation updates.

### Using `transform`

```js
element.style.transform = `translateX(${x}px)`;
```

The document layout does not necessarily need to change.

Potentially:

```text
Style
 ↓
Composite
```

after the relevant layer has been prepared.

Therefore:

> **Transforms are commonly preferred for animations when the visual effect can be achieved without changing layout.**

---

# 13. `opacity`

Similarly:

```css
opacity: 0.5;
```

can often be handled efficiently during compositing.

This is why:

```text
transform
opacity
```

are commonly considered compositor-friendly properties.

Example:

```css
.box {
  transition:
    transform 300ms,
    opacity 300ms;
}
```

They are often preferable to animating:

```text
width
height
top
left
margin
```

when the desired visual effect can be achieved with transforms/opacity.

---

# 14. GPU Acceleration

Avoid saying:

> "`transform` always uses the GPU."

That's too absolute.

A better interview answer:

> **Transforms and opacity can often be handled efficiently by the compositor, and compositing may involve GPU-accelerated operations depending on the browser, platform, and rendering path.**

Modern browsers have sophisticated rendering architectures, and CPU/GPU involvement depends on the situation.

---

# 15. Compositing Layers

Browsers can create separate layers for certain elements.

Things that can contribute to layer creation include:

```text
transform
opacity
position: fixed
certain animations
```

However:

> **More layers are not automatically better.**

Layers consume memory and have management costs.

Therefore, don't blindly try to create layers everywhere.

---

# 16. `will-change`

Example:

```css
.box {
  will-change: transform;
}
```

This tells the browser that a property is likely to change.

It can allow the browser to prepare for an upcoming change.

But excessive use can hurt performance because additional layers/resources may consume memory.

Therefore:

> **Use `will-change` sparingly and intentionally.**

---

# 17. Reflow vs Repaint vs Composite

Useful interview-level generalization:

| Change | Layout? | Paint? | Composite? |
|---|---:|---:|---:|
| `width` | Often | Usually | Yes |
| `height` | Often | Usually | Yes |
| `margin` | Often | Usually | Yes |
| `background-color` | No | Often | Yes |
| `color` | No | Often | Yes |
| `transform` | Usually no | Often no after preparation | Often |
| `opacity` | Usually no | Often no after preparation | Often |

**Important:** These are generalizations, not absolute browser guarantees.

---

# 18. Layout Thrashing

Layout thrashing happens when JavaScript repeatedly alternates between:

1. DOM/style writes that invalidate layout.
2. Synchronous layout reads that require up-to-date layout information.

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
Read → force layout
 ↓
Write
 ↓
Read → force layout
 ↓
Write
 ↓
Read → force layout
```

Repeated forced layout can hurt performance.

---

# 19. Layout-Related Reads

Examples of DOM APIs that can require current layout information include:

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

can require style/layout work depending on what is being queried and the current state of the document.

---

# 20. Better Pattern: Separate Reads and Writes

Instead of:

```js
for (const element of elements) {
  element.style.width = "500px";

  console.log(element.offsetWidth);
}
```

prefer, where appropriate:

```js
for (const element of elements) {
  element.style.width = "500px";
}

for (const element of elements) {
  console.log(element.offsetWidth);
}
```

Conceptually:

```text
Writes
 ↓
Writes
 ↓
Writes
 ↓
Layout
 ↓
Reads
 ↓
Reads
 ↓
Reads
```

This reduces unnecessary forced synchronous layout.

---

# 21. Read → Write → Read Problem

The problematic pattern is often:

```text
READ
WRITE
READ
WRITE
READ
WRITE
```

when reads require fresh layout.

A better pattern is often:

```text
READ
READ
READ

WRITE
WRITE
WRITE
```

This is an important DOM-performance principle.

---

# 22. React and Browser Rendering

React optimizes how application state changes become DOM updates.

The browser then handles those DOM/style changes through its own rendering pipeline.

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

React optimization and browser rendering optimization are related, but they are different layers.

---

# 23. Interview: Why `transform` Instead of `left`?

Strong answer:

> Changing `left` can trigger layout because it changes the element's geometry. A transform generally doesn't affect document layout and can often be handled by the compositor, making it more efficient for animations.

---

# 24. Interview: What Is Layout Thrashing?

Strong answer:

> Layout thrashing occurs when JavaScript repeatedly alternates between DOM writes that invalidate layout and synchronous layout reads such as `offsetWidth` or `getBoundingClientRect()`. The browser may be forced to recalculate layout repeatedly, hurting performance.

---

# 25. Interview: Layout vs Paint

> **Layout** determines the geometry and position of elements. **Paint** determines the visual drawing of those elements, such as colors, text, borders, shadows, and images.

---

# 26. Interview: Why Are `transform` and `opacity` Animation-Friendly?

> They generally don't require changing document layout and can often be handled efficiently by the compositor, reducing the amount of layout and paint work required during animation.

---

# 27. Core Mental Model

```text
             RENDER TREE
                  ↓
               LAYOUT
           "WHERE + SIZE?"
                  ↓
                PAINT
             "WHAT TO DRAW?"
                  ↓
             COMPOSITE
          "COMBINE THE LAYERS"
                  ↓
                PIXELS
```

### Geometry change

```text
Geometry change
    ↓
Potential Layout
    ↓
Potential Paint
    ↓
Composite
```

### Visual-only change

```text
Visual-only change
    ↓
Potential Paint
    ↓
Composite
```

### Compositor-friendly change

```text
Compositor-friendly change
    ↓
Potential Composite
```

---

# 28. Performance Hierarchy

A useful simplified model:

```text
Layout + Paint + Composite
        ↑
      expensive

Paint + Composite
        ↑
     cheaper

Composite
        ↑
   often cheapest
```

This is a useful mental model, **not an absolute cost guarantee**. Actual cost depends on the page, browser, device, and amount of work involved.

---

# 29. Chapter 4 Cheat Sheet

### Layout

> Determines geometry — position and size.

### Reflow

> Common term for recalculating layout after a change.

### Paint

> Draws visual content such as text, colors, borders, shadows, and images.

### Repaint

> Re-drawing visual content after a change that doesn't necessarily require layout.

### Composite

> Combines rendered layers into the final image.

### `transform`

> Changes visual geometry without normally changing document layout; often compositor-friendly.

### `opacity`

> Often compositor-friendly.

### `will-change`

> Hint that a property is likely to change; use sparingly.

### Layout thrashing

> Repeated layout-invalidating writes mixed with synchronous layout reads.

### Performance principle

> **Batch reads and writes; avoid unnecessary layout work; prefer compositor-friendly properties for animations when appropriate.**
