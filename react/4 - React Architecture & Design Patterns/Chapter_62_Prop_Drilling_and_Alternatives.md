# Handbook 4 — React Architecture & Design Patterns

# Chapter 62 — Prop Drilling & Alternatives

## Core Idea

**Prop drilling** means passing data through intermediate components that do not need the data themselves, simply so a deeper component can receive it.

Example:

```text
App
 ↓ user
Dashboard
 ↓ user
Sidebar
 ↓ user
UserProfile
```

If only `UserProfile` uses `user`, the intermediate components are merely transporting it.

---

## 1. Simple Example

```jsx
function App() {
  const user = {
    name: "Sonu"
  };

  return <Dashboard user={user} />;
}

function Dashboard({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <h2>{user.name}</h2>;
}
```

Here:

- Dashboard does not use `user`
- Sidebar does not use `user`
- UserProfile actually needs `user`

That is prop drilling.

---

## 2. Prop Drilling Is Not the Same as Normal Prop Passing

Do not call every parent → child prop passing prop drilling.

This is normal:

```jsx
<Profile user={user} />
```

if `Profile` actually needs `user`.

Prop drilling becomes problematic when data travels through multiple unrelated layers:

```text
Parent
 ↓
A
 ↓
B
 ↓
C
 ↓
Actual consumer
```

where A, B, and C don't use the data.

### Key principle

> **Props are not bad. Unnecessary prop propagation is the problem.**

---

## 3. Why Prop Drilling Can Become a Problem

In a large tree, you may end up with:

```jsx
<ComponentA
  user={user}
  theme={theme}
  permissions={permissions}
  settings={settings}
/>
```

then:

```jsx
<ComponentB
  user={user}
  theme={theme}
  permissions={permissions}
  settings={settings}
/>
```

and so on.

Intermediate components become coupled to data they don't actually care about.

This can make:

- component APIs noisy
- refactoring harder
- ownership less clear
- unrelated components responsible for transporting data

---

# Alternative 1 — Composition

Before reaching for Context, consider **component composition**.

Suppose:

```text
App
 ↓
Dashboard
 ↓
Sidebar
 ↓
UserProfile
```

Only UserProfile needs `user`.

Instead of passing `user` through every layer:

```jsx
function App() {
  const user = {
    name: "Sonu"
  };

  return (
    <Dashboard
      sidebar={
        <UserProfile user={user} />
      }
    />
  );
}
```

Dashboard can simply render the supplied UI:

```jsx
function Dashboard({ sidebar }) {
  return (
    <div>
      <main>Dashboard</main>
      {sidebar}
    </div>
  );
}
```

Now:

```text
user
 ↓
UserProfile
```

while Dashboard does not need to understand the user data.

### Why composition helps

It allows the component with the data to directly create the UI that needs it.

The intermediate component receives UI rather than unrelated application data.

---

# Alternative 2 — Context

When many components genuinely need the same value, Context can be appropriate.

Create context:

```jsx
const UserContext = createContext(null);
```

Provide it:

```jsx
function App() {
  const user = {
    name: "Sonu"
  };

  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}
```

Consume it:

```jsx
function UserProfile() {
  const user = useContext(UserContext);

  return <h2>{user.name}</h2>;
}
```

The intermediate components no longer need:

```jsx
user={user}
```

### Context data flow

Props:

```text
App
 ↓
Dashboard
 ↓
Sidebar
 ↓
UserProfile
```

Context:

```text
          UserProvider
               ↓
       ┌───────┴───────┐
       ↓               ↓
   Dashboard       UserProfile
                       ↓
                  useContext()
```

---

# Don't Use Context for Everything

Context is not automatically the answer whenever props appear.

Ask:

```text
Can composition solve the problem?
```

If yes:

```text
→ Composition
```

If many components genuinely need the same value:

```text
→ Context may be appropriate
```

If state is complex, broadly shared, and needs advanced state-management capabilities:

```text
→ Consider external state management
```

Prefer the simplest solution that fits the problem.

---

# Alternative 3 — Custom Hooks

Custom Hooks can provide a cleaner API around shared behavior or Context.

Example:

```jsx
function useAuth() {
  return useContext(AuthContext);
}
```

A component can then write:

```jsx
const user = useAuth();
```

instead of:

```jsx
const user = useContext(AuthContext);
```

The Custom Hook is not necessarily replacing Context.

It can simply hide the Context implementation detail:

```text
Context
   ↓
Custom Hook
   ↓
Components
```

---

# Alternative 4 — External State Management

Large applications may have shared state used by many unrelated areas.

Examples include:

- Redux
- Zustand
- other external stores

Conceptually:

```text
          Store
        ↙   ↓   ↘
      A     B     C
```

Components can access shared state without passing props through unrelated intermediate components.

But:

> **Do not introduce a global store merely because two components share state.**

Start local, lift when needed, and broaden the state-management mechanism only when the architecture justifies it.

---

# 4. Decision Hierarchy

Use this mental model:

```text
Need to share data?
        ↓
Can it stay local?
        ↓
      Yes
        ↓
   Local state
```

If not:

```text
Multiple related components?
        ↓
Lift state to closest common parent
```

If the data now has to travel through unrelated layers:

```text
Can composition solve it?
        ↓
      Yes
        ↓
    Composition
```

