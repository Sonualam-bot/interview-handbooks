# Handbook 6 — React Ecosystem
# Chapter 83 — State Management Landscape

## 1. What Is State?

State represents information that can change over time and influence what the user sees or how the application behaves.

Examples include modal visibility, selected tabs, user information, API results, pagination, and form submission status.

A useful mental model is:

```text
UI = f(State)
```

React renders UI based on state. This is a mental model, not a literal description of every internal operation.

## 2. Why Does State Management Become a Problem?

As an application grows, multiple components may need to read or update the same logical information.

Common challenges:

1. **Ownership:** Who owns the state?
2. **Sharing:** Which components need it?
3. **Synchronization:** How do consumers stay consistent?
4. **Updates:** How should state change?
5. **Lifecycle:** How long should it survive?
6. **Server synchronization:** What happens when remote data changes?

A state-management library is useful when its abstractions solve problems the application actually has.

## 3. React Already Provides State Management

| Mechanism | Primary purpose |
|---|---|
| `useState` | Local component state |
| `useReducer` | State transitions through a reducer and actions |
| Props | Pass data from parent to child |
| Context | Make values available to descendants |
| Custom Hooks | Reuse stateful logic |

If only one component needs a value, local state is often enough.

**Principle:** use the simplest mechanism that correctly represents the state and its ownership.

## 4. Local State vs Shared State

### Local state

State is local when one component or a small isolated subtree owns and uses it.

Examples: tooltip visibility, dropdown visibility, temporary input values, and a widget's active tab.

```jsx
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(open => !open)}>
        Toggle
      </button>
      {isOpen && <div>Dropdown content</div>}
    </div>
  );
}
```

### Shared state

Shared state is needed when multiple components use the same logical information.

```text
Navbar ───────┐
ProductCard ──┼── Cart state
CartDrawer ───┤
CartSummary ──┘
```

Possible approaches:
- Lift state to a common parent
- Pass props
- Use Context
- Use an external store such as Redux or Zustand

**Shared state does not automatically mean global state.** Share state only across the scope that needs it.

## 5. How State Sharing Evolves

- **Local state:** one component needs the value.
- **Lifted state:** multiple siblings need the same state, so move it to their nearest common owner.
- **External store:** when sharing and update coordination become complex, a store may help.

An external store is not the inevitable final stage of every application.

## 6. Client State vs Server State

### Client state

State primarily owned by the client application.

Examples:
- Theme preference
- Whether a sidebar is open
- Temporary product selections
- An unsaved multi-step wizard

### Server state

Data owned or managed by a remote system. The client holds a copy, but the server remains authoritative.

Examples:
- Products in a database
- User orders
- Account details
- Inventory availability
- API notifications

Server state introduces concerns such as network latency, failures, loading states, stale data, cache invalidation, refetching, duplicate requests, pagination, and synchronization with changes made by others.

A `useState` array can hold API results, but `useState` alone does not provide a complete caching and synchronization strategy.

## 7. Why Use a Server-State Library?

A manually managed API state might look like:

```js
{
  products: [],
  isLoading: false,
  error: null,
  lastFetchedAt: null
}
```

You would still need policies for fetching, cache reuse, stale data, invalidation, and mutations.

**TanStack Query** and **SWR** provide abstractions for these recurring remote-data concerns.

```text
Redux / Zustand
    ↓
Client-owned application state

TanStack Query / SWR
    ↓
Remote data, caching and synchronization
```

This is a useful default, not a strict rule. Redux Toolkit also offers RTK Query for server-state concerns.

## 8. URL State

Consider:

```text
/products?category=shoes&page=2&sort=price
```

This URL represents category, page, and sort state.

URL state is useful when state should survive refreshes, be bookmarkable, or be shareable.

| State | Good default location |
|---|---|
| Tooltip visibility | Local React state |
| Selected products shared across components | Shared client state |
| Current product page | URL state |
| Product records fetched from an API | Server-state cache |
| Form values while editing | Form state |
| Authenticated session | Authentication/session mechanism |

These are starting points, not rigid rules.

## 9. Form State

Forms often manage field values, validation errors, touched/dirty status, submission status, field registration, and reset behavior.

A small form can use React state. A complex form may benefit from **React Hook Form**, which specializes in form behavior rather than serving as the global store for an entire application.

## 10. State-Management Landscape

