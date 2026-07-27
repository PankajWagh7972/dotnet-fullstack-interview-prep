I think you mean **"What new things were introduced in React after React 14?"**

There actually **was no React 14**. The major React releases went like this:

* React 15
* React 16 (Fiber)
* React 17
* React 18
* React 19 (current major release)

If you're interviewing as a **5–6 years Full Stack .NET + React developer**, these are the important changes you should know.

---

# React 16 (Fiber Architecture)

This was one of the biggest changes.

## Fiber Reconciliation

Before React 16:

* Rendering was synchronous.
* Large UI updates could block the browser.

React Fiber introduced:

* Interruptible rendering
* Better scheduling
* Faster UI updates
* Foundation for Concurrent Rendering

---

## Error Boundaries

Before React 16:

One component crash could crash the entire app.

Now:

```jsx
class ErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    console.log(error);
  }

  render() {
    return this.props.children;
  }
}
```

---

## Fragments

Instead of:

```jsx
<div>
   <Header />
   <Content />
</div>
```

You can write:

```jsx
<>
   <Header />
   <Content />
</>
```

No unnecessary DOM element.

---

# React 17

React 17 introduced very few new APIs.

Main goal:

Upgrade React without breaking applications.

Major improvement:

## New Event Delegation

Before:

```text
document
    ↑
React Events
```

After:

```text
Root Container
       ↑
React Events
```

This allows multiple React versions on the same page more easily.

---

# React 18 (Very Important)

Most interview questions come from React 18.

---

## Concurrent Rendering

React can pause rendering.

```text
User typing

↓

Urgent Update

↓

Background Rendering
```

UI remains responsive.

---

## Automatic Batching

Before React 18:

```jsx
setCount(c => c + 1);
setLoading(true);
```

Two renders.

Now:

One render.

Less rendering = better performance.

---

## createRoot()

Old:

```jsx
ReactDOM.render(<App />, root);
```

New:

```jsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));

root.render(<App />);
```

---

## useTransition()

Allows low-priority updates.

Example:

User types in search.

```text
Typing

↓

Immediate UI update

↓

Large search list updates later
```

Example:

```jsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
    setProducts(filteredProducts);
});
```

Typing stays smooth.

---

## useDeferredValue()

Useful for expensive rendering.

```jsx
const deferredSearch = useDeferredValue(searchText);
```

The input updates immediately while the filtered list can lag slightly.

---

## Suspense Improvements

Before:

Mostly for lazy loading.

```jsx
<Suspense fallback={<Spinner />}>
    <Products />
</Suspense>
```

Now it better supports asynchronous rendering patterns.

---

# React 19 (Latest)

---

## Actions

Instead of manually managing loading and error state:

Old:

```jsx
const handleSubmit = async () => {
    setLoading(true);

    await save();

    setLoading(false);
}
```

New:

React provides **Actions** to simplify async form workflows.

---

## useActionState()

```jsx
const [state, formAction] = useActionState(action, initialState);
```

Useful for forms.

---

## useOptimistic()

Very popular interview topic.

Instead of:

```text
Click Like

↓

Wait API

↓

Update UI
```

React can do:

```text
Click Like

↓

Update UI Immediately

↓

Call API

↓

Rollback if API fails
```

Example:

```jsx
const [optimisticTodos, addOptimistic] =
    useOptimistic(todos);
```

Excellent for chat apps and social media.

---

## use()

One of the biggest additions.

Instead of:

```jsx
const data = await fetch(...);
```

React allows:

```jsx
const products = use(productsPromise);
```

Works with Suspense.

---

## Document Metadata

Instead of:

```jsx
import Helmet from "react-helmet";
```

React supports:

```jsx
<title>Products</title>

<meta name="description" />
```

No external library needed in supported environments.

---

## Better Server Components

React 19 improves React Server Components (commonly used with frameworks like Next.js).

---

# Features You Should Know for Interviews

| Feature                         | React Version | Why Important              |
| ------------------------------- | ------------- | -------------------------- |
| Hooks (`useState`, `useEffect`) | 16.8          | Core React development     |
| Context API                     | 16.3          | State sharing              |
| Fragments                       | 16            | Cleaner DOM                |
| Error Boundaries                | 16            | Error handling             |
| Fiber                           | 16            | Rendering architecture     |
| Concurrent Rendering            | 18            | Performance                |
| Automatic Batching              | 18            | Fewer renders              |
| `createRoot()`                  | 18            | New app initialization     |
| `useTransition()`               | 18            | Responsive UI              |
| `useDeferredValue()`            | 18            | Optimize expensive updates |
| `useOptimistic()`               | 19            | Optimistic UI updates      |
| `useActionState()`              | 19            | Async form handling        |
| `use()`                         | 19            | Async data with Suspense   |

---

# Interview Questions

### Q1. What is Concurrent Rendering?

It allows React to interrupt long rendering tasks, prioritize urgent updates (like typing), and resume background work later, making the UI more responsive.

---

### Q2. What is Automatic Batching?

React groups multiple state updates into a single render to improve performance. In React 18, batching works across more asynchronous scenarios than before.

---

### Q3. Difference between `useTransition()` and `useDeferredValue()`?

| `useTransition()`                     | `useDeferredValue()`                                |
| ------------------------------------- | --------------------------------------------------- |
| Marks state updates as low priority   | Defers the use of a value                           |
| You control when updates are deferred | React defers updates automatically based on a value |
| Useful for navigation and filtering   | Useful for expensive rendering based on user input  |

---

### Q4. What is `useOptimistic()`?

It updates the UI immediately before the server confirms the operation. If the server request fails, the optimistic update can be reverted.

---

### Q5. What is React Fiber?

Fiber is React's reconciliation engine introduced in React 16. It enables interruptible rendering, task prioritization, and is the foundation for features like Concurrent Rendering.

For a **5–6 year React developer**, interviewers will expect you to be comfortable with **Hooks, Context API, React 18 features (Concurrent Rendering, Automatic Batching, `useTransition`), and newer React 19 hooks like `useOptimistic` and `useActionState`**, while also understanding when and why to use them in real applications.
