`useEffect` and `useMemo` are two of the most frequently asked React Hooks in interviews. The key difference is:

* **useEffect** → Perform **side effects** (API calls, subscriptions, timers, DOM updates).
* **useMemo** → **Cache expensive calculations** to avoid unnecessary recomputation.

---

# 1. useEffect

## Purpose

Run code after a component renders.

Common uses:

* API calls
* Database requests
* Timers
* Event listeners
* WebSocket connections
* Updating document title

---

## Real-Time Example 1: Load Employees from API

Suppose your .NET API is:

```text
GET /api/employees
```

React:

```javascript
import { useState, useEffect } from "react";
import axios from "axios";

function EmployeeList() {

    const [employees, setEmployees] = useState([]);

    useEffect(() => {

        loadEmployees();

    }, []);

    async function loadEmployees() {

        const response = await axios.get("/api/employees");

        setEmployees(response.data);
    }

    return (
        <>
            {employees.map(e =>
                <p key={e.id}>{e.name}</p>
            )}
        </>
    );
}
```

### Flow

```text
Component Loaded

↓

useEffect()

↓

API Call

↓

State Updated

↓

Component Re-rendered
```

---

## Real-Time Example 2: Search Employees

```javascript
const [search, setSearch] = useState("");

useEffect(() => {

    if (search.length >= 3) {

        searchEmployee();

    }

}, [search]);
```

Whenever `search` changes:

```text
P

↓

No API

↓

Pa

↓

No API

↓

Pan

↓

API Called
```

This avoids unnecessary requests for very short input.

---

## Real-Time Example 3: Auto Refresh Dashboard

```javascript
useEffect(() => {

    const interval = setInterval(() => {

        loadDashboard();

    }, 10000);

    return () => clearInterval(interval);

}, []);
```

Every 10 seconds:

```text
Dashboard

↓

Refresh

↓

Refresh

↓

Refresh
```

---

## Real-Time Example 4: SignalR / WebSocket

```javascript
useEffect(() => {

    connection.start();

    connection.on("ReceiveOrder", order => {

        setOrders(prev => [...prev, order]);

    });

    return () => {

        connection.stop();

    };

}, []);
```

Without the cleanup function, multiple connections could remain open after navigating away from the page.

---

# useMemo

## Purpose

Cache the result of an expensive calculation so it only recomputes when its dependencies change.

---

## Real-Time Example 1: Large Employee Search

Suppose you have 50,000 employees.

```javascript
const filteredEmployees = useMemo(() => {

    return employees.filter(e =>
        e.name.toLowerCase().includes(search.toLowerCase())
    );

}, [employees, search]);
```

Without `useMemo`:

```text
User Types

↓

Filter 50,000 Employees

↓

Component Re-renders

↓

Filter Again

↓

Slow
```

With `useMemo`:

```text
Filter Once

↓

Cached

↓

Reuse Cached Result

↓

Fast
```

---

## Real-Time Example 2: Expensive Dashboard Calculation

Imagine thousands of sales records.

```javascript
const totalRevenue = useMemo(() => {

    console.log("Calculating...");

    return orders.reduce((sum, order) =>
        sum + order.amount,
        0
    );

}, [orders]);
```

Now changing unrelated state won't trigger the calculation.

```javascript
const [theme, setTheme] = useState("light");
```

Changing the theme:

```text
Theme Changed

↓

Component Re-render

↓

Revenue NOT Recalculated
```

---

## Real-Time Example 3: Sorting Large Data

```javascript
const sortedEmployees = useMemo(() => {

    return [...employees].sort(
        (a, b) => a.salary - b.salary
    );

}, [employees]);
```

Sorting 100,000 rows repeatedly is expensive. `useMemo` prevents unnecessary sorting when the data hasn't changed.

---

# Difference

### Without useMemo

```javascript
const total = orders.reduce((sum, order) =>
    sum + order.amount,
    0);
```

Every render:

```text
Render

↓

Calculate

↓

Render

↓

Calculate

↓

Render

↓

Calculate
```

---

### With useMemo

