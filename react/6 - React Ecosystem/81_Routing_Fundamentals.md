# Handbook 6 — React Ecosystem
# Chapter 81 — Routing Fundamentals

## 1. What Problem Does Routing Solve?

Routing is the process of deciding what application/resource should respond to a given URL.

```text
/                  → Home
/products          → Products
/products/42       → Product Details
/login             → Login
```

A route is essentially a mapping between a URL pattern and application behavior/view.

---

## 2. What Happens When You Enter a URL?

For a traditional website:

```text
Browser
   ↓
DNS
   ↓
Server
   ↓
HTTP request
   ↓
Server decides what resource to return
   ↓
HTTP response
   ↓
Browser renders document
```

Example:

```http
GET /products HTTP/1.1
Host: example.com
```

The server receives `/products` and decides what response to return.

---

## 3. Traditional Multi-Page Applications

Historically:

```text
Request /products
       ↓
Server
       ↓
HTML
       ↓
Browser
```

Navigating to another page:

```text
/products/42
       ↓
Server
       ↓
New HTML document
       ↓
Browser
```

The browser performs a new document navigation.

---

## 4. Single Page Applications

In an SPA, JavaScript can update the UI without downloading a completely new HTML document for every navigation.

Initial load:

```text
Browser
   ↓
GET /
   ↓
Server
   ↓
HTML + JavaScript
   ↓
React application starts
```

Client-side navigation:

```text
User clicks /products
        ↓
JavaScript changes URL
        ↓
Router determines matching route
        ↓
React renders Products
```

This is **client-side routing**.

---

## 5. Server-Side Routing vs Client-Side Routing

### Server-side routing

```text
URL
 ↓
Server
 ↓
Route matching
 ↓
Response
 ↓
Browser
```

The server decides what resource/document to return.

### Client-side routing

```text
URL
 ↓
Browser JavaScript
 ↓
Route matching
 ↓
React UI
```

The client application decides what UI to render.

---

## 6. Who Changes the URL?

The browser provides APIs for controlling navigation.

For SPA routing, the **History API** is especially important.

```js
window.history.pushState({}, "", "/products");
```

This changes the browser URL without performing a normal full-page navigation.

---

## 7. `pushState()` Does Not Render React

```js
window.history.pushState({}, "", "/products");
```

only changes browser history/URL state.

It does **not** automatically tell React to render `Products`.

Conceptually:

```text
pushState()
    ↓
URL changes
    ↓
Router detects navigation
    ↓
Router determines matching route
    ↓
React renders corresponding UI
```

---

## 8. Browser History

Browsers maintain navigation history:

```text
/
↓
/products
↓
/products/42
```

Pressing Back moves from:

```text
/products/42
```

to:

```text
/products
```

The application needs to respond to that navigation.

---

## 9. Important History API Methods

### `pushState()`

Adds a history entry.

```js
history.pushState({}, "", "/products");
```

### `replaceState()`

Replaces the current history entry.

```js
history.replaceState({}, "", "/login");
```

### `back()`

```js
history.back();
```

Moves backward in history.

### `forward()`

```js
history.forward();
```

Moves forward in history.

---

## 10. `popstate`

When the user moves through browser history using Back/Forward, the browser can emit a `popstate` event.

```js
window.addEventListener("popstate", () => {
  console.log("History navigation occurred");
});
```

Conceptually:

```text
User clicks Back
       ↓
Browser changes history entry
       ↓
popstate
       ↓
Router detects navigation
       ↓
Route matching
       ↓
UI updates
```

---

## 11. The Core Routing Problem

A client-side router coordinates:

```text
          URL
           │
           ↓
     Route matching
           │
           ↓
       UI rendering
```

Navigation adds:

```text
User action
    ↓
Change history
    ↓
URL changes
    ↓
Match route
    ↓
Render correct UI
```

This is the foundation on which React Router builds.

---

## 12. Route Matching

Suppose the application defines:

```text
/
/products
/products/:id
/login
```

The router receives:

```text
/products/42
```

It determines that:

```text
/products/:id
```

matches and extracts:

```text
id = 42
```

Routing therefore requires matching URLs against route patterns.

---

## 13. Dynamic Segments

A route can contain a dynamic segment:

```text
/products/:id
```

These URLs can match:

```text
/products/1
/products/42
/products/999
```

For:

```text
/products/42
```

the router extracts:

```text
id = 42
```

The value can then be used by the application:

```jsx
<Product id="42" />
```

---

## 14. Query Parameters

Consider:

```text
/products?category=shoes&page=2
```

There are two concepts.

### Path

```text
/products
```

### Query string

```text
?category=shoes&page=2
```

Query parameters commonly represent:

- Filtering
- Sorting
- Pagination
- Search
- UI state

---

## 15. Path Parameters vs Query Parameters

### Path parameter

Usually identifies **which resource**:

```text
/products/42
```

```text
id = 42
```

### Query parameter

Usually modifies **how the resource is viewed/requested**:

```text
/products?page=2&sort=price
```

```text
page = 2
sort = price
```

This is a useful convention, not an absolute rule.

---

## 16. Hash Routing

Another approach uses the URL fragment:

```text
example.com/#/products
```

Everything after `#` is the URL fragment.

The server essentially receives:

```text
/
```

while JavaScript can interpret:

```text
/products
```

from the fragment.

Examples:

```text
/
/#/products
/#/products/42
```

