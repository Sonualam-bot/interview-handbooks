# Chapter 45 — Prototype Chain

> **JavaScript OOP Handbook**

---

# What You'll Learn

- What the Prototype Chain is
- How Property Lookup Works
- `Object.prototype`
- The End of the Chain
- Method Inheritance
- Shadowing
- Interview Questions

---

# Introduction

In the previous chapter, we learned that every object has an internal
`[[Prototype]]` (accessible through the legacy `__proto__` accessor).

The **prototype chain** is what JavaScript follows whenever a property cannot
be found directly on an object.

Understanding this lookup mechanism is one of the most common JavaScript
interview topics.

---

# Building a Prototype Chain

```js
function Animal() {}

Animal.prototype.eat = function () {
  console.log("Eating...");
};

function Dog(name) {
  this.name = name;
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function () {
  console.log("Woof!");
};

const dog = new Dog("Bruno");
```

Conceptually:

```text
dog
 │
 ▼
Dog.prototype
 │
 ▼
Animal.prototype
 │
 ▼
Object.prototype
 │
 ▼
null
```

---

# Property Lookup

When JavaScript evaluates:

```js
dog.bark();
```

It searches in this order:

1. `dog`
2. `Dog.prototype`
3. `Animal.prototype`
4. `Object.prototype`
5. `null`

The search stops as soon as the property is found.

---

# Inherited Methods

```js
dog.eat();
```

Even though `eat()` does not exist on `dog` or `Dog.prototype`,
JavaScript finds it on `Animal.prototype`.

---

# Object.prototype

Almost every ordinary object ultimately inherits from
`Object.prototype`.

```js
const user = {
  name: "Sonu"
};

console.log(user.toString());
```

`toString()` works because it exists on `Object.prototype`.

---

# End of the Chain

```js
console.log(
  Object.getPrototypeOf(Object.prototype)
);
```

Output:

```text
null
```

`Object.prototype` is the final object in the prototype chain.

---

# Property Shadowing

If an object defines a property that also exists on its prototype,
the object's own property wins.

```js
const animal = {
  sound: "Generic"
};

const dog = Object.create(animal);

dog.sound = "Woof";

console.log(dog.sound);
```

Output:

```text
Woof
```

The inherited property is shadowed.

---

# Checking Ownership

Use `hasOwnProperty()` to determine whether a property belongs
directly to the object.

```js
console.log(dog.hasOwnProperty("sound"));
```

Output:

```text
true
```

Inherited properties return `false`.

---

# Visualizing Lookup

```text
dog.run()

│

├── Exists on dog?
│      ❌
│
├── Exists on Dog.prototype?
│      ❌
│
├── Exists on Animal.prototype?
│      ✅
│
└── Execute Method
```

---

# Common Mistakes

### Confusing Inheritance with Copying

Methods are **not copied** into each object.

They are shared through the prototype chain.

---

### Assuming Every Object Has Every Property

Objects inherit only what exists along their prototype chain.

---

# Interview Questions

### What is the prototype chain?

A linked sequence of prototype objects that JavaScript searches during property lookup.

---

### When does property lookup stop?

When the property is found or the chain reaches `null`.

---

### Why does `toString()` work on almost every object?

Because it is inherited from `Object.prototype`.

---

# Key Takeaways

- Objects inherit behavior through the prototype chain.
- Property lookup proceeds from the object to its prototypes.
- `Object.prototype` is the root of most prototype chains.
- Own properties shadow inherited properties.
- Prototype chains enable efficient code reuse.

---

# Next Chapter

**Chapter 46 — Classes**
