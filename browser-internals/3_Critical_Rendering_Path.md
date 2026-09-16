# Chapter 3 — Critical Rendering Path

## 1. What is the Critical Rendering Path?

The **Critical Rendering Path (CRP)** is the sequence of steps the browser performs to convert web resources into pixels on the screen.

High-level flow:

```text
HTML + CSS + JavaScript
        ↓
       DOM
        ↓
      CSSOM
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

HTML parsing, CSS loading, and JavaScript execution can interact with this process and delay rendering.

---

## 2. HTML Parsing

The browser receives HTML and parses it incrementally.

```text
HTML bytes
    ↓
HTML Parser
    ↓
DOM
```

The browser does not necessarily wait for the entire HTML document before beginning to process what it has received.

---

## 3. Resource Discovery

While parsing HTML, the browser discovers external resources:

```html
<link rel="stylesheet" href="style.css">
<script src="app.js"></script>
<img src="hero.png">
```

The browser discovers resources such as:

```text
style.css
app.js
hero.png
```

and can begin fetching them.

A resource cannot be fetched until the browser discovers it, so resource discovery is important for performance.

---

## 4. CSS and the Critical Rendering Path

The browser parses CSS into the CSSOM:

```text
CSS
 ↓
CSS Parser
 ↓
CSSOM
```

Stylesheets are **render-blocking resources by default**.

Render-blocking does not mean the browser cannot download anything else or that HTML parsing must always stop completely.

It means a stylesheet can prevent the browser from producing the rendered page until the necessary CSS is available.

---

## 5. Why CSS Can Block Rendering

If the browser painted an unstyled page first and CSS arrived later, it might need to redo styling and visual work.

Conceptually:

```text
Unstyled page
     ↓
Style Calculation
     ↓
Layout
     ↓
Paint
```

Waiting for relevant CSS helps the browser avoid unnecessarily rendering an incomplete styled page.

---

## 6. JavaScript and HTML Parsing

A classic JavaScript script can interrupt HTML parsing.

```html
<h1>Hello</h1>

<script src="app.js"></script>

<p>Welcome</p>
```

The browser generally:

```text
Pause HTML parsing
       ↓
Fetch script if necessary
       ↓
Execute JavaScript
       ↓
Continue HTML parsing
```

This happens because JavaScript can modify the document while it is being parsed.

JavaScript can:

- Modify the DOM
- Remove nodes
- Insert nodes
- Change attributes
- Change styles
- Manipulate the document

---

## 7. `defer`

```html
<script src="app.js" defer></script>
```

With `defer`, the browser can download the script while HTML parsing continues.

```text
HTML parsing ─────────────────────→ complete
       │
       └── script download ───────────────→
                                           ↓
                                  script executes
```

### Key idea

> **`defer` downloads the script without blocking HTML parsing and executes it after HTML parsing is complete.**

Deferred scripts preserve their document order.

```html
<script src="a.js" defer></script>
<script src="b.js" defer></script>
```

Execution:

```text
a.js
 ↓
b.js
```

after parsing completes.

---

## 8. `async`

```html
<script src="analytics.js" async></script>
```

With `async`, the script downloads in parallel and executes as soon as it is ready.

```text
HTML parsing ─────────────────────────→
                         \ script download
              ↓
         download complete
              ↓
        execute immediately
              ↓
      parsing may be interrupted
```

### Key idea

> **`async` downloads the script in parallel and executes it as soon as it is ready.**

---

## 9. `async` vs `defer`

| Feature | `async` | `defer` |
|---|---|---|
| Downloads in parallel | Yes | Yes |
| Blocks HTML parsing while downloading | No | No |
| Can interrupt HTML parsing for execution | Yes | No |
| Executes after HTML parsing | Not necessarily | Yes |
| Execution order between multiple scripts | Not guaranteed | Preserves document order |
| Good for | Independent scripts | DOM-dependent application scripts |

### Mental model

```text
async
→ Download it and run whenever it is ready.

defer
→ Download it, but wait until parsing is finished.
```

---

## 10. `DOMContentLoaded`

```js
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM ready");
});
```

`DOMContentLoaded` fires when the document has been completely parsed and deferred scripts have executed.

It does **not** mean every image or other resource has finished loading.

---

## 11. `load`

```js
window.addEventListener("load", () => {
  console.log("Page resources loaded");
});
```

`load` represents a later page-loading milestone and waits for the page's resources to finish loading as applicable.

Therefore:

```text
DOMContentLoaded
→ Document parsed + deferred scripts completed