```javascript
const total = useMemo(() => {

    return orders.reduce((sum, order) =>
        sum + order.amount,
        0);

}, [orders]);
```

```text
First Render

↓

Calculate

↓

Cache

↓

Next Render

↓

Use Cache
```

---

# Can We Call an API Inside useMemo?

❌ No.

```javascript
const data = useMemo(() => {

    axios.get("/api/users");

}, []);
```

This is incorrect because `useMemo` is intended for synchronous calculations, not side effects.

Use `useEffect` instead:

```javascript
useEffect(() => {

    loadUsers();

}, []);
```

---

# Enterprise Example

Suppose you're building an insurance application.

### useEffect

```text
Policy Screen Opened

↓

Load Policies API

↓

Load Customer API

↓

Load Claims API
```

### useMemo

```text
50000 Policies

↓

Calculate Total Premium

↓

Calculate Active Policies

↓

Calculate Claims

↓

Cache Results
```

If the user only changes the UI theme, the expensive calculations are reused instead of running again.

---

# Interview Difference

| useEffect             | useMemo                                       |
| --------------------- | --------------------------------------------- |
| Used for side effects | Used for memoization                          |
| API calls             | Expensive calculations                        |
| Timers                | Filtering                                     |
| Event listeners       | Sorting                                       |
| WebSockets            | Aggregation                                   |
| Async operations      | Synchronous calculations                      |
| Runs after render     | Runs during render and returns a cached value |

---

# Interview Answer (30 seconds)

> `useEffect` is used to perform side effects such as calling APIs, subscribing to events, setting timers, or interacting with external systems. It runs after the component renders and can re-run when specified dependencies change.
>
> `useMemo` is used to optimize performance by memoizing the result of expensive synchronous computations, such as filtering, sorting, or aggregating large datasets. It recalculates the value only when its dependencies change, reducing unnecessary work during re-renders.

### Easy way to remember

* **useEffect** → "**Do something**" after rendering (API call, timer, subscription).
* **useMemo** → "**Remember something**" expensive (filtered list, sorted data, calculated totals).


`useCallback` is another React performance optimization hook that is often asked together with `useMemo`.

> **`useCallback` memoizes a function** so that React doesn't create a new function instance on every render.

## Why do we need `useCallback`?

Every time a React component re-renders, **all functions inside it are recreated**.

Example:

```javascript id="q3jyr7"
function Parent() {

    const [count, setCount] = useState(0);

    const handleClick = () => {
        console.log("Button Clicked");
    };

    return (
        <>
            <button onClick={() => setCount(count + 1)}>
                Count
            </button>

            <Child onClick={handleClick} />
        </>
    );
}
```

Every time `count` changes:

```text id="1s2sba"
Parent Render

↓

New handleClick()

↓

Child Re-render
```

Even though the logic didn't change.

---

# Real-World Example 1: Employee List (React.memo)

Suppose you have 5,000 employees.

### Parent Component

```javascript id="7b5zkv"
import { useState, useCallback } from "react";
import EmployeeList from "./EmployeeList";

function App() {

    const [theme, setTheme] = useState("light");

    const handleSelect = useCallback((id) => {

        console.log("Selected Employee:", id);

    }, []);

    return (
        <>
            <button
                onClick={() =>
                    setTheme(theme === "light" ? "dark" : "light")
                }
            >
                Change Theme
            </button>

            <EmployeeList onSelect={handleSelect} />
        </>
    );
}
```

---

### Child Component

```javascript id="xj57sn"
import React from "react";

const EmployeeList = React.memo(({ onSelect }) => {

    console.log("Employee List Rendered");

    return (
        <button onClick={() => onSelect(10)}>
            Select Employee
        </button>
    );

});

export default EmployeeList;
```

### Without `useCallback`

Changing only the theme:

```text id="emc1rm"
Parent Render

↓

New handleSelect()

↓

EmployeeList Render
```

Even though the employee list didn't change.

---

### With `useCallback`

```text id="yrx9nw"
Parent Render

↓

Same handleSelect Reference

↓

EmployeeList Doesn't Re-render
```

