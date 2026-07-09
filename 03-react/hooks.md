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


This is a common React interview question for **5+ years of experience**. Interviewers usually expect you to know **what `useReducer` is, how it works internally, why it exists, and when it is a better choice than `useState`**.

---

# What is `useReducer`?

`useReducer` is a React Hook used to manage **complex state logic**. Instead of updating state directly, you **dispatch actions** to a **reducer function**, which calculates and returns the next state.

It follows the same concept as Redux, but it's built into React and is intended for component-level or local state management.

### Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

* **state** → Current state
* **dispatch** → Function used to send actions
* **reducer** → Function that determines how state changes
* **initialState** → Initial value of the state

---

# How It Works

```text
User Clicks Button

↓

dispatch({ type: "increment" })

↓

Reducer Receives Current State + Action

↓

Reducer Returns New State

↓

React Updates UI
```

Unlike `useState`, where you directly set the value, `useReducer` centralizes update logic in one place.

---

# Basic Example

```jsx
import { useReducer } from "react";

const initialState = { count: 0 };

function reducer(state, action) {
    switch (action.type) {
        case "increment":
            return { count: state.count + 1 };

        case "decrement":
            return { count: state.count - 1 };

        case "reset":
            return initialState;

        default:
            return state;
    }
}

export default function Counter() {
    const [state, dispatch] = useReducer(reducer, initialState);

    return (
        <>
            <h1>{state.count}</h1>

            <button onClick={() => dispatch({ type: "increment" })}>
                +
            </button>

            <button onClick={() => dispatch({ type: "decrement" })}>
                -
            </button>

            <button onClick={() => dispatch({ type: "reset" })}>
                Reset
            </button>
        </>
    );
}
```

---

# How `useState` Would Look

```jsx
const [count, setCount] = useState(0);

const increment = () => setCount(count + 1);
const decrement = () => setCount(count - 1);
const reset = () => setCount(0);
```

This is simple and perfectly fine for small state.

---

# Why `useReducer`?

Imagine a form with multiple fields:

```text
Employee Form

Name

Email

Department

Salary

Address

Skills

Experience

Validation Errors

Loading

Success
```

With `useState`:

```jsx
const [name, setName] = useState("");
const [email, setEmail] = useState("");
const [department, setDepartment] = useState("");
const [salary, setSalary] = useState(0);
const [errors, setErrors] = useState({});
const [loading, setLoading] = useState(false);
const [success, setSuccess] = useState(false);
```

Managing many related pieces of state becomes harder.

With `useReducer`:

```jsx
const initialState = {
    name: "",
    email: "",
    department: "",
    salary: 0,
    errors: {},
    loading: false,
    success: false
};
```

One reducer can handle all updates in a predictable way.

---

# Real-Time Use Cases

## 1. Complex Forms

```text
Registration Form

↓

User Types

↓

dispatch({ type: "UPDATE_NAME", payload: "Pankaj" })

↓

Reducer Updates State

↓

UI Re-renders
```

Benefits:

* All update logic is centralized.
* Easier to validate and reset the form.
* Simpler to maintain as the form grows.

---

## 2. Shopping Cart

State:

```jsx
{
    items: [],
    total: 0,
    discount: 0
}
```

Actions:

```text
ADD_ITEM

REMOVE_ITEM

UPDATE_QUANTITY

CLEAR_CART

APPLY_DISCOUNT
```

Reducer:

```jsx
switch(action.type){
    case "ADD_ITEM":
    case "REMOVE_ITEM":
    case "UPDATE_QUANTITY":
}
```

Instead of spreading logic across many `setState` calls, everything is handled in one place.

---

## 3. Authentication

```text
Login

↓

Loading

↓

Success

↓

User Info

↓

Error
```

State:

```jsx
{
    user: null,
    loading: false,
    error: null,
    isAuthenticated: false
}
```

Actions:

```text
LOGIN_REQUEST

LOGIN_SUCCESS

LOGIN_FAILURE

LOGOUT
```

This approach makes authentication state transitions clear and easy to test.

---

## 4. Multi-Step Wizard

```text
Step 1

↓

Step 2

↓

Step 3

↓

Submit
```

Actions:

