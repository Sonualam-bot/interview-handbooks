# Chapter 31 — JavaScript Memory Model

> **Objects & Memory Handbook**

---

# What You'll Learn

- What memory is
- How JavaScript stores data
- Stack vs Heap
- Execution Context and Memory
- Variable Allocation
- Memory Lifecycle
- Why Understanding Memory Matters

---

# Why Learn the Memory Model?

When people start learning JavaScript, they write:

```js
let age = 25;
```

Internally, the JavaScript engine allocates memory, associates the variable with that memory, decides where the value should live, and later reclaims it when it is no longer needed.

Understanding this chapter is the foundation for:

- Primitive vs Reference Types
- Objects
- Object Copying
- Garbage Collection

---

# What is Memory?

Memory is temporary storage used while a program executes.

Whenever you create variables, functions, objects or arrays, the JavaScript engine allocates memory.

---

# JavaScript Memory (Conceptual Model)

```text
Memory
├── Stack
└── Heap
```

This is the conceptual model commonly used in interviews.

---

# Stack Memory

The stack conceptually stores:

- Execution Contexts
- Local Variables
- Primitive Values
- Return Addresses

Characteristics:

- Fast
- Small
- LIFO (Last In, First Out)
- Automatically managed

---

# Heap Memory

The heap typically stores:

- Objects
- Arrays
- Functions

Characteristics:

- Large
- Dynamic
- Managed by the Garbage Collector

---

# Execution Context and Memory

Whenever a function executes, JavaScript creates a new execution context.

```js
function greet(name) {
  const message = `Hello ${name}`;
  return message;
}
```

Memory is allocated for:

- Parameters
- Local variables
- Function declarations

When execution completes, the execution context is removed from the call stack.

---

# Memory Lifecycle

Every value goes through three stages:

1. Allocation
2. Usage
3. Release

Memory that is no longer reachable becomes eligible for garbage collection.

---

# Why This Matters

The JavaScript memory model explains:

- Why objects behave differently from primitive values.
- Why copying objects can be confusing.
- Why garbage collection exists.
- Why memory leaks happen.

These concepts are explored in the following chapters.

---

# Common Interview Questions

### Does JavaScript use Stack and Heap?

Conceptually, yes. This is the standard interview mental model.

### Does JavaScript manage memory automatically?

Yes. The engine allocates and reclaims memory automatically.

### What happens when a function finishes?

Its execution context is removed from the call stack, and unreachable memory becomes eligible for garbage collection.

---

# Key Takeaways

- Memory stores program data during execution.
- JavaScript conceptually uses Stack and Heap memory.
- Every function call creates an execution context.
- Variables are labels used to access values in memory.
- This chapter lays the foundation for the remaining Objects & Memory handbook.

---

# Next Chapter

**Chapter 32 — Primitive vs Reference Types**
