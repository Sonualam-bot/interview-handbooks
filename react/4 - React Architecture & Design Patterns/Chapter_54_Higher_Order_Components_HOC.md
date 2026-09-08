# Chapter 54 — Higher-Order Components (HOC)

**Handbook 4 — React Architecture & Design Patterns**

## 1. What is a Higher-Order Component?

A **Higher-Order Component (HOC)** is a function that takes a component and returns a new enhanced component.

```jsx
const EnhancedComponent = withSomething(Component);
```

Mental model:

```text
Component → HOC → Enhanced Component
```

The HOC adds reusable behavior without modifying the original component.

## 2. Basic Example

```jsx
function withLoading(Component) {
  return function WithLoading({ isLoading, ...props }) {
    if (isLoading) {
      return <p>Loading...</p>;
    }

    return <Component {...props} />;
  };
}
```

Usage:

```jsx
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);
```

The HOC handles loading behavior while `UserList` focuses on displaying users.

## 3. HOCs Are Functions, Not Components

`withLoading` is a function that **returns a React component**.

```text
withLoading
    ↓
takes Component
    ↓
returns EnhancedComponent
```

## 4. Why Were HOCs Created?

Before Hooks, HOCs were a common way to share reusable behavior.

```jsx
const ProtectedDashboard = withAuth(Dashboard);
const ProtectedProfile = withAuth(Profile);
const ProtectedSettings = withAuth(Settings);
```

> **HOCs reuse component behavior by wrapping components.**

## 5. HOCs Can Inject Props

```jsx
function withUser(Component) {
  return function WithUser(props) {
    const user = getCurrentUser();

    return <Component {...props} user={user} />;
  };
}
```

The HOC calculates the user and injects it into the wrapped component.

## 6. Props Forwarding

A good HOC generally forwards unrelated props:

```jsx
function withLoading(Component) {
  return function WithLoading({ isLoading, ...props }) {
    if (isLoading) {
      return <p>Loading...</p>;
    }

    return <Component {...props} />;
  };
}
```

```text
isLoading → consumed by HOC
everything else → forwarded to Component
```

## 7. Don't Mutate the Original Component

Avoid modifying the component you receive.

❌

```jsx
function withLoading(Component) {
  Component.prototype.isLoading = true;
  return Component;
}
```

✅

```jsx
function withLoading(Component) {
  return function EnhancedComponent(props) {
    return <Component {...props} />;
  };
}
```

The HOC should create a new component rather than modifying the original.

## 8. HOC Composition

Multiple HOCs can be combined:

```jsx
const EnhancedComponent =
  withAuth(
    withLogging(
      withLoading(Component)
    )
  );
```

Conceptually:

```text
Component
   ↓
withLoading
   ↓
withLogging
   ↓
withAuth
   ↓
EnhancedComponent
```

This is essentially function composition.

## 9. Naming Convention

HOCs commonly use the `with...` convention:

```jsx
withAuth()
withLoading()
withLogging()
withPermissions()
withTheme()
withAnalytics()
```

## 10. HOC vs Normal Component Wrapper

### Normal wrapper

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

Usage:

```jsx
<Card>
  <User />
</Card>
```

The wrapper is part of the UI tree.

### HOC

```jsx
const EnhancedUser = withAuth(User);
```

The HOC is primarily an abstraction for enhancing a component.

## 11. Main Drawback: Wrapper Hell

Multiple HOCs can create deeply nested component trees:

```jsx
withAuth(
  withLogging(
    withTheme(
      withPermissions(Component)
    )
  )
);
```

Conceptually:

```text
Auth
 └── Logging
      └── Theme
           └── Permissions
                └── Component
```

This can make debugging and understanding the application harder.

## 12. Prop Collisions

A HOC may inject a prop that conflicts with a consumer-supplied prop.

```jsx
function withUser(Component) {
  return function Enhanced(props) {
    const user = getCurrentUser();

    return <Component {...props} user={user} />;
  };
}
```

Here the HOC's `user` can overwrite `props.user`.

HOCs therefore need clear conventions around which props they own and which they forward.

## 13. Static Properties Can Be Lost

Wrapping a component can make static properties on the original component inaccessible.

```jsx
Component.someStaticValue = 123;

const Enhanced = withSomething(Component);
```

`Enhanced.someStaticValue` may not exist.

Libraries historically used utilities such as `hoist-non-react-statics` when static properties needed to be copied.

Interview takeaway:

> **Wrapping a component can hide static properties of the original component.**

## 14. HOCs and Refs

A `ref` is special in React and isn't treated like an ordinary prop.

An HOC that needs to preserve refs may use ref forwarding:

```jsx
const Enhanced = forwardRef((props, ref) => (
  <Component {...props} ref={ref} />
));
```

## 15. HOCs vs Render Props

### HOC

```jsx
const Enhanced = withAuth(Component);
```

**Wrap the component.**

### Render prop

```jsx
<Auth>
  {(user) => <Component user={user} />}
</Auth>
```

**Give a function that decides what to render.**

Mental model:

```text
HOC
→ enhance/wrap a component

Render Prop
→ give a function that receives reusable state/behavior
```

## 16. HOCs vs Custom Hooks

### HOC

```jsx
const Enhanced = withUser(Profile);
```

The component is wrapped.

### Custom Hook

```jsx
function Profile() {
  const user = useUser();

  return <h1>{user.name}</h1>;
}
```

The component directly consumes reusable logic.

For many new use cases, custom hooks are simpler because they avoid:

- extra wrapper components
- wrapper nesting
- prop injection
- HOC naming/debugging complexity

HOCs are still important to understand because existing React codebases and libraries may use them.

## 17. Interview Answer

> **A Higher-Order Component is a function that takes a React component and returns a new enhanced component. It was commonly used to reuse component logic before Hooks. HOCs can inject props or add behavior, but they can also introduce wrapper nesting, prop collisions, and debugging complexity. In modern React, custom hooks often provide a simpler alternative for sharing logic.**

## 18. Why "Higher-Order"?

The idea comes from **higher-order functions** in JavaScript.

A higher-order function takes or returns a function:

```js
function createMultiplier(x) {
  return function multiply(y) {
    return x * y;
  };
}
```

Similarly:

```jsx
function withFeature(Component) {
  return EnhancedComponent;
}
```

It takes a component and returns another component.

## 19. Quick Comparison

| Pattern | Core Idea |
|---|---|
| Composition | Combine components |
| Compound Components | Components coordinate around shared state |
| Render Props | Function controls rendering |
| HOC | Wrap/enhance a component |
| Custom Hook | Reuse stateful logic directly |

## 20. Key Takeaways

1. **HOC = function that takes a component and returns an enhanced component.**
2. Common naming convention: `withSomething`.
3. HOCs can inject props and reusable behavior.
4. Good HOCs generally forward unrelated props.
5. Don't mutate the original component.
6. Multiple HOCs can be composed.
7. HOCs can cause wrapper hell and prop collisions.
8. Static properties and refs require special consideration.
9. HOCs were especially important before Hooks.
10. **Custom Hooks are often the preferred modern solution for logic reuse.**

### One-line memory trick

> **HOC = "Give me a component, I'll give you an enhanced component."**

## 🎯 Interview Priority

### 🔥🔥🔥 Know this

- HOC definition
- HOC syntax
- Why HOCs were used
- Prop forwarding
- Don't mutate the original component
- HOC vs Render Props
- HOC vs Custom Hooks
- Wrapper hell / drawbacks

### Lower priority

- Static property hoisting details
- Advanced ref handling
- Historical HOC libraries