```text
NEXT_STEP

PREVIOUS_STEP

RESET

UPDATE_FORM
```

A reducer keeps the navigation and form updates organized.

---

# `useState` vs `useReducer`

| `useState`                        | `useReducer`                       |
| --------------------------------- | ---------------------------------- |
| Best for simple state             | Best for complex state             |
| Direct state updates              | State changes go through a reducer |
| Easy to learn                     | Better for complex business logic  |
| Good for a few state variables    | Good for many related state values |
| Update logic can become scattered | Update logic is centralized        |

---

# Can `useReducer` Replace Redux?

Not completely.

`useReducer` manages **local component state**.

Redux manages **global application state** shared across many unrelated components.

A common pattern is to combine `useReducer` with the Context API for medium-sized applications, but Redux (or similar libraries) is still useful for large applications with advanced global state needs.

---

# When Should You Use `useReducer`?

Use it when:

* State has multiple related values.
* State transitions depend on the previous state.
* You have many different update actions.
* Business logic becomes difficult to manage with multiple `useState` calls.
* You want predictable, centralized update logic.

Avoid it for simple state like:

```jsx
const [count, setCount] = useState(0);
```

Using `useReducer` there would add unnecessary complexity.

---

# Interview Answer (5+ Years)

> `useReducer` is a React Hook used to manage complex state logic by centralizing state updates in a reducer function. Instead of updating state directly, components dispatch actions, and the reducer returns the next state based on the current state and the action. It's particularly useful for forms with many fields, shopping carts, authentication flows, and multi-step wizards where state transitions are interconnected. Compared to `useState`, `useReducer` improves maintainability and makes state changes more predictable by keeping update logic in a single place.


Here's a **complete, interview-friendly example** of `useReducer`. This is the type of example you can easily explain in an interview because it demonstrates multiple actions (`Increment`, `Decrement`, `Reset`) and clearly shows the flow of `dispatch → reducer → state`.

## Complete Example

```jsx
import React, { useReducer } from "react";

// Initial state
const initialState = {
  count: 0,
};

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return {
        ...state,
        count: state.count + 1,
      };

    case "DECREMENT":
      return {
        ...state,
        count: state.count - 1,
      };

    case "RESET":
      return initialState;

    default:
      return state;
  }
}

function Counter() {
  // useReducer Hook
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h1>Counter using useReducer</h1>

      <h2>{state.count}</h2>

      <button
        onClick={() => dispatch({ type: "INCREMENT" })}
      >
        Increment
      </button>

      <button
        onClick={() => dispatch({ type: "DECREMENT" })}
        style={{ marginLeft: "10px" }}
      >
        Decrement
      </button>

      <button
        onClick={() => dispatch({ type: "RESET" })}
        style={{ marginLeft: "10px" }}
      >
        Reset
      </button>
    </div>
  );
}

export default Counter;
```

---

# How It Works

### 1. Initial State

```jsx
const initialState = {
  count: 0,
};
```

This is the starting state.

---

### 2. Reducer Function

```jsx
function counterReducer(state, action) {
```

The reducer always receives:

* **Current state**
* **Action object**

Example action:

```jsx
{
  type: "INCREMENT"
}
```

---

### 3. Dispatch Action

```jsx
dispatch({ type: "INCREMENT" });
```

`dispatch` sends an action to the reducer.

---

### 4. Reducer Executes

```jsx
case "INCREMENT":
    return {
        ...state,
        count: state.count + 1
    };
```

React updates the state with the returned object.

---

### 5. Component Re-renders

```jsx
<h2>{state.count}</h2>
```

The updated count is displayed automatically.

---

# Flow Diagram

```text
User Clicks "Increment"
          │
          ▼
dispatch({ type: "INCREMENT" })
          │
          ▼
counterReducer(state, action)
          │
          ▼
Returns New State
{ count: 1 }
          │
          ▼
React Updates State
          │
          ▼
Component Re-renders
```

---

# Real-Time Example: Shopping Cart

A reducer becomes even more useful when you have many related actions.

```jsx
const initialState = {
  items: [],
  total: 0,
};
```

Reducer:

