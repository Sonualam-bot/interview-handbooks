# Chapter 3 Notes --- React Elements

## Definition

A React Element is an immutable JavaScript object describing UI.

## Deep Dive `[NEW]`

### Why a Plain Object, Not a DOM Node

If `createElement('button', {}, 'Click')` returned a real `<button>`
DOM node immediately, two problems follow: (1) React couldn't compare
"what should the UI look like" against "what does it currently look
like" without expensive DOM reads, since the description and the
actual thing would be the same object; (2) creating elements would be
as expensive as touching the real DOM, even for elements that never
end up changing.

By making an element a cheap, plain JS object — just `{ type, props
}` — React gets a lightweight, disposable description it can create
by the thousands every render, diff against the previous tree, and
throw away, without ever touching the browser until it knows exactly
what needs to change.

### Immutability Is Load-Bearing, Not Stylistic

Because an element is immutable, React can hold a reference to "last
render's tree" and compare it against "this render's tree" without
worrying that someone mutated the old tree out from under it mid-diff.
If elements were mutable, comparing old vs. new would require
defensive copies everywhere — immutability is what makes cheap
structural comparison possible in the first place.

### Traced Example

``` jsx
<button className="primary">Click</button>
```

compiles to (roughly):

``` js
jsx('button', { className: 'primary', children: 'Click' })
```

which returns:

``` js
{
  type: 'button',
  key: null,
  props: { className: 'primary', children: 'Click' }
}
```

This object is never mutated after creation. If `className` needs to
change, React doesn't edit this object — the component function runs
again and produces a *new* element object with `className:
'secondary'`. The two objects are then compared (see Virtual DOM /
Reconciliation).

### Element vs Component vs Instance

This one distinction untangles most early confusion:

-   **Element** — the plain object describing what to render
    (disposable, recreated every render).
-   **Component** — the function/class definition
    (`function Button() {}`) — exists once in your source.
-   **Instance** — React's internal bookkeeping (a fiber node) for one
    particular `<Button />` in the tree, which is what actually
    persists state across renders (see Components and State notes).

## JSX Pipeline

``` text
JSX
 ↓
Compiler
 ↓
React.createElement()/jsx()
 ↓
React Element
```

## Example

``` js
{ type:'button', props:{ children:'Click'} }
```

## Key Points

-   JSX is not HTML.
-   Components return React Elements.
-   React Elements are immutable.

## Flashcards

-   React.createElement returns? → React Element
