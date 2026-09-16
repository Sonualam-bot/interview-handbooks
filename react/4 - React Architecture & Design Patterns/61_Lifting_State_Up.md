# Handbook 4 — React Architecture & Design Patterns

# Chapter 61 — Lifting State Up

## Core Idea

Lifting state up means moving state from a child into its **closest common parent** when multiple components need to share or coordinate that state.

The parent becomes the **single source of truth**.

```text
        Parent
       /      \
    Input    Display
       \      /
       shared state
```

---

## Deep Dive `[NEW]`

### "Receives Props It Doesn't Use" Actually Means "Re-renders For No Reason"

Section 13 notes that lifting too far makes unrelated intermediate
components "receive props they don't actually use" — the precise
performance consequence is stronger than that phrasing suggests. Every
component sitting on the path between the new state owner and the
actual consumer re-executes its render function on every update to
that state — not because it reads the prop, but simply because its
parent re-rendered and passed it *something* new to render (a
recreated `<Search />` element, forwarded down through `Layout` and
`Dashboard`). Reconciliation still has to walk into `Layout` and
`Dashboard`'s subtrees to confirm nothing meaningfully changed for
them, even if neither one ever touches the lifted state directly (see
Handbook 1/2, Reconciliation). `React.memo` can stop this — but only
if `Layout`/`Dashboard` are wrapped in it *and* their own props happen
to be unchanged, which is rarely true once they're the ones forwarding
the ever-changing state down further. This is the mechanical reason
"lift to the lowest common owner, not higher" is a real performance
rule, not only an organizational one: every extra level between the
state and its real consumers is an extra subtree paying a
reconciliation cost for a value it never displays.

## 1. Why Lift State Up?

Suppose:

```text
Parent
 ├── Input
 └── Display
```

If Input owns the value but Display also needs it, the siblings cannot naturally share that local state.

Instead:

```text
Parent
 ├── value
 ├── setValue
 ├── Input
 └── Display
```

Now the parent coordinates both components.

---

## 2. Basic Pattern

```jsx
function Parent() {
  const [value, setValue] = useState("");

  return (
    <>
      <Input
        value={value}
        onChange={setValue}
      />

      <Display value={value} />
    </>
  );
}
```

The parent owns the state while children receive it through props.

---

## 3. Data Flow

The most important flow is:

```text
Parent state
     ↓
   props
     ↓
  Child
     ↓
User interaction
     ↓
callback prop
     ↓
Parent setState
     ↓
new state
     ↓
new props
     ↓
Children update
```

Example:

```text
value = "React"
    ↓
Input receives value
    ↓
User types "React Hooks"
    ↓
Input calls onChange(...)
    ↓
Parent setValue(...)
    ↓
Parent state changes
    ↓
Display receives "React Hooks"
```

The parent coordinates the siblings.

---

## 4. Parent Owns State, Child Owns Interaction

A useful distinction:

- **Parent owns state**
- **Child handles interaction and reports changes**

Example:

```jsx
function Input({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={e => onChange(e.target.value)}
    />
  );
}
```

The Input does not own the state.

It reports:

> "The user typed this."

The parent decides how the state should change.

---

## 5. Connection to Controlled Components

Lifting state up frequently creates a controlled component:

```jsx
function Parent() {
  const [value, setValue] = useState("");

  return (
    <Input
      value={value}
      onChange={setValue}
    />
  );
}
```

Here:

```text
Parent
  ↓
owns state

Input
  ↓
controlled by parent
```

So:

```text
Lifting State Up
        ↓
Parent becomes state owner
        ↓
Child can become controlled
```

---

## 6. Example — Temperature Converter

Suppose:

```text
TemperatureConverter
 ├── CelsiusInput
 └── FahrenheitInput
```

If each input owns its own state:

```text
CelsiusInput
 └── celsius

FahrenheitInput
 └── fahrenheit
```

they can become inconsistent.

Instead, lift the state:

```text
TemperatureConverter
 └── temperature
      ├── CelsiusInput
      └── FahrenheitInput
```

Now:

```text
User changes Celsius
       ↓
Parent state updates
       ↓
Fahrenheit is derived
       ↓
Both inputs stay consistent
```

---

## 7. Single Source of Truth

Lifting state prevents multiple components from maintaining conflicting copies.

Instead of:

```text
Celsius → owns one value
Fahrenheit → owns another value
```

use:

```text
Parent
  ↓
one source of truth
```

Other representations can be derived from that state.

---

## 8. Avoid Duplicating Derived State

If a value can be calculated from existing state, do not automatically create another state variable.

Instead of:

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [fullName, setFullName] = useState("");
```

prefer:

```jsx
const fullName = `${firstName} ${lastName}`;
```

General principle:

> **Do not create duplicated state when the value can be derived from existing state.**

---

## 9. Closest Common Parent

Suppose:

```text
App
 └── Dashboard
      ├── SearchBox
      └── SearchResults
