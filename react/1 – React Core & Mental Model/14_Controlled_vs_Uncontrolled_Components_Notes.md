# 14_Controlled_vs_Uncontrolled_Components_Notes

## Definition

The difference is **who owns the source of truth**.

-   Controlled → React State
-   Uncontrolled → Browser DOM

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
