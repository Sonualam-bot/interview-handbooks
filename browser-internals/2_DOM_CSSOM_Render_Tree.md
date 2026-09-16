# Chapter 2 — DOM, CSSOM & Render Tree

## 1. HTML → DOM

The browser receives HTML as source text and parses it into a structured tree called the **DOM (Document Object Model)**.

Example:

```html
<body>
  <h1>Hello Sonu</h1>
  <p>Welcome to my page</p>
</body>
```

Conceptually:

```text
Document
└── html
    └── body
        ├── h1
        │   └── "Hello Sonu"
        └── p
            └── "Welcome to my page"
```

### Key idea

> HTML is the source document; the DOM is the browser's structured representation of that document.

JavaScript interacts with the DOM:

```js
const heading = document.querySelector("h1");
heading.textContent = "Hello!";
```

---

## 2. DOM ≠ HTML Source

HTML is source text:

```html
<h1>Hello</h1>
```

The DOM is the browser's object/tree representation of that document.

JavaScript works with the DOM rather than directly manipulating the original HTML source.

---

# 3. CSS → CSSOM

The browser also parses CSS.

Example:

```css
.title {
  color: red;
  font-size: 32px;
}

p {
  color: blue;
}
```

The browser parses the stylesheet into a representation called the **CSSOM (CSS Object Model)**.

Conceptually:

```text
CSS
 ↓
CSS Parser
 ↓
CSSOM
```

### Mental model

- **DOM** → What document structure/content exists?
- **CSSOM** → What CSS rules/style information exist?

---

# 4. DOM + CSSOM → Render Tree

The browser uses information from the DOM and CSSOM to determine what participates in rendering.

```text
DOM + CSSOM
     ↓
Render Tree
```

The **Render Tree** represents the objects that participate in rendering.

Important:

> The Render Tree is not simply a copy of the DOM.

---

# 5. `display: none`

Consider:

```html
<body>
  <h1>Hello</h1>
  <p>Visible</p>
  <p style="display: none">Hidden</p>
</body>
```

The DOM contains all three elements:

```text
body
├── h1
├── p
└── p (display:none)
```

But the `display:none` element does not participate in the rendered output/layout.

### Important

`display:none`:

- Does NOT remove the element from the DOM.
- Does NOT allow the element to take layout space.
- Makes the element not participate in rendering/layout.

JavaScript can still find it:

```js
document.querySelector("p");
```

---

# 6. `visibility: hidden`

```css
.hidden {
  visibility: hidden;
}
```

The element:

- Remains in the DOM.
- Remains part of layout.
- Occupies its layout space.
- Is not visually visible.

### Comparison

| CSS | In DOM? | Layout space? | Visible? |
|---|---:|---:|---:|
| `display: none` | Yes | No | No |
| `visibility: hidden` | Yes | Yes | No |
| `opacity: 0` | Yes | Yes | No |
| Normal element | Yes | Yes | Yes |

### Interview trap

`display:none` and `visibility:hidden` are **not equivalent**.

---

# 7. `opacity: 0`

```css
.box {
  opacity: 0;
}
```

The element:

- Exists in the DOM.
- Participates in layout.
- Is rendered but visually transparent.

Therefore:

```text
display:none
    ≠
visibility:hidden
    ≠
opacity:0
```

---

# 8. Style Calculation

The browser needs to determine which styles actually apply to each element.

For example:

```css
p {
  color: blue;
}

.article p {
  color: red;
}
```

For:

```html
<div class="article">
  <p>Hello</p>
</div>
```

the browser must resolve the applicable styles.

This involves concepts such as:

- Selector matching
- Cascade
- Specificity
- Inheritance
- Computed values

This process is generally called **style calculation**.

---

# 9. Computed Style

The browser eventually determines computed style information for an element.

JavaScript can inspect it:

```js
const styles = getComputedStyle(element);

console.log(styles.width);
```

This becomes important later because reading certain layout/style information can cause the browser to perform additional work.

---

# 10. Complete Rendering Pipeline

The simplified pipeline is:

```text
HTML
 ↓
Parsing
 ↓
DOM

CSS
 ↓
Parsing
 ↓
CSSOM

DOM + CSSOM
 ↓
Style Calculation
 ↓
Render Tree
 ↓
Layout
 ↓
Paint
 ↓
Composite
 ↓
Pixels
```

---

# 11. What Each Stage Means

### DOM

> Represents the document structure/content.

### CSSOM

> Represents parsed CSS rules/style information.

### Style Calculation

> Determines the styles that apply to elements.

### Render Tree

> Represents what participates in rendering.

### Layout

> Determines geometry: position and size.

### Paint

> Determines the visual drawing work.

### Composite

> Combines rendered layers into the final output.

---

# 12. JavaScript and DOM Changes

JavaScript can modify the DOM:

```js
element.textContent = "New text";
```

or styles:

```js
element.style.width = "500px";
```

The browser then determines what rendering work is necessary.

A simplified example:

```text
DOM/CSS change
      ↓
Style calculation
      ↓
Layout
      ↓
Paint
      ↓
Composite
```

But:

> Not every change necessarily requires every stage.

The browser tries to do only the work required by the change.

This becomes important when studying **reflow, repaint, and compositing**.

---

# 13. Pseudo-elements

CSS can create rendered content that does not correspond to a normal HTML element.

Example:

```css
.title::before {
  content: "🔥";
}
```

There is no corresponding:

```html
<span>🔥</span>
```

in the HTML.

This reinforces the idea that:

> The Render Tree is not simply the DOM copied into another structure.

---

# 14. Interview Mental Model

Think about the pipeline as questions:

```text
HTML
 ↓
"What exists?"
 ↓
DOM

CSS
 ↓
"What styling rules exist?"
 ↓
CSSOM

DOM + CSSOM
 ↓
"What participates in rendering?"
 ↓
Render Tree

Render Tree
 ↓
"Where and how big?"
 ↓
Layout

Layout
 ↓
"What should be visually drawn?"
 ↓
Paint

Paint
 ↓
"How should layers be combined?"
 ↓
Composite
```

---

# 15. High-Value Interview Traps

### Q: Does `display:none` remove an element from the DOM?

**No.**

The DOM node still exists and JavaScript can access it.

### Q: Does `display:none` take layout space?

**No.**

### Q: Does `visibility:hidden` take layout space?

**Yes.**

### Q: Is the DOM the same as the Render Tree?

**No.**

The DOM represents document structure, while the Render Tree represents what participates in rendering.

### Q: What is the CSSOM?

The browser's representation of parsed CSS rules/style information.

### Q: What happens when JavaScript changes the DOM?

The browser determines the necessary rendering work. Depending on the change, this can involve style calculation, layout, paint, and/or compositing.

---

# 16. Chapter 2 Cheat Sheet

```text
HTML → DOM
CSS  → CSSOM

DOM + CSSOM
     ↓
Style Calculation
     ↓
Render Tree
     ↓
Layout
     ↓
Paint
     ↓
Composite
     ↓
Pixels
```

### Remember

```text
display:none
→ DOM exists
→ no layout space
→ doesn't participate in rendering/layout

visibility:hidden
→ DOM exists
→ layout space exists
→ invisible

opacity:0
→ DOM exists
→ layout space exists
→ visually transparent
```

## Core interview sentence

> The browser parses HTML into the DOM and CSS into the CSSOM, calculates the styles that apply, builds the rendering representation, and then performs layout, paint, and compositing to produce the final pixels.
