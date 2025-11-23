---
title: JavaScript - The Ultimate Guide
date:
description: A deep dive into the V8 Engine, Event Loop, Closures, Prototypes, and Async patterns.
tags:
  - javascript
  - web
  - frontend
  - backend
  - nodejs
  - LLM
---

> [!abstract] The Philosophy
> JavaScript is a **high-level, single-threaded, garbage-collected, interpreted (JIT-compiled), prototype-based, multi-paradigm** language.
> Unlike C++, you don't manage memory. Unlike Python, it runs in the browser. Unlike Java, it uses Prototypes, not Classes.

## 1. The Runtime (Under the Hood) ⚙️
JavaScript does not run alone. It runs inside a **Host Environment** (Browser or Node.js). The most famous engine is Google's **V8** (written in C++).

### Memory Heap & Call Stack
1.  **Memory Heap:** Unstructured memory where objects and variables are allocated. Garbage Collection (Mark & Sweep) happens here.
2.  **Call Stack:** Where the code actually executes. JS is **Single Threaded**, meaning it has only **ONE** stack. It can only do one thing at a time.

### The Event Loop (How Async Works)
If JS is single-threaded, how does `setTimeout` or `fetch` work without freezing the UI?
**Answer:** The Event Loop.



1.  **Call Stack:** Runs synchronous code (e.g., `console.log`).
2.  **Web APIs (Browser) / C++ APIs (Node):** When you call `setTimeout`, the stack hands it off to the Web API. The timer runs *outside* the JS thread.
3.  **Callback Queue (Task Queue):** When the timer finishes, the callback is pushed here.
4.  **The Loop:** The Event Loop checks: *Is the Stack empty?* If yes, it pushes the first item from the Queue to the Stack.

> [!danger] Interview Question
> **Microtasks vs Macrotasks:**
> * **Microtasks:** Promises (`.then`), `queueMicrotask`. **High Priority**.
> * **Macrotasks:** `setTimeout`, `setInterval`. **Low Priority**.
> * The Engine drains the *entire* Microtask queue before running a single Macrotask.

---

## 2. Variables & Scope 📦

### `var` vs `let` vs `const`
| Keyword | Scope | Hoisting | Reassignable? |
| :--- | :--- | :--- | :--- |
| `var` | Function | Yes (initialized `undefined`) | Yes |
| `let` | Block `{}` | Yes (TDZ*) | Yes |
| `const` | Block `{}` | Yes (TDZ*) | No (but objects are mutable) |

* **TDZ (Temporal Dead Zone):** Variables exist but cannot be accessed until the line of code is executed.

### Hoisting
JS moves declarations to the top.
```javascript
console.log(a); // undefined (No error!)
var a = 5;

console.log(b); // ReferenceError (TDZ)
let b = 5;
````

---

## 3. Types & Coercion (The "Wat" Moments)

### Primitives (Stack)

`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.

- Passed by **Value**.
    

### Reference Types (Heap)

`Object`, `Array`, `Function`.

- Passed by **Reference** (Pointer).
    

JavaScript

```
// Reference Copying
let a = [1, 2];
let b = a; // b points to the same memory address as a
b.push(3);
console.log(a); // [1, 2, 3] - 'a' changed!
```

### Coercion (Implicit Casting)

The engine tries to be helpful, often with disastrous results.

JavaScript

```
1 + "2"   // "12"  (String concatenation)
"5" - 1   // 4     (Math subtraction)
[] + []   // ""    (Arrays to string is "", "" + "" = "")
[] == ![] // true  (Don't ask.)
```

> [!tip] Rule
> 
> Always use === (Strict Equality) which checks Type and Value. Never use ==.

---

## 4. Functions & Closures 🔒

### First-Class Citizens

Functions are just objects. You can pass them as arguments, return them, and assign properties to them.

### Closures

A closure is a function that "remembers" its outer variables even after the outer function has returned. It is like a backpack of data.

JavaScript

```
function createCounter() {
    let count = 0; // "Private" variable
    return function() {
        count++;
        return count;
    }
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
// You cannot access 'count' directly from here.
```

### The `this` Keyword

The value of `this` depends on **how** the function is called, not where it is defined.

1. **Global:** `window` (or `undefined` in strict mode).
    
2. **Object Method:** The object itself.
    
3. **Constructor:** The new instance.
    
4. **Arrow Function:** Inherits `this` from the parent scope (Lexical Scoping). **This is why we use Arrow functions in React.**
    

---

## 5. Prototypes (OOP in JS) 🧬

JavaScript does not have Classes (internally). It has Prototypes.

When you look for a property on an object, if it's not found, JS looks at the object's __proto__, then that object's __proto__, all the way to null.

JavaScript

```
// The "Old" Way
function Dog(name) {
    this.name = name;
}
Dog.prototype.bark = function() {
    console.log("Woof!");
}

// The "New" Way (Syntactic Sugar)
class Dog {
    constructor(name) {
        this.name = name;
    }
    bark() {
        console.log("Woof!");
    }
}
```

_Note: Under the hood, the `class` version compiles down to the `prototype` version._

---

## 6. Asynchronous Programming ⏳

### Phase 1: Callback Hell

Nesting functions inside functions.

JavaScript

```
getData(function(a) {
    getMore(a, function(b) {
        // ...
    });
});
```

### Phase 2: Promises (ES6)

A placeholder for a future value.

- States: `Pending` -> `Resolved` OR `Rejected`.
    

### Phase 3: Async / Await (ES8)

Makes async code look synchronous.

JavaScript

```
async function fetchData() {
    try {
        const response = await fetch('/api/user');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Failed", error);
    }
}
```

---

## 7. Modern JS Patterns (ES6+) ✨

### Destructuring

JavaScript

```
const user = { id: 1, name: "Habib", role: "Admin" };
const { name, role } = user; // Extract variables
```

### Spread Operator (...)

JavaScript

```
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4]; // [1, 2, 3, 4]
const clone = { ...user }; // Shallow copy of object
```

### Modules (ESM)

JavaScript

```
// lib.js
export const add = (a, b) => a + b;

// main.js
import { add } from './lib.js';
```

---

## 8. Security (The Hacker Angle) 🛡️

### XSS (Cross-Site Scripting)

If you trust user input, they can inject JS into your site.

- **Attack:** `<img src=x onerror=alert(1)>`
    
- **Defense:** Always escape HTML. Use `textContent` instead of `innerHTML`. Use React (which escapes by default).
    

### Prototype Pollution

A vulnerability specific to JS. If an attacker can modify `Object.prototype`, they can affect **every object** in the application.

JavaScript

```
// Malicious payload
let payload = JSON.parse('{"__proto__": {"isAdmin": true}}');
// Merging this into an object might pollute the global prototype
```

---

## Linked Notes

- [[Node-Backend-Architecture]]
    
- [[React-Deep-Dive]]
    
- [[Web-Security-Basics]]
    
- [[C-Programming]] - Comparing V8 (C++) to JS.