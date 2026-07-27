Absolutely. Below are **40 additional senior React interview questions (101–140)** with **answers and code snippets**. These are the types of questions frequently asked in product companies and MNCs.

---

# 101. Why shouldn't we mutate state directly?

### Wrong

```jsx
const [user, setUser] = useState({
    name: "Pankaj",
    age: 28
});

user.age = 29;
```

React doesn't know the object changed.

### Correct

```jsx
setUser(prev => ({
    ...prev,
    age: 29
}));
```

**Interview Answer**

React detects state changes by comparing references. Mutating the same object keeps the reference unchanged, so React may not re-render.

---

# 102. Why use Functional Updates?

```jsx
setCount(count + 1);
setCount(count + 1);
```

Result:

```
1
```

Correct:

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

Result:

```
2
```

**Why?**

Each update receives the latest state.

---

# 103. Why are keys important?

Wrong

```jsx
users.map(user =>
    <User user={user} />
)
```

Correct

```jsx
users.map(user =>
    <User key={user.id} user={user} />
)
```

React uses keys to identify items during reconciliation.

---

# 104. Why shouldn't index be key?

```jsx
users.map((user,index)=>

<User key={index}/>

)
```

If list order changes:

```text
A
B
C
```

↓

Delete A

↓

```text
B
C
```

React may reuse the wrong component.

Use

```jsx
key={user.id}
```

---

# 105. Explain React Reconciliation

```
Old Virtual DOM

↓

New Virtual DOM

↓

Compare

↓

Only update differences

↓

Real DOM
```

---

# 106. Why use React.memo?

```jsx
const Child = React.memo(function Child({name}){

    console.log("Render");

    return <h1>{name}</h1>;

});
```

Parent renders

↓

Child won't render

if props remain same.

---

# 107. Why React.memo sometimes fails?

```jsx
<Child
    user={{
        name:"Pankaj"
    }}
/>
```

Every render creates

new object.

Use

```jsx
const user=useMemo(()=>({

name:"Pankaj"

}),[]);
```

---

# 108. Difference between useMemo & React.memo

### useMemo

Memoizes

```
Value
```

### React.memo

Memoizes

```
Component
```

---

# 109. What is stale closure?

```jsx
useEffect(()=>{

setInterval(()=>{

console.log(count);

},1000);

},[]);
```

Always prints

```
0
```

Fix

```jsx
useEffect(()=>{

const id=setInterval(()=>{

setCount(prev=>prev+1);

},1000);

return ()=>clearInterval(id);

},[]);
```

---

# 110. Infinite useEffect loop

Wrong

```jsx
useEffect(()=>{

setCount(count+1);

});
```

Every render

↓

setCount

↓

Render

↓

Infinite loop

---

# 111. Correct dependency

```jsx
useEffect(()=>{

loadUsers();

},[]);
```

Runs once.

---

# 112. useEffect with dependency

```jsx
useEffect(()=>{

loadProduct(id);

},[id]);
```

Runs only when

```
id
```

changes.

---

# 113. Cleanup Example

```jsx
useEffect(()=>{

window.addEventListener("resize",resize);

return ()=>{

window.removeEventListener("resize",resize);

}

},[]);
```

---

# 114. Prevent Memory Leak

```jsx
useEffect(()=>{

const controller=new AbortController();

fetch(url,{
signal:controller.signal
});

return ()=>controller.abort();

},[]);
```

---

# 115. Controlled Input

```jsx
const[name,setName]=useState("");

<input

value={name}

onChange={(e)=>setName(e.target.value)}

/>
```

---

# 116. Uncontrolled Input

```jsx
const ref=useRef();

<input ref={ref}/>
```

Read

```jsx
ref.current.value
```

---

# 117. Debounce Search

```jsx
const [search,setSearch]=useState("");

useEffect(()=>{

const timer=setTimeout(()=>{

loadProducts(search);

},500);

return ()=>clearTimeout(timer);

},[search]);
```

---

# 118. useCallback Example

```jsx
const save=useCallback(()=>{

console.log("Save");

},[]);
```

Useful with

```
React.memo
```

---

# 119. Lazy Loading

```jsx
const Home=React.lazy(()=>import("./Home"));
```

```jsx
<Suspense fallback={<Loader/>}>

<Home/>

</Suspense>
```

