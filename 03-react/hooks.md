Understanding the **React Hooks Lifecycle** is a very common interview topic for React developers with **5+ years of experience**. While functional components don't have traditional lifecycle methods like class components, Hooks (`useEffect`, `useLayoutEffect`) let you perform the same lifecycle operations.

---

# React Hooks Lifecycle

Think of the lifecycle in **three phases**:

```text
1. Mounting (Component Created)

↓

2. Updating (Props/State Changed)

↓

3. Unmounting (Component Destroyed)
```

---

# 1. Mounting Phase

The component is created and added to the DOM.

```jsx
function App() {
    console.log("Render");

    useLayoutEffect(() => {
        console.log("useLayoutEffect");
    }, []);

    useEffect(() => {
        console.log("useEffect");
    }, []);

    return <h1>Hello React</h1>;
}
```

### Execution Order

```text
Component Function Executes (Render)

↓

Virtual DOM Created

↓

Real DOM Updated

↓

useLayoutEffect

↓

Browser Paint

↓

useEffect
```

Console Output

```text
Render

useLayoutEffect

useEffect
```

---

# 2. Updating Phase

Whenever **state** or **props** change.

```jsx
function Counter() {

    const [count, setCount] = useState(0);

    console.log("Render");

    useEffect(() => {
        console.log("Effect");
    });

    return (
        <>
            <h1>{count}</h1>

            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
        </>
    );
}
```

Click button

```text
setState()

↓

Component Re-renders

↓

DOM Updated

↓

Cleanup Previous Effect

↓

Run New Effect
```

Console

```text
Render

Cleanup

Effect
```

React always cleans up the previous effect before running the new one (if a cleanup function is returned).

---

# 3. Unmounting Phase

When the component is removed from the DOM.

```jsx
useEffect(() => {

    console.log("Mounted");

    return () => {
        console.log("Cleanup");
    };

}, []);
```

Output

```text
Mounted

↓

Component Removed

↓

Cleanup
```

This cleanup is where you should release resources.

Examples:

* Remove event listeners
* Clear intervals/timeouts
* Close WebSocket connections
* Abort API requests

---

# Effect Dependencies

## 1. No Dependency Array

```jsx
useEffect(() => {

    console.log("Runs");

});
```

Runs

```text
Mount

↓

Every Re-render
```

Equivalent to

```text
componentDidMount +

componentDidUpdate
```

---

## 2. Empty Dependency Array

```jsx
useEffect(() => {

    console.log("Runs Once");

}, []);
```

Runs

```text
Only Once

↓

On Mount
```

Equivalent to

```text
componentDidMount
```

---

## 3. Dependency Array

```jsx
useEffect(() => {

    console.log("Count Changed");

}, [count]);
```

Runs

```text
Mount

↓

Whenever count changes
```

---

# Complete Lifecycle Flow

```text
Initial Render

↓

Component Function Executes

↓

Virtual DOM

↓

Real DOM Updated

↓

useLayoutEffect

↓

Browser Paint

↓

useEffect

========================

State Changes

↓

Render Again

↓

DOM Updated

↓

Cleanup Previous Effect

↓

useLayoutEffect

↓

Browser Paint

↓

useEffect

========================

Component Removed

↓

Cleanup Function Executes
```

---

# Class Lifecycle vs Hooks

| Class Component         | Functional Component                                   |
| ----------------------- | ------------------------------------------------------ |
| constructor             | `useState()` initialization                            |
| componentDidMount       | `useEffect(() => {}, [])`                              |
| componentDidUpdate      | `useEffect(() => {})` or `useEffect(() => {}, [deps])` |
| componentWillUnmount    | Cleanup function returned from `useEffect`             |
| shouldComponentUpdate   | `React.memo()`                                         |
| getSnapshotBeforeUpdate | `useLayoutEffect()` (for layout-related work)          |

---

# Real-World Example

Imagine a **Chat Application**.

