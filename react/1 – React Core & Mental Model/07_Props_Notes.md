# 07_Props_Notes

## Definition

Props (Properties) are read-only inputs passed from a parent component
to configure a child component.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why Read-Only Is Enforced, Not Just Convention

If a child could mutate the props object it received, the parent's
element tree from *this* render and the props object the child holds
would diverge — the parent thinks it passed `{name: "Sonu"}`, but the
child mutated it to `{name: "changed"}` in place. Now two parts of the
tree disagree about the current value of the same conceptual piece of
data, with no single source of truth. React's re-render model depends
on "props reflect what the parent most recently decided to pass"
being reliably true; mutation breaks that guarantee. This is why
props being read-only isn't a style preference — it's what keeps
one-way data flow actually one-way.

### Why One Props Object, Not Separate Arguments

`Button(name, age, onClick, className, ...)` doesn't scale and has no
room for optional/named parameters that don't collide positionally. A
single object gives React (and you) a stable, inspectable, spreadable
unit — `{...props}`, default values via destructuring, and
prop-drilling patterns (Lifting State Up, Composition) all depend on
props being one addressable object rather than loose arguments.

### children Is Not Magic

`<Card><h1>Hello</h1></Card>` is exactly equivalent to `<Card
children={<h1>Hello</h1>} />` — the compiler treats nested JSX as
sugar for a `children` prop, nothing more. This is *why* composition
(see file 15) works through ordinary props: there's no separate
"slot" mechanism in React, just this one convention, which is also
what lets you pass components through props with other names (e.g.
`<Layout header={<Header />} />`) using the exact same mechanism.

### Traced Example

``` jsx
<UserCard name="Sonu" age={26} />
```

becomes the element:

``` js
{ type: UserCard, props: { name: "Sonu", age: 26 } }
```

React later calls `UserCard({ name: "Sonu", age: 26 })`. If
`UserCard` does `props.name = "changed"` internally, it mutates the
object React is holding as "what this fiber was rendered with" — on
the next render, React still compares against the *parent's* new
props object, so the mutation is silently lost and creates a
one-render flash of wrong data at best, a hard-to-trace bug at worst.
That's the concrete failure mode behind "never mutate props."

------------------------------------------------------------------------

## Mental Model

``` text
Parent Component
        ↓
Props
        ↓
Child Component
        ↓
React Elements
```

------------------------------------------------------------------------

## Core Concepts

### Why Props?

-   Make components reusable.
-   Allow multiple instances of the same component with different data.

### Internal Flow

``` text
<UserCard name="Sonu" age={26} />
        ↓
React Element
{
  type: UserCard,
  props: {
    name: "Sonu",
    age: 26
  }
}
        ↓
React executes:
UserCard(props)
```

### Props are Read-only

-   Never mutate props.
-   Parent passes new props when data changes.

### Props vs State

  Props            State
  ---------------- ---------------------
  Parent-owned     Component-owned
  Read-only        Updated with setter
  External input   Internal memory

### Children Prop

``` jsx
<Card>
  <h1>Hello</h1>
</Card>
```

becomes:

``` js
props = {
  children: <h1>Hello</h1>
}
```

------------------------------------------------------------------------

## Execution Flow

``` text
Parent Executes
      ↓
Creates Child React Element
      ↓
Props attached
      ↓
React executes Child(props)
      ↓
Returns more React Elements
```

------------------------------------------------------------------------

## Interview Nuggets

-   Props are inputs to components.
-   React passes a single props object.
-   `children` is just another prop created automatically.
-   Data flows from parent to child.

------------------------------------------------------------------------

## Common Mistakes

❌ Mutating props.

✅ Treat props as immutable.

❌ Confusing props with state.

✅ Props come from the parent; state belongs to the component instance.

------------------------------------------------------------------------

## Flashcards

**Q:** What are props?

**A:** Read-only inputs passed from parent to child.

**Q:** What is the `children` prop?

**A:** Everything nested between a component's opening and closing tags.

**Q:** Why can two instances of the same component display different
data?

**A:** Each instance receives its own props object.

------------------------------------------------------------------------

## 30-Second Revision

-   Props = component inputs.
-   Props are immutable.
-   React passes one props object.
-   Parent → Child data flow.
-   `children` is a normal prop.