This is where `useCallback` provides the biggest benefit.

---

# Real-World Example 2: Search Box

```javascript id="kzq3yn"
const searchEmployee = useCallback(async (text) => {

    const response =
        await axios.get("/api/employees?search=" + text);

    setEmployees(response.data);

}, []);
```

```javascript id="0wt4yo"
<SearchBox onSearch={searchEmployee} />
```

If the parent re-renders because of unrelated state changes, the `SearchBox` won't re-render just because the callback reference changed.

---

# Real-World Example 3: Data Grid

Suppose you're using a grid component.

```javascript id="pcglte"
<DataGrid

    rows={employees}

    onRowClick={handleRowClick}

/>
```

```javascript id="oxtgxh"
const handleRowClick = useCallback((row) => {

    navigate("/employee/" + row.id);

}, [navigate]);
```

Without `useCallback`, every parent render creates a new `onRowClick` function, causing the grid to receive a new prop and potentially re-render.

---

# Real-World Example 4: E-Commerce Cart

```javascript id="gn4jlwm"
const addToCart = useCallback((productId) => {

    dispatch({
        type: "ADD",
        payload: productId
    });

}, []);
```

```javascript id="v0gbfj"
<ProductCard

    product={item}

    onAdd={addToCart}

/>
```

If only the cart count changes:

```text id="lhwvml"
App Render

↓

Same addToCart()

↓

Product Cards Don't Re-render
```

This can save a lot of rendering work when displaying many products.

---

# Difference Between `useMemo` and `useCallback`

### useMemo

Caches a **value**.

```javascript id="otqesg"
const filteredEmployees = useMemo(() => {

    return employees.filter(e =>
        e.active);

}, [employees]);
```

Returns:

```text id="srvqlu"
Array
```

---

### useCallback

Caches a **function**.

```javascript id="vqutbl"
const saveEmployee = useCallback(() => {

    console.log("Save");

}, []);
```

Returns:

```text id="mp73i6"
Function
```

---

# `useCallback` Internally

```javascript id="ecjlwm"
const fn = useCallback(() => {

    console.log("Hello");

}, []);
```

is conceptually similar to:

```javascript id="a8l7q5"
const fn = useMemo(() => {

    return () => {

        console.log("Hello");

    };

}, []);
```

`useCallback(fn, deps)` is essentially a convenience wrapper around `useMemo(() => fn, deps)`.

---

# When Should You Use `useCallback`?

✅ Use it when:

* Passing callbacks to components wrapped with `React.memo`
* Passing handlers to expensive child components
* Avoiding unnecessary re-renders in large component trees
* Stable callbacks required by custom hooks or third-party libraries

❌ Don't use it everywhere. It also has a small cost. Memoize only when it prevents meaningful re-renders.

---

# `useEffect` + `useCallback`

A stable callback also helps when it's used as a dependency.

```javascript id="8gfmbl"
const loadEmployees = useCallback(async () => {

    const response = await axios.get("/api/employees");

    setEmployees(response.data);

}, []);
```

```javascript id="zs8ylv"
useEffect(() => {

    loadEmployees();

}, [loadEmployees]);
```

Because `loadEmployees` is memoized, the effect won't re-run just because the component re-rendered.

---

# Interview Summary

| Hook          | Purpose                                                 | Returns    |
| ------------- | ------------------------------------------------------- | ---------- |
| `useEffect`   | Perform side effects (API calls, subscriptions, timers) | Nothing    |
| `useMemo`     | Cache expensive computed values                         | A value    |
| `useCallback` | Cache function references                               | A function |

---

## Interview Answer (30 seconds)

> `useCallback` is used to memoize a function so that React preserves the same function reference across renders unless its dependencies change. It's primarily used when passing callbacks to child components wrapped with `React.memo`, or when a stable function reference is needed to avoid unnecessary re-renders or repeated effects. A common example is passing `onClick`, `onSearch`, or `onRowClick` handlers to large lists or data grids, where recreating the function on every render would cause unnecessary child component updates.
