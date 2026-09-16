# Chapter 4 Notes --- Virtual DOM

## Definition

Virtual DOM is React's in-memory tree of React Elements.

## Deep Dive `[NEW]`

### Why Not Just Update the Real DOM Directly From State?

You could write `if (count changed) document.querySelector('p').textContent
= count`. That's exactly the imperative approach React exists to avoid
(see Declarative vs Imperative). The other extreme — re-render
everything into the DOM from scratch on every state change — is
correct but catastrophically slow: real DOM nodes are heavyweight
objects that trigger layout/style recalculation, and re-creating a
whole tree loses focus, scroll position, input state, and CSS
transition state.

The Virtual DOM is the middle path: build a *cheap* in-memory
description of what the UI should look like (React Elements), compare
it to the previous cheap description, and only touch the *expensive*
real DOM for the parts that actually differ.

### Reconciliation Is a Heuristic, Not a Perfect Diff

A mathematically optimal tree-diff algorithm is O(n³) for n nodes —
far too slow to run on every keystroke. React's reconciler uses two
cheap heuristics that are "good enough" almost all the time:

1.  **Different element types at the same position → discard the
    whole subtree and rebuild.** `<div>` → `<span>` at the same spot
    doesn't try to figure out what's reusable inside; it discards and
    remounts. This is why swapping element types resets state.
2.  **Same type → compare props, keep the DOM node, recurse into
    children.** `<div className="a">` → `<div className="b">`
    patches just the `className` attribute on the existing node.

For lists, `key` is what lets the heuristic match children across
renders by identity instead of position (see Lists & Keys).

### Fiber: Why "Commit" Is Its Own Step

Modern React (Fiber architecture) splits work into two phases, which
is why the pipeline below has a separate Commit stage at all:

-   **Render/Reconciliation phase** — can be paused, resumed, or
    thrown away (e.g. a newer update supersedes it). No visible side
    effects yet.
-   **Commit phase** — synchronous, uninterruptible, actually mutates
    the DOM.

This split is why "Render ≠ DOM Update ≠ Browser Paint" — each is a
genuinely separate phase with different performance characteristics
and interruptibility guarantees, not just a conceptual nicety.

### Traced Example

``` text
Old element tree: <p>Count: 3</p>
New element tree: <p>Count: 4</p>

Diff: same type "p", props.children differs ("3" vs "4")
      → schedule: update this text node's content
Commit: textNode.textContent = "4"
```

Nothing else in the DOM is touched — no new node is created, no
attribute other than the changed text is written.

## Pipeline

``` text
State
 ↓
React Elements
 ↓
Virtual DOM
 ↓
Reconciliation
 ↓
Commit
 ↓
Real DOM
```

## Key Points

-   Virtual DOM is not browser DOM.
-   React compares React Element trees.
-   Render ≠ DOM Update.
-   DOM Update ≠ Browser Paint.

## Flashcards

-   Reconciliation compares? → React Element trees.
