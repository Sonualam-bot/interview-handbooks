# Chapter 82 — React Router

## 1. Why React Router Exists

A URL identifies a location or resource in an application. In a React single-page application (SPA), the browser usually loads the application shell once, and React renders different screens as the URL changes.

Without a routing library, you might inspect `window.location.pathname` yourself:

```jsx
function App() {
  const path = window.location.pathname;

  if (path === "/") return <HomePage />;
  if (path === "/products") return <ProductsPage />;
  if (path === "/about") return <AboutPage />;

  return <NotFoundPage />;
}
```

This quickly becomes insufficient. Reading `pathname` does not automatically make React rerender when the URL changes. You would need to manage history events, dynamic route matching, nested layouts, query parameters, navigation, redirects, and not-found states yourself.

**React Router** coordinates the browser location with the React UI. Its broad flow is:

```text
Browser URL
    ↓
React Router reads location
    ↓
Matches the route configuration
    ↓
Builds the matched route hierarchy
    ↓
Renders the corresponding React elements
```

The URL and the UI remain connected, while navigation can happen without a full document reload.

## 2. Install and Set Up

For a typical browser-based React app:

```bash
npm install react-router-dom
```

A minimal setup:

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/products" element={<ProductsPage />} />
        <Route path="/about" element={<AboutPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

- `BrowserRouter` connects routing to the browser's History API.
- `Routes` selects the best matching route branch.
- `Route` describes a path and the UI element associated with it.

The router should generally wrap the portion of the app that needs routing. In a typical SPA, that means wrapping the app near its root.

## 3. `BrowserRouter`: The Browser Location Connection

`BrowserRouter` uses normal URL paths, such as:

- `/`
- `/products`
- `/products/42`
- `/settings/profile`

It uses browser history so navigation can update the URL without requesting an entirely new HTML document for each internal navigation.

This does **not** remove the need for server configuration. If someone directly opens or refreshes `/products/42`, the web server must be configured to serve the SPA entry document for that path; otherwise, the server may return a 404 before React Router runs.

### BrowserRouter versus hash routing

With `BrowserRouter`, the route is represented in the path:

```text
https://example.com/products
```

With hash-based routing, it is represented after `#`:

```text
https://example.com/#/products
```

Hash routing can be useful when server fallback configuration cannot be changed. `BrowserRouter` generally produces cleaner URLs when the server is configured correctly.

## 4. `<Link>` versus `<a>`

Use React Router's `<Link>` for normal internal navigation:

```jsx
import { Link } from "react-router-dom";

function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/products">Products</Link>
      <Link to="/about">About</Link>
    </nav>
  );
}
```

A normal anchor such as `<a href="/products">` asks the browser to navigate to that URL as a document navigation. In a conventional SPA setup, this usually reloads the document.

`<Link>` intercepts an ordinary internal navigation and lets the router update history and render the matching route without a full reload.

Use a regular `<a>` when navigation should genuinely leave the SPA or point to an external resource. Avoid using a router link for an external destination.

## 5. Programmatic Navigation with `useNavigate`

Sometimes navigation follows an action rather than a click on a link: a form succeeds, a user completes onboarding, or an operation requires redirecting.

```jsx
import { useNavigate } from "react-router-dom";

function CreateProductForm() {
  const navigate = useNavigate();

  async function handleSubmit(event) {
    event.preventDefault();

    // Submit data to the server here.
    const productId = "42";

    navigate(`/products/${productId}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Create product</button>
    </form>
  );
}
```

Common uses:

```jsx
navigate("/dashboard");       // Navigate to a path
navigate(-1);                 // Go back in history
navigate(1);                  // Go forward in history
navigate("/login", { replace: true }); // Replace current history entry
```

Use declarative links for navigation that the user can see and choose. Use `useNavigate` when navigation is a consequence of application logic.

**Important:** `navigate(-1)` assumes there is a meaningful previous history entry. If the user opened the page directly, going back may leave the app.

## 6. Dynamic Routes and `useParams`

A route can contain a parameter:

```jsx
import { Routes, Route, useParams } from "react-router-dom";

function ProductDetails() {
  const { productId } = useParams();

  return <h1>Product: {productId}</h1>;
}

function App() {
  return (
    <Routes>
      <Route path="/products/:productId" element={<ProductDetails />} />
    </Routes>
  );
}
```

For `/products/42`, `useParams()` returns an object containing `productId: "42"`.

Route parameters are strings. If the value represents a number, validate and convert it deliberately:

```jsx
const { productId } = useParams();
const id = Number(productId);

