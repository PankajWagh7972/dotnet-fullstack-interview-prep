For a **5–7 years React + .NET Full Stack** interview, these are the kinds of questions that companies like Microsoft, Deloitte, BNY, JPMorgan, Accenture, EPAM, Cognizant, LTIMindtree, and product companies commonly ask.

---

# React Fundamentals

## Q1. What is React?

**Answer:**

React is an open-source JavaScript library for building user interfaces. It follows a component-based architecture and uses a Virtual DOM to efficiently update the UI.

---

## Q2. Why is React faster than traditional DOM manipulation?

**Answer:**

React uses a Virtual DOM. When state changes, React compares the new Virtual DOM with the previous one (reconciliation) and updates only the changed elements in the real DOM.

---

## Q3. What is JSX?

**Answer:**

JSX is JavaScript XML. It allows writing HTML-like syntax inside JavaScript.

```jsx
const element = <h1>Hello React</h1>;
```

JSX is converted into `React.createElement()` during compilation.

---

## Q4. Functional Component vs Class Component

| Functional         | Class                  |
| ------------------ | ---------------------- |
| Uses Hooks         | Uses lifecycle methods |
| Less boilerplate   | More code              |
| Preferred          | Mostly legacy          |
| Better performance | Slightly heavier       |

---

## Q5. Props vs State

| Props              | State                |
| ------------------ | -------------------- |
| Read-only          | Mutable              |
| Passed from parent | Managed by component |
| External data      | Internal data        |

---

# Component Lifecycle

## Q6. Explain the lifecycle of a functional component.

**Answer:**

Instead of lifecycle methods, React uses Hooks.

Equivalent mapping:

```text
Mount
↓

useEffect(() => {}, [])

Update
↓

useEffect(() => {}, [dependency])

Unmount
↓

return () => {}
```

---

## Q7. How do you run code only once?

```jsx
useEffect(() => {
    loadData();
}, []);
```

---

## Q8. How do you clean up resources?

```jsx
useEffect(() => {
    const id = setInterval(loadData, 1000);

    return () => clearInterval(id);
}, []);
```

---

# useState

## Q9. What is useState?

Allows a functional component to manage state.

```jsx
const [count, setCount] = useState(0);
```

---

## Q10. Why shouldn't we modify state directly?

Wrong:

```jsx
count++;
```

Correct:

```jsx
setCount(count + 1);
```

React only re-renders when state is updated through its setter.

---

# useEffect

## Q11. When does useEffect execute?

```jsx
useEffect(() => {})
```

Runs after every render.

---

```jsx
useEffect(() => {}, [])
```

Runs only once.

---

```jsx
useEffect(() => {}, [id])
```

Runs when `id` changes.

---

## Q12. Why does useEffect run twice in development?

Because React Strict Mode intentionally invokes certain lifecycle logic twice in development to help detect side effects. This does **not** happen in production.

---

# useMemo

## Q13. What is useMemo?

Caches the result of an expensive calculation.

```jsx
const total = useMemo(() => calculateTotal(items), [items]);
```

---

## Q14. When should you use useMemo?

Use it for expensive computations that don't need to run on every render.

Examples:

* Sorting
* Filtering
* Large calculations

---

# useCallback

## Q15. What is useCallback?

Caches a function.

```jsx
const handleClick = useCallback(() => {
    save();
}, []);
```

---

## Q16. Difference between useMemo and useCallback?

| useMemo        | useCallback       |
| -------------- | ----------------- |
| Memoizes value | Memoizes function |
| Returns result | Returns function  |

---

# useRef

## Q17. What is useRef?

Stores a mutable value without causing re-renders.

Example:

```jsx
const inputRef = useRef();
```

---

## Q18. Common use cases for useRef

* Focus input
* Store timer ID
* Previous value
* DOM access

---

# useReducer

## Q19. When should you use useReducer?

When state is complex.

```jsx
dispatch({
    type: "ADD_ITEM"
});
```

Better than many `useState` calls.

---

# Context API

## Q20. Why use Context?

Avoids Prop Drilling.

Instead of:

```text
App

↓

Parent

↓

Child

↓

GrandChild
```

Context allows:

```text
App

↓

Context

↓

Any Component
```

---

## Q21. Context vs Redux

| Context          | Redux              |
| ---------------- | ------------------ |
| Small state      | Large applications |
| Built into React | External library   |
| Simpler          | More scalable      |

---

# React.memo

## Q22. What is React.memo?

Prevents unnecessary re-rendering.

```jsx
export default React.memo(ProductCard);
```

---

## Q23. When won't React.memo help?

If props change every render (for example, a new object or function is created each time), `React.memo` cannot prevent re-renders.

---

# Rendering

## Q24. What causes a component to re-render?

* State changes
* Props change
* Context changes
* Parent re-renders (unless optimized)

---

## Q25. Virtual DOM vs Real DOM

| Virtual DOM                   | Real DOM           |
| ----------------------------- | ------------------ |
| Lightweight JavaScript object | Actual browser DOM |
| Fast diffing                  | Expensive updates  |

---

## Q26. What is Reconciliation?

The process React uses to compare the old Virtual DOM with the new Virtual DOM and determine the minimal changes needed in the real DOM.

---

# React Router

