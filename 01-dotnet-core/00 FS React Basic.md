# React Lifecycle, Hooks, Working & Optimisation

*Comprehensive Guide for .NET Full‑Stack Developers*

This document covers React's internal working, all built‑in hooks with practical examples, lifecycle equivalents, and performance optimisation techniques essential for building efficient front‑ends that integrate with .NET back‑ends.

---

## Table of Contents

1. [How React Works Under the Hood](#how-react-works-under-the-hood)
2. [React Lifecycle (Class Components vs Hooks)](#react-lifecycle-class-components-vs-hooks)
3. [All Built‑in Hooks Explained with Examples](#all-built-in-hooks-explained-with-examples)
4. [Custom Hooks](#custom-hooks)
5. [Performance Optimisation Techniques](#performance-optimisation-techniques)
6. [React with ASP.NET Core Integration Tips](#react-with-aspnet-core-integration-tips)

---

## How React Works Under the Hood

### 1. The Virtual DOM and Reconciliation

**Answer:**  
React maintains a lightweight in‑memory representation of the real DOM called the **Virtual DOM (VDOM)** . When state or props change, React creates a new VDOM tree and compares it with the previous one using a **diffing algorithm**. This process is called **Reconciliation**.

**Steps:**
1. **Render phase:** React builds a new VDOM tree (can be interrupted).
2. **Commit phase:** React applies the minimal necessary changes to the real DOM (cannot be interrupted).

**Key optimisations in diffing:**
- If element types differ, React destroys the old tree and builds a new one.
- `key` prop helps identify which child items have changed.

**Example:**
```jsx
// Efficient update because React knows these are different components
<div> → <span>   // Recreates entire subtree

// Efficient list update using keys
{items.map(item => <Item key={item.id} />)}
```

---

### 2. React Fiber Architecture

**Answer:**  
**Fiber** is the re‑implementation of React's core algorithm introduced in React 16. It enables:
- **Incremental rendering** – splitting work into chunks and spreading it over multiple frames.
- **Prioritisation** – high‑priority updates (user input) can interrupt low‑priority ones (data fetching).
- **Concurrency** – features like `Suspense` and `useTransition`.

**Fiber Node:** A JavaScript object representing a unit of work. React builds a **fiber tree** that mirrors the component tree.

---

## React Lifecycle (Class Components vs Hooks)

### 3. Lifecycle Methods in Class Components

| Phase             | Methods                                          |
|-------------------|--------------------------------------------------|
| **Mounting**      | `constructor`, `getDerivedStateFromProps`, `render`, `componentDidMount` |
| **Updating**      | `getDerivedStateFromProps`, `shouldComponentUpdate`, `render`, `getSnapshotBeforeUpdate`, `componentDidUpdate` |
| **Unmounting**    | `componentWillUnmount`                           |
| **Error Handling**| `componentDidCatch`, `getDerivedStateFromError`  |

**Example:**
```jsx
class ProductDetail extends React.Component {
  componentDidMount() {
    this.fetchProduct(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchProduct(this.props.id);
    }
  }
  componentWillUnmount() {
    this.cancelRequest();
  }
}
```

---

### 4. Lifecycle Equivalents with Hooks

| Class Method                     | Hook Equivalent                                           |
|----------------------------------|-----------------------------------------------------------|
| `constructor`                    | Function body / `useState` initialisation                 |
| `componentDidMount`              | `useEffect(() => {...}, [])`                              |
| `componentDidUpdate`             | `useEffect(() => {...}, [dependencies])`                  |
| `componentWillUnmount`           | `useEffect` cleanup function `return () => {...}`         |
| `shouldComponentUpdate`          | `React.memo` + `useMemo` / `useCallback`                  |
| `getDerivedStateFromProps`       | State update directly during render (or `useMemo`)        |
| `getSnapshotBeforeUpdate`        | `useLayoutEffect` (rare)                                  |
| Error boundaries                 | `componentDidCatch` (no hook equivalent; use class)       |

**Example – Lifecycle using `useEffect`:**
```jsx
function ProductDetail({ id }) {
  useEffect(() => {
    // Mount + update when id changes
    const controller = new AbortController();
    fetchProduct(id, controller.signal);

    // Cleanup (unmount or before next effect)
    return () => controller.abort();
  }, [id]); // dependency array controls updates

  return <div>...</div>;
}
```

---

## All Built‑in Hooks Explained with Examples

### 5. `useState` – State Management

**Purpose:** Add state to functional components.

```jsx
const [state, setState] = useState(initialValue);
// Functional update when new state depends on previous
setState(prev => prev + 1);
```

**Example:**
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

### 6. `useEffect` – Side Effects

**Purpose:** Perform side effects (data fetching, subscriptions, DOM manipulation).

```jsx
useEffect(() => {
  // Effect code
  return () => { /* cleanup */ };
}, [dependencies]);
```

**Empty `[]`** → runs once after mount.  
**No array** → runs after every render.  
**Array with values** → runs when any value changes.

**Example – Fetch with cleanup:**
```jsx
useEffect(() => {
  let ignore = false;
  async function fetchData() {
    const result = await api.getData();
    if (!ignore) setData(result);
  }
  fetchData();
  return () => { ignore = true; };
}, [query]);
```

---

### 7. `useContext` – Consume Context

**Purpose:** Access context values without prop drilling.

```jsx
const ThemeContext = React.createContext('light');

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

**Full example with provider:**
```jsx
const AuthContext = createContext(null);
function App() {
  const [user, setUser] = useState(null);
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      <Dashboard />
    </AuthContext.Provider>
  );
}
function Dashboard() {
  const { user } = useContext(AuthContext);
  return <div>Welcome {user?.name}</div>;
}
```

---

### 8. `useReducer` – Complex State Logic

**Purpose:** Manage state with a reducer function (similar to Redux). Best for complex state transitions.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

**Example:**
```jsx
const initialState = { count: 0 };
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    case 'reset': return { count: action.payload };
    default: return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset', payload: 0 })}>Reset</button>
    </>
  );
}
```

---

### 9. `useCallback` – Memoize Functions

**Purpose:** Return a memoized version of a callback function that only changes if dependencies change. Prevents unnecessary re‑creation of functions.

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**When to use:**
- Passing callbacks to optimised child components that rely on reference equality (`React.memo`).
- Callbacks used in `useEffect` dependencies.

**Example:**
```jsx
const Parent = () => {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // This function is recreated only when `count` changes
  const handleClick = useCallback(() => {
    console.log('Count is:', count);
  }, [count]);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <input value={text} onChange={e => setText(e.target.value)} />
      <MemoizedChild onClick={handleClick} />
    </>
  );
};

const Child = ({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Log Count</button>;
};
const MemoizedChild = React.memo(Child);
```

---

### 10. `useMemo` – Memoize Expensive Calculations

**Purpose:** Returns a memoized value. Re‑computes only when dependencies change.

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**Example:**
```jsx
function ProductList({ products, filterTerm }) {
  const filteredProducts = useMemo(() => {
    console.log('Filtering...');
    return products.filter(p => 
      p.name.toLowerCase().includes(filterTerm.toLowerCase())
    );
  }, [products, filterTerm]);

  return (
    <ul>
      {filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

**Note:** Do not overuse `useMemo` for trivial calculations – the overhead of memoisation may outweigh the benefit.

---

### 11. `useRef` – Mutable Reference

**Purpose:** Hold a mutable value that persists across renders **without** causing re‑renders when changed. Also used to reference DOM elements.

```jsx
const refContainer = useRef(initialValue);
```

**Use cases:**
- Access DOM nodes (`ref={inputRef}`).
- Store previous values.
- Hold interval/timeout IDs.
- Store values that shouldn't trigger re‑renders.

**Example – DOM focus:**
```jsx
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => inputEl.current.focus();
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus</button>
    </>
  );
}
```

**Example – Tracking previous props:**
```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; });
  return ref.current;
}
```

---

### 12. `useImperativeHandle` – Customise Instance Value

**Purpose:** Customise the instance value exposed when using `ref` with `forwardRef`. Rarely needed.

```jsx
useImperativeHandle(ref, () => ({
  customMethod: () => { ... }
}), [dependencies]);
```

**Example – Child component exposing method:**
```jsx
const FancyInput = React.forwardRef((props, ref) => {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => inputRef.current.value = ''
  }));
  return <input ref={inputRef} {...props} />;
});

// Parent usage:
const parentRef = useRef();
<FancyInput ref={parentRef} />
parentRef.current.focus();
```

---

### 13. `useLayoutEffect` – Synchronous Effect

**Purpose:** Similar to `useEffect` but fires **synchronously after all DOM mutations** and before browser paint. Use for DOM measurements or visual updates that must happen before paint.

```jsx
useLayoutEffect(() => {
  // DOM measurements
  const rect = element.getBoundingClientRect();
  // Adjust state before paint
}, [dependencies]);
```

**When to use:** Measuring layout, synchronously updating DOM to avoid flicker.

**Note:** `useEffect` is preferred for most cases as it doesn't block painting.

---

### 14. `useDebugValue` – Label Custom Hooks in DevTools

**Purpose:** Display a label for custom hooks in React DevTools.

```jsx
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  useDebugValue(isOnline ? 'Online' : 'Offline');
  return isOnline;
}
```

---

### 15. `useId` – Generate Unique IDs (React 18+)

**Purpose:** Generate stable, unique IDs for accessibility attributes.

```jsx
const id = useId();
return (
  <>
    <label htmlFor={id}>Name</label>
    <input id={id} type="text" />
  </>
);
```

---

### 16. `useTransition` – Mark Non‑Urgent Updates (React 18+)

**Purpose:** Mark state updates as non‑blocking transitions to keep UI responsive.

```jsx
const [isPending, startTransition] = useTransition();

const handleChange = (e) => {
  setInputValue(e.target.value); // urgent update
  startTransition(() => {
    setFilteredResults(heavyFilter(e.target.value)); // non-urgent
  });
};

return <div>{isPending && 'Loading...'}</div>;
```

---

### 17. `useDeferredValue` – Defer Re‑rendering (React 18+)

**Purpose:** Accept a value and return a new copy that will lag behind urgent updates.

```jsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => heavyFilter(deferredQuery), [deferredQuery]);
```

---

## Custom Hooks

### 18. Creating Reusable Logic with Custom Hooks

**Purpose:** Extract component logic into reusable functions.

**Naming convention:** Must start with `use`.

**Example – `useFetch` custom hook:**
```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);
    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') setError(err);
      })
      .finally(() => setLoading(false));
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage:
function ProductList() {
  const { data, loading, error } = useFetch('/api/products');
  if (loading) return <Spinner />;
  if (error) return <Error />;
  return <div>{data.map(...)}</div>;
}
```

**Example – `useLocalStorage` hook:**
```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.log(error);
    }
  };

  return [storedValue, setValue];
}
```

---

## Performance Optimisation Techniques

### 19. `React.memo` – Prevent Unnecessary Re‑renders

**Purpose:** Memoize a component; only re‑render if props change (shallow comparison).

```jsx
const MyComponent = React.memo(function MyComponent(props) {
  /* only re-renders if props change */
});
```

**Custom comparison function:**
```jsx
React.memo(Component, (prevProps, nextProps) => {
  return prevProps.id === nextProps.id; // return true to skip re-render
});
```

**When to use:** Components that render often with the same props.

---

### 20. `useCallback` and `useMemo` for Stable References

**Summary:**
- `useCallback`: memoize functions → stable callback references.
- `useMemo`: memoize computed values → avoid expensive recalculations.

**Common mistake:** Passing inline functions to memoized children without `useCallback` defeats memoization.

```jsx
// ❌ Bad: new function every render, child re-renders
<MemoChild onClick={() => doSomething(id)} />

// ✅ Good: stable function reference
const handleClick = useCallback(() => doSomething(id), [id]);
<MemoChild onClick={handleClick} />
```

---

### 21. Code Splitting with `React.lazy` and `Suspense`

**Purpose:** Split bundle into smaller chunks, loaded on demand.

```jsx
import React, { Suspense } from 'react';
const ProductPage = React.lazy(() => import('./ProductPage'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <ProductPage />
    </Suspense>
  );
}
```

**Route‑based splitting:**
```jsx
const Home = lazy(() => import('./Home'));
const Products = lazy(() => import('./Products'));

<Routes>
  <Route path="/" element={<Suspense fallback={<Spinner />}><Home /></Suspense>} />
  <Route path="/products" element={<Suspense fallback={<Spinner />}><Products /></Suspense>} />
</Routes>
```

---

### 22. Virtualising Long Lists with `react‑window`

**Purpose:** Render only visible items in large lists to improve performance.

```jsx
import { FixedSizeList as List } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style}>Row {index}</div>
);

function VirtualizedList() {
  return (
    <List height={400} itemCount={1000} itemSize={35} width={300}>
      {Row}
    </List>
  );
}
```

---

### 23. Avoid Inline Functions and Objects in JSX

**Problem:** Inline objects/functions create new references on every render, causing unnecessary re‑renders or breaking memoization.

```jsx
// ❌ Bad: new object every render
<Component style={{ margin: 10 }} />

// ✅ Good: define outside
const style = { margin: 10 };
<Component style={style} />

// ❌ Bad: new function every render
<input onChange={e => setValue(e.target.value)} />

// ✅ Good: use useCallback if needed, or just accept it for simple handlers
const handleChange = useCallback(e => setValue(e.target.value), []);
<input onChange={handleChange} />
```

---

### 24. Use Production Build and DevTools Profiler

- Always test performance with **production build** (`npm run build`).
- Use **React DevTools Profiler** to identify slow components.
- Check for unnecessary re‑renders with "Highlight updates" option.

---

### 25. Debouncing and Throttling Expensive Operations

**Debounce example for search input:**
```jsx
import { useMemo } from 'react';
import { debounce } from 'lodash';

function SearchInput() {
  const handleSearch = useMemo(
    () => debounce((query) => fetchResults(query), 300),
    []
  );

  return <input onChange={e => handleSearch(e.target.value)} />;
}
```

---

## React with ASP.NET Core Integration Tips

### 26. Environment‑Specific API URLs

**In React (Vite/Create React App):**
```jsx
const API_BASE = import.meta.env.VITE_API_URL || 'https://localhost:5001';
// or process.env.REACT_APP_API_URL
```

**In ASP.NET Core:** Configure CORS to allow the React dev server origin.

---

### 27. Authentication Flow with JWT

**React side:**
- Store token in `localStorage` (or better, HTTP‑only cookie with BFF pattern).
- Include token in `Authorization` header.

```jsx
const fetchWithAuth = (url, options = {}) => {
  const token = localStorage.getItem('token');
  return fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
      ...options.headers
    }
  });
};
```

**Better approach (Backend‑For‑Frontend):** Use cookie authentication with `SameSite=Strict` and CSRF protection.

---

### 28. Proxy API Requests in Development

**In `vite.config.js` (Vite):**
```js
export default {
  server: {
    proxy: {
      '/api': 'https://localhost:5001'
    }
  }
};
```

**In Create React App (`package.json`):**
```json
"proxy": "https://localhost:5001"
```

This avoids CORS issues during local development.

---

*These notes provide a comprehensive understanding of React's inner workings, hooks, and performance patterns essential for building scalable front‑ends integrated with .NET back‑ends.*
