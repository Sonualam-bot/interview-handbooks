# Chapter 2 Notes --- Declarative vs Imperative

## Declarative vs Imperative

Imperative = HOW Declarative = WHAT

## Deep Dive `[NEW]`

### Same Goal, Two Ways to Get There

Take "highlight the selected item in a list."

**Imperative** (how):

``` js
for (const el of items) {
  if (el.dataset.id === selectedId) el.classList.add('selected');
  else el.classList.remove('selected');
}
```

You are personally responsible for walking every element and deciding
its state, every time selection changes, from any code path that can
change it.

**Declarative** (what):

``` jsx
{items.map(item => (
  <li className={item.id === selectedId ? 'selected' : ''} />
))}
```

You state the rule once. The result is re-derived whenever the render
function runs — there is no "remove the class from the previously
selected item" step, because there is no "previous" state to track by
hand.

### Why This Distinction Matters More Than It Sounds

The imperative version has a hidden invariant: *every place that
changes `selectedId` must remember to re-run this exact loop.* Miss
one, and the highlight goes stale. The declarative version has no
such invariant — it's re-evaluated as a byproduct of the state change
flowing through the render pipeline, not because a specific piece of
code remembered to call it. This is the actual mechanism behind
"declarative improves consistency" (below) — consistency isn't a
vague virtue, it's the direct result of removing "remember to update
this" as a requirement.

### Where Imperative Code Still Lives

React doesn't eliminate imperative code — it pushes it to the
boundary. Something, somewhere, still calls `appendChild` /
`setAttribute` on real DOM nodes. React's reconciler does that work
for you, imperatively, based on the declarative tree you handed it.
You write declarative code; React executes imperative code on your
behalf. That's the seam behind "Render ≠ DOM Update" below.

## Pipeline

``` text
State
 ↓
Component
 ↓
React
 ↓
DOM
```

## Key Points

-   Declarative improves consistency.
-   Manual DOM updates don't scale.
-   Render ≠ DOM Update.

## Interview Nuggets

React manages DOM; it doesn't eliminate it.

## Flashcards

-   Declarative? → Describe desired UI.
