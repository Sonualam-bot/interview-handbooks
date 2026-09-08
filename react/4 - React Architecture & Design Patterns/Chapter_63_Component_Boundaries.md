# Handbook 4 — React Architecture & Design Patterns

# Chapter 63 — Component Boundaries

## Core Idea

A **component boundary** is the point where we decide that a responsibility belongs in a separate component.

The goal is not simply smaller components.

> **Good component boundaries make responsibilities clearer.**

---

## 1. Why Component Boundaries Matter

Good boundaries can help:

- isolate responsibilities
- reduce coupling
- improve reuse
- make testing easier
- make changes safer
- make large applications easier to understand
- clarify state and behavior ownership

A component boundary is an architectural boundary. Components interact across it through interfaces such as:

- props
- callbacks
- Context
- composition

---

## 2. Clear Component Responsibilities

A component doing many unrelated things becomes difficult to reason about.

Instead of:

```text
UserPage
 ├── data fetching
 ├── form validation
 ├── modal management
 ├── table rendering
 ├── profile rendering
 └── permissions
```

we might create:

```text
UserPage
 ├── UserProfile
 ├── UserTable
 ├── UserForm
 └── UserModal
```

Reusable behavior may also be extracted into Custom Hooks.

---

## 3. Don't Split Everything

Good architecture does **not** mean every piece of JSX needs its own component.

Over-fragmentation can create:

```text
Dashboard
 └── Header
      └── HeaderTitle
           └── HeaderTitleText
                └── HeaderTitleSpan
```

If those boundaries provide no meaningful value, they add complexity.

Instead of:

> "Can I make this a component?"

ask:

> **"Should this responsibility have its own boundary?"**

---

## 4. Signals That a Boundary May Be Useful

A section may be a good candidate when it has:

### A distinct responsibility

```text
Search
Analytics
Notifications
```

### Independent understanding

The section can be understood without knowing the internals of its parent.

### Reuse

For example:

```text
UserAvatar
```

is used by:

```text
Header
Profile
Comments
```

### Its own state or behavior

```text
FilterPanel
 └── filter state
```

### A parent that is becoming difficult to understand

A very large component may benefit from meaningful boundaries.

### Independent change

If a section changes frequently and independently, a boundary can help isolate that change.

---

## 5. Component Boundaries and State Colocation

This connects directly to Chapter 60.

Suppose:

```text
Dashboard
 └── Search
      └── search state
```

The search state is colocated with Search.

Therefore:

```text
Component boundary
        ↓
State boundary
```

Good boundaries often make good state-ownership boundaries.

If multiple components need the state, it may instead need to move to their closest common parent.

---

## 6. Component Boundaries and Lifting State

Example:

```text
Dashboard
 ├── SearchBox
 └── SearchResults
```

Both need:

```text
query
```

So:

```text
Dashboard
 └── query
      ├── SearchBox
      └── SearchResults
```

The components remain separate. The state simply lives at the appropriate common owner.

---

## 7. Component Boundaries and Prop Drilling

Suppose:

```text
App
 ↓
Dashboard
 ↓
Layout
 ↓
Sidebar
 ↓
UserProfile
```

and `user` must travel through every layer.

This may indicate that the component structure deserves reconsideration.

Ask:

- Could composition solve it?
- Are the components structured around the right responsibilities?
- Would Context be appropriate?
- Is there a more natural component boundary?

Prop drilling can sometimes be a symptom of boundaries that need reconsideration.

---

## 8. Boundaries Based on Responsibility

A meaningful architecture might be:

```text
Dashboard
 ├── SearchSection
 ├── AnalyticsSection
 └── RecentActivity
```

These names communicate intent:

```text
SearchSection
→ search responsibility

AnalyticsSection
→ analytics responsibility

RecentActivity
→ activity responsibility
```

---

## 9. Boundaries Based on Change

Ask:

> **What parts of this UI are likely to change independently?**

For example:

```text
PaymentForm
 ├── PaymentFields
 ├── PaymentSummary
 └── PaymentButton
```

If PaymentSummary evolves independently, separating it can make changes more localized.

---

## 10. Boundaries Based on Reuse

A genuinely reusable component naturally forms a boundary.

Example:

```text
ProductCard
```

used by:

```text
Home
Search
Recommendations
```

However:

> Do not extract something solely because you might reuse it someday.

Build boundaries around real conceptual or architectural value.

---

## 11. Boundaries Based on State Ownership

Suppose:

```text
UserModal
 ├── isOpen
 ├── formData
 ├── validation
 └── submit behavior
```

This is a natural boundary because the UI, state, and behavior belong to the same feature.

General principle:

> **Colocate state and behavior with the component responsible for them.**

---

## 12. Presentational and Behavioral Boundaries

The Container / Presentational pattern from Chapter 58 is another way to create boundaries.

```text
UserListContainer
      ↓
data + behavior
      ↓
UserList
      ↓
UI
```

Modern React may instead use a Custom Hook:

```text
UserList
   ↓
useUsers()
   ↓
data / behavior
```

