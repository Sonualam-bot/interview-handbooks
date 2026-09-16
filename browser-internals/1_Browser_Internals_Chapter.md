# Browser Internals — Chapter 1
## Browser Architecture & Page Loading

## 1. Why Browser Internals Matter

Frontend code ultimately has to become **pixels displayed on the screen**.

Understanding the browser rendering pipeline helps explain:

- Rendering performance
- Slow pages
- Janky animations
- Expensive DOM updates
- Layout/reflow
- JavaScript blocking
- Network performance
- Core Web Vitals

---

## 2. High-Level Browser Architecture

A modern browser is composed of multiple processes/components.

Simplified model:

```text
                         Browser
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ↓                 ↓                 ↓
   Browser Process     Renderer Process   Network Process
                            │
                            ↓
                       Web Page
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
         JavaScript       DOM/CSS      Rendering
           Engine                       Pipeline
```

The exact architecture differs between browsers, but this model is useful for understanding frontend behavior.

---

## 3. Browser Process

The browser process handles browser-level responsibilities such as:

- Browser UI
- Tabs/windows
- Navigation
- Coordinating other processes
- Security/isolation responsibilities

Your React application does not normally execute in the browser process itself.

---

## 4. Renderer Process

The renderer process is responsible for processing a webpage.

Conceptually, it deals with:

- HTML
- CSS
- JavaScript
- DOM
- Rendering

For frontend developers, this is the most relevant part of browser architecture.

---

## 5. JavaScript Engine

A browser contains a JavaScript engine.

For example, Chrome uses **V8**.

Its job is to execute JavaScript.

Example:

```js
const x = 10;
const y = 20;

console.log(x + y);
```

The JavaScript engine executes this code.

This connects to the async JavaScript model:

```text
JavaScript
    ↓
Call Stack
    ↓
Browser APIs / runtime
    ↓
Task & Microtask queues
    ↓
Event Loop
```

The JavaScript engine also interacts with browser-provided capabilities such as:

```js
document
fetch()
setTimeout()
localStorage
WebSocket
```

These are exposed through the browser/runtime environment.

---

## 6. Rendering Engine / Rendering Pipeline

The browser needs to turn HTML and CSS into something that can be displayed.

Simplified pipeline:

```text
HTML
 ↓
DOM

CSS
 ↓
CSSOM

DOM + CSSOM
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

This is the core mental model for browser rendering.

---

## 7. HTML → DOM

The browser receives HTML:

```html
<html>
  <body>
    <h1>Hello</h1>
    <p>Welcome</p>
  </body>
</html>
```

It parses the HTML and constructs the **DOM (Document Object Model)**.

Conceptually:

```text
Document
└── html
    └── body
        ├── h1
        │   └── "Hello"
        │
        └── p
            └── "Welcome"
```

The DOM is:

> A tree representation of the document structure that scripts and browser internals can work with.

JavaScript can manipulate it:

```js
document.querySelector("h1").textContent = "Hello Sonu";
```

A DOM change can cause additional rendering work.

---

## 8. CSS → CSSOM

The browser also parses CSS.

Example:

```css
h1 {
  color: red;
}

p {
  font-size: 16px;
}
```

The browser creates a representation called the **CSSOM — CSS Object Model**.

Conceptually:

```text
CSS
 ↓
Parser
 ↓
CSSOM
```

The CSSOM represents the browser's understanding of CSS rules and styles.

---

## 9. DOM + CSSOM → Render Tree

The DOM and CSSOM are used together to construct the **Render Tree**.

Important distinction:

> The DOM represents document structure; the Render Tree represents what participates in rendering.

Example:

```html
<div>
  <p>Hello</p>
  <p style="display: none">Secret</p>
</div>
```

Both `<p>` elements exist in the DOM, but the `display: none` element does not participate in the rendered output.

The details of the Render Tree are covered in Chapter 2.

---

## 10. Render Tree → Layout

Once the browser knows what needs to be rendered, it calculates:

> Where should everything go, and how large should everything be?

The Layout stage calculates things such as:

- Width
- Height
- Position
- X/Y coordinates
- Relationships between elements

Example:

```css
.box {
  width: 500px;
  height: 200px;
}
```

The browser needs to calculate the actual geometry of the element.

Layout is also commonly called **reflow**.

---

## 11. Layout → Paint

After calculating geometry, the browser determines:

> What should these things actually look like?

This is the **Paint** stage.

Painting involves visual details such as:

- Text
- Colors
- Backgrounds
- Borders
- Shadows
- Images

Mental model:

```text
Layout
 ↓