| Tool | Main role | Typical use |
|---|---|---|
| `useState` | Local component state | Modal visibility, toggles |
| `useReducer` | Structured local state transitions | Multi-step UI, complex transitions |
| Context | Distribute values through a subtree | Theme, locale, shared dependencies |
| Redux Toolkit | Structured shared client state | Application-wide workflows |
| Zustand | Lightweight shared client state | Shared state with a small API |
| TanStack Query | Server-state lifecycle and cache | Fetching, caching, invalidating API data |
| SWR | Server-state fetching and revalidation | Remote data with cache/revalidation patterns |
| React Hook Form | Form state and interactions | Complex forms |
| URL / React Router | Navigation-related state | Filters, pagination, search parameters |

Connect each tool to the problem it solves instead of memorizing the table.

## 11. Context Is Not Automatically a Replacement for Redux

Context makes a value available to descendants without passing it through every intermediate component.

```jsx
const ThemeContext = createContext("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  return <Toolbar />;
}

function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Toolbar</div>;
}
```

Context does not automatically provide every feature of a dedicated store, such as a standardized action/reducer architecture, selective subscriptions to individual state slices, built-in server-data cache invalidation, or specialized debugging and async workflows.

Context may be sufficient. The decision depends on the complexity and update patterns of the state.

## 12. How to Choose Where State Belongs

```text
Does only one component need it?
        │
       Yes ──→ Local state
        │
        No
        ↓
Is it naturally represented by the URL?
        │
       Yes ──→ URL / router state
        │
        No
        ↓
Is it authoritative data from a server?
        │
       Yes ──→ Server-state solution
        │
        No
        ↓
Is it shared client-owned state?
        ↓
Start with lifting state / Context
        ↓
Use Redux Toolkit or Zustand if
shared-state complexity justifies a store
```

Forms may use dedicated form-state libraries. Authentication has its own security and session requirements. Use this flow as a reasoning tool, not an inflexible rule.

## 13. Example: E-Commerce Application

| Information | Likely owner | Why |
|---|---|---|
| Product catalog | Server-state cache | Server is authoritative; caching and refetching matter |
| Current page and filters | URL | Shareable and restorable navigation state |
| Cart shared across components | Shared client state or server, depending on design | Multiple components need a consistent view |
| Modal visibility | Local state | Narrow scope and simple lifecycle |
| Checkout field values | Form library or local form state | Validation, dirty state, submission |
| Theme | Context or preference store | Many components may need it |
| Order history | Server-state cache | Remote and potentially changing data |

One application can use several state-management tools at once. This is often cleaner than forcing every kind of state into one global store.

## 14. Common Mistakes

1. **Putting everything in a global store:** increases coupling and makes local behavior harder to reason about.
2. **Treating server state like ordinary client state:** remote data needs loading, error, caching, freshness, and invalidation policies.
3. **Using Context for every shared value:** broad, frequently changing context can trigger many consumer updates.
4. **Keeping URL-representable state only in memory:** shareable filters or pagination may belong in the URL.
5. **Adding a library before understanding the problem:** a modal does not need Redux.
6. **Duplicating the same state:** independent copies can drift out of sync. Prefer a clear source of truth and derive values when practical.

## 15. Interview Questions

### Q1. What is state management in React?

State management is the way an application owns, updates, shares, and synchronizes changing information that influences its behavior or UI.

### Q2. When should you use Redux or Zustand?

When client-owned state needs to be shared across components and the coordination complexity justifies a centralized store. The choice depends on architecture, team needs, update patterns, and tooling.

### Q3. What is the difference between client state and server state?

Client state is primarily owned by the client application. Server state is authoritative on a remote system, and the client holds a copy that can become stale and needs synchronization.

### Q4. Why use TanStack Query if React already has `useState` and `useEffect`?

Those Hooks can implement fetching, but they do not by themselves provide a complete shared cache, freshness, invalidation, and refetching strategy. TanStack Query provides abstractions for those concerns.

### Q5. Is Context a state-management library?

Context is a React mechanism for distributing values through a component tree. It can be part of a state-management solution, but it does not automatically provide every feature of a dedicated store or server-state library.

### Q6. Should all state be global?

No. Keep state local when its scope is local, put navigation-related state in the URL when appropriate, and use shared stores when multiple components need coordinated access.

## 16. Final Mental Model

```text
                    Application State
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
      Client State     Server State      URL State
          │                │                │
    useState,         TanStack Query,   React Router,
    useReducer,       SWR, RTK Query    search params
    Redux, Zustand
          │
          └───────────────┬────────────────┘
                          ↓
                      React UI
```

Forms may use dedicated form-state tools, and authentication requires its own session/security model.

> **Choose the state-management mechanism based on who owns the data, who needs it, how long it should live, and how it changes—not based on which library is most popular.**

## Next Chapter

**Chapter 84 — Redux Fundamentals**

We will derive why Redux introduces a store, actions, reducers, and a predictable update flow—and what problems those concepts are intended to solve.
