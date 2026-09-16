# Chapter 1 Notes --- What is React?

## One-Line Definition

React is a JavaScript library for building user interfaces using
reusable, declarative components.

## Deep Dive `[NEW]`

### The Problem Before React

Before React (the jQuery/vanilla-DOM era), UI updates were imperative:
code described every DOM mutation step by step — find the node, check
its current state, decide what changed, mutate it. As an app grew, the
number of possible UI states multiplied, and each transition needed
its own hand-written synchronization code. Two symptoms followed:

1.  **State/DOM drift** — in-memory data and the actual DOM slowly
    went out of sync because some code path forgot to update a node.
2.  **Combinatorial handler growth** — every new piece of interactive
    state required auditing all the places that might need to change
    because of it.

This is the disease React was written to cure — not "the DOM is
slow" (a common misconception, already flagged below), but "manually
keeping DOM in sync with data doesn't scale past a certain
complexity."

### The Reasoning Behind UI = f(state)

If you say "here is my data, and here is a pure function describing
what the UI should look like for any given data," the drift problem
disappears structurally: no code path can forget to update the UI,
because the UI is *derived*, not *maintained*. Same idea as a
spreadsheet formula — you don't write "when B2 changes, update C2";
you write `C2 = B2 * 2` once, and it stays true.

React's job is to take the *output* of that function (a description
of the desired UI) and compute the minimal set of real DOM operations
to make the actual DOM match. That step — reconciliation — is what
makes `UI = f(state)` practical instead of re-creating the whole DOM
tree on every change (see Virtual DOM notes).

### Why "Library," Not "Framework"

A framework calls your code (inversion of control over the whole app:
routing, data fetching, build tooling). React only provides the
rendering model; you assemble everything else yourself, and you call
React's APIs — not the other way around for app structure. This is
why React can be dropped into a single widget on an existing
server-rendered page, which a framework couldn't do.

### Traced Example

``` text
data:  { count: 3 }
f:     count => <p>Count: {count}</p>
UI:    <p>Count: 3</p>

data changes: { count: 4 }
f runs again → <p>Count: 4</p>
React compares "Count: 3" vs "Count: 4" text node → patches just that text node
```

Nowhere in this flow was there an instruction to "update the
paragraph." Only a description of what the paragraph should say for a
given count.

## Core Ideas

-   Solves UI management complexity.
-   UI = f(State).
-   Component-based architecture.
-   React is a library.

## Mental Model

``` text
State
 ↓
Component
 ↓
UI
```

## Interview Nuggets

-   React is a library.
-   React focuses on UI.
-   State is the source of truth.

## Common Mistakes

-   React exists because DOM is slow ❌
-   React exists to keep UI synchronized ✅

## Flashcards

-   React or Framework? → Library
-   UI=f(?) → State