```jsx
function ChatRoom({ roomId }) {

    useEffect(() => {

        console.log("Connect to server");

        return () => {
            console.log("Disconnect");
        };

    }, [roomId]);

    return <div>Chat Room</div>;
}
```

### Mount

```text
Open Chat

↓

Connect to WebSocket
```

### Update

```text
Switch Room

↓

Disconnect Old Room

↓

Connect New Room
```

### Unmount

```text
Leave Chat

↓

Disconnect WebSocket
```

This prevents memory leaks and stale subscriptions.

---

# Interview Answer (5+ Years)

> Functional components don't have explicit lifecycle methods like class components. Instead, React Hooks manage lifecycle behavior. On **mount**, the component renders, `useLayoutEffect` runs after the DOM update but before the browser paints, and `useEffect` runs after the paint. On **updates**, React re-renders the component, cleans up the previous effect if necessary, and then runs the new effect. On **unmount**, React executes the cleanup function returned from `useEffect` or `useLayoutEffect`, making it the right place to remove event listeners, clear timers, close WebSocket connections, or cancel pending requests.
>

This is a very common React interview question, especially for **5+ years of experience**. Interviewers usually expect you to explain **when each hook runs, why React provides both, and where to use them**.

---

# Short Answer

| useEffect                                    | useLayoutEffect                                       |
| -------------------------------------------- | ----------------------------------------------------- |
| Runs **after the browser paints the screen** | Runs **before the browser paints the screen**         |
| Does not block painting                      | Blocks painting until it finishes                     |
| Used for API calls, subscriptions, timers    | Used for DOM measurements and synchronous DOM updates |
| Better for performance                       | Can hurt performance if overused                      |

---

# React Rendering Lifecycle

Let's understand what happens internally.

```text
State Changes

↓

React renders Virtual DOM

↓

React updates Real DOM

↓

useLayoutEffect Runs

↓

Browser Paints UI

↓

useEffect Runs
```

The key difference is **when the effect executes relative to the browser painting the updated UI**.

---

# useEffect

`useEffect` runs **after** React has updated the DOM **and after the browser has painted the screen**.

```jsx
useEffect(() => {
    console.log("useEffect");
}, []);
```

Sequence:

```text
Render

↓

Update DOM

↓

Browser Paint

↓

useEffect
```

Since it doesn't block rendering, it provides a smoother user experience.

---

## Real-world Example: Fetching Data

```jsx
function Users() {
    const [users, setUsers] = useState([]);

    useEffect(() => {
        fetch("/api/users")
            .then(res => res.json())
            .then(setUsers);
    }, []);

    return <UserList users={users} />;
}
```

The user sees the page immediately, while the data loads in the background.

This is the most common use case.

---

## Other Common Uses

* API calls
* Event listeners
* WebSocket connections
* Timers (`setInterval`, `setTimeout`)
* Analytics
* Logging

---

# useLayoutEffect

`useLayoutEffect` runs **after the DOM has been updated but before the browser paints the screen**.

```jsx
useLayoutEffect(() => {
    console.log("useLayoutEffect");
}, []);
```

Sequence:

```text
Render

↓

Update DOM

↓

useLayoutEffect

↓

Browser Paint
```

Since it runs before painting, it can safely read layout information and make synchronous DOM changes without the user seeing intermediate states.

---

# Real-world Example: Measuring an Element

Suppose you need to know the height of a `<div>`.

```jsx
function Box() {
    const ref = useRef();

    useLayoutEffect(() => {
        console.log(ref.current.offsetHeight);
    }, []);

    return <div ref={ref}>Hello</div>;
}
```

Here, the DOM exists, but the browser hasn't painted yet, so the measurement is accurate and there is no visual flicker.

---

# Another Example: Tooltip Positioning

Imagine a tooltip.

Without `useLayoutEffect`:

```text
Render Tooltip

↓

Browser Paint

↓

Measure Position

↓

Move Tooltip

↓

Second Paint
```

The user briefly sees the tooltip in the wrong position (a flicker).

With `useLayoutEffect`:

```text
Render Tooltip

↓

Measure Position

↓

Move Tooltip

↓

Browser Paint
```

The tooltip appears in the correct position immediately.

---

# Example Comparison

### useEffect

```jsx
useEffect(() => {
    document.title = "Dashboard";
}, []);
```

This update isn't visually critical, so it's fine after the paint.

---

### useLayoutEffect

```jsx
useLayoutEffect(() => {
    const width = divRef.current.offsetWidth;

    if (width < 500) {
        divRef.current.style.fontSize = "14px";
    }
}, []);
```

The size is measured and adjusted before the user sees it.

---

# Why Not Always Use useLayoutEffect?

Because it **blocks the browser from painting**.

Imagine this:

```jsx
useLayoutEffect(() => {
    // Heavy calculation
    for (let i = 0; i < 1000000000; i++) {}
}, []);
```

The browser cannot paint until this work finishes, making the UI feel frozen.

`useEffect` avoids this problem by allowing the browser to paint first.

---

# Common Interview Question

### Which is faster?

Neither hook is inherently faster.

* `useEffect` is **better for overall performance** because it doesn't block painting.
* `useLayoutEffect` executes **earlier**, but can slow down rendering if it does heavy work.

---

# When Should You Use Which?

### Use `useEffect` (90–95% of cases)

* API calls
* Timers
* Event listeners
* Logging
* Analytics
* WebSocket connections
* Updating the document title

### Use `useLayoutEffect` (only when necessary)

* Measuring DOM elements (`offsetWidth`, `offsetHeight`, `getBoundingClientRect`)
* Calculating layouts
* Scroll restoration
* Positioning tooltips, popovers, or modals
* Preventing UI flicker caused by immediate DOM adjustments

---

# Interview Answer (5+ Years)

> `useEffect` runs after React updates the DOM and the browser has painted the screen, making it ideal for side effects like API calls, subscriptions, timers, and event listeners because it doesn't block rendering. `useLayoutEffect` runs synchronously after the DOM is updated but before the browser paints. It's used when you need to read layout information or synchronously modify the DOM, such as measuring element sizes or positioning tooltips, to avoid visible flicker. Since `useLayoutEffect` blocks painting, it should be reserved for layout-related work, while `useEffect` should be the default choice for most side effects.


In modern React, **`React.memo()`** is the functional component equivalent of `PureComponent`. It is commonly used to optimize performance by preventing unnecessary re-renders.

Let's look at a few real-world scenarios.

---

# Use Case 1: Product List (E-commerce)

Suppose you have an online shopping application.

```text
App
 ├── Search Box
 ├── Cart Count
 └── ProductList
       ├── ProductCard
       ├── ProductCard
       └── ProductCard
```

Whenever the cart count changes, the parent (`App`) re-renders.

Without `React.memo()`:

```jsx
function ProductCard({ product }) {
    console.log(product.name, "Rendered");

    return (
        <div>
            {product.name} - ₹{product.price}
        </div>
    );
}
```

Even though the products haven't changed, every `ProductCard` renders again.

Imagine there are **1000 products**.

```
Cart Count Changed

↓

App Rendered

↓

1000 ProductCards Render Again ❌
```

---

Using `React.memo()`:

```jsx
const ProductCard = React.memo(function ProductCard({ product }) {
    console.log(product.name, "Rendered");

    return (
        <div>
            {product.name} - ₹{product.price}
        </div>
    );
});
```

Now,

```
Cart Count Changed

↓

App Rendered

↓

Product Props Same

↓

No ProductCard Render ✅
```

Huge performance improvement.

---

# Use Case 2: Dashboard

Imagine a dashboard.

```
Dashboard

├── Header
├── Sidebar
├── User Profile
├── Notifications
├── Sales Graph
└── Footer
```

Every 5 seconds,

```
Notifications
```

are refreshed.

Without memoization,

```
Header
Sidebar
User Profile
Sales Graph
Footer
```

also render again.

But these components never changed.

```jsx
const Header = React.memo(() => {
    console.log("Header Render");
    return <h1>Dashboard</h1>;
});
```