## Q27. Difference between Link and useNavigate

`<Link>` is used for navigation in JSX.

`useNavigate()` is used inside JavaScript logic.

---

## Q28. How do you create protected routes?

Check authentication before rendering.

```jsx
if (!isLoggedIn)
    return <Navigate to="/login" />;
```

---

# API Calls

## Q29. Where should API calls be made?

Usually inside `useEffect()` or a custom hook.

---

## Q30. How do you cancel API requests?

Using `AbortController`.

```jsx
const controller = new AbortController();
```

---

# Custom Hooks

## Q31. Why create custom hooks?

To reuse stateful logic.

Example:

```jsx
useFetch()

useAuth()

useDebounce()
```

---

# Performance

## Q32. How do you optimize React performance?

* React.memo
* useMemo
* useCallback
* Lazy Loading
* Code Splitting
* Pagination
* Virtualization
* Debouncing

---

## Q33. What is Lazy Loading?

```jsx
const Home = React.lazy(() => import("./Home"));
```

Loads components only when needed.

---

## Q34. What is Code Splitting?

Breaking a large JavaScript bundle into smaller chunks that load on demand.

---

# Forms

## Q35. Controlled vs Uncontrolled Components

Controlled:

```jsx
<input
 value={name}
 onChange={...}
/>
```

Uncontrolled:

```jsx
<input ref={inputRef}/>
```

---

# Error Handling

## Q36. What is an Error Boundary?

A React component that catches rendering errors in its child component tree and displays a fallback UI instead of crashing the entire application.

---

# Authentication

## Q37. JWT Authentication Flow

```text
Login

↓

JWT Token

↓

Store Token

↓

API Calls

↓

Authorization Header
```

---

## Q38. Where should JWT be stored?

Prefer **HttpOnly cookies** for better security against XSS. `localStorage` is simple but more vulnerable if malicious scripts execute. The choice depends on your application's architecture and security requirements.

---

# React 18

## Q39. What is Automatic Batching?

Multiple state updates are grouped into one render.

Before:

Two renders.

After React 18:

One render.

---

## Q40. What is Concurrent Rendering?

React can interrupt low-priority rendering work to keep the UI responsive for higher-priority interactions like typing or clicking.

---

## Q41. What is useTransition?

Marks updates as low priority.

Useful for:

* Search
* Filters
* Large tables

---

## Q42. What is useDeferredValue?

Defers rendering based on a changing value to keep the UI responsive during expensive updates.

---

# React 19

## Q43. What is useOptimistic?

Immediately updates the UI before the server confirms success.

Great for:

* Chat
* Like button
* Comments

---

## Q44. What is useActionState?

Simplifies handling async form submissions by managing action state and results.

---

# Design Patterns

## Q45. What is a Higher Order Component (HOC)?

A function that takes a component and returns an enhanced component.

```jsx
withAuthorization(Component)
```

---

## Q46. What is a Render Prop?

A component shares logic by accepting a function as a prop and calling it to render UI.

---

# React + .NET

## Q47. How does React communicate with .NET?

```text
React

↓

Axios

↓

.NET Web API

↓

SQL Server
```

Usually using REST APIs (or SignalR for real-time features).

---

## Q48. How do you secure React APIs?

* JWT Authentication
* HTTPS
* Role-based Authorization
* CORS configuration
* Input validation

---

# Scenario Questions

## Q49. Search API is called on every key press. How do you optimize?

Use:

* Debouncing
* `useDeferredValue`
* `useTransition`

---

## Q50. Large table has 50,000 rows. What do you do?

* Pagination
* Virtualization (`react-window` / `react-virtualized`)
* Lazy Loading
* Memoization

---

## Q51. Parent component re-renders unnecessarily. How do you fix it?

* Wrap child with `React.memo`
* Memoize callbacks with `useCallback`
* Memoize expensive values with `useMemo`
* Avoid creating new objects/functions inside JSX unless necessary

---

## Q52. How do you share login information across the app?

Use:

* Context API (small to medium apps)
* Redux Toolkit (large apps)
* Store only the necessary authentication state

---

## Q53. When would you choose Context API over Redux?

Use Context when the shared state is relatively simple (theme, current user, language). Use Redux Toolkit when you have complex application state, advanced debugging needs, or many features sharing state.

---

# Frequently Asked Senior-Level Questions

1. Explain React's rendering process.
2. How does React Fiber work?
3. Difference between `useMemo` and `React.memo`.
4. Explain reconciliation.
5. Why does React use keys?
6. How do you optimize a slow React application?
7. How do you prevent unnecessary re-renders?
8. How do you implement role-based authorization in React?
9. How do you manage global state in enterprise applications?
10. Explain the complete React component lifecycle using Hooks.
11. What are stale closures in React Hooks, and how do you avoid them?
12. How do you handle race conditions when multiple API requests are in flight?
13. What is the difference between client-side rendering (CSR), server-side rendering (SSR), and static site generation (SSG)?
14. How do you structure a large React application for maintainability?
15. How would you debug a React application with performance issues?

These 50+ questions cover the majority of React topics that interviewers expect from **5–7 years experienced React/.NET Full Stack developers**. If you can confidently answer these and explain the reasoning behind your choices with examples from your projects, you'll be well prepared for senior React interview rounds.
