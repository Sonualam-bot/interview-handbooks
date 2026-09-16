# Chapter 52 — Compound Components `[NEW]`

**Handbook 4 — React Architecture & Design Patterns**

---

## 1. What Is the Compound Components Pattern?

Compound components are a set of components that work together to form one cohesive UI, sharing implicit state without the consumer having to wire that state through props by hand.

```jsx
<Select>
  <Select.Option value="a">Option A</Select.Option>
  <Select.Option value="b">Option B</Select.Option>
</Select>
```

`Select` and `Select.Option` are separate components, but they communicate with each other behind the scenes — the consumer never passes `selectedValue` or `onSelect` down to each `Option` manually.

> **Compound components split one piece of UI into cooperating parts that share state implicitly, while the consumer controls composition — which children to render, in what order, and with what markup in between.**

---

## 2. The Problem It Solves

Without this pattern, a component with several related, configurable parts tends to grow into one large component controlled entirely through props:

```jsx
<Select
  options={[
    { label: "Option A", value: "a" },
    { label: "Option B", value: "b" }
  ]}
  value={value}
  onChange={setValue}
  renderOption={(opt) => <span>{opt.label}</span>}
  optionClassName="..."
  disabledOptions={["b"]}
/>
```

Every new capability — a divider between groups of options, an icon before a label, a disabled state, custom styling per option — adds another prop, and the props keep multiplying because there's only one component (`Select`) trying to describe every possible internal layout through configuration (see Component Composition, Chapter 51, section 6 on boolean/config props hitting the same wall). The consumer is stuck configuring the *inside* of `Select` from the outside, through an ever-growing API surface, instead of just writing the markup they want directly.

---

## 3. Basic Example, Built From Scratch

```jsx
const SelectContext = createContext(null);

function Select({ value, onChange, children }) {
  return (
    <SelectContext.Provider value={{ value, onChange }}>
      <div className="select">{children}</div>
    </SelectContext.Provider>
  );
}

function Option({ value, children }) {
  const { value: selectedValue, onChange } = useContext(SelectContext);
  const isSelected = value === selectedValue;

  return (
    <div
      className={isSelected ? "option selected" : "option"}
      onClick={() => onChange(value)}
    >
      {children}
    </div>
  );
}

Select.Option = Option;
```

Usage:

```jsx
function App() {
  const [value, setValue] = useState("a");

  return (
    <Select value={value} onChange={setValue}>
      <Select.Option value="a">Option A</Select.Option>
      <Select.Option value="b">Option B</Select.Option>
    </Select>
  );
}
```

`Select` owns the shared state (`value`, `onChange`) and exposes it through Context. Each `Option` reads that Context to know whether it's selected and how to report a click — the consumer never threads `value`/`onChange` into each `Option` by hand.

---

## 4. How the Children Actually Communicate: Two Real Mechanisms

There are two different techniques historically used to implement this pattern, and understanding both — and why the second replaced the first — is the core of this chapter.

### Mechanism A (Legacy): `React.Children.map` + `cloneElement`

```jsx
function Select({ value, onChange, children }) {
  return (
    <div className="select">
      {React.Children.map(children, (child) =>
        cloneElement(child, {
          isSelected: child.props.value === value,
          onSelect: onChange
        })
      )}
    </div>
  );
}
```

Here, `Select` walks its own `children` prop (the React Elements passed to it, not yet rendered), and injects extra props (`isSelected`, `onSelect`) into each one before rendering them. This works, but has real structural limits:

- **It only works one level deep.** If a consumer wraps an `Option` inside a `<div>` for styling, `React.Children.map` sees the `<div>` as the child — not the `Option` inside it — and the injected props go to the wrong element.
- **It couples `Select` to knowing exactly what its children are.** `Select` has to assume every child is an `Option` (or specifically detect and skip anything else), which quietly limits how consumers can structure their markup.

### Mechanism B (Modern): Context

The example in Section 3 uses Context instead — `Select` doesn't need to know anything about its children at all. It just provides a value; any descendant, at any depth, wrapped in any amount of intermediate markup, can read that value with `useContext`. This is the real reason Context-based compound components became the standard approach once Context (and later Hooks) matured: **the sharing mechanism no longer depends on direct parent-child prop injection**, so consumers are free to nest, wrap, and rearrange the pieces however they want, and the shared state still reaches every descendant that asks for it.

```jsx
<Select value={value} onChange={setValue}>
  <div className="option-group">
    <Select.Option value="a">Option A</Select.Option>   {/* still works — Context doesn't care about depth */}
  </div>
</Select>
```

---

## 5. Why This Isn't Just "Composition With Extra Steps"

Plain composition (Chapter 51) passes data explicitly, through props or `children` — the parent decides exactly what each child receives. Compound components go further: the *children themselves* actively participate in a shared piece of state that no single explicit prop carries between them. `Select.Option` doesn't receive `value`/`onChange` as props at all in the Context version — it reaches into a shared Context that `Select` set up. This is the specific new capability compound components add on top of composition: implicit, ambient coordination between siblings that would otherwise need to pass data sideways through a mutual parent on every render.

---

## 6. Namespacing via Static Properties

```jsx
Select.Option = Option;
```