Now only

```
Notifications
```

re-renders.

Everything else stays untouched.

---

# Use Case 3: Chat Application

```
Chat App

├── Contact List
├── Chat Window
└── User Profile
```

When typing a message:

```
Hello...
```

The state changes every keystroke.

Without memoization

```
Contact List
User Profile
```

also render every time.

With

```jsx
const ContactList = React.memo(ContactList);
```

only

```
Chat Window
```

updates.

---

# Use Case 4: Employee Management System

Suppose your page contains

```
Employee List
Attendance Widget
Leave Balance
Company News
```

Attendance refreshes every minute.

Without `React.memo()`

```
Employee List
```

renders again although employees didn't change.

Since Employee List might contain hundreds of rows, this wastes CPU time.

---

# Use Case 5: Data Grid

Imagine a table with 10,000 rows.

```
Customers

Row1
Row2
...
Row10000
```

A search box is outside the table.

Every key press

```
a
ab
abc
abcd
```

updates the parent state.

Without memoization

```
10000 rows render again.
```

With

```jsx
const CustomerRow = React.memo(CustomerRow);
```

only rows whose props actually change will render.

This is a huge optimization.

---

# When `React.memo()` Does **Not** Work

A common mistake is passing a new object or function on every render.

```jsx
<ProductCard
    product={product}
    style={{ color: "red" }}   // New object every render
/>
```

or

```jsx
<ProductCard
    onClick={() => addToCart(product)} // New function every render
/>
```

Every render creates new references.

```
Old object != New object
```

So `React.memo()` thinks the props changed and re-renders anyway.

### Fix

Use `useMemo()` and `useCallback()`.

```jsx
const style = useMemo(() => ({ color: "red" }), []);

const handleClick = useCallback(() => {
    addToCart(product);
}, [product]);
```

Now the references remain stable, allowing `React.memo()` to skip unnecessary renders.

---

# Interview Answer (5+ Years)

> We use `React.memo()` when a child component receives the same props frequently while the parent re-renders due to unrelated state changes. Typical examples include product cards in an e-commerce app, dashboard widgets, chat contact lists, table rows, and employee lists. `React.memo()` performs a shallow comparison of props and skips rendering if they haven't changed, improving performance. However, it is effective only when prop references remain stable, so it is often used together with `useCallback()` and `useMemo()` for function and object props.

This is one of the most frequently asked React interview questions for **5+ years of experience** because it tests your understanding of **performance optimization** and **rendering behavior**.

---

# What is `useCallback`?

`useCallback` is a React Hook that **memoizes (caches) a function** so that React returns the **same function instance** between renders unless one of its dependencies changes.

### Syntax

```jsx
const memoizedFunction = useCallback(() => {
    // function logic
}, [dependencies]);
```

Without `useCallback`, a new function is created on every render.

---

# Why Do We Need `useCallback`?

In JavaScript, every time a component renders, a new function is created.

```jsx
function App() {

    const handleClick = () => {
        console.log("Clicked");
    };

    return <button onClick={handleClick}>Click</button>;
}
```

Even though the function body is the same:

```text
Render 1

handleClick ---> Memory Address A
```

```text
Render 2

handleClick ---> Memory Address B
```

```text
Render 3

handleClick ---> Memory Address C
```

Every render creates a **new function reference**.

Normally, this is not a problem.

The issue arises when you pass that function to a memoized child component.

---

# Problem Without `useCallback`

### Parent

```jsx
import React, { useState } from "react";

function App() {

    const [count, setCount] = useState(0);

    const handleClick = () => {
        console.log("Button clicked");
    };

    return (
        <>
            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>

            <Child onClick={handleClick} />
        </>
    );
}
```

### Child

```jsx
const Child = React.memo(({ onClick }) => {

    console.log("Child Rendered");

    return (
        <button onClick={onClick}>
            Child Button
        </button>
    );
});
```

### What Happens?

Click **Increment**.

The parent re-renders.

A new `handleClick` function is created.