"Where and how big?"
 ↓
Paint
 ↓
"What should it look like?"
```

---

## 12. Paint → Composite

Modern browsers can divide rendering work into layers.

The **compositing** stage combines those layers into the final image.

Conceptually:

```text
Layer 1
Layer 2
Layer 3
   ↓
Composite
   ↓
Final frame
```

Compositing becomes particularly important for:

- `transform`
- `opacity`
- Animations
- Compositor layers
- GPU-related rendering

These topics are covered in more detail later.

---

## 13. Final Result: Pixels

The simplified rendering process is:

```text
HTML
 ↓
DOM
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

The final result is what appears on the screen.

---

## 14. Where JavaScript Fits

JavaScript is not simply the thing that "adds functionality."

JavaScript can modify structures involved in rendering.

Example:

```js
element.textContent = "Hello";
```

This changes the DOM.

Or:

```js
element.style.width = "500px";
```

This changes styling/geometry.

The browser then determines what rendering work is necessary.

Conceptually:

```text
             JavaScript
                  ↓
           DOM / CSS change
                  ↓
        Browser determines
        necessary rendering work
                  ↓
       Layout / Paint / Composite
                  ↓
                Pixels
```

Important:

> The browser does not necessarily redo the entire rendering pipeline after every change.

It determines what work is actually required.

---

## 15. Complete Mental Model

```text
                         WEB PAGE
                            │
             ┌──────────────┴──────────────┐
             ↓                             ↓
           HTML                           CSS
             ↓                             ↓
            DOM                           CSSOM
             └──────────────┬──────────────┘
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

JavaScript can enter the process by modifying DOM or styles:

```text
                     JavaScript
                          │
                          ↓
                  DOM / CSS changes
                          │
                          ↓
                Browser determines
                required rendering work
                          │
                 ┌────────┴────────┐
                 ↓                 ↓
               Layout            Paint
                 └────────┬────────┘
                          ↓
                       Composite
                          ↓
                        Pixels
```

---

## 16. Interview Cheat Sheet

### What is the DOM?

A tree representation of the HTML document.

### What is the CSSOM?

A representation of the CSS rules/styles parsed by the browser.

### What is the Render Tree?

A representation of the content that participates in rendering, combining relevant DOM information with styling information.

### What is Layout?

Calculating the geometry, size, and position of elements.

### What is Paint?

Determining the visual content that needs to be drawn, such as text, colors, borders, backgrounds, and images.

### What is Composite?

Combining rendered layers to produce the final frame displayed on screen.

### What happens when JavaScript modifies the DOM?

The browser determines what rendering work is necessary; depending on the change, this may involve layout, paint, and/or compositing.

---

## 17. Common Interview Traps

### "DOM is the rendered page."

No.

```text
HTML → DOM
```

DOM is a data structure representing the document.

### "Render Tree = DOM."

No.

The Render Tree represents what participates in rendering.

### "Every DOM change causes reflow."

No.

Different changes can trigger different amounts of rendering work.

### "Paint puts pixels directly on the screen."

Not quite.

The simplified pipeline is:

```text
Layout → Paint → Composite → Screen
```

### "JavaScript runs after rendering."

Not necessarily.

JavaScript can execute during page loading and can modify DOM/styles, causing additional rendering work.

---

## 18. One Diagram to Memorize

```text
HTML ──→ DOM ──────┐
                   ├──→ Render Tree
CSS  ──→ CSSOM ────┘
                         ↓
                       Layout
                         ↓
                        Paint
                         ↓
                     Composite
                         ↓
                       Pixels
```

And:

```text
JavaScript
    ↓
DOM / CSS changes
    ↓
Browser determines required work
    ↓
Layout → Paint → Composite
```

---

## Chapter 1 Status

**DONE**

Next chapter:

**Chapter 2 — DOM + CSSOM + Render Tree**

Topics:
- DOM construction
- CSSOM construction
- Style calculation
- Computed styles
- Render Tree
- `display: none`
- `visibility: hidden`
- pseudo-elements
- what participates in rendering
