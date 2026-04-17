# JavaScript Essentials for .NET Full‑Stack Developers

*Core JavaScript Concepts, ES6+, Async Patterns, and DOM Manipulation*

This document covers JavaScript fundamentals and modern features essential for building robust front‑ends that interact with ASP.NET Core back‑ends. It bridges the gap between C# and JavaScript paradigms.

---

## Table of Contents

1. [JavaScript vs C# – Key Differences](#javascript-vs-c--key-differences)
2. [Variables and Data Types](#variables-and-data-types)
3. [Functions and Scope](#functions-and-scope)
4. [Objects and Prototypes](#objects-and-prototypes)
5. [Arrays and Iteration Methods](#arrays-and-iteration-methods)
6. [ES6+ Features You Must Know](#es6-features-you-must-know)
7. [Asynchronous JavaScript](#asynchronous-javascript)
8. [DOM Manipulation and Events](#dom-manipulation-and-events)
9. [Error Handling and Debugging](#error-handling-and-debugging)
10. [Modules and Module Bundlers](#modules-and-module-bundlers)
11. [JavaScript in the Browser vs Node.js](#javascript-in-the-browser-vs-nodejs)
12. [TypeScript – A Quick Introduction](#typescript--a-quick-introduction)

---

## JavaScript vs C# – Key Differences

| Feature               | C# (.NET)                                          | JavaScript                                            |
|-----------------------|----------------------------------------------------|-------------------------------------------------------|
| **Type System**       | Statically typed                                   | Dynamically typed                                     |
| **Compilation**       | Compiled to IL / machine code                      | Interpreted or JIT‑compiled                           |
| **OOP Model**         | Classical inheritance                              | Prototypal inheritance                                |
| **Concurrency**       | Multithreading (`Task`, `async/await`)             | Single‑threaded event loop with async callbacks       |
| **`this` binding**    | Always refers to current instance                  | Determined by **call site** (can change)              |
| **Null/Undefined**    | `null`                                             | `null` and `undefined` (two distinct empty values)    |
| **Equality**          | `==` value equality, `ReferenceEquals()`           | `==` loose equality, `===` strict equality            |
| **Scope**             | Block‑scoped                                       | Function‑scoped (`var`) or block‑scoped (`let`/`const`) |

---

## Variables and Data Types

### 1. Declaring Variables: `var`, `let`, `const`

| Keyword   | Scope         | Reassignable | Hoisted           | Temporal Dead Zone |
|-----------|---------------|--------------|-------------------|---------------------|
| `var`     | Function      | Yes          | Yes (undefined)   | No                  |
| `let`     | Block         | Yes          | Yes (not init)    | Yes                 |
| `const`   | Block         | No           | Yes (not init)    | Yes                 |

```javascript
// var - function scoped
function demoVar() {
    if (true) {
        var x = 10;
    }
    console.log(x); // 10 (accessible outside block)
}

// let - block scoped
function demoLet() {
    if (true) {
        let y = 20;
    }
    console.log(y); // ReferenceError: y is not defined
}

// const - block scoped, cannot reassign
const PI = 3.14159;
PI = 3.14; // TypeError: Assignment to constant variable

const person = { name: 'John' };
person.name = 'Jane'; // OK - object properties can change
```

---

### 2. Primitive vs Reference Types

| Primitive Types                          | Reference Types               |
|------------------------------------------|-------------------------------|
| `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint` | `Object`, `Array`, `Function`, `Date`, etc. |

```javascript
// Primitives - copied by value
let a = 5;
let b = a;
b = 10;
console.log(a); // 5 (unchanged)

// Reference - copied by reference
let obj1 = { name: 'Alice' };
let obj2 = obj1;
obj2.name = 'Bob';
console.log(obj1.name); // 'Bob' (changed)
```

---

### 3. Type Coercion and Truthy/Falsy

**Falsy values:** `false`, `0`, `''`, `null`, `undefined`, `NaN`.  
Everything else is truthy.

```javascript
console.log(5 == '5');   // true (loose equality, coerces types)
console.log(5 === '5');  // false (strict equality, no coercion)

// Common pitfalls
console.log(0 == false);     // true
console.log('' == false);    // true
console.log(null == undefined); // true
console.log(null === undefined); // false
```

**Use `===` and `!==` to avoid surprises.**

---

## Functions and Scope

### 4. Function Declarations vs Expressions vs Arrow Functions

```javascript
// Function Declaration (hoisted)
function add(a, b) {
    return a + b;
}

// Function Expression (not hoisted)
const subtract = function(a, b) {
    return a - b;
};

// Arrow Function (lexical this, no arguments object)
const multiply = (a, b) => a * b;

// Arrow with single parameter
const square = x => x * x;

// Arrow returning object (wrap in parentheses)
const createPerson = (name, age) => ({ name, age });
```

---

### 5. The `this` Keyword in Different Contexts

| Context                       | `this` Value                                                                 |
|-------------------------------|------------------------------------------------------------------------------|
| Global scope                  | `window` (browser) / `global` (Node) / `undefined` (strict mode)             |
| Object method                 | The object itself                                                            |
| Constructor (`new`)           | The newly created instance                                                   |
| Event handler                 | The element that received the event                                          |
| Arrow function                | Lexical – inherits from outer scope                                          |
| `call` / `apply` / `bind`     | Explicitly set value                                                         |

```javascript
const user = {
    name: 'Alice',
    greet() {
        console.log(`Hello, ${this.name}`);
    },
    greetArrow: () => {
        console.log(`Hello, ${this.name}`); // 'this' is from outer scope
    }
};
user.greet();      // Hello, Alice
user.greetArrow(); // Hello, undefined (outer this is window/global)
```

---

### 6. Closures

A **closure** is a function that retains access to its outer scope even after the outer function has returned.

```javascript
function createCounter() {
    let count = 0;
    return {
        increment: () => ++count,
        decrement: () => --count,
        getCount: () => count
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2
// 'count' variable is private, only accessible via returned methods
```

**Common use:** Module pattern, factory functions, event handlers with preserved state.

---

## Objects and Prototypes

### 7. Object Literals and Property Shorthand

```javascript
const name = 'Alice';
const age = 30;

// ES5
const person1 = { name: name, age: age };

// ES6 shorthand
const person2 = { name, age };

// Computed property names
const prop = 'status';
const obj = {
    [prop]: 'active',
    [`get${prop}`]() { return this[prop]; }
};
```

---

### 8. Prototypal Inheritance

Every object has an internal link to another object called its **prototype**. When accessing a property, JavaScript looks up the prototype chain.

```javascript
const parent = {
    greet() { console.log('Hello from parent'); }
};

const child = Object.create(parent);
child.sayHi = function() { console.log('Hi from child'); };

child.greet(); // "Hello from parent" (inherited)
console.log(child.hasOwnProperty('greet')); // false
console.log(child.hasOwnProperty('sayHi')); // true
```

**Constructor functions and `prototype` property:**
```javascript
function Person(name) {
    this.name = name;
}
Person.prototype.greet = function() {
    console.log(`Hello, I'm ${this.name}`);
};

const alice = new Person('Alice');
alice.greet(); // "Hello, I'm Alice"
```

---

### 9. Classes (Syntactic Sugar over Prototypes)

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    speak() {
        console.log(`${this.name} makes a sound.`);
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);
        this.breed = breed;
    }
    speak() {
        console.log(`${this.name} barks.`);
    }
}

const rex = new Dog('Rex', 'German Shepherd');
rex.speak(); // "Rex barks."
```

---

## Arrays and Iteration Methods

### 10. Common Array Methods (Functional Style)

| Method       | Purpose                                               | Returns                     | Mutates Original? |
|--------------|-------------------------------------------------------|-----------------------------|-------------------|
| `map`        | Transform each element                                | New array                   | No                |
| `filter`     | Select elements that pass test                        | New array                   | No                |
| `reduce`     | Accumulate to single value                            | Single value                | No                |
| `forEach`    | Execute function for each element                     | `undefined`                 | No (but can)      |
| `find`       | Get first element that passes test                    | Element or `undefined`      | No                |
| `some`       | Check if any element passes test                      | `boolean`                   | No                |
| `every`      | Check if all elements pass test                       | `boolean`                   | No                |
| `includes`   | Check if array contains value                         | `boolean`                   | No                |

```javascript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(n => n * 2);           // [2, 4, 6, 8, 10]
const evens = numbers.filter(n => n % 2 === 0);    // [2, 4]
const sum = numbers.reduce((acc, n) => acc + n, 0); // 15
const firstEven = numbers.find(n => n % 2 === 0);  // 2
const hasNegative = numbers.some(n => n < 0);      // false
const allPositive = numbers.every(n => n > 0);     // true
```

---

### 11. Spread and Rest Operators (`...`)

```javascript
// Spread - expand array/object
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1,2,3,4,5,6]

const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, ...obj1 }; // { c:3, a:1, b:2 }

// Rest - gather remaining arguments
function sum(...numbers) {
    return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4); // 10

// Rest in destructuring
const [first, second, ...rest] = [10, 20, 30, 40, 50];
console.log(rest); // [30, 40, 50]
```

---

## ES6+ Features You Must Know

### 12. Destructuring Assignment

```javascript
// Array destructuring
const colors = ['red', 'green', 'blue'];
const [primary, secondary] = colors;
console.log(primary); // 'red'

// Object destructuring
const user = { name: 'Alice', age: 30, city: 'NY' };
const { name, age } = user;
console.log(name); // 'Alice'

// Renaming and defaults
const { name: userName, country = 'USA' } = user;
console.log(userName); // 'Alice'
console.log(country);  // 'USA'

// Nested destructuring
const response = { data: { user: { id: 1 } } };
const { data: { user: { id } } } = response;
```

---

### 13. Template Literals

```javascript
const name = 'Alice';
const age = 30;

// String interpolation
console.log(`Hello, my name is ${name} and I am ${age} years old.`);

// Multi‑line strings
const html = `
    <div>
        <h1>${name}</h1>
        <p>Age: ${age}</p>
    </div>
`;

// Tagged templates (advanced)
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => 
        `${result}${str}<strong>${values[i] || ''}</strong>`, '');
}
console.log(highlight`Hello ${name}, age ${age}`);
```

---

### 14. Optional Chaining (`?.`) and Nullish Coalescing (`??`)

```javascript
const user = { profile: { name: 'Alice' } };

// Optional chaining - safe access
console.log(user.profile?.name);        // 'Alice'
console.log(user.settings?.theme);      // undefined (no error)

// Nullish coalescing - fallback for null/undefined only
const theme = user.settings?.theme ?? 'light';
console.log(theme); // 'light'

// Difference from || (logical OR)
const count = 0;
console.log(count || 10);   // 10 (0 is falsy)
console.log(count ?? 10);   // 0 (0 is not null/undefined)
```

---

## Asynchronous JavaScript

### 15. Callbacks and the Event Loop

JavaScript is single‑threaded. Long operations (network, timers) use callbacks queued by the **event loop**.

```javascript
console.log('Start');

setTimeout(() => {
    console.log('Timeout');
}, 0);

Promise.resolve().then(() => console.log('Promise'));

console.log('End');

// Output:
// Start
// End
// Promise
// Timeout
```

**Microtasks** (Promises) run before **macrotasks** (setTimeout).

---

### 16. Promises

A **Promise** represents an asynchronous operation that may complete in the future.

```javascript
const fetchData = () => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = true;
            if (success) {
                resolve({ id: 1, name: 'Product' });
            } else {
                reject('Error fetching data');
            }
        }, 1000);
    });
};

// Using .then/.catch
fetchData()
    .then(data => console.log(data))
    .catch(err => console.error(err))
    .finally(() => console.log('Done'));

// Promise chaining
fetchData()
    .then(data => fetchDetails(data.id))
    .then(details => console.log(details))
    .catch(err => console.error(err));
```

**Static methods:**
```javascript
Promise.all([promise1, promise2])     // Waits for all, rejects if any fails
Promise.allSettled([p1, p2])          // Waits for all, never rejects
Promise.race([p1, p2])                // Resolves/rejects with first settled
Promise.any([p1, p2])                 // Resolves with first fulfilled
```

---

### 17. Async/Await

Syntactic sugar over Promises for cleaner code.

```javascript
async function getProduct() {
    try {
        const response = await fetch('https://api.example.com/products/1');
        if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
        const data = await response.json();
        console.log(data);
        return data;
    } catch (error) {
        console.error('Failed:', error);
        throw error; // re-throw if needed
    }
}

// Using with Promise.all
async function loadAll() {
    const [users, products] = await Promise.all([
        fetch('/api/users').then(r => r.json()),
        fetch('/api/products').then(r => r.json())
    ]);
    return { users, products };
}
```

---

### 18. Fetch API and Working with REST APIs

```javascript
// GET
const response = await fetch('/api/products');
const products = await response.json();

// POST
const newProduct = { name: 'Laptop', price: 999.99 };
const postResponse = await fetch('/api/products', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newProduct)
});
const created = await postResponse.json();

// PUT
await fetch(`/api/products/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(updatedProduct)
});

// DELETE
await fetch(`/api/products/${id}`, { method: 'DELETE' });

// Handling errors
try {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
} catch (err) {
    // Network error or thrown error
}
```

---

## DOM Manipulation and Events

### 19. Selecting and Modifying Elements

```javascript
// Selectors
const element = document.getElementById('myId');
const elements = document.getElementsByClassName('myClass');
const firstPara = document.querySelector('p');
const allParas = document.querySelectorAll('.myClass');

// Modifying content
element.textContent = 'New text';
element.innerHTML = '<strong>Bold text</strong>';
element.setAttribute('data-id', '123');

// Classes
element.classList.add('active');
element.classList.remove('hidden');
element.classList.toggle('dark');
element.classList.contains('active'); // true/false

// Styles
element.style.backgroundColor = 'red';
element.style.display = 'none';

// Creating and appending elements
const newDiv = document.createElement('div');
newDiv.textContent = 'Hello';
document.body.appendChild(newDiv);

// Removing
element.remove();
```

---

### 20. Event Handling

```javascript
// Traditional
button.onclick = function() { console.log('Clicked'); };

// addEventListener (preferred)
button.addEventListener('click', (event) => {
    console.log('Clicked', event.target);
    event.preventDefault(); // prevent default behavior (e.g., form submit)
    event.stopPropagation(); // stop event bubbling
});

// Event delegation (handling events on dynamic elements)
document.querySelector('.list').addEventListener('click', (e) => {
    if (e.target.matches('.item')) {
        console.log('Item clicked:', e.target.textContent);
    }
});

// Common events
input.addEventListener('input', (e) => console.log(e.target.value));
form.addEventListener('submit', handleSubmit);
window.addEventListener('resize', handleResize);
document.addEventListener('DOMContentLoaded', init);
```

---

### 21. Working with Forms

```javascript
const form = document.querySelector('form');

form.addEventListener('submit', async (e) => {
    e.preventDefault();
    
    // Get form data
    const formData = new FormData(form);
    const data = Object.fromEntries(formData.entries());
    
    // Or access individual inputs
    const name = document.querySelector('#name').value;
    
    // Validate
    if (!name.trim()) {
        showError('Name is required');
        return;
    }
    
    // Submit to API
    try {
        const response = await fetch('/api/submit', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        });
        if (response.ok) {
            form.reset();
            showSuccess('Saved!');
        }
    } catch (error) {
        showError(error.message);
    }
});
```

---

## Error Handling and Debugging

### 22. Try/Catch and Custom Errors

```javascript
try {
    const result = riskyOperation();
} catch (error) {
    console.error('Caught:', error.message);
    // Optionally re-throw
    throw new Error(`Operation failed: ${error.message}`);
} finally {
    cleanup();
}

// Custom error types
class ValidationError extends Error {
    constructor(message, field) {
        super(message);
        this.name = 'ValidationError';
        this.field = field;
    }
}
throw new ValidationError('Invalid email', 'email');
```

---

### 23. Debugging Tools

- **Browser DevTools:** Breakpoints, `debugger` statement, Console tab.
- **`console` methods:** `log`, `warn`, `error`, `table`, `group`, `time`.

```javascript
console.table(products);
console.group('User Details');
console.log('Name:', name);
console.groupEnd();
console.time('fetch');
await fetchData();
console.timeEnd('fetch');
debugger; // Execution stops here when DevTools open
```

---

## Modules and Module Bundlers

### 24. ES Modules (`import` / `export`)

```javascript
// utils.js (named exports)
export const formatDate = (date) => date.toLocaleDateString();
export const API_URL = 'https://api.example.com';

// Default export
export default function log(message) { console.log(message); }

// Importing
import log, { formatDate, API_URL } from './utils.js';
import * as utils from './utils.js';
```

**Dynamic imports (lazy loading):**
```javascript
const module = await import('./heavyModule.js');
module.doSomething();
```

---

### 25. Module Bundlers (Vite / Webpack)

Modern React projects use **Vite** or **Create React App** (Webpack). They bundle JavaScript files, handle assets, and provide hot reloading.

**Basic Vite configuration (`vite.config.js`):**
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    server: {
        port: 3000,
        proxy: {
            '/api': 'https://localhost:5001'
        }
    }
});
```

---

## JavaScript in the Browser vs Node.js

| Environment | Global Object   | Modules               | DOM    | File System |
|-------------|-----------------|-----------------------|--------|-------------|
| **Browser** | `window`        | ES Modules (modern)   | Yes    | No (limited)|
| **Node.js** | `global`        | CommonJS / ES Modules | No     | Yes (`fs`)  |

**In Node.js:**
```javascript
const fs = require('fs'); // CommonJS
import fs from 'fs';      // ES Modules (with "type":"module")
```

---

## TypeScript – A Quick Introduction

TypeScript adds static typing to JavaScript, making it more familiar to C# developers.

```typescript
interface Product {
    id: number;
    name: string;
    price: number;
    category?: string; // optional
}

async function fetchProduct(id: number): Promise<Product> {
    const response = await fetch(`/api/products/${id}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json() as Promise<Product>;
}

// Usage
const product = await fetchProduct(1);
console.log(product.name.toUpperCase());
```

**Basic types:** `string`, `number`, `boolean`, `Array<T>`, `[string, number]` (tuple), `enum`, `any`, `unknown`, `void`, `never`.

**Benefits for .NET developers:**
- Compile‑time type checking.
- Better IDE support (IntelliSense).
- Easier refactoring.
- Interfaces and generics similar to C#.

---

*This JavaScript guide covers the essentials a .NET full‑stack developer needs to be productive with front‑end development. Combine this knowledge with React or Angular to build complete web applications.*
