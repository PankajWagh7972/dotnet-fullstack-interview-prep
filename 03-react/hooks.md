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
