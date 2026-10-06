# Handbook 5 — React Performance Engineering

# Chapter 75 — Virtualization

## 1. The Problem Virtualization Solves

Imagine a list containing **100,000 items**.

A normal React list might create 100,000 React elements and DOM nodes even though most are not visible.

**Virtualization solves this by rendering only the items currently visible, plus a small buffer around them.**

```text
100,000 data items
        ↓
Determine visible range
        ↓
Render only visible items
        ↓
Update rendered window while scrolling
```

---

## 2. What Is Virtualization?

Virtualization is a rendering optimization where only the items currently visible in a scrollable viewport, plus a small **overscan** buffer, are mounted in the DOM.

Example:

```text
Data:
1  2  3  4  5  ... 100,000

Viewport:
     500 - 520

Rendered:
     ~490 - 530
```

The exact range depends on the configuration.

---

## 3. Why Does It Improve Performance?

The problem with huge lists is not necessarily the amount of data. It is the amount of **UI that React and the browser must render and maintain**.

Without virtualization:

```text
100,000 records
      ↓
100,000 React elements
      ↓
100,000 DOM nodes
```

With virtualization:

```text
100,000 records
      ↓
~30 visible/overscan items
      ↓
~30 DOM nodes
```

This dramatically reduces browser DOM work.

---

## 4. DOM Size Matters

A large DOM can increase:

- memory usage
- style calculation
- layout work
- painting work
- rendering overhead

Virtualization keeps the DOM small even when the underlying dataset is huge.

Remember:

```text
Data size ≠ DOM size

100,000 records → ~30 DOM rows
```

---

## 5. Virtualization vs Pagination

They solve different problems.

### Pagination

Controls how much data the user accesses or receives at a time.

```text
100,000 records
      ↓
Page 1 → 50 records
Page 2 → 50 records
Page 3 → 50 records
```

### Virtualization

Controls how many UI elements are rendered at once.

```text
100,000 records in memory
      ↓
Only ~30 rows mounted
```

**Pagination controls data access. Virtualization controls rendering.**

They can be combined.

---

## 6. Core Mental Model

A virtualized list needs to determine which items should currently be rendered.

It uses information such as:

- scroll position
- viewport height
- item size
- total item count
- overscan

```text
Scroll position
      +
Viewport size
      +
Item dimensions
      ↓
Visible range
      ↓
Render only that range
```

As the user scrolls:

```text
Scroll
  ↓
Visible range changes
  ↓
Old items removed
  ↓
New items rendered
```

---

## 7. How Can Scrolling Still Work?

If only 30 rows exist in the DOM, how can the scrollbar represent 100,000 rows?

The virtualization system creates a **virtual total height** representing the entire list.

For fixed-height rows:

```text
rowHeight = 40px

100,000 × 40
=
4,000,000px
```

The browser behaves as if the complete list occupies that much vertical space, while only visible rows are mounted.

---

## 8. Positioning Visible Items

For fixed-height rows, positioning is straightforward:

```jsx
style={{
  position: "absolute",
  top: index * rowHeight
}}
```

If:

```text
rowHeight = 40px
```

then:

```text
item 0 → 0px
item 1 → 40px
item 2 → 80px
item 3 → 120px
```

This allows a small number of DOM nodes to represent a much larger virtual list.

---

## 9. Overscan

If virtualization rendered only the exact visible items, fast scrolling could cause visible gaps.

Instead, it usually renders a small buffer around the viewport.

```text
Viewport:
500 - 520

Overscan:
490 - 530
```

The extra items are called **overscan**.

Overscan helps:

- reduce visible gaps
- make scrolling smoother
- reduce immediate item replacement/churn

---

## 10. Overscan Tradeoff

More overscan is not automatically better.

### Too little

```text
smaller DOM
+
more frequent item replacement
```

### Too much

```text
larger DOM
+
more rendering work
```

Therefore:

> Overscan is a tradeoff between smooth scrolling and rendering work.

---

## 11. Fixed Height vs Variable Height

### Fixed-height items

```text
rowHeight = 40px

top = index × rowHeight
```

Position calculation is simple.

### Variable-height items

```text
Row 1 → 40px
Row 2 → 80px
Row 3 → 55px
Row 4 → 100px
```

The system needs additional work such as:

- estimating sizes
- measuring actual sizes
- caching measurements
- recalculating positions

Therefore, variable-height virtualization is more complex.

---

## 12. Virtualization Libraries

Common React virtualization libraries include:

- `react-window`
- TanStack Virtual

They can handle concerns such as:

- visible range calculation
- item positioning
- overscan
- scrolling
- fixed/variable dimensions depending on configuration

---

## 13. Virtualization Is Not Data Fetching

A common interview mistake is:

> "Virtualization means loading only the visible data from the server."

Not necessarily.

You can have:

```text
API returns:
100,000 records

Memory:
100,000 records

DOM:
~30 rows
```

Virtualization controls rendering.

To reduce data transferred from the server, you might use:

- pagination
- cursor pagination
- infinite scrolling
- server-side filtering
- server-side search

