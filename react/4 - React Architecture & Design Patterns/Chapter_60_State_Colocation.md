# Handbook 4 — React Architecture & Design Patterns

# Chapter 60 — State Colocation

## 1. Core Idea

**State colocation** means keeping state as close as possible to the components that actually use it.

The fundamental question is:

> **Where should this state live?**

Rule of thumb:

> **Keep state as local as possible, and lift it only when necessary.**

---

## 2. Why State Colocation Matters

Putting state unnecessarily high in the tree can create:

- unnecessary prop passing
- component coupling
- broader render work
- unclear ownership
- unnecessary state-management complexity

Prefer:

```text
App
 ├── Header
 ├── Sidebar
 └── Profile
      └── local state
```

over putting Profile-only state in App.

The goal is not to make all state local at any cost; it is to avoid making state ownership broader than necessary.

---

## 3. The Decision Process

Ask:

### Who needs this state?

**One component:**

```text
→ Keep it local.
```

**Multiple siblings:**

```text
→ Lift it to their closest common parent.
```

**Many distant components:**

```text
→ Consider Context or an external store,
  depending on the application's needs.
```

---

## 4. State Colocation and Re-renders

Example:

```jsx
function App() {
  const [name, setName] = useState("");

  return (
    <>
      <Header />
      <Profile name={name} />
    </>
  );
}
```

If `name` changes, App renders again.

If the state only belongs to Profile:

```jsx
function Profile() {
  const [name, setName] = useState("");

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>{name}</p>
    </>
  );
}
```

the ownership is more localized.

**Important:** Do not reduce this to "higher state always means worse performance." React's reconciliation and memoization determine the actual work. The architectural principle is to avoid unnecessarily broad state ownership.

---

## 5. When State Should Move Up

Suppose:

```text
Parent
 ├── SearchBox
 └── SearchResults
```

Initially SearchBox owns:

```jsx
const [query, setQuery] = useState("");
```

But now both SearchBox and SearchResults need `query`.

Lift it to their closest common parent:

```jsx
function Parent() {
  const [query, setQuery] = useState("");

  return (
    <>
      <SearchBox
        query={query}
        setQuery={setQuery}
      />
      <SearchResults query={query} />
    </>
  );
}
```

---

## 6. State Colocation vs Lifting State Up

These concepts are complementary.

### State colocation

> Keep state close to where it is used.

### Lifting state up

> Move state to a common parent when multiple components need to share it.

Think:

```text
One component
    ↓
Colocate

Multiple siblings
    ↓
Lift to closest common parent
```

---

## 7. Example: Modal State

If only a Settings feature needs modal state:

```jsx
function SettingsButton() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>
        Settings
      </button>

      {isOpen && <SettingsModal />}
    </>
  );
}
```

There is no reason to move `isOpen` to App if nothing else needs it.

---

## 8. Feature-Level Colocation

Colocation can happen at the feature level too:

```text
features/
  search/
    Search.jsx
    SearchResults.jsx
    searchApi.js
    searchUtils.js
```

The broader principle is:

> **Keep related state, logic, and UI close to the feature that owns them.**

This becomes increasingly useful as an application grows.

---

## 9. Don't Over-Colocate

Colocation does **not** mean every component must own all of its own state.

Example:

```text
Dashboard
 ├── Cart
 └── Checkout
```

If both need the same cart state, keeping it only inside Cart may create awkward communication.

Instead:

```text
Dashboard
 ├── Cart
 └── Checkout

Dashboard owns cart state
```

Dashboard is the appropriate common owner.

---

## 10. State Colocation and Context

Context is useful when many distant components need the same state.

For example:

```text
ThemeProvider
      ↓
     App
      ↓
 many components
```

But do not automatically put local state into Context.

Prefer:

```text
Local requirement
    ↓
Local state
```

rather than:

```text
Local requirement
    ↓
Global Context
```

Use broader state-management mechanisms when the sharing requirements justify them.

---

## 11. Example: Search State

Tree:

```text
App
 └── Dashboard
      ├── Header
      ├── SearchBox
      ├── SearchResults
      └── Profile
```

`query` is needed by:

```text
SearchBox
SearchResults
```

The best owner is:

```text
Dashboard
```

because it is their closest common parent.

Data flow:

```text
Dashboard state: query
       ↓
SearchBox props
       ↓
User types
       ↓
Dashboard setQuery()
       ↓
Dashboard state changes
       ↓
new query
       ↓
SearchResults props
```

---

## 12. Examples of Correct State Placement

### A. Search text

Used by:

```text
SearchBox
SearchResults
```

**Owner:** Dashboard.

Reason:

```text
SearchBox + SearchResults
        ↓
closest common owner
        ↓
Dashboard
```

### B. Profile dropdown

Used only by:

```text
Profile
```

**Owner:** Profile.

```text
Profile
 └── isDropdownOpen
```

### C. Theme

Used by many distant components.

**Possible owner:** App-level state or a Theme Context/Provider.

The choice depends on how broadly it is consumed and whether prop passing becomes awkward.

### D. Temporary SearchBox input state

Used only by SearchBox.

**Owner:** SearchBox.

```text
SearchBox
 └── inputValue
```

---

# 🧪 Exercise — Find the Correct State Owner

Given:

```text
App
 └── Dashboard
      ├── Header
      ├── SearchBox
      ├── SearchResults
      └── Profile
```

Determine where each state should live:

1. Search text
2. Profile dropdown open/closed
3. Theme
4. Temporary SearchBox input state

Then explain **why**.

### Data-flow challenge

For search state, trace:

```text
User types
    ↓
?
    ↓
?
    ↓
SearchResults
```

Expected reasoning:

```text
SearchBox + SearchResults
        ↓
closest common owner
        ↓
Dashboard
```

Full flow:

```text
User types
    ↓
SearchBox onChange
    ↓
Dashboard setQuery()
    ↓
Dashboard state changes
    ↓
query passed to SearchResults
    ↓
SearchResults renders with new query
```

---

# 🎯 Interview Checklist

- [ ] Definition of state colocation
- [ ] Why state should generally stay local
- [ ] State placement and ownership
- [ ] Closest common parent
- [ ] Relationship with lifting state up
- [ ] Avoiding unnecessary global state
- [ ] When Context/external state may be appropriate
- [ ] Feature-level colocation
- [ ] Avoiding unnecessarily broad state ownership

---

# 🧠 Final Mental Model

```text
Who needs the state?
       ↓
One component?
       ↓
Keep it local.

Multiple siblings?
       ↓
Lift to closest common parent.

Many distant components?
       ↓
Consider Context / external store.
```

> **Keep state local by default; lift it only when sharing requires it.**
