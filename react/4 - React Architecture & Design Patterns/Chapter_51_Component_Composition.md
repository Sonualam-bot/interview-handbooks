# Chapter 51 — Component Composition

## Handbook
**Handbook 4 — React Architecture & Design Patterns**

## 1. Core Mental Model

Composition means building larger components by combining smaller reusable components.

```text
Small reusable components
        ↓
     composed
        ↓
Larger component
```

Common mechanisms:
- `children`
- component props

> **Composition = combine smaller components to build larger UI.**

## 2. The Problem Composition Solves

Instead of one giant component:

```jsx
function Dashboard() {
  // 500 lines...
}
```

compose focused components:

```text
Dashboard
 ├── Header
 ├── Sidebar
 ├── Search
 ├── UserPanel
 └── Content
```

## 3. `children` as Composition

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

Usage:

```jsx
<Card>
  <UserProfile />
</Card>
```

`Card` controls the structure while the consumer controls the content.

The same component can accept:

```jsx
<Card><ProductDetails /></Card>
<Card><Settings /></Card>
```

## 4. Composition Through Props

```jsx
function Layout({ sidebar, content }) {
  return (
    <div>
      <aside>{sidebar}</aside>
      <main>{content}</main>
    </div>
  );
}
```

Usage:

```jsx
<Layout
  sidebar={<Sidebar />}
  content={<Dashboard />}
/>
```

## 5. Composition vs Inheritance

### Composition

```text
Component A
   +
Component B
   +
Component C
   ↓
Larger component
```

### Inheritance

One type derives from another:

```js
class Animal {
  eat() {}
}

class Dog extends Animal {
  bark() {}
}
```

React generally favors composition over inheritance.

> **Composition combines smaller components; inheritance derives one type from another.**

## 6. Boolean Props

A component with many configuration flags can become difficult to maintain:

```jsx
<Card
  showHeader
  showFooter
  showActions
  showAvatar
/>
```

Composition can be cleaner:

```jsx
<Card>
  <CardHeader />
  <CardBody />
  <CardFooter />
</Card>
```

The consumer decides which pieces are included.

> **Composition can reduce configuration-heavy components by allowing consumers to compose the exact UI they need.**

## 7. Composition vs Prop Drilling

### Composition

```text
Parent
 ↓
passes UI/component
 ↓
Child renders it
```

### Prop drilling

```text
Parent
 ↓
data
 ↓
Intermediate
 ↓
data
 ↓
Deep Child
```

Composition can reduce some prop drilling by allowing the parent to construct the UI.

## 8. Specialization Through Composition

```jsx
function Button({ children }) {
  return <button className="button">{children}</button>;
}

function DeleteButton() {
  return <Button>Delete</Button>;
}
```

`DeleteButton` composes `Button`; it does not inherit from it.

## 9. Composition and State

Composition does **not** automatically share state.

A child's state belongs to that child:

```text
Parent
 ↓
Child
 ↓
Child's state
```

If the parent needs to control it, state can be lifted:

```text
Parent
 ↓
state
 ↓
props
 ↓
Child
```

> **Composition reuses UI structure and logic; it does not automatically share component state.**

## 10. Compound Components

Compound components are an advanced composition pattern:

```jsx
<Select>
  <Select.Option />
  <Select.Option />
</Select>
```

Remember:

> **Compound components build on composition.**

# Interview Questions

### Q1. What is component composition?
> Building larger components by combining smaller reusable components. `children` and component props are common mechanisms.

### Q2. Why is `<Card><UserProfile /></Card>` composition?
> `Card` controls the container/structure while the consumer controls the content through `children`.

### Q3. Composition vs inheritance?
> Composition combines smaller components to build larger UI. Inheritance derives one type from another. React generally favors composition.

### Q4. Why can composition reduce boolean-heavy components?
> Consumers can compose exactly the UI they need instead of making one component handle many combinations of configuration flags.

### Q5. Does composition share state?
> No. State belongs to the component that owns it unless it is explicitly lifted or shared through another mechanism.

# Quick Revision

```text
Composition
→ combine smaller components
→ build larger UI
```

```text
children
→ common composition mechanism
```

```text
component props
→ another composition mechanism
```

```text
React
→ generally favors composition over inheritance
```

```text
Composition
≠
automatic state sharing
```

### Interview line

> **Composition means combining smaller reusable components to build larger UI, giving consumers flexibility over what gets rendered without relying on inheritance.**

**Chapter 51 — COMPLETE ✅**