---

## 17. History Routing vs Hash Routing

### History-based routing

```text
example.com/products
```

Uses the History API.

### Hash-based routing

```text
example.com/#/products
```

Uses the URL fragment.

History routing provides cleaner URLs, but introduces an important deployment consideration.

---

## 18. The SPA Refresh Problem

Suppose the React application uses:

```text
/products
```

Client-side navigation works.

Then the user refreshes.

The browser sends:

```http
GET /products
```

to the server.

If the server does not know how to handle `/products`, it may return:

```text
404 Not Found
```

Why?

Because during a hard refresh:

```text
Browser
   ↓
GET /products
   ↓
Server
```

React has not started yet.

---

## 19. Why SPA Deployment Needs Server Configuration

During client-side navigation:

```text
React application
      ↓
Router handles /products
      ↓
Products component
```

During refresh:

```text
Browser
   ↓
GET /products
   ↓
Server
```

Therefore:

> Client-side routing does not eliminate the need for server configuration.

The server generally needs to serve the SPA entry document for application routes.

Conceptually:

```text
/products
/users
/settings
```

can fall back to:

```text
index.html
```

Then:

```text
index.html
   ↓
React starts
   ↓
Client router reads current URL
   ↓
Correct UI renders
```

---

## 20. Complete SPA Navigation Flow

### Initial page load

```text
Browser requests /
        ↓
Server returns index.html
        ↓
Browser downloads JavaScript
        ↓
React starts
        ↓
Router reads current URL
        ↓
Route matching
        ↓
Correct UI rendered
```

### Client-side navigation

```text
User clicks Products
        ↓
Router intercepts navigation
        ↓
History updated
        ↓
URL becomes /products
        ↓
Router matches /products
        ↓
React renders Products
```

### Browser Back

```text
User clicks Back
        ↓
Browser moves through history
        ↓
popstate
        ↓
Router detects new URL
        ↓
Route matching
        ↓
React renders matching UI
```

### Browser refresh

```text
Refresh /products
        ↓
Browser requests GET /products
        ↓
Server serves SPA entry
        ↓
React starts
        ↓
Router reads /products
        ↓
Products UI renders
```

---

## 21. What Does a Router Provide?

A routing library generally handles:

```text
Router
 ├── URL observation
 ├── Navigation
 ├── History management
 ├── Route matching
 ├── Dynamic parameters
 ├── Nested routes
 ├── Navigation state
 └── Rendering matching UI
```

React Router provides abstractions around these concepts.

---

## 22. Why Use a Routing Library?

A primitive router could be:

```js
const routes = {
  "/": Home,
  "/products": Products,
  "/login": Login
};
```

Then:

```js
const Component = routes[window.location.pathname];
```

But production applications need:

- Dynamic routes
- Nested routes
- Route parameters
- Query parameters
- Navigation
- Back/forward handling
- Redirects
- Route protection
- Lazy loading
- Error handling
- Data loading
- Nested layouts

A routing library provides these abstractions instead of rebuilding them manually.

---

## 23. URL Is Application State

The URL is a form of application state represented in the browser's address bar.

For example:

```text
/products?page=2&sort=price
```

contains state that can influence what the UI displays.

Conceptually:

```text
URL
 ↓
Router
 ↓
Route state
 ↓
React component tree
 ↓
UI
```

This is one reason routing and state management are closely related.

---

## 24. Routing and React Are Separate Concepts

React's core concern is rendering UI based on state:

```text
UI = f(state)
```

Routing introduces another source of application state:

```text
URL = navigation state
```

The router connects them:

```text
URL
 ↓
Router
 ↓
React state/tree
 ↓
UI
```

React itself is not the routing system.

---

# 25. Interview Questions

### Q1. What is client-side routing?

> Client-side routing is a navigation mechanism where the browser-side application changes the URL and renders the corresponding UI without requiring a full document reload for every navigation.

### Q2. Does `history.pushState()` render React?

**No.**

It changes browser history and the URL. The router/application must respond to the navigation and update the UI.

### Q3. Why does the SPA refresh problem happen?

Because client-side navigation is handled by JavaScript, but a browser refresh sends the current URL directly to the server before React starts. The server must therefore serve the SPA entry document for application routes.

### Q4. What is a dynamic route?

```text
/products/:id
```

matches:

```text
/products/42
```

and extracts:

```text
id = 42
```

### Q5. Path parameter vs query parameter?

```text
/products/42
```

usually identifies a resource.

```text
/products?page=2&sort=price
```

usually modifies how the resource is requested/viewed.

### Q6. What is `popstate`?

A browser event associated with moving through history entries, such as Back/Forward navigation.

---

# 🧠 Final Mental Model

```text
                    BROWSER
                       │
                       │ URL
                       ↓
                    ROUTER
                       │
              ┌────────┴────────┐
              ↓                 ↓
        Route Matching      Navigation
              │                 │
              ↓                 ↓
        React UI Tree      History API
              │                 │
              └────────┬────────┘
                       ↓
                      UI
```

### Traditional Web

```text
URL → Server → HTML → Browser
```

### SPA Navigation

```text
URL → Client Router → React UI
          │
          └── History API
```

### SPA Refresh

```text
URL
 ↓
Server
 ↓
SPA entry
 ↓
React
 ↓
Client Router
 ↓
UI
```

> **Core idea:** Routing connects the browser's URL/history with the application's UI.
