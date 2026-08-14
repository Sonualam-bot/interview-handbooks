
# Objects & Memory Handbook
## Chapter 1 — JavaScript Memory Model

> **Objective**
>
> Understand how JavaScript stores values in memory, how variables reference those values, and why mastering the memory model is essential for understanding objects, functions, closures, prototypes, and garbage collection.

---

# 1. Why Memory Matters

Every JavaScript program manipulates data.

When you write:

```js
let age = 26;
let user = { name: "Sonu" };
```

JavaScript must decide:

- Where values are stored
- How variables reference them
- What happens when values change
- When memory can be reclaimed

Understanding these questions is the foundation for advanced JavaScript.

---

# 2. JavaScript Memory Areas (Conceptual Model)

## Stack

Stores conceptually:

- Primitive values
- Function execution contexts
- Local variables
- References to heap objects

Characteristics:

- Fast
- Automatic cleanup
- LIFO

---

## Heap

Stores:

- Objects
- Arrays
- Functions
- Maps
- Sets
- Class instances

Characteristics:

- Dynamic memory
- Garbage collected

---

# 3. Memory Diagram

```text
        Stack

age = 26

user -----------+
                |
                v

        Heap

{
  name: "Sonu"
}
```

The variable `user` stores a reference, not the object itself.

---

# 4. Primitive Values

Primitive types:

- string
- number
- bigint
- boolean
- undefined
- symbol
- null

Example:

```js
let age = 26;
let score = age;

score = 30;
```

Changing `score` does not affect `age`.

---

# 5. Reference Values

Reference values include:

- Objects
- Arrays
- Functions
- Maps
- Sets

Example:

```js
let user = { name: "Sonu" };
let admin = user;
```

Memory:

```text
Stack

user -----+
          |
admin ----+
          |
          v

Heap

{
  name: "Sonu"
}
```

Both variables point to the same object.

---

# 6. Assignment

Primitive assignment copies the value.

```js
let a = 10;
let b = a;
```

Reference assignment copies the reference.

```js
let a = { value: 10 };
let b = a;
```

---

# 7. Mutation vs Reassignment

Mutation:

```js
user.name = "Rahul";
```

Reassignment:

```js
user = {
  name: "Amit"
};
```

Mutation changes object contents.

Reassignment changes what the variable points to.

---

# 8. Mental Model

Primitive:

```text
age -> 26
```

Reference:

```text
user

↓

Reference

↓

Heap Object
```

---

# 9. Common Misconceptions

- Variables do not store objects.
- Variables store references to objects.
- Objects are not copied during assignment.
- References are copied.

---

# 10. Interview Questions

1. Why are objects called reference types?
2. What is stored inside an object variable?
3. Why does changing one object affect another variable?
4. Explain Stack vs Heap.
5. Difference between mutation and reassignment.

---

# Chapter Summary

- Stack and Heap are conceptual memory models.
- Primitive values are copied by value.
- Objects live in the Heap.
- Variables hold references to objects.
- Assignment copies references.
- Mutation and reassignment are different operations.

---

## Next Chapter

**Chapter 2 — Primitive vs Reference Types**
