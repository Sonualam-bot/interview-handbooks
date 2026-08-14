# 05_JSX_Deep_Dive_Notes

## Definition

JSX (JavaScript XML) is a syntax extension for JavaScript used to
describe React Elements. It is **not HTML** and is compiled into
JavaScript before execution.

------------------------------------------------------------------------

## Mental Model

``` text
JSX
↓
Compiler
↓
jsx() / React.createElement()
↓
React Element
↓
Virtual DOM
↓
Reconciliation
↓
Commit
↓
Real DOM
```

------------------------------------------------------------------------

## Core Concepts

### JSX is NOT HTML

-   JSX is JavaScript syntax.
-   Browsers cannot execute JSX directly.
-   JSX is compiled before execution.

### JSX is Syntax Sugar

``` jsx
<h1>Hello</h1>
```

Conceptually becomes:

``` js
jsx("h1", {
  children: "Hello"
});
```

which creates a React Element.

### JSX Produces React Elements

``` js
{
  type: "h1",
  props: {
    children: "Hello"
  }
}
```

### Uppercase vs Lowercase

`<div />` → `type: "div"`

`<Button />` → `type: Button`

### JavaScript Inside JSX

Valid expressions:

``` jsx
{name}
{count + 1}
{condition ? "A" : "B"}
{items.map(...)}
```

Invalid statements:

``` jsx
if (...) {}
for (...) {}
switch (...) {}
```

### One Root Element

Components return one root React Element.

Use a Fragment for multiple siblings.

------------------------------------------------------------------------

## Execution Flow

``` text
React Executes Component
        ↓
JavaScript Evaluates Expressions
        ↓
JSX Transform
        ↓
React Elements
        ↓
Virtual DOM
```

------------------------------------------------------------------------

## Interview Nuggets

-   JSX is not HTML.
-   JSX is not executed by the browser.
-   JSX compiles into JavaScript.
-   Components return one root React Element.
-   JavaScript evaluates expressions before React creates React
    Elements.

------------------------------------------------------------------------

## Common Mistakes

❌ JSX creates DOM nodes.

✅ JSX creates React Elements.

❌ React evaluates JSX expressions.

✅ JavaScript evaluates expressions first.

❌ Components return multiple root elements.

✅ Components return one root React Element.

------------------------------------------------------------------------

## Flashcards

**Q:** What is JSX?

**A:** A syntax extension for JavaScript that describes React Elements.

**Q:** Does JSX become HTML?

**A:** No. It becomes JavaScript.

**Q:** Why uppercase component names?

**A:** They identify custom React components.

**Q:** Why can't `if` be used directly inside JSX?

**A:** Because JSX accepts expressions, not statements.

**Q:** Who evaluates `{count + 1}`?

**A:** JavaScript.

------------------------------------------------------------------------

## 30-Second Revision

-   JSX is JavaScript.
-   JSX is syntax sugar.
-   JSX compiles into JavaScript.
-   JSX creates React Elements.
-   JavaScript evaluates expressions first.
-   Components return one root React Element.
