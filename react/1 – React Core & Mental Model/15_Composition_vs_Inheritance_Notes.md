# 15_Composition_vs_Inheritance_Notes

## Definition

React **prefers composition over inheritance**. Complex UIs are built by
combining smaller, reusable components rather than extending existing
ones.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why Inheritance Fails For UI Specifically

Class inheritance works well when subtypes are genuinely "is-a"
specializations of one shared, stable interface (`Dog extends
Animal`). UI components rarely fit that shape: a `Modal` isn't a
specialized `Card`, and a `Card` isn't a specialized `Panel` — they
*contain* each other, or sit beside each other, in different
arrangements depending on the screen. Forcing that into inheritance
means picking one rigid hierarchy upfront
(`SpecialCard extends Card extends Panel`) that has to anticipate
every future combination, and adding a new combination later often
means restructuring the whole chain. Composition sidesteps the
question entirely: nothing has to predict the hierarchy, because
pieces are assembled at the call site, per use, not fixed at
definition time.

### children Is Just props, Which Is Just an Object

The reason composition "just works" in React without a special
templating/slot system is that it rides entirely on the mechanism
already covered in Props notes: nested JSX becomes `props.children`,
full stop. `<Card><Header/></Card>` and
`<Layout header={<Header/>} />` are the *same trick* — passing an
element as the value of a prop — just with `children` being the
implicit, positional version of it. Composition isn't a separate
React feature layered on top of props; it's props, applied to values
that happen to be other elements.

### Recursive Resolution, Why It Terminates

``` text
<Card><Header/></Card>
  React executes Card(props) → Card returns <div>{props.children}</div>
  → that's <div><Header/></div>
  React executes Header(props) → Header returns <header>...</header>, which
  contains only DOM element types (string types like "header", "div"), not
  component types.
```

React keeps calling component functions on whatever they return, only
stopping recursion at element types that are strings ("div",
"header") — a string type has no further function to call; it's a
direct instruction to create a real DOM node. Every component tree,
no matter how deeply composed, bottoms out for this reason: the
recursion is structurally guaranteed to end at native elements,
because that's the only kind of element type React can't recurse into
further.

### Why "Prefer Composition" Isn't Anti-OOP Dogma

React function components can't be meaningfully subclassed anyway
(there's no instance to extend), so "prefer composition" isn't really
a stylistic opinion — it's a description of the only mechanism that
was actually available once components became functions returning
element trees rather than class hierarchies.

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