load
→ Later resource-loading milestone
```

They are not equivalent.

---

## 12. Critical Rendering Path

Conceptually:

```text
HTML arrives
     ↓
HTML parsing
     ↓
DOM
     ↓
CSS parsing
     ↓
CSSOM
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

JavaScript can affect this process:

```text
Classic script
     ↓
Can pause HTML parsing
     ↓
Execute
     ↓
Parsing continues
```

While:

```text
defer
     ↓
Download in parallel
     ↓
HTML parsing completes
     ↓
Execute
```

And:

```text
async
     ↓
Download in parallel
     ↓
Execute whenever ready
     ↓
May interrupt parsing
```

---

## 13. What Does "Critical" Mean?

A critical path consists of work that must happen before a particular user-visible result can be produced.

For initial rendering, conceptually:

```text
HTML
 ↓
DOM
 ↓
CSS
 ↓
CSSOM
 ↓
Render Tree
 ↓
Layout
 ↓
Paint
```

If something unnecessarily delays this work, the first visible rendering can be delayed.

### Core performance principle

> **Reduce, defer, parallelize, or eliminate unnecessary work on the critical path.**

---

## 14. Bad Critical Path Example

```html
<head>
  <link rel="stylesheet" href="huge.css">
  <script src="huge.js"></script>
</head>
```

A possible sequence:

```text
HTML parsing
     ↓
CSS request
     ↓
CSS download
     ↓
JS request
     ↓
JS download
     ↓
JS execution
     ↓
HTML parsing continues
     ↓
DOM complete
     ↓
Render
```

A lot of work can delay the page.

---

## 15. Better Approach

For an application script that does not need to execute during parsing:

```html
<head>
  <link rel="stylesheet" href="critical.css">
  <script src="app.js" defer></script>
</head>
```

Conceptually:

```text
HTML parsing ─────────────────────→ DOM complete
       │
       ├── CSS download
       │
       └── JS download
                    ↓
              JS executes
```

The browser can perform more work in parallel.

### Core principle

> **Do not unnecessarily serialize work that can happen in parallel.**

---

## 16. Performance Mindset

The browser's goal is not simply to finish everything quickly.

From a user perspective, important goals include:

- Get useful content on screen quickly.
- Keep the page responsive.
- Avoid unnecessary work before the first useful render.

This leads into:

- First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)
- Core Web Vitals
- Resource prioritization
- `preload`
- Code splitting
- Lazy loading
- Caching

---

## 17. Interview Answer: "How Does a Browser Render a Page?"

> The browser parses HTML into the DOM and CSS into the CSSOM. It performs style calculation and constructs the rendering representation, then calculates layout, paints visual content, and composites layers into pixels. During this process, resources such as CSS and JavaScript can affect the critical rendering path. Classic scripts can block HTML parsing, while `defer` scripts execute after parsing and `async` scripts execute as soon as they are downloaded.

---

## 18. Common Interview Traps

### Does JavaScript always block rendering?

**No.**

Classic synchronous scripts can block HTML parsing. `async` and `defer` change script behavior, and the actual rendering impact depends on the script and where it appears.

### Does `async` mean the script blocks nothing?

**No.**

Its download does not block HTML parsing, but its execution can interrupt parsing.

### Does `defer` download the script after HTML parsing?

**No.**

The script can download while HTML is being parsed. It waits to execute until parsing completes.

### Does `DOMContentLoaded` wait for images?

**No.**

It represents the document-parsing milestone plus completion of deferred scripts.

### Are `DOMContentLoaded` and `load` the same?

**No.**

They represent different points in the page-loading lifecycle.

---

# 19. Chapter 3 Cheat Sheet

## Rendering

```text
HTML
 ↓
DOM

CSS
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

## Scripts

```text
<script>
→ Can pause HTML parsing
→ Download + execute
→ Parsing resumes
```

```text
<script defer>
→ Download in parallel
→ HTML parsing completes
→ Execute
```

```text
<script async>
→ Download in parallel
→ Execute as soon as ready
→ May interrupt parsing
```

## Remember

> **async = execute when ready**

> **defer = execute after parsing**

> **Critical rendering path = work that affects getting pixels onto the screen**

> **Performance optimization = reduce, defer, parallelize, or eliminate unnecessary critical-path work**