if (!Number.isInteger(id) || id <= 0) {
  return <p>Invalid product ID.</p>;
}
```

A route parameter identifies a resource or route segment. It does not automatically fetch the resource or authorize access to it. The component or data-loading layer must perform those tasks.

## 7. Nested Routes and `<Outlet>`

Many pages share a layout. For example, a dashboard may have a sidebar and header while its main panel changes between overview, reports, and settings.

Nested routes model this hierarchy directly:

```jsx
import { Routes, Route, Outlet, Link } from "react-router-dom";

function DashboardLayout() {
  return (
    <div className="dashboard">
      <aside>
        <Link to="/dashboard">Overview</Link>
        <Link to="/dashboard/reports">Reports</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </aside>

      <main>
        <Outlet />
      </main>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardOverview />} />
        <Route path="reports" element={<ReportsPage />} />
        <Route path="settings" element={<SettingsPage />} />
      </Route>
    </Routes>
  );
}
```

The parent route renders `DashboardLayout`. `<Outlet>` marks where the matched child route renders.

For `/dashboard/reports`, the result is conceptually:

```text
DashboardLayout
├── Sidebar and shared header
└── Outlet
    └── ReportsPage
```

Child paths are relative to the parent in this configuration. Writing `path="reports"` under `/dashboard` creates `/dashboard/reports`.

### Index routes

An index route renders at the parent's exact path:

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardOverview />} />
  <Route path="reports" element={<ReportsPage />} />
</Route>
```

The index route renders for `/dashboard`; it is not a child URL segment of its own.

## 8. Route Parameters versus Query Parameters

These encode different kinds of information.

| Type | Example | Typical meaning |
|---|---|---|
| Path parameter | `/products/42` | Which product/resource |
| Query parameter | `/products?category=shoes&page=2` | How to filter, sort, or paginate a collection |

Read query parameters with `useSearchParams`:

```jsx
import { useSearchParams } from "react-router-dom";

function ProductsPage() {
  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get("category") ?? "all";
  const page = Number(searchParams.get("page") ?? "1");

  function goToNextPage() {
    const next = new URLSearchParams(searchParams);
    next.set("page", String(page + 1));
    setSearchParams(next);
  }

  return (
    <>
      <p>Category: {category}</p>
      <p>Page: {page}</p>
      <button onClick={goToNextPage}>Next page</button>
    </>
  );
}
```

Query parameters are strings or `null`; parse and validate them before using them as numbers, enums, or other structured values.

A useful rule: put state in the URL when users should be able to bookmark it, share it, or restore it by refreshing. Examples include search terms, filters, sorting, and pagination.

## 9. URL State versus Component State

Not all state belongs in the URL.

**URL state** is appropriate when it identifies the current navigable view or should survive a refresh and be shareable:

- Search query
- Selected category
- Page number
- Sort order
- Selected resource ID

**Component state** is appropriate for short-lived interface details:

- Whether a dropdown is open
- Hover state
- A temporary tooltip
- An unsaved input value that should not yet affect the URL

The choice depends on product behavior. For example, a search input may use local state while the user types and update the URL when the search is submitted or debounced.

Avoid storing the same source of truth independently in both the URL and local state unless there is a clear synchronization strategy.

## 10. Protected Routes: A UI Pattern, Not a Security Boundary

A simple protected-route wrapper can conditionally render a page:

```jsx
import { Navigate, Outlet, useLocation } from "react-router-dom";

function RequireAuth({ isAuthenticated }) {
  const location = useLocation();

  if (!isAuthenticated) {
    return (
      <Navigate
        to="/login"
        replace
        state={{ from: location }}
      />
    );
  }

  return <Outlet />;
}
```

Usage:

```jsx
<Routes>
  <Route path="/login" element={<LoginPage />} />

  <Route element={<RequireAuth isAuthenticated={isAuthenticated} />}>
    <Route path="/dashboard" element={<DashboardPage />} />
    <Route path="/settings" element={<SettingsPage />} />
  </Route>
</Routes>
```

`<Navigate>` performs declarative navigation. `replace` avoids leaving the protected URL as the immediate history entry. The location can be stored so the app can return the user to the page they originally requested after login.

However, **client-side route protection is not authorization**. Users can inspect or alter client-side code and requests. The backend must authenticate requests and authorize access to protected data and operations.

Also consider loading states: if authentication status is initially unknown, do not immediately redirect before the app has resolved the session.

## 11. `useLocation`

`useLocation` exposes information about the current location, including `pathname`, `search`, `hash`, `state`, and a location `key`.

