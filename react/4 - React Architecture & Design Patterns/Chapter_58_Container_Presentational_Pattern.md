# Chapter 58 — Container / Presentational Pattern

**Handbook 4 — React Architecture & Design Patterns**

---

## 1. The Problem

A component can end up doing two different jobs:

### Data / behavior

- fetching data
- managing state
- effects
- event handlers
- business logic

### Presentation

- rendering
- layout
- styling
- displaying data

Example:

```jsx
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/user")
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, []);

  if (loading) {
    return <p>Loading...</p>;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

The component mixes data/behavior with UI.

---

## 2. The Container / Presentational Idea

Separate the responsibilities:

```text
Container
    ↓
handles data + behavior
    ↓
Presentational Component
    ↓
renders UI
```

### Container

Usually responsible for:

- fetching data
- state
- effects
- event handlers
- business logic

### Presentational component

Usually responsible for:

- rendering
- layout
- styling
- displaying props

---

## 3. Example

### Presentational component

```jsx
function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

It doesn't care where `user` came from.

It simply receives:

```jsx
user={user}
```

### Container

```jsx
function UserProfileContainer() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch("/api/user")
      .then(res => res.json())
      .then(setUser);
  }, []);

  if (!user) {
    return <p>Loading...</p>;
  }

  return <UserProfile user={user} />;
}
```

Now:

```text
UserProfileContainer
        ↓
"How do I get the user?"
        ↓
      user
        ↓
UserProfile
        ↓
"How do I display the user?"
```

---

## 4. Why Is This Useful?

Suppose the same data needs multiple presentations:

```text
User data
   ↓
 ┌───────────────┐
 ↓               ↓
UserCard      UserProfile
```

The data/behavior can be separated from how the data is displayed.

This can improve:

- separation of concerns
- reuse
- readability
- testing

---

## 5. Container = Smart Component

Older React terminology sometimes calls the container a:

> **Smart component**

Because it knows about:

- data
- state
- behavior
- application logic

Presentational components were sometimes called:

> **Dumb components**

⚠️ Don't interpret "dumb" literally.

It simply describes responsibility.

---

## 6. The Data Flow

This is one of the most important parts.

```text
API / external source
        ↓
   Container
        ↓
   state/data
        ↓
      props
        ↓
Presentational Component
        ↓
       UI
```

For user interaction:

```text
User clicks
    ↓
Presentational component
    ↓
callback prop
    ↓
Container
    ↓
state/business logic
    ↓
new props
    ↓
Presentational component re-renders
```

Communication can therefore go both directions:

```text
Data:
Container ─────────→ Presentational

Events:
Presentational ────→ Container
```

---

## 7. Example with an Event

### Container

```jsx
function CounterContainer() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count => count + 1);
  }

  return (
    <Counter
      count={count}
      onIncrement={increment}
    />
  );
}
```

### Presentational

```jsx
function Counter({ count, onIncrement }) {
  return (
    <div>
      <p>{count}</p>

      <button onClick={onIncrement}>
        +
      </button>
    </div>
  );
}
```

Data flow:

```text
count
  ↓
Container
  ↓ props
Counter
  ↓
renders count

User clicks
  ↓
onIncrement
  ↓
Container
  ↓
setCount()
  ↓
new count
  ↓
Counter receives new props
```

This is clean unidirectional data flow.

---

## 8. Presentational Components Are Often Easier to Reuse

Consider:

```jsx
function UserCard({ user, onSelect }) {
  return (
    <article onClick={() => onSelect(user)}>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </article>
  );
}
```

This component doesn't care whether the user came from:

- an API
- Context
- a state store
- a local array
- a test fixture

It only knows:

```text
"I receive user and onSelect."
```

That makes it highly reusable.

---

## 9. Presentational Components Are Easier to Test

Instead of mocking APIs:

```jsx
<UserCard
  user={{
    name: "User",
    email: "user@example.com"
  }}
  onSelect={mockFn}
/>
```

You can test directly:

```text
Given these props
       ↓
Does the correct UI render?
```

The test doesn't need to care about data fetching.

---

## 10. Major Limitation

**Do not blindly split every component into Container + Presentational.**

For a tiny component:

```jsx
function Button({ children, onClick }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

Creating:

```text
ButtonContainer
Button
```

would likely make the code worse.

You would introduce unnecessary abstraction.

---

## 11. Modern React Changed This Pattern

Container / Presentational became especially popular before Hooks.

Historically:

```text
Container
    ↓
Presentational
```

was a common way to separate logic and UI.

Modern React lets us extract logic with Custom Hooks.

Example:

```jsx
function useUser() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch("/api/user")
      .then(res => res.json())
      .then(setUser);
  }, []);

  return user;
}
```

Then:

```jsx
function UserProfile() {
  const user = useUser();

  if (!user) {
    return <p>Loading...</p>;
  }

  return <UserCard user={user} />;
}
```

Now:

```text
Custom Hook
    ↓
reusable behavior

Component
    ↓
UI
```

We don't necessarily need a separate container component.

---

## 12. Is Container / Presentational Obsolete?

**No.**

The strict version is less necessary than it once was.

The architectural principle remains valuable:

> **Separate responsibilities when doing so improves clarity and reuse.**

Modern React provides more tools for achieving that separation:

- Custom Hooks
- Context
- Composition
- Feature modules
- State management
- Server/client boundaries

Don't memorize:

> "Every component needs a container."

Instead remember:

> **Separate behavior from presentation when the separation provides real value.**

---

## 13. Container / Presentational vs Custom Hook

### Container pattern

```text
Container
  ↓