The implementation can vary.

The architectural principle remains:

> **Separate responsibilities when doing so makes the system easier to understand.**

---

## 13. Component Boundary ≠ File Boundary

You can have multiple components in one file:

```jsx
function UserProfile() {}

function Avatar() {}

function UserName() {}
```

Those are still component boundaries.

Likewise:

```text
Avatar.jsx
UserName.jsx
UserStatus.jsx
```

does not automatically mean better architecture.

Remember:

```text
More files
≠
Better architecture
```

---

## 14. Component Boundaries and Testing

Good boundaries can make testing easier.

Instead of reasoning about one enormous Dashboard, you can reason about:

```text
SearchBox
UserTable
Profile
```

independently.

Each component can have a narrower responsibility and a more focused test surface.

---

## 15. Component Boundaries and Reusability

Useful boundaries can exist at different levels.

### UI primitives

```text
Button
Modal
Dropdown
Input
Card
```

### Feature/domain components

```text
UserProfile
OrderSummary
SearchResults
```

Both can be valuable when they represent meaningful responsibilities.

---

## 16. Don't Over-Abstract

Over-abstraction can make architecture harder to understand.

For example:

```text
Dashboard
 └── GenericSection
      └── GenericContainer
           └── GenericWrapper
                └── GenericContent
```

If each abstraction exists only to make code look reusable, the system becomes harder to navigate.

A good boundary should have a reason.

---

## 17. Component Boundaries and Performance

Component boundaries can sometimes help performance by making it easier to:

- localize state
- isolate expensive UI
- memoize specific components
- avoid unnecessary work

But:

> **Do not create components solely for performance.**

Creating a component does not automatically make React render less.

The primary reason for a boundary should usually be:

> **Clear responsibility and maintainability.**

---

## 18. Architectural Smell

Imagine:

```text
Component
 ├── 15 pieces of state
 ├── 20 handlers
 ├── 10 effects
 ├── 800 lines of JSX
 └── 30 props
```

This does not automatically mean "split it."

But it is a signal to ask:

> **Are multiple responsibilities living inside one component?**

Look for meaningful boundaries rather than splitting mechanically.

---

## 19. Three Questions for Boundary Decisions

### Question 1

> **Does this have a distinct responsibility?**

If yes → candidate.

### Question 2

> **Does it have its own state or behavior?**

If yes → candidate.

### Question 3

> **Does it change, get reused, or need to be reasoned about independently?**

If yes → strong candidate.

None of these is an absolute rule. Architecture requires judgment.

---

## Interview Answer

> **I generally create component boundaries around meaningful responsibilities, independently understandable or reusable UI, and areas with their own state or behavior. I also consider whether parts of the UI are likely to change independently. I avoid splitting components purely for the sake of having smaller files because excessive abstraction can make the code harder to understand. The goal is to create boundaries that reduce coupling and make ownership and responsibilities clear.**

---

# Exercise — Refactor the Dashboard

Imagine:

```text
Dashboard.jsx

800 lines
12 useState calls
8 event handlers
4 effects
Search UI
User profile
Analytics
Notifications
Settings modal
```

A reasonable starting point:

```text
Dashboard
 ├── SearchSection
 ├── Analytics
 ├── Notifications
 ├── UserProfile
 └── SettingsModal
```

Now decide where state should live.

### Search

If only SearchSection needs search state:

```text
SearchSection
 └── search state
```

If SearchResults also needs it:

```text
Dashboard
 └── query
      ├── SearchSection
      └── SearchResults
```

### Settings modal

If the modal and its state are tightly coupled:

```text
SettingsModal
 └── relevant state/behavior
```

unless another component needs to control the state.

### Analytics filters

If only Analytics needs the filters:

```text
Analytics
 └── filter state
```

unless other parts of Dashboard need those filters.

---

# Exercise 2 — Find Bad Boundaries

Consider:

```text
Dashboard
 └── GenericContainer
      └── GenericSection
           └── GenericWrapper
                └── Search
```

Answer:

1. Which boundaries provide real value?
2. Which are probably unnecessary?
3. Where should Search state live?
4. If SearchResults also needs the query, where should the state move?
5. If five unrelated features need the query, what alternatives could you consider?

---

# Interview Checklist

- [ ] What a component boundary represents
- [ ] Boundaries based on responsibility
- [ ] State and behavior boundaries
- [ ] Reusable component boundaries
- [ ] Components that change independently
- [ ] Relationship with state colocation
- [ ] Relationship with lifting state
- [ ] Relationship with prop drilling
- [ ] Avoiding over-abstraction
- [ ] Component boundary ≠ file boundary
- [ ] Components are not created merely for performance

---

# Final Mental Model

When looking at a large component, ask:

```text
Does this section have a distinct responsibility?
             ↓
Does it have its own state/behavior?
             ↓
Does it change or get reused independently?
             ↓
Would separating it reduce coupling?
             ↓
Create a boundary if the answer is meaningful.
```

> **Good component boundaries aren't about making components smaller. They're about making responsibilities clearer.**
