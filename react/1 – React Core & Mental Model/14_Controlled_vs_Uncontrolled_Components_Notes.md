# 14_Controlled_vs_Uncontrolled_Components_Notes

## Definition

The difference is **who owns the source of truth**.

-   Controlled → React State
-   Uncontrolled → Browser DOM

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### The Actual Trade-off, Not Just "Who Owns It"

Every re-render triggered by a controlled input costs something:
React runs the component function, diffs, and commits — for every
keystroke, of a form that might have thirty fields. For most UIs this
is fast enough to be free, but not literally free. That's *why*
uncontrolled components are a real option, not a legacy pattern: for
very large forms, for third-party non-React widgets that expect to own
their own DOM state (a canvas library, a map widget), or for values
you only need to *read once* (e.g. on submit) rather than track
continuously, paying a re-render per keystroke to keep React's state
in sync is pure overhead with no benefit — React never needed to know
the value until the very end.

### Why ref, Specifically

A `ref` (`useRef`) gives a stable handle to the actual DOM node
without asking React to track its value as state (state is what
triggers re-renders — see State notes; a ref changing does not).
Reading `inputRef.current.value` at submit time asks the DOM directly
for whatever the browser has been holding onto the whole time — the
DOM was always the source of truth, React just never bothered to ask
until it needed the value.

### Traced Comparison

``` text
Controlled:
  keystroke → onChange → setState → re-render → React writes value prop back to DOM
  (DOM value is always a reflection of React state)

Uncontrolled:
  keystroke → browser updates its own internal input value directly
  (React does nothing — no event handler even required)
  ...later, on submit:
  handleSubmit → reads inputRef.current.value → gets whatever the DOM currently holds
```

Nothing computed the uncontrolled input's value on every keystroke —
native browser input behavior did all of that work for free, which is
exactly the performance case for choosing it.

### Why File Inputs Must Be Uncontrolled

`<input type="file">`'s value is a `FileList` the browser controls for
security reasons (JavaScript cannot programmatically set what file a
user "selected"). Since React can never legally write a `value` back
into a file input, controlling it is structurally impossible — you
can only read via `ref`, making this the one input type that's
uncontrolled by necessity, not preference.

------------------------------------------------------------------------

## Mental Model

``` text
Controlled:
User Types → Browser Event → React onChange → setState() → React Stores State → Re-render → New React Element Tree → Reconciliation → Commit → DOM

Uncontrolled:
User Types → Browser Updates DOM → Value Stored in DOM → React Reads ref.current.value when needed
```

------------------------------------------------------------------------

## Core Concepts

### Controlled

-   Uses `value` and `onChange`.
-   React state owns the value.
-   Every update triggers a re-render.

### Uncontrolled

-   Uses `ref`.
-   Browser DOM owns the value.
-   React reads it only when required.

------------------------------------------------------------------------

## Interview Nuggets

-   Controlled = State is the source of truth.
-   Uncontrolled = DOM is the source of truth.
-   Controlled components are preferred for validation and predictable
    UI.
-   File inputs are commonly uncontrolled.

------------------------------------------------------------------------

## Common Mistakes

-   Controlled components do **not** update the DOM directly; they
    update state first.
-   Uncontrolled components are valid React patterns for specific use
    cases.

------------------------------------------------------------------------

## Flashcards

**Q:** Source of truth in controlled components?

**A:** React state.

**Q:** Source of truth in uncontrolled components?

**A:** Browser DOM.

**Q:** Why are controlled components preferred?

**A:** Predictable state, validation, and synchronization.

------------------------------------------------------------------------

## 30-Second Revision

-   Controlled → State owns value.
-   Uncontrolled → DOM owns value.
-   Controlled re-renders on input.
-   Uncontrolled uses refs.