---

## 14. Virtualization + Infinite Scrolling

These can work together.

### Infinite scrolling

Controls:

> When should more data be fetched?

### Virtualization

Controls:

> How many loaded items should be rendered?

```text
Initial API request
      ↓
Load 1,000 items
      ↓
Virtualization renders ~30
      ↓
User scrolls
      ↓
Load more data
      ↓
Virtualization still renders only the visible window
```

So:

```text
Infinite scrolling → data loading strategy
Virtualization     → rendering strategy
```

---

## 15. Virtualization + React.memo

They optimize different things.

```text
Virtualization
    ↓
Fewer mounted/rendered items

React.memo
    ↓
Fewer unnecessary component renders
```

They can be combined, but don't automatically use both. Measure first.

---

## 16. Interview Example

Suppose an application displays:

```text
100,000 users
```

Each row contains:

- avatar
- name
- email
- status
- actions

A naive implementation might create 100,000 rows.

A virtualized implementation might render:

```text
Visible rows: 20
Overscan: 10 above + 10 below

Total DOM rows ≈ 40
```

As the user scrolls:

```text
Visible range changes
        ↓
Old rows leave the window
        ↓
New rows enter the window
        ↓
DOM stays relatively small
```

---

## 17. Conceptual Virtualized List

A simplified mental model:

```jsx
function VirtualList({ items }) {
  const visibleItems = calculateVisibleItems();

  return (
    <div className="viewport">
      <div
        className="virtual-container"
        style={{ height: totalHeight }}
      >
        {visibleItems.map(item => (
          <Row key={item.id} item={item} />
        ))}
      </div>
    </div>
  );
}
```

The exact implementation is less important than the architecture:

```text
Total virtual space
        +
Small rendered window
        +
Positioned visible items
```

---

## 18. What Virtualization Does NOT Solve

Virtualization does not automatically solve:

### Expensive row rendering

If each visible row performs expensive work, virtualization only limits how many rows exist.

### Large API responses

If the server sends 100,000 records, virtualization does not reduce network transfer.

### Expensive data processing

If you perform a huge calculation before rendering:

```js
const processed = expensiveOperation(items);
```

virtualization does not automatically optimize that calculation.

You may need:

- memoization
- server-side processing
- better algorithms
- caching
- pagination

---

## 19. When Should You Consider Virtualization?

Good candidates:

- large tables
- large lists
- logs
- chat histories
- search results
- file explorers
- large dropdowns
- grids
- feeds with thousands of items

Especially when profiling shows rendering or DOM size is a bottleneck.

---

## 20. When You Probably Don't Need It

For a small list:

```text
20 items
50 items
100 simple items
```

virtualization may add unnecessary complexity.

A normal React list is often sufficient.

> Don't introduce virtualization just because the list is an array. Use it when the rendering cost justifies the complexity.

---

## 21. Interview Definition

> **Virtualization is a rendering optimization where only the items currently visible in a scrollable viewport, plus a small overscan buffer, are mounted in the DOM. The virtualization system calculates the visible range and updates the rendered window as the user scrolls. This reduces DOM size, memory usage, and rendering work for large collections.**

---

## 22. Common Interview Follow-ups

### Is virtualization the same as pagination?

No.

```text
Pagination → controls data access
Virtualization → controls rendering
```

### Does virtualization reduce API calls?

Not by itself. It can be combined with pagination or infinite scrolling.

### Why do we need overscan?

To render a small buffer around the viewport and reduce gaps/churn during scrolling.

### Why are variable-height rows harder?

Because the system cannot simply calculate:

```text
top = index × rowHeight
```

It needs measurements or estimates.

### Can virtualization and React.memo be used together?

Yes.

```text
Virtualization → fewer mounted/rendered items
React.memo     → fewer unnecessary component renders
```

---

## 23. Mental Model

Remember virtualization as:

```text
Huge dataset
     ↓
Determine visible range
     ↓
Add overscan
     ↓
Render small window
     ↓
Position items inside virtual space
     ↓
Update window while scrolling
```

Core idea:

```text
100,000 records
        ↓
~30 DOM rows
```

---

## 24. Golden Rules

1. Virtualization renders only a small visible window of a large collection.
2. Overscan adds a buffer around the visible range.
3. Virtualization reduces DOM size and rendering work.
4. Virtualization is different from pagination.
5. Virtualization does not inherently reduce API calls.
6. Infinite scrolling controls data loading; virtualization controls rendering.
7. Fixed-height rows are easier to virtualize.
8. Variable-height rows require measurement or estimation.
9. `React.memo` and virtualization solve different performance problems.
10. Small lists usually don't need virtualization.
11. Measure performance before introducing it.
12. The key distinction is **data size vs DOM size**.

---

## 25. Quick Interview Summary

```text
Virtualization
      ↓
Render only visible items
      +
Overscan buffer
      ↓
Keep DOM small
      ↓
Reduce rendering/layout/memory overhead
```

### One-line answer

> **Virtualization keeps the DOM small by rendering only the visible portion of a large collection instead of mounting every item at once.**
