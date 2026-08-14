# 15_Composition_vs_Inheritance_Notes

## Definition

React **prefers composition over inheritance**. Complex UIs are built by
combining smaller, reusable components rather than extending existing
ones.

------------------------------------------------------------------------

## Mental Model

``` text
Small Components
        ↓
Composition
        ↓
Large UI
```

------------------------------------------------------------------------

## Core Concepts

### What is Composition?

Composition means building complex components by combining simpler ones.

``` jsx
<Card>
  <Header />
  <Profile />
  <Actions />
</Card>
```

------------------------------------------------------------------------

### JSX to React Elements

``` jsx
<Card>
  <Header />
  <Profile />
  <Actions />
</Card>
```

Conceptually becomes:

``` js
{
  type: Card,
  props: {
    children: [
      { type: Header, props: {} },
      { type: Profile, props: {} },
      { type: Actions, props: {} }
    ]
  }
}
```

------------------------------------------------------------------------

### Recursive Component Resolution

``` text
App()
    ↓
Card React Element
    ↓
Execute Card()
    ↓
<div>
    ↓
Execute Header()
    ↓
<header>
    ↓
Execute Profile()
    ↓
<main>
    ↓
Execute Actions()
    ↓
<button>
```

React continues recursively until only native DOM elements remain.

------------------------------------------------------------------------

### Final Conceptual React Element Tree

``` js
{
  type: "div",
  props: {
    className: "card",
    children: [
      {
        type: "header",
        props: { children: "Header" }
      },
      {
        type: "main",
        props: { children: "Profile" }
      },
      {
        type: "button",
        props: { children: "Edit" }
      }
    ]
  }
}
```

------------------------------------------------------------------------

## Composition vs Inheritance

  Composition                 Inheritance
  --------------------------- ------------------------------
  Combines components         Extends classes
  Uses `children` and props   Uses `extends`
  Preferred in React          Rarely used for UI
  Flexible and reusable       Can create rigid hierarchies

------------------------------------------------------------------------

## Interview Nuggets

-   React recommends composition over inheritance.
-   `children` is the most common composition mechanism.
-   Components can also be composed by passing components as props.
-   React recursively resolves the component tree into native DOM
    elements.

------------------------------------------------------------------------

## Common Mistakes

❌ Build one giant component.

✅ Split UI into focused, reusable components.

❌ Think inheritance is React's preferred reuse model.

✅ Prefer composition for reusable UI.

------------------------------------------------------------------------

## Flashcards

**Q:** Why does React prefer composition?

**A:** Because UIs are naturally built by combining independent
components, making them more reusable and maintainable.

**Q:** What enables flexible composition?

**A:** `props.children` and passing components through props.

**Q:** What does React recursively resolve?

**A:** The component tree until only browser (DOM) elements remain.

------------------------------------------------------------------------

## 30-Second Revision

-   Build large UIs from small reusable components.
-   Composition is React's preferred reuse model.
-   `children` enables flexible composition.
-   React recursively executes components until only native DOM elements
    remain.
