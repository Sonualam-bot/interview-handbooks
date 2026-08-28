# Chapter 42 — Custom Hooks

## 1. Core Mental Model

A custom Hook is a reusable JavaScript function that:

- starts with `use`
- can call other Hooks
- extracts reusable stateful/effectful React logic

Example:

```jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  function increment() {
    setCount(c => c + 1);
  }

  return { count, increment };
}
```

Usage:

```jsx
function Counter() {
  const { count, increment } = useCounter();

  return (
    <button onClick={increment}>
      {count}
    </button>
  );
}
```

> **Custom Hooks reuse logic, not state.**

---

## 2. The Most Important Idea

A custom Hook does **not** share state automatically.

If two components call:

```jsx
const counterA = useCounter();
const counterB = useCounter();
```

they have separate state.

Conceptually:

```text
Component A
 ↓
useCounter()
 ↓
State A

Component B
 ↓
useCounter()
 ↓
State B
```

The logic is shared.

The state is not.

---

## 3. Why Custom Hooks Exist

Without a custom Hook, the same stateful logic may be duplicated across components.

Instead of repeating:

```jsx
const [loading, setLoading] = useState(false);
const [data, setData] = useState(null);
const [error, setError] = useState(null);

// fetch logic...
```

extract it:

```jsx
function useFetch(url) {
  // reusable React logic
}
```

Then:

```jsx
const result = useFetch(url);
```

---

## 4. Custom Hook = Logic Extraction

Mental model:

```text
Component
   ↓
uses custom Hook
   ↓
Hook contains reusable logic
   ↓
Hook uses React Hooks
```

A custom Hook is not a special new React state mechanism.

It is a reusable function that follows the Rules of Hooks.

---

## 5. Custom Hooks Can Use Other Hooks

Example:

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return isOnline;
}
```

Usage:

```jsx
const isOnline = useOnlineStatus();
```

The component gets the reusable behavior without needing to know the implementation details.

---

## 6. Custom Hooks Follow the Rules of Hooks

If a custom Hook calls:

```jsx
useState();
useEffect();
useContext();
useRef();
```

it must follow the same Rules of Hooks.

Hooks must be called:

- at the top level
- consistently across renders
- from React components or custom Hooks

Don't do:

```jsx
function useCounter(condition) {
  if (condition) {
    useState(0);
  }
}
```

This violates the Rules of Hooks.

---

## 7. Why the Name Starts With `use`

Examples:

```text
useCounter
useFetch
useOnlineStatus
useAuth
useDebounce
```

The `use` prefix communicates:

> **This function is a custom Hook and follows Hook rules.**

It also allows Hooks tooling to recognize it.

---

## 8. Custom Hooks Can Return Anything

A custom Hook can return:

### Value

```jsx
return isOnline;
```

### Array

```jsx
return [count, increment];
```

### Object

```jsx
return {
  count,
  increment,
  decrement
};
```

Choose the API that makes sense for the consumer.

---

## 9. Example — `useToggle`

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = () => {
    setValue(v => !v);
  };

  return [value, toggle];
}
```

Usage:

```jsx
function Modal() {
  const [isOpen, toggle] = useToggle(false);

  return (
    <>
      <button onClick={toggle}>
        Toggle
      </button>

      {isOpen && <div>Modal</div>}
    </>
  );
}
```

The reusable part is the toggle state logic.

---

## 10. Example — `usePrevious`

```jsx
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}
```

Usage:

```jsx
const previousCount = usePrevious(count);
```

The Hook logic is reusable, but each component gets its own ref instance.

---

## 11. Custom Hooks Don't Need to Be Large

A custom Hook can be tiny:

```jsx
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);

  return [
    value,
    () => setValue(v => !v)
  ];
}
```

Or more complex:

```text
useFetch
useWebSocket
useDebounce
useForm
useOnlineStatus
```

The purpose isn't size.

The purpose is:

> **Reuse a piece of stateful React behavior.**

---

## 12. Custom Hooks vs Utility Functions

### Utility function

```jsx
function formatCurrency(amount) {
  return `$${amount}`;
}
```

General JavaScript logic. It doesn't use Hooks.

### Custom Hook

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  // effect logic...

  return width;
}
```

It encapsulates React-specific behavior.

Mental model:

```text
Utility
→ general JavaScript logic

Custom Hook
→ reusable React stateful/effectful logic
```

---

## 13. Custom Hooks Don't Automatically Share Data

Suppose:

```jsx
function useCounter() {
  const [count, setCount] = useState(0);

  return [count, setCount];
}
```

Then:

```jsx
function A() {
  const [count] = useCounter();
}

function B() {
  const [count] = useCounter();
}
```

This does **not** mean:

```text
A count === B count
```

Instead:

```text
A → own Hook state
B → own Hook state
```

To share actual state, you need an appropriate shared-state mechanism such as:

```text
lifting state up
Context
external state management
```

depending on the problem.

---

## 14. Custom Hooks Can Compose

A custom Hook can call another custom Hook:

```jsx
function useUser() {
  // ...
}

function useUserPermissions() {
  const user = useUser();

  // derive permissions...

  return permissions;
}
```

This is called **Hook composition**.

Mental model:

```text
useUserPermissions
       ↓
    useUser
       ↓
   React Hooks
```

---

## 15. Custom Hook and Component Lifecycle

A custom Hook does not have its own independent lifecycle.

Its Hooks participate in the lifecycle of the component calling it.

For example:

```jsx
function Component() {
  useMyHook();
}
```

If `useMyHook()` contains:

```jsx
useEffect(...)
```

that effect participates in the component's lifecycle.

Think:

```text
Component
   ↓
Custom Hook
   ↓
useEffect
```

not:

```text
Component lifecycle
+
separate Hook lifecycle
```

---

## 16. Interview Questions

### Q1. What is a custom Hook?

> A reusable function whose name starts with `use` and which can use other React Hooks to encapsulate reusable stateful or effectful logic.

### Q2. Do custom Hooks share state?

> No. Each component calling the custom Hook gets its own Hook state.

### Q3. Why must custom Hooks start with `use`?

> It communicates that the function is a Hook and allows Hooks tooling to recognize and enforce Hook rules.

### Q4. Can a custom Hook call another Hook?

> Yes. Custom Hooks can compose other Hooks.

### Q5. Custom Hook vs utility function?

> A utility function is general JavaScript logic; a custom Hook encapsulates reusable React Hook-based behavior.

### Q6. Does a custom Hook have its own lifecycle?

> No. Its Hooks participate in the lifecycle of the component that calls it.

---

# Quick Revision

```text
Custom Hook
→ reusable React logic
```

```text
Name
→ starts with use
```

```text
Can use Hooks
→ yes
```

```text
Can call another custom Hook
→ yes
```

```text
Shares logic
→ yes
```

```text
Shares state automatically
→ no
```

```text
Each component calls Hook
→ gets its own Hook state
```

```text
Utility function
→ general JS logic

Custom Hook
→ React-specific reusable logic
```

---

# Final Mental Model

```text
             Component A
                  ↓
             useCounter()
                  ↓
               State A


             Component B
                  ↓
             useCounter()
                  ↓
               State B
```

**Same logic. Separate state.**

> **Custom Hooks let you reuse stateful logic, not state itself.**

**Chapter 42 — COMPLETE**