```jsx
function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      // Add item to cart
      break;

    case "REMOVE_ITEM":
      // Remove item
      break;

    case "UPDATE_QUANTITY":
      // Update quantity
      break;

    case "CLEAR_CART":
      // Empty cart
      break;

    default:
      return state;
  }
}
```

Instead of managing several `useState` hooks, all cart-related logic stays in one reducer.

---

## Interview Tip

When explaining `useReducer`, remember this sequence:

```text
Initial State
      ↓
useReducer()
      ↓
dispatch(action)
      ↓
Reducer receives state + action
      ↓
Reducer returns new state
      ↓
React re-renders the component
```

This clearly demonstrates that **`dispatch` never updates the state directly**. It simply sends an action, and the **reducer decides how the state should change**. This predictable flow is the main advantage of `useReducer`.


For a **5+ years React interview**, don't just show `fetch()` inside `useEffect`. Explain **loading state**, **error handling**, **cleanup**, and **race condition prevention**.

---

# Example 1: Using `useEffect` + `fetch` (Most Common)

```jsx
import { useEffect, useState } from "react";

function Users() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUsers() {
      try {
        setLoading(true);

        const response = await fetch(
          "https://jsonplaceholder.typicode.com/users",
          {
            signal: controller.signal,
          }
        );

        if (!response.ok) {
          throw new Error("Failed to fetch users");
        }

        const data = await response.json();
        setUsers(data);
      } catch (err) {
        // Ignore aborted requests
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchUsers();

    return () => {
      controller.abort();
    };
  }, []);

  if (loading) return <h2>Loading...</h2>;

  if (error) return <h2>{error}</h2>;

  return (
    <div>
      <h2>Users</h2>

      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  );
}

export default Users;
```

---

# Execution Flow

```text
Component Mounts
       │
       ▼
useEffect Executes
       │
       ▼
Loading = true
       │
       ▼
API Call
       │
       ▼
Success?
   /         \
 Yes         No
  │           │
setUsers    setError
  │           │
  └──────┬────┘
         ▼
Loading = false
         ▼
UI Updates
```

---

# Why `AbortController`?

Suppose the user navigates away before the API finishes.

Without cleanup:

```text
Component Unmounted

↓

API Response Arrives

↓

setState()

↓

Warning / Memory Leak
```

With cleanup:

```jsx
return () => controller.abort();
```

The request is canceled, preventing state updates on an unmounted component.

---

# Example 2: Using Axios

```jsx
import axios from "axios";
import { useEffect, useState } from "react";

function Users() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    async function loadUsers() {
      try {
        const response = await axios.get(
          "https://jsonplaceholder.typicode.com/users"
        );

        setUsers(response.data);
      } catch (err) {
        console.log(err);
      }
    }

    loadUsers();
  }, []);

  return (
    <>
      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </>
  );
}
```

---

# Real-Time Example: Employee Management System

```text
Employee Page

↓

Loading Spinner

↓

GET /api/employees

↓

Success

↓

Employee Table
```

```jsx
useEffect(() => {
    loadEmployees();
}, []);
```

While loading:

```jsx
{loading && <Spinner />}
```

If error:

```jsx
{error && <ErrorMessage />}
```

Otherwise:

```jsx
<EmployeeTable employees={employees} />
```

---

# Best Practices

* Show a loading indicator while fetching data.
* Handle API errors gracefully.
* Cancel requests on component unmount (`AbortController` for `fetch`).
* Use `async/await` for cleaner code.
* Avoid unnecessary re-fetching by setting the correct dependency array.
* For large applications, prefer libraries like **React Query (TanStack Query)** or **SWR** because they provide caching, background refetching, retries, and request deduplication.

---

# Interview Answer (5+ Years)

> In React, asynchronous data is typically loaded inside `useEffect` using `async/await`. I maintain separate state for the fetched data, loading status, and errors using `useState`. While the request is in progress, I display a loading indicator. If the request succeeds, I update the data state; if it fails, I show an appropriate error message. I also clean up in-flight requests using `AbortController` (or cancellation mechanisms provided by the HTTP client) to prevent updating state after a component unmounts. In production applications, I often use libraries like **TanStack Query (React Query)** for features such as caching, automatic retries, and background data synchronization.