Attaching `Option` as a static property on `Select` is a naming convention, not a requirement of the pattern — `Select` and `Option` could just as easily be two separately exported components (`import { Select, Option } from "./select"`). The dot-notation exists purely for ergonomics: it visually signals "this component only makes sense inside a `Select`," and it avoids polluting a module's exports with generically-named pieces (`Option` on its own is a common, collision-prone name; `Select.Option` is not).

---

## 7. Compound Components vs Render Props

Both patterns let a component own logic while the consumer controls rendering — the difference is *where* the flexibility lives. A render prop hands the consumer a single function and one insertion point:

```jsx
<Toggle render={(on) => <span>{on ? "On" : "Off"}</span>} />
```

Compound components instead expose **multiple, independently-placeable pieces**, each able to read shared state on its own:

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Tab index={0}>One</Tabs.Tab>
    <Tabs.Tab index={1}>Two</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel index={0}>Content One</Tabs.Panel>
    <Tabs.Panel index={1}>Content Two</Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

A render prop can't easily express "these two independent trees of markup, rendered in totally different places in the DOM, both need access to the same internal state" — you'd need two separate render props, manually wired to the same state. Compound components handle this naturally because every piece independently subscribes to the same Context.

---

## 8. Compound Components vs Prop Drilling

The pattern is, in one sense, a deliberate alternative to prop drilling *within one logical component's own subtree* (contrast with Chapter 62, which covers prop drilling across a whole application). Instead of `Select` receiving a `children` array and manually forwarding `value`/`onChange` into each one via `cloneElement` (Mechanism A above, with all its depth limitations), Context lets every participating piece read the shared state directly, regardless of how deeply a consumer nests it.

---

## 9. Real-World Examples

This pattern is the backbone of most modern headless UI libraries — Radix UI, React Aria, Headless UI — precisely because those libraries need to expose *unstyled, structurally flexible* building blocks (`Dialog.Trigger`, `Dialog.Content`, `Dialog.Close`) that consumers can rearrange, wrap in their own markup, and style completely freely, while the library still coordinates open/close state, focus management, and accessibility attributes behind the scenes via Context. It also mirrors how native HTML elements like `<select>`/`<option>` or `<table>`/`<tr>`/`<td>` are structured: several distinct tags that only make sense combined, coordinating implicitly through the browser's own rendering rules.

---

## 10. The Limitation: Children Must Be Used Inside Their Provider

```jsx
<Select.Option value="a">Option A</Select.Option>   {/* ❌ outside any <Select> */}
```

Because `Option` depends on `SelectContext`, using it outside a `Select` means `useContext(SelectContext)` returns whatever default value `createContext` was given (often `null`), causing a runtime error the moment `Option` tries to read `.value`/`.onChange` off it. This is a real trade-off of the pattern: the pieces are not independently usable components — they're fragments of one larger, implicit contract, and nothing at the type level (without TypeScript discipline) stops a consumer from using one outside its required context.

---

## 11. When Should You Use This Pattern?

- A UI element naturally has several structurally-related, user-arrangeable parts (tabs, accordions, selects, menus, dialogs).
- You're building a reusable/shared component library where consumers need real layout flexibility, not just configuration through props.
- The number of configuration props on a single component is growing because it's trying to describe internal structure that would be more natural as actual JSX.

## 12. When Shouldn't You Use It?

- For a simple component with no meaningfully separate sub-parts — this adds Context and indirection for no benefit.
- When the "parts" never need independent placement — if they're always rendered together in a fixed layout, plain composition or even a single component is simpler.

---

## 13. Interview Questions

### Q1. What is the Compound Components pattern?

> A set of components that work together to form one UI, sharing implicit state (usually via Context) so the consumer can compose and arrange the pieces freely without manually wiring shared state through props.

### Q2. How do the pieces typically share state?

> Through React Context — the parent component provides shared state and handlers, and any descendant piece can read them with `useContext`, regardless of nesting depth.

### Q3. Why did Context replace `React.Children.map` + `cloneElement` for this pattern?

> `cloneElement` only injects props one level deep and requires the parent to know exactly what its children are; Context lets any descendant at any depth participate, so consumers can freely nest and wrap the pieces.

### Q4. How is this different from a render prop?

> A render prop exposes one function and one insertion point. Compound components expose multiple independently-placeable pieces that each subscribe to the same shared state on their own.

### Q5. What's the main limitation?

> The individual pieces (e.g. `Select.Option`) aren't independently usable — they depend on being rendered inside their corresponding provider, and using one outside it typically breaks at runtime.

---

# Quick Revision

```text
Parent component (Select)
      ↓
Provides shared state via Context
      ↓
Child pieces (Select.Option, at any depth)
      ↓
Read shared state via useContext
      ↓
Consumer freely arranges/wraps the pieces in JSX
```

- Compound components share implicit state between cooperating pieces.
- Modern implementations use Context, not `cloneElement` — Context has no depth limit.
- The dot-notation (`Select.Option`) is a naming convention, not a mechanism.
- More flexible than render props for multiple, independently-placed pieces.
- The trade-off: pieces only work inside their required provider.

# Final Mental Model

> **A single configurable component describes its internals through props. Compound components let the consumer describe those internals directly in JSX, while Context quietly keeps the pieces talking to each other.**

**Chapter 52 — COMPLETE**
