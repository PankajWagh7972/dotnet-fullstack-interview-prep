This is one of the most common React interview questions for **5+ years of experience**. Interviewers expect you to know:

* What React Router is
* How routing works internally
* Difference between client-side and server-side routing
* Dynamic routing
* Nested routes
* Route parameters
* Protected routes

---

# What is React Router?

**React Router** is a library that enables **client-side routing** in React applications.

Instead of requesting a new HTML page from the server for every URL, React Router updates the URL and renders different React components **without refreshing the page**.

Example:

```text
Home
/products
/products/101
/about
/contact
```

All of these are handled by React.

---

# How React Router Works Internally

When you click a link:

```text
User Clicks Link

↓

React Router intercepts click

↓

history.pushState()

↓

Browser URL Changes

↓

No Page Refresh

↓

React matches Route

↓

Corresponding Component Renders
```

Notice:

* No server request
* No page reload
* Only component changes

---

# Installation

```bash
npm install react-router-dom
```

---

# Basic Routing

## App.jsx

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./Home";
import About from "./About";
import Contact from "./Contact";

function App() {
  return (
    <BrowserRouter>

      <Routes>

        <Route path="/" element={<Home />} />

        <Route path="/about" element={<About />} />

        <Route path="/contact" element={<Contact />} />

      </Routes>

    </BrowserRouter>
  );
}

export default App;
```

---

# Navigation

Instead of

```html
<a href="/about">About</a>
```

Use

```jsx
import { Link } from "react-router-dom";

<Link to="/about">
    About
</Link>
```

Why?

`<a>` refreshes the page.

`<Link>` changes URL without reload.

---

# Dynamic Routing

Suppose you have products.

```text
Product 101

Product 102

Product 103
```

Instead of creating

```text
/product101

/product102

/product103
```

Use one route.

```jsx
<Route
    path="/products/:id"
    element={<ProductDetails />}
/>
```

`:id` is called a **Route Parameter**.

---

# Reading Route Parameters

```jsx
import { useParams } from "react-router-dom";

function ProductDetails() {

    const { id } = useParams();

    return (
        <h1>
            Product ID : {id}
        </h1>
    );
}
```

If URL is

```text
/products/101
```

Output

```text
Product ID : 101
```

If

```text
/products/250
```

Output

```text
Product ID : 250
```

Same component handles every product.

---

# Real-Time Example

Employee Management System

```text
Employees

↓

Click Employee

↓

/employees/15
```

Route

```jsx
<Route
    path="/employees/:employeeId"
    element={<EmployeeDetails />}
/>
```

Component

```jsx
import { useParams } from "react-router-dom";

function EmployeeDetails() {

    const { employeeId } = useParams();

    return (
        <h1>
            Employee : {employeeId}
        </h1>
    );
}
```

Then

```jsx
useEffect(() => {

    fetch(`/api/employees/${employeeId}`);

}, [employeeId]);
```

The API fetches only the selected employee.

---

# Nested Routing

Suppose Dashboard

```text
Dashboard

├── Profile

├── Orders

├── Settings
```

Routes

```jsx
<Route path="/dashboard" element={<Dashboard />}>

    <Route path="profile" element={<Profile />} />

    <Route path="orders" element={<Orders />} />

    <Route path="settings" element={<Settings />} />

</Route>
```

Inside Dashboard

```jsx
import { Outlet } from "react-router-dom";

function Dashboard() {

    return (

        <div>

            <Sidebar />

            <Outlet />

        </div>

    );
}
```

`Outlet` renders child routes.

---

# Programmatic Navigation

Instead of clicking a link.

```jsx
import { useNavigate } from "react-router-dom";

function Login() {

    const navigate = useNavigate();

    const login = () => {

        // Login Success

        navigate("/dashboard");
    };

    return (
        <button onClick={login}>
            Login
        </button>
    );
}
```

---

# Protected Route

```jsx
import { Navigate } from "react-router-dom";

function ProtectedRoute({ children }) {

    const isLoggedIn = true;

    if (!isLoggedIn) {

        return <Navigate to="/login" />;
    }

    return children;
}
```

Usage

```jsx
<Route
    path="/dashboard"
    element={
        <ProtectedRoute>
            <Dashboard />
        </ProtectedRoute>
    }
/>
```

---

# Route Flow

```text
Browser URL

↓

React Router

↓

Match Route

↓

Extract Parameters

↓

Render Component

↓

Component Fetches Data

↓

Display UI
```

---

# BrowserRouter vs HashRouter

| BrowserRouter                 | HashRouter                                                  |
| ----------------------------- | ----------------------------------------------------------- |
| Uses HTML5 History API        | Uses URL hash (`#`)                                         |
| Clean URLs                    | URLs contain `#`                                            |
| Requires server configuration | No server configuration needed                              |
| Most commonly used            | Used for static hosting when server rewrite isn't available |

Example:

BrowserRouter

```text
/products/100
```

HashRouter

```text
#/products/100
```

---

# Common Interview Questions

### Why use `<Link>` instead of `<a>`?

Because `<Link>` performs **client-side navigation** without refreshing the page, while `<a>` causes a full page reload.

---

### What is `useParams()`?

It reads **dynamic route parameters** from the URL.

Example

```text
/products/:id
```

returns

```jsx
const { id } = useParams();
```

---

### What is `useNavigate()`?

It is used for **programmatic navigation**, such as redirecting a user after login or form submission.

---

### Difference between Route Params and Query Params

| Route Params              | Query Params                    |
| ------------------------- | ------------------------------- |
| `/products/101`           | `/products?id=101`              |
| Access with `useParams()` | Access with `useSearchParams()` |

---

# Interview Answer (5+ Years)

> React Router enables client-side routing in React applications by mapping URL paths to React components without reloading the page. It uses the browser's History API to update the URL and render the matching component. Dynamic routing is implemented using route parameters, such as `/products/:id`, where `:id` is a placeholder. Inside the component, the parameter can be accessed using the `useParams()` hook to fetch or display data for that specific resource. React Router also supports nested routes using `<Outlet>`, programmatic navigation with `useNavigate()`, and protected routes for authentication, making it suitable for building scalable single-page applications.