---

# 120. Dynamic Import

```jsx
const module=await import("./math");
```

Loads

only when needed.

---

# 121. Error Boundary

```jsx
class ErrorBoundary extends React.Component{

componentDidCatch(error){

console.log(error);

}

}
```

Catches rendering errors.

---

# 122. Context Example

```jsx
const UserContext=createContext();
```

Provider

```jsx
<UserContext.Provider

value={user}>

<App/>

</UserContext.Provider>
```

Consume

```jsx
const user=useContext(UserContext);
```

---

# 123. Prop Drilling

```
App

↓

Parent

↓

Child

↓

GrandChild
```

Context solves this.

---

# 124. useReducer Example

```jsx
const reducer=(state,action)=>{

switch(action.type){

case"INC":

return{

count:state.count+1

};

}

};

const[state,dispatch]=useReducer(reducer,{

count:0

});
```

---

# 125. React Router Protected Route

```jsx
function ProtectedRoute({children}){

if(!isLoggedIn)

return <Navigate to="/login"/>;

return children;

}
```

---

# 126. Nested Route

```jsx
<Route path="/dashboard">

<Route index element={<Home/>}/>

<Route

path="users"

element={<Users/>}

/>

</Route>
```

---

# 127. useNavigate

```jsx
const navigate=useNavigate();

navigate("/dashboard");
```

---

# 128. useParams

```jsx
const{id}=useParams();
```

Route

```
/products/10
```

Returns

```
10
```

---

# 129. useLocation

```jsx
const location=useLocation();
```

Useful for

breadcrumbs

analytics

redirects.

---

# 130. Axios Interceptor

```jsx
axios.interceptors.request.use(config=>{

config.headers.Authorization=

`Bearer ${token}`;

return config;

});
```

---

# 131. Upload File

```jsx
const formData=new FormData();

formData.append("file",file);

await axios.post(

"/upload",

formData

);
```

---

# 132. Multiple API Calls

```jsx
const[users,products]=await Promise.all([

getUsers(),

getProducts()

]);
```

Runs concurrently.

---

# 133. Custom Hook

```jsx
function useCounter(){

const[count,setCount]=

useState(0);

return{

count,

increment:()=>setCount(c=>c+1)

};

}
```

---

# 134. Theme Context

```jsx
const ThemeContext=createContext();
```

```jsx
const theme=useContext(

ThemeContext

);
```

---

# 135. React StrictMode

```jsx
<React.StrictMode>

<App/>

</React.StrictMode>
```

Runs extra checks

Development only.

---

# 136. React Portal

```jsx
createPortal(

<Modal/>,

document.body

);
```

Useful for

* Modal
* Tooltip
* Popup

---

# 137. Forward Ref

```jsx
const Input=

forwardRef((props,ref)=>

<input

ref={ref}

/>

);
```

Parent

```jsx
inputRef.current.focus();
```

---

# 138. useImperativeHandle

```jsx
useImperativeHandle(ref,()=>({

focus(){

inputRef.current.focus();

}

}));
```

Expose methods to parent.

---

# 139. Optimistic UI

```jsx
setLikes(l=>l+1);

await api.like();

catch{

setLikes(l=>l-1);

}
```

User sees

instant update.

---

# 140. Search Optimization

```jsx
const deferredSearch=

useDeferredValue(search);
```

or

```jsx
const[isPending,startTransition]=

useTransition();

startTransition(()=>{

setProducts(filtered);

});
```

Keeps typing smooth.

---

# ⭐ Senior-Level Follow-up Questions (Very Common)

* Explain the React rendering lifecycle from state update to DOM update.
* Why does React compare object references instead of deep comparison?
* How does `React.memo` work internally?
* When would `useMemo` hurt performance?
* Explain stale closures with a real example.
* How would you implement an infinite scrolling list?
* How would you cache API responses?
* How would you prevent duplicate API requests?
* How would you build a reusable data table component?
* How would you structure a React application with 100+ screens?
* Explain CSR vs SSR vs SSG.
* How do you implement role-based authorization?
* How do you handle optimistic updates with rollback?
* How do you debug unnecessary re-renders?
* How would you migrate a JavaScript React project to TypeScript?

These questions are representative of what senior React developers are expected to discuss beyond basic hooks and components.