```text
Old Function != New Function
```

`React.memo()` performs a shallow comparison.

It sees:

```text
Previous onClick !== Current onClick
```

So the child re-renders unnecessarily.

---

# Solution Using `useCallback`

```jsx
const handleClick = useCallback(() => {
    console.log("Button clicked");
}, []);
```

Now:

```text
Render 1

handleClick ---> Address A
```

```text
Render 2

handleClick ---> Address A
```

```text
Render 3

handleClick ---> Address A
```

The function reference stays the same.

Since `Child` receives the same prop reference, `React.memo()` skips re-rendering.

---

# Real-Time Use Cases

## 1. Large Data Table

Imagine an employee management system.

```text
App

↓

EmployeeTable

↓

EmployeeRow (1000 rows)
```

Each row receives:

```jsx
<EmployeeRow
    employee={emp}
    onDelete={handleDelete}
/>
```

Without `useCallback`:

Every parent render creates a new `handleDelete`.

All 1000 rows re-render.

With:

```jsx
const handleDelete = useCallback((id) => {
    deleteEmployee(id);
}, []);
```

Rows only re-render if their actual props change.

---

## 2. E-Commerce Product List

```text
Product List

↓

Product Card

↓

Add to Cart Button
```

```jsx
<ProductCard
    product={product}
    onAdd={handleAdd}
/>
```

Changing the cart count re-renders the parent.

Without `useCallback`, every product card receives a new `onAdd` function and re-renders.

With `useCallback`, the function reference remains stable, allowing memoized product cards to avoid unnecessary renders.

---

## 3. Dashboard

```text
Dashboard

├── Header
├── Sidebar
├── Chart
├── User Profile
└── Notifications
```

The dashboard refreshes notifications every few seconds.

Memoized child components receiving callback props won't re-render unnecessarily if those callbacks are wrapped in `useCallback`.

---

# Dependency Example

```jsx
const handleIncrement = useCallback(() => {
    setCount(count + 1);
}, [count]);
```

Whenever `count` changes:

```text
Old Function

↓

New Function Created
```

Because the dependency changed.

---

# When NOT to Use `useCallback`

Many developers overuse it.

❌ Don't do this:

```jsx
const sayHello = useCallback(() => {
    console.log("Hello");
}, []);
```

If the function:

* isn't passed to child components,
* isn't a dependency of another hook,
* isn't causing performance issues,

then `useCallback` adds unnecessary complexity.

---

# `useCallback` vs `useMemo`

| `useCallback`                       | `useMemo`                              |
| ----------------------------------- | -------------------------------------- |
| Memoizes a **function**             | Memoizes a **computed value**          |
| Returns a function                  | Returns the result of a computation    |
| Used for stable callback references | Used to avoid expensive recalculations |

Example:

```jsx
const handleClick = useCallback(() => {
    console.log("Click");
}, []);
```

```jsx
const sortedUsers = useMemo(() => {
    return users.sort();
}, [users]);
```

---

# Common Interview Questions

### Does `useCallback` improve performance?

Not always.

It only helps when:

* Passing callbacks to `React.memo()` components.
* Using callbacks as dependencies in `useEffect`, `useMemo`, or another `useCallback`.
* Avoiding unnecessary function recreation where it impacts rendering.

Otherwise, it can introduce unnecessary overhead.

---

### Does `useCallback` stop parent re-renders?

No.

It only keeps the function reference stable. The parent still re-renders when its state or props change.

---

# Interview Answer (5+ Years)

> `useCallback` is a React Hook that memoizes a function and returns the same function reference across renders unless its dependencies change. Its primary purpose is to prevent unnecessary re-renders of memoized child components (`React.memo`) and to provide stable callback references when used as dependencies in other hooks like `useEffect` or `useMemo`. A common real-world use case is passing event handlers such as `onDelete`, `onEdit`, or `onAddToCart` to large lists or tables. However, `useCallback` should be used only when it provides a measurable performance benefit, as overusing it can make code more complex without improving performance.
