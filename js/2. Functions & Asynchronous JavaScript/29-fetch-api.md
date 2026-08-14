# Chapter 29 — Fetch API

> *"Every modern web application talks to servers. The Fetch API is JavaScript's standard way of asking another computer for information."*

---

# A Mystery

Consider this code.

```js
const response = await fetch("/api/users");
```

At first glance, it almost looks like:

```js
const response = getUsers();
```

But they're very different.

One returns data immediately.

The other begins a network request that may take milliseconds—or several seconds.

So what exactly happens when you call `fetch()`?

---

# Becoming the JavaScript Engine

You encounter:

```js
fetch("/api/users");
```

You cannot contact another computer yourself.

Instead, you ask the runtime (the browser or Node.js) to perform the HTTP request.

Immediately, you receive a Promise.

```text
JavaScript

↓

Runtime starts network request

↓

Promise returned immediately

↓

Program continues executing
```

Later, when the response arrives, the Promise settles and your callback or `await` continues execution.

---

# What Is the Fetch API?

The **Fetch API** is a built-in JavaScript API for making HTTP requests.

It allows your application to:

- retrieve data,
- send data,
- update resources,
- delete resources,

using standard HTTP methods.

---

# A Simple GET Request

```js
const response = await fetch("/api/users");

const users = await response.json();

console.log(users);
```

Notice there are two asynchronous operations:

1. Waiting for the HTTP response.
2. Reading and parsing the response body.

Both return Promises.

---

# Sending Data

Fetching isn't only about reading.

You can also send data.

```js
await fetch("/api/users", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        name: "Sonu"
    })
});
```

Here the browser sends an HTTP POST request containing JSON.

---

# The Response Object

`fetch()` does **not** directly return your JSON.

It returns a **Response** object.

```js
const response = await fetch("/api/users");
```

From that object you can access:

```js
response.status
response.ok
response.headers
response.json()
response.text()
```

This separation lets you inspect metadata before reading the body.

---

# Error Handling

A common misconception is:

> "fetch throws an error for every unsuccessful HTTP status."

It doesn't.

For example:

```text
404 Not Found
500 Internal Server Error
```

still produce a successful network request.

You should check:

```js
if (!response.ok) {
    throw new Error("Request failed");
}
```

Network failures themselves reject the Promise.

---

# A Real-World Analogy

Imagine ordering a package online.

You first receive confirmation that the delivery truck has arrived.

Only after opening the package do you see what's inside.

`fetch()` behaves similarly.

The Response object is the package.

Calling `response.json()` is opening it.

---

# React Connection

Fetching data is one of the most common tasks in React.

Examples include:

```js
useEffect(() => {
    async function loadUsers() {
        const response = await fetch("/api/users");
        const users = await response.json();

        setUsers(users);
    }

    loadUsers();
}, []);
```

Libraries such as React Query and SWR are built on the same underlying idea: making requests with Promises and updating the UI when results arrive.

---

# Key Takeaways

- `fetch()` is JavaScript's standard HTTP API.
- It immediately returns a Promise.
- The resolved value is a Response object, not parsed JSON.
- Reading the body (`json()`, `text()`, etc.) is another asynchronous step.
- Always check `response.ok` when handling HTTP errors.

---

> **Next Chapter:** *HTTP Fundamentals — Understanding Requests, Responses, Headers, and Status Codes*