```

Both need:

```text
query
```

The state should usually live in:

```text
Dashboard
```

because Dashboard is their closest common parent.

```text
Dashboard
 └── query
      ├── SearchBox
      └── SearchResults
```

This is the direct connection with state colocation:

```text
State Colocation
→ Keep state close.

Lifting State Up
→ Move it to the closest common owner when sharing requires it.
```

---

## 10. Lifting State Up Does Not Mean Global State

This is important.

If:

```text
Dashboard
 ├── SearchBox
 └── SearchResults
```

need shared state, simply putting:

```jsx
const [query, setQuery] = useState("");
```

in Dashboard is enough.

You do not automatically need:

- Redux
- Context
- Zustand
- another global store

Therefore:

```text
Shared state
≠
Global state
```

Shared state can remain local to a feature.

---

## 11. When Should You Lift State?

Lift state when:

- sibling components need the same state
- multiple components must stay synchronized
- one component's interaction affects another
- you need a single source of truth
- a common parent should coordinate the behavior

Example:

```text
Accordion
 ├── Panel A
 ├── Panel B
 └── Panel C
```

If only one panel should be open, the Accordion can own:

```text
openPanel
```

---

## 12. When Shouldn't You Lift State?

Do not lift state simply because you can.

If only:

```text
SearchBox
```

needs:

```text
isFocused
```

keep it in SearchBox.

Do not move it unnecessarily to:

```text
Dashboard
```

or:

```text
App
```

This would violate the state-colocation principle.

---

## 13. Lifting Too Far

Suppose:

```text
App
 └── Layout
      └── Dashboard
           └── Search
```

If state is needed by distant branches, lifting it all the way to App can cause unrelated intermediate components to receive props they don't actually use.

This can lead to:

> **Prop drilling**

That is the topic of the next chapter.

So lifting state is correct when sharing requires it, but the state should still live at the **lowest appropriate common owner**.

---

## 14. Example — Accordion

Structure:

```text
Accordion
 ├── Panel 1
 ├── Panel 2
 └── Panel 3
```

Requirement:

> Only one panel may be open at a time.

If each panel owns its own `open` state, multiple panels can be open simultaneously.

Instead:

```text
Accordion
 └── openPanel
      ├── Panel 1
      ├── Panel 2
      └── Panel 3
```

Flow:

```text
User clicks Panel 3
       ↓
Panel 3 → onClick
       ↓
Accordion setOpenPanel(3)
       ↓
Accordion state changes
       ↓
new props
       ↓
Panel 3 receives "open"
       ↓
Other panels receive "closed"
```

The parent owns the rule because the rule affects multiple children.

---

## Interview Answer

> **Lifting state up means moving state from a child component into their closest common parent when multiple components need to share or coordinate that state. The parent becomes the single source of truth and passes the state and callbacks down through props. This keeps related components synchronized and avoids duplicated state.**

---

# Exercise — Shared Search State

Build:

```text
Dashboard
 ├── SearchBox
 └── SearchResults
```

Requirements:

- Dashboard owns `query`
- SearchBox receives `query` and `onChange`
- SearchResults receives `query`
- Typing in SearchBox updates SearchResults

Expected flow:

```text
User types
    ↓
SearchBox onChange
    ↓
Dashboard setQuery()
    ↓
Dashboard state changes
    ↓
new query prop
    ↓
SearchResults
    ↓
renders results for new query
```

Questions:

1. Why shouldn't SearchBox own `query`?
2. Why shouldn't SearchResults own `query`?
3. Why is Dashboard the correct owner?
4. What problems could arise if both children maintained their own copies?

---

# Exercise — Accordion

Build:

```text
Accordion
 ├── Panel A
 ├── Panel B
 └── Panel C
```

Requirement:

> Only one panel may be open at a time.

Think through:

```text
Where does openPanel live?
        ↓
Who changes it?
        ↓
How does a Panel communicate a click?
        ↓
How does Accordion tell Panels which one is open?
```

Expected architecture:

```text
Accordion
   │
   ├── openPanel
   ├── Panel A
   ├── Panel B
   └── Panel C
```

---

# Interview Checklist

- [ ] Definition of lifting state up
- [ ] Why sibling components may need a shared parent state owner
- [ ] Closest common parent
- [ ] Single source of truth
- [ ] Parent owns state, children receive props
- [ ] Callback props for child → parent communication
- [ ] Relationship with controlled components
- [ ] Avoiding duplicated state
- [ ] Shared state does not necessarily mean global state
- [ ] When not to lift state
- [ ] Relationship between state colocation and lifting state
- [ ] Why lifting state too far can cause prop drilling

---

# Final Mental Model

```text
State belongs to one component
        ↓
Keep it there.

Multiple components need it
        ↓
Find closest common parent
        ↓
Lift state there.

State has to travel through
many unrelated components
        ↓
You may have a prop-drilling problem.
```

> **Lift state to the closest common parent that needs to coordinate the components using that state.**
