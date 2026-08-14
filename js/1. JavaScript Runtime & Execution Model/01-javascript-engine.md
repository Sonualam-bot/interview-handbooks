# Chapter 1 — The JavaScript Engine

> *Don't start by asking what JavaScript is. Start by asking what must happen before JavaScript can execute even a single line.*

---

# A Mystery

Imagine you've just written your very first JavaScript program.

```js
console.log("Hello, World!");
```

You click **Run**.

The output appears instantly.

But before that message appeared, something invisible happened.

The JavaScript Engine had already begun working.

Who reads your code?

Who understands it?

Who decides what each statement means?

The answer is the JavaScript Engine.

---

# Why This Chapter Exists

Your code never executes itself.

A JavaScript file is only text.

Someone has to read it, understand it, prepare it, and finally execute it.

That someone is the JavaScript Engine.

---

# Think Like the Engine

Imagine you are the engine.

A developer gives you:

```js
let name = "Sonu";

function greet() {
  console.log(name);
}

greet();
```

Can you immediately execute it?

No.

First you must prepare.

You need to know where variables will live, where functions belong, how function calls work, and how execution will be tracked.

Only then can execution begin.

---

# What Is the JavaScript Engine?

The JavaScript Engine is software responsible for converting JavaScript source code into executable instructions.

Examples:

| Environment | Engine |
| --- | --- |
| Chrome | V8 |
| Node.js | V8 |
| Firefox | SpiderMonkey |
| Safari | JavaScriptCore |

Although implementations differ, they all follow the ECMAScript specification.

---

# Looking Ahead

Everything else in this handbook grows from this chapter.

Next, we'll answer the question:

> **Where does the engine keep everything it needs while a program is running?**

That leads us to the Execution Context.

---

# React Connection

React components are ordinary JavaScript functions.

Before React can render anything, the JavaScript Engine must execute those functions.

Understanding the engine is the first step toward understanding React internals.

---

# Key Takeaways

- JavaScript files are only text until an engine executes them.
- The JavaScript Engine reads, prepares, and executes programs.
- Preparation happens before execution.
- The next chapter introduces the Execution Context.