```jsx
import { useLocation } from "react-router-dom";

function CurrentLocation() {
  const location = useLocation();

  return (
    <pre>
      {JSON.stringify(
        {
          pathname: location.pathname,
          search: location.search,
          hash: location.hash,
        },
        null,
        2
      )}
    </pre>
  );
}
```

It is useful when UI behavior depends on the current location, such as analytics, scroll restoration, active navigation, or reacting to navigation changes. Be intentional about effects that run when location changes so they do not cause unnecessary work or loops.

## 12. Not-Found Routes

A catch-all route can render a not-found page:

```jsx
<Routes>
  <Route path="/" element={<HomePage />} />
  <Route path="/products" element={<ProductsPage />} />
  <Route path="*" element={<NotFoundPage />} />
</Routes>
```

A not-found route handles URLs that do not match the configured route tree on the client. The server still needs suitable behavior for direct requests and refreshes.

## 13. Common Mistakes

1. **Using `<a>` for every internal link.** This can trigger full document reloads instead of SPA navigation.
2. **Expecting `pushState` alone to rerender React.** The History API changes browser history; the router coordinates that change with React.
3. **Forgetting server fallback configuration.** Client-side routing cannot handle a direct request if the server never serves the app.
4. **Treating route params as numbers.** Params are strings and should be parsed and validated.
5. **Confusing path and query params.** A resource ID commonly belongs in the path; filters and pagination commonly belong in the query string.
6. **Forgetting `<Outlet>`.** A nested child route needs an outlet in the parent layout to render at the intended location.
7. **Using programmatic navigation for every link.** Prefer `<Link>` when the user is choosing a destination.
8. **Assuming protected routes secure the backend.** Server-side authorization is still required.
9. **Duplicating URL state in local state without synchronization.** This can create conflicting values.
10. **Redirecting while authentication is unresolved.** Distinguish unauthenticated from still-loading session state.

## 14. Interview Questions and Answers

### Q1. What problem does React Router solve?

It maps browser locations to React UI and coordinates navigation, route matching, history, nested layouts, parameters, and redirects without requiring a full document reload for each internal navigation.

### Q2. What is the difference between `<Link>` and `useNavigate`?

`<Link>` represents a user-visible navigation choice. `useNavigate` performs navigation from application logic, such as after a successful form submission.

### Q3. What is the purpose of `<Outlet>`?

It defines where the matched child route renders inside a parent route's layout.

### Q4. What is an index route?

A child route that renders at the exact URL of its parent, without adding another path segment.

### Q5. How do path params differ from query params?

Path params are part of the route path and commonly identify a specific resource. Query params follow `?` and commonly describe filters, sorting, search, or pagination.

### Q6. Why might refreshing a nested route return 404?

The browser requests that URL directly from the server. If the server is not configured to serve the SPA entry document for client routes, React Router never gets a chance to match the route.

### Q7. Are protected routes secure?

They are a client-side UI control. The backend must still authenticate requests and authorize protected operations and data.

### Q8. When should state be in the URL?

When it should be bookmarkable, shareable, restorable on refresh, or part of the navigable view, such as filters, search, sorting, and pagination.

### Q9. What does `navigate(path, { replace: true })` do?

It replaces the current history entry rather than adding another entry, which can be useful for redirects where returning to the previous route would be undesirable.

### Q10. Does `useParams()` fetch data?

No. It reads values matched from the URL. The application uses those values to fetch data or select the relevant resource.

## 15. Practice Exercises

1. Build routes for `/`, `/products`, `/products/:productId`, and a not-found page.
2. Create a dashboard layout with nested Overview, Reports, and Settings routes using `<Outlet>`.
3. Add `category`, `page`, and `sort` query parameters to a product listing. Make pagination update the URL.
4. Implement a login redirect that remembers the requested location and returns there after authentication.
5. Explain why a client-side route may work through in-app navigation but fail on refresh.
6. Decide which of these belong in the URL and justify your answer: open modal, search query, selected product, sidebar expanded, page number.

## Final Mental Model

React Router is the layer connecting **the browser's location and history** to **the React route tree**.

- The URL describes the current navigable location.
- Route configuration describes which UI corresponds to that location.
- Nested routes express shared layouts.
- `<Outlet>` renders matched child routes.
- `<Link>` handles user-initiated internal navigation.
- `useNavigate` handles navigation caused by application logic.
- Params identify route segments; query params represent additional view state.
- The server must support direct requests to SPA routes.
- Client-side route guards improve UX but do not replace backend authorization.