owns behavior
  ↓
passes props
  ↓
Presentational component
```

### Custom Hook

```text
Component
  ↓
calls Custom Hook
  ↓
gets behavior/data
  ↓
renders UI
```

The Custom Hook lets you reuse the **logic itself**.

The Container pattern separates and reuses the **component responsibilities/structure**.

---

## 14. Container / Presentational vs Compound Components

### Container / Presentational

Separates:

```text
behavior
    ↓
presentation
```

### Compound Components

Organizes related components around shared behavior:

```jsx
<Tabs>
  <Tabs.List />
  <Tabs.Panel />
</Tabs>
```

Therefore:

```text
Container / Presentational
→ separation of responsibilities

Compound Components
→ coordinated component API
```

---

## 15. When Should You Use This Pattern?

Good situations include:

### Complex data-driven UI

```text
API
 ↓
state
 ↓
business logic
 ↓
UI
```

### Multiple presentations of the same data

```text
same data
  ↓
Card
Table
List
Chart
```

### Complex interaction logic

When a component has a lot of:

- event handlers
- state transitions
- data transformation
- side effects

Separating these from presentation can make the code easier to reason about.

---

## 16. When Should You NOT Use It?

Avoid it when:

- the component is tiny
- there is no meaningful logic/presentation separation
- the abstraction doesn't improve reuse
- it creates unnecessary files/components
- the split makes navigation harder

Architectural principle:

> **Abstraction has a cost.**

Don't introduce architecture simply because a pattern exists.

---

## 17. Interview Answer

> **The Container / Presentational pattern is a React architectural pattern where container components handle data, state, and behavior, while presentational components focus primarily on rendering UI based on props. It can improve separation of concerns and reusability. With modern React, Custom Hooks often provide a simpler way to reuse logic, so the pattern shouldn't be applied rigidly to every component.**

---

## 18. Mental Model

Remember:

```text
              DATA / BEHAVIOR
                     ↓
                CONTAINER
                     ↓
                   props
                     ↓
             PRESENTATIONAL
                     ↓
                    UI
```

Events travel back:

```text
UI interaction
      ↓
callback prop
      ↓
Container
      ↓
state update
      ↓
new props
      ↓
UI re-render
```

---

# 🧪 Exercise — Build a Container + Presentational Counter

The goal is to understand the **data flow**, not just create two files.

## Step 1 — Create the presentational component

Create:

```jsx
function Counter({ count, onIncrement, onDecrement }) {
  // render the UI
}
```

It should:

- display `count`
- have an `Increment` button
- have a `Decrement` button
- contain **no state**
- contain **no API calls**
- contain **no business logic**

---

## Step 2 — Create the container

Create:

```jsx
function CounterContainer() {
  // state + behavior
}
```

It should:

```jsx
const [count, setCount] = useState(0);
```

Create:

```jsx
function increment() {
  // update count
}

function decrement() {
  // update count
}
```

Then pass them:

```jsx
<Counter
  count={count}
  onIncrement={increment}
  onDecrement={decrement}
/>
```

---

## Step 3 — Trace the data flow

Start with:

```text
count = 0
```

Ask yourself:

### Initial render

```text
CounterContainer
      ↓
count = 0
      ↓
props
      ↓
Counter
      ↓
UI displays 0
```

### User clicks Increment

Trace:

```text
1. User clicks Increment
2. Counter's button calls onIncrement
3. onIncrement points to Container's increment()
4. increment() calls setCount()
5. Container state becomes 1
6. Container re-renders
7. Counter receives count={1}
8. Counter renders 1
```

---

## Step 4 — Trace the reverse direction

This is the important architectural flow:

```text
                 DATA
                  ↓
Container ─────────────→ Presentational
                              ↓
                             UI
                              ↓
                         User click
                              ↓
                       callback prop
                              ↓
Container ←───────────────────┘
      ↓
   setState
      ↓
 new props
      ↓
     UI
```

### 🔥 Check yourself

Answer these without looking at the notes:

**Q1. Who owns the `count` state?**

→ The Container.

**Q2. Does the Presentational component know where `count` came from?**

→ No.

**Q3. How does the Presentational component tell the Container that the user clicked?**

→ Through a callback prop.

**Q4. Does the Presentational component call `setCount()` directly?**

→ No. It calls the callback supplied by the Container.

**Q5. What causes the Presentational component to display the new count?**

→ The Container updates state → re-renders → passes new props → Presentational component re-renders.

---

## Step 5 — Extension Challenge

Now imagine the counter has this rule:

> The count cannot go below zero.

Where should that rule live?

Think before implementing.

The important architectural question is:

```text
Is "count cannot be negative"
a UI concern?
or
a behavior/business rule?
```

A good answer is:

> It belongs in the behavior/logic layer, so the Container can enforce it.

Then implement:

```jsx
function decrement() {
  setCount(count => Math.max(0, count - 1));
}
```

The Presentational component should remain unaware of the rule.

---

# 🎯 Interview Priority

### 🔥🔥🔥 Know this

- Container vs Presentational responsibilities
- Data flow between them
- Callback flow back to the Container
- Why separation helps
- Smart vs presentational terminology
- Container/Presentational vs Custom Hooks
- Why modern React often uses Hooks instead
- When **not** to use the pattern

### Lower priority

- Historical terminology
- Creating containers for every component
- Strict adherence to the pattern

---

## One-line Memory Trick

> **Container asks "How does it work?" — Presentational asks "How does it look?"**
