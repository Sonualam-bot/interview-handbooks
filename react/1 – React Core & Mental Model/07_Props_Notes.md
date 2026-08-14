# 07_Props_Notes

## Definition

Props (Properties) are read-only inputs passed from a parent component
to configure a child component.

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