If many components need the same value:

```text
Context may be appropriate
```

If shared state becomes broad and complex:

```text
External state management may be appropriate
```

---

# 5. Prop Drilling Is Not Always Bad

Do not say:

> "Prop drilling should always be avoided."

That is too strong.

This is completely normal:

```jsx
<Profile user={user} />
```

And even:

```text
Parent
 ↓
Child
 ↓
Grandchild
```

can be perfectly reasonable if the prop is meaningful to the components in that relationship.

The real problem is:

> **Unnecessary propagation through unrelated layers.**

---

# 6. Example — Theme

Suppose:

```text
App
 └── Layout
      ├── Header
      ├── Dashboard
      └── Footer
```

Many components need:

```text
theme
```

Passing it through every intermediate component can become cumbersome.

Context can make sense:

```text
ThemeProvider
      ↓
    Layout
   ↙  ↓  ↘
Header Dashboard Footer
```

Each consumer can access the theme directly.

---

# 7. Example — User Data and Composition

Suppose:

```text
App
 └── Dashboard
      └── Sidebar
           └── UserProfile
```

Only UserProfile needs `user`.

Instead of:

```jsx
<Dashboard user={user} />
```

```jsx
<Sidebar user={user} />
```

```jsx
<UserProfile user={user} />
```

composition could allow:

```jsx
<Dashboard
  sidebar={<UserProfile user={user} />}
/>
```

Now Dashboard receives UI rather than needing to understand `user`.

---

# 8. Prop Drilling and Component Boundaries

Repeated prop drilling can sometimes indicate that component boundaries aren't ideal.

If data repeatedly travels through unrelated layers, ask:

> **Are these components structured around the right responsibilities?**

Sometimes the solution isn't Context.

Sometimes the solution is simply restructuring the component tree.

This leads into:

**Chapter 63 — Component Boundaries.**

---

# 9. Prop Drilling vs Lifting State Up

These are different concepts.

### Lifting State Up

Answers:

> **Where should shared state live?**

```text
Child A
Child B
   ↓
Common parent owns state
```

### Prop Drilling

Answers:

> **How far does that data have to travel through the tree?**

```text
Parent
 ↓
Unrelated A
 ↓
Unrelated B
 ↓
Actual consumer
```

Therefore you can have:

```text
Good state ownership
+
Bad prop propagation
```

---

# 10. Architecture Connection

Our sequence is:

```text
59 Controlled vs Uncontrolled
        ↓
Who owns state?

60 State Colocation
        ↓
Where should state live?

61 Lifting State Up
        ↓
What if multiple components need it?

62 Prop Drilling & Alternatives
        ↓
What if shared data travels too far?

63 Component Boundaries
        ↓
Are the components structured correctly?
```

The goal is not to memorize isolated patterns.

The goal is to understand how to make decisions about **state ownership, data flow, and component architecture**.

---

# Interview Answer

> **Prop drilling is the practice of passing data through intermediate components that don't need the data themselves, simply so a deeper component can access it. Passing props itself isn't a problem; prop drilling becomes problematic when data travels through many unrelated layers and increases coupling. Depending on the situation, we can use component composition, Context, Custom Hooks, or external state management. The simplest appropriate solution should be preferred rather than automatically using global state.**

---

# Exercise 1 — Identify the Problem

Consider:

```text
App
 └── Dashboard
      └── Layout
           └── Sidebar
                └── UserProfile
```

Only `UserProfile` needs:

```text
user
```

Currently:

```jsx
<App user={user} />

<Dashboard user={user} />

<Layout user={user} />

<Sidebar user={user} />

<UserProfile user={user} />
```

Answer:

1. Which component actually needs `user`?
2. Which components are only transporting it?
3. Is this prop drilling?
4. Could composition solve it?
5. At what point would Context become reasonable?

---

# Exercise 2 — Choose the Alternative

### A

```text
Parent
 ├── Button
 └── Modal
```

Only these components need the value.

Think:

> Can ordinary state + props solve this?

---

### B

```text
App
 └── Dashboard
      └── Settings
           └── ThemeToggle
```

Many components across the application need the theme.

Think:

> Context may be appropriate.

---

### C

```text
App
 └── Layout
      └── Card
           └── UserAvatar
```

Only UserAvatar needs the user object.

Think:

> Could composition eliminate the unnecessary prop chain?

---

### D

A large application has complex shared state used by many unrelated features.

Think:

> An external state-management solution may be appropriate.

---

# Interview Checklist

- [ ] Definition of prop drilling
- [ ] Why prop drilling can become problematic
- [ ] Prop drilling ≠ all prop passing
- [ ] Composition as an alternative
- [ ] Context as an alternative
- [ ] Custom Hooks around Context
- [ ] External state management
- [ ] Don't use global state unnecessarily
- [ ] Prop drilling can indicate poor component boundaries
- [ ] Difference between lifting state and prop drilling

---

# Final Mental Model

```text
Need to share data?
        ↓
Keep it local if possible
        ↓
Otherwise lift it
        ↓
If props must travel through many
unrelated components
        ↓
Prop drilling
        ↓
Consider:
  • Composition
  • Context
  • Custom Hooks
  • External state management
```

> **Don't eliminate props just because they're props. Eliminate unnecessary prop propagation.**
