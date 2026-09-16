# 05_JSX_Deep_Dive_Notes

## Definition

JSX (JavaScript XML) is a syntax extension for JavaScript used to
describe React Elements. It is **not HTML** and is compiled into
JavaScript before execution.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why a Syntax Extension Instead of Plain `createElement()` Calls

You could write every UI in raw `createElement` calls — React does
not require JSX. But nested UI expressed as nested function calls
becomes unreadable fast:

``` js
createElement('div', null,
  createElement('h1', null, 'Title'),
  createElement('p', null, 'Body'))
```

JSX exists purely so the *shape* of the code visually matches the
*shape* of the UI tree it describes. It buys nothing at runtime —
it's a compile-time convenience that disappears entirely before the
browser ever sees it.

### Why Expressions Only, Never Statements

`{}` in JSX is a slot for a single JavaScript **value** — because
under the hood, everything inside becomes an *argument* to a function
call (`jsx(type, props)`). A function argument must evaluate to a
value; `if`, `for`, and `switch` don't evaluate to anything, they
*execute*. That's not a React rule bolted on — it's a direct
consequence of JSX compiling to function calls, which is why `if` has
to be pulled out above `return` as a statement, while ternaries,
`&&`, and function calls all work inline (see Conditional Rendering).

### Why Exactly One Root Element

A component function returns one value. `jsx()` calls compose into a
single nested object graph — there's no way to "return two objects"
from one function call without wrapping them in something. A
`Fragment` (`<>...</>`) is that wrapper: it's a real element type that
produces no DOM node of its own, existing solely so multiple siblings
can be handed back as one value.

### Traced Example: What the Compiler Actually Produces

``` jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

compiles to roughly:

``` js
function Greeting({ name }) {
  return jsx('h1', { children: ['Hello, ', name, '!'] });
}
```

`{name}` was evaluated by plain JavaScript *before* `jsx()` was ever
called — React never sees the expression `{name}`, only its already-
computed result. This is the real mechanism behind "JavaScript
evaluates expressions before React creates React Elements": there is
no other order possible, because `{name}` is just an argument being
passed into a function call, and arguments are always evaluated
before the call executes.

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
