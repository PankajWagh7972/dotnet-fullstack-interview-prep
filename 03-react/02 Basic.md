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

Excellent. Here's the **next batch (141–180)** with **interview questions, explanations, and code snippets**. These focus on **advanced React concepts**, exactly what companies ask for **5–8 years experienced React developers**.

---

# 141. What is Lifting State Up?

When multiple components need the same data, move the state to their nearest common parent.

### Before

```text
Parent
 ├── ChildA (count)
 └── ChildB (count)
```

Both have separate state.

### After

```text
Parent (count)
 ├── ChildA
 └── ChildB
```

### Code

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <ChildA count={count} />
      <ChildB count={count} />
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </>
  );
}
```

---

# 142. What is Prop Drilling?

Passing props through multiple components unnecessarily.

```text
App
 │
 ▼
Layout
 │
 ▼
Dashboard
 │
 ▼
UserCard
```

Instead use Context.

---

# 143. How does Context API work?

### Create Context

```jsx
const UserContext = createContext();
```

### Provider

```jsx
<UserContext.Provider value={user}>
    <App />
</UserContext.Provider>
```

### Consumer

```jsx
const user = useContext(UserContext);
```

---

# 144. Context vs Redux

| Context        | Redux Toolkit          |
| -------------- | ---------------------- |
| Theme          | Large applications     |
| Language       | Complex business logic |
| Logged-in user | API caching            |
| Simple state   | Debugging & middleware |

---

# 145. Why isn't Context a replacement for Redux?

Because Context re-renders all consuming components when its value changes, while Redux allows components to subscribe only to the specific slices of state they need.

---

# 146. What is Redux?

A predictable state management library.

```text
Component

↓

Dispatch

↓

Reducer

↓

Store

↓

UI Update
```

---

# 147. Redux Flow

```text
User Click

↓

dispatch()

↓

Action

↓

Reducer

↓

Store Updated

↓

React Re-render
```

---

# 148. Redux Toolkit Example

```jsx
const counterSlice = createSlice({

    name: "counter",

    initialState: 0,

    reducers: {

        increment: state => state + 1,

        decrement: state => state - 1

    }

});
```

---

# 149. useSelector

```jsx
const count = useSelector(state => state.counter);
```

Reads data.

---

# 150. useDispatch

```jsx
const dispatch = useDispatch();

dispatch(increment());
```

Updates store.

---

# 151. What is createAsyncThunk?

Simplifies async Redux actions.

```jsx
export const getUsers = createAsyncThunk(

    "users/get",

    async () => {

        return await api.getUsers();

    }

);
```

---

# 152. Extra Reducers

```jsx
extraReducers: builder => {

builder.addCase(

getUsers.fulfilled,

(state,action)=>{

state.users = action.payload;

});

}
```

---

# 153. What problem does Redux Toolkit solve?

* Boilerplate
* Immutable updates
* Async handling
* Cleaner reducers

---

# 154. Explain Immutability

Wrong

```jsx
state.user.name = "John";
```

Correct

```jsx
return {

...state,

user:{

...state.user,

name:"John"

}

};
```

Redux Toolkit uses Immer internally, allowing mutation-like syntax while producing immutable updates.

---

# 155. What is React Query / TanStack Query?

A library for managing server state.

Instead of

```jsx
useEffect()

fetch()

loading

error
```

Use

```jsx
useQuery()
```

---

# 156. Example

```jsx
const { data, isLoading } = useQuery({

    queryKey: ["users"],

    queryFn: getUsers

});
```

---

# 157. Benefits

* Caching
* Retry
* Background refresh
* Pagination
* Infinite scroll
* Deduplication

---

# 158. useMutation

```jsx
const mutation = useMutation({

mutationFn: createUser

});
```

Used for

* POST
* PUT
* DELETE

---

# 159. Optimistic Update

```jsx
mutation.mutate(user,{
onSuccess(){

},
onError(){

}
});
```

Update UI before API completes.

---

# 160. Why React Query instead of Redux?

Redux manages client state.

React Query manages server state.

---

# 161. What is Suspense?

Displays fallback while loading.

```jsx
<Suspense

fallback={<Spinner/>}>

<Home/>

</Suspense>
```

---

# 162. What is React.lazy?

```jsx
const Dashboard = React.lazy(

()=>import("./Dashboard")

);
```

Loads component only when needed.

---

# 163. Difference

| Lazy            | Suspense        |
| --------------- | --------------- |
| Loads component | Displays loader |

---

# 164. Error Boundary

```jsx
class ErrorBoundary extends React.Component{

componentDidCatch(error){

console.log(error);

}

}
```

---

# 165. What errors can't Error Boundaries catch?

* Event handler errors
* Async code (e.g., `setTimeout`)
* Server-side rendering errors
* Errors inside the error boundary itself

---

# 166. What is ForwardRef?

```jsx
const Input = forwardRef(

(props,ref)=>

<input ref={ref}/>

);
```

---

# 167. Parent Access

```jsx
const ref = useRef();

<Input ref={ref}/>
```

---

# 168. useImperativeHandle

```jsx
useImperativeHandle(

ref,

()=>({

focus(){

inputRef.current.focus();

}

})

);
```

---

# 169. What is Portal?

```jsx
createPortal(

<Modal/>,

document.body

);
```

Useful for

* Modals
* Popups
* Toasts

---

# 170. Why Portal?

Avoids CSS issues like:

* overflow:hidden
* z-index conflicts

---

# 171. What is Hydration?

```text
Server HTML

↓

Browser

↓

React attaches events
```

---

# 172. CSR vs SSR

| CSR                   | SSR                |
| --------------------- | ------------------ |
| Browser renders       | Server renders     |
| Slower first load     | Faster first paint |
| Better for dashboards | Better for SEO     |

---

# 173. What is SSG?

Pages are generated at build time.

Very fast.

---

# 174. When SSR?

* SEO
* Marketing pages
* Blogs

---

# 175. When CSR?

* Admin panel
* Dashboard
* Internal applications

---

# 176. What is Code Splitting?

Instead of

```text
2 MB JS
```

Split

```text
Home.js

Dashboard.js

Admin.js
```

---

# 177. Dynamic Import

```jsx
const page = await import("./Dashboard");
```

---

# 178. Tree Shaking

Unused code removed during build.

Example

```javascript
import { add } from "./math";
```

Only `add()` is included.

---

# 179. Why Bundle Size Matters?

Large bundle

↓

Slow download

↓

Slow parsing

↓

Poor performance

---

# 180. How do you optimize React performance?

**Answer**

* React.memo
* useMemo
* useCallback
* Lazy Loading
* Code Splitting
* Virtualization
* Debouncing
* Throttling
* Pagination
* Image optimization
* React Query caching
* Avoid unnecessary re-renders
* Split large components into smaller ones
* Profile with React DevTools

---

# ⭐ Senior Interview Scenarios

### Scenario 1

**Interviewer:** Your parent component renders every second. The child table has 10,000 rows. How do you optimize it?

**Answer:**

* Wrap the table with `React.memo`.
* Memoize row data with `useMemo`.
* Memoize callbacks with `useCallback`.
* Use virtualization (`react-window`).
* Fetch and cache data with TanStack Query if applicable.

---

### Scenario 2

**Interviewer:** API takes 5 seconds. Users click the button multiple times.

**Answer:**

* Disable the button while the request is in progress.
* Track loading state.
* Use request deduplication (e.g., TanStack Query where appropriate).
* Cancel stale requests with `AbortController` if needed.

```jsx
const [loading, setLoading] = useState(false);

const save = async () => {
    if (loading) return;

    setLoading(true);

    try {
        await api.save();
    } finally {
        setLoading(false);
    }
};
```

---

### Scenario 3

**Interviewer:** Search is lagging when typing.

**Answer:**

Use:

* Debouncing
* `useDeferredValue`
* `useTransition`
* Memoized filtering with `useMemo`

---

These are exactly the kinds of discussions interviewers use to assess senior React developers. In total, you now have **180 React interview questions**. The next batch (181–220) would cover **TypeScript with React, React Hook Form, Testing (Jest & React Testing Library), accessibility, WebSockets/SignalR, authentication flows, and advanced system design scenarios**, which are also common in senior React interviews.

Excellent. This batch (**181–220**) covers **TypeScript + React**, **React Hook Form**, **Authentication**, **Testing**, and **Real Enterprise Interview Questions**. These are frequently asked in companies like Microsoft, JPMorgan, Goldman Sachs, EPAM, BNY, Deloitte, Accenture, Infosys, and TCS Digital.

---

# React + TypeScript (181–195)

---

# 181. Why use TypeScript in React?

### Answer

Benefits:

* Type Safety
* Better IntelliSense
* Compile-time errors
* Easier refactoring
* Better documentation
* Reduces runtime bugs

---

# 182. Define Props Interface

```tsx
interface UserProps {
    name: string;
    age: number;
}

function User({ name, age }: UserProps) {
    return (
        <>
            <h2>{name}</h2>
            <p>{age}</p>
        </>
    );
}
```

---

# 183. Optional Props

```tsx
interface ButtonProps {
    title: string;
    disabled?: boolean;
}
```

Usage

```tsx
<Button title="Save" />
```

---

# 184. Typing useState

```tsx
const [count, setCount] = useState<number>(0);
```

Object

```tsx
interface User {
    id: number;
    name: string;
}

const [user, setUser] = useState<User | null>(null);
```

---

# 185. Typing useRef

```tsx
const inputRef = useRef<HTMLInputElement>(null);
```

Focus

```tsx
inputRef.current?.focus();
```

---

# 186. Typing Events

```tsx
const handleChange = (
    e: React.ChangeEvent<HTMLInputElement>
) => {
    console.log(e.target.value);
};
```

---

# 187. Typing API Response

```tsx
interface Product {
    id: number;
    name: string;
}

const [products, setProducts] =
    useState<Product[]>([]);
```

---

# 188. Generic Component

```tsx
interface TableProps<T> {
    data: T[];
}

function Table<T>({ data }: TableProps<T>) {
    return <div>{data.length}</div>;
}
```

---

# 189. Typing Children

```tsx
interface LayoutProps {
    children: React.ReactNode;
}
```

---

# 190. Union Types

```tsx
type Status =
    | "Loading"
    | "Success"
    | "Failed";
```

---

# 191. Enum

```tsx
enum UserRole {
    Admin,
    User,
    Manager
}
```

---

# 192. Record Type

```tsx
const permissions:
Record<string, boolean> = {

Admin: true,

User: false

};
```

---

# 193. Partial

```tsx
interface User{

name:string;

age:number;

}
```

```tsx
Partial<User>
```

Makes

all properties optional.

---

# 194. Pick

```tsx
type UserSummary =

Pick<User,"name"|"age">;
```

---

# 195. Omit

```tsx
type PublicUser =

Omit<User,"password">;
```

---

# React Hook Form (196–205)

---

# 196. Why React Hook Form?

Benefits

* Better performance
* Less re-rendering
* Built-in validation
* Easy integration

---

# 197. Basic Example

```tsx
const {

register,

handleSubmit

}=useForm();

return(

<form onSubmit={handleSubmit(console.log)}>

<input

{...register("name")}

/>

<button>

Save

</button>

</form>

);
```

---

# 198. Required Validation

```tsx
<input

{...register(

"email",

{

required:true

}

)}

/>
```

---

# 199. Pattern Validation

```tsx
register("email",{

pattern:

/^\S+@\S+$/

});
```

---

# 200. Display Error

```tsx
{

errors.email &&

<p>

Invalid Email

</p>

}
```

---

# 201. Default Values

```tsx
const{

register

}=useForm({

defaultValues:{

name:"Pankaj"

}

});
```

---

# 202. Reset Form

```tsx
reset();
```

---

# 203. Controller Example

Material UI

```tsx
<Controller

name="city"

control={control}

render={({field})=>

<TextField

{...field}

/>

}

/>
```

---

# 204. Watch Field

```tsx
const password=

watch("password");
```

---

# 205. Trigger Validation

```tsx
await trigger();
```

---

# Authentication (206–212)

---

# 206. JWT Flow

```text
Login

↓

API

↓

Access Token

↓

React

↓

Authorization Header

↓

.NET API
```

---

# 207. Axios Authorization

```tsx
axios.interceptors.request.use(

config=>{

config.headers.Authorization=

`Bearer ${token}`;

return config;

});
```

---

# 208. Refresh Token

Flow

```text
Access Token Expired

↓

401

↓

Refresh Token

↓

New Token

↓

Retry Request
```

---

# 209. Protected Route

```tsx
function ProtectedRoute({

children

}){

return token

? children

: <Navigate to="/login"/>;

}
```

---

# 210. Role-Based Authorization

```tsx
if(user.role!=="Admin"){

return <AccessDenied/>;

}
```

---

# 211. Logout

```tsx
localStorage.removeItem(

"token"

);

navigate("/login");
```

---

# 212. Auto Logout

```tsx
setTimeout(()=>{

logout();

},tokenExpiry);
```

---

# Testing (213–220)

---

# 213. What is Jest?

JavaScript testing framework.

---

# 214. React Testing Library

Focuses on

testing

user behavior

instead of

implementation details.

---

# 215. Render Test

```tsx
render(<Login/>);

expect(

screen.getByText("Login")

).toBeInTheDocument();
```

---

# 216. Button Click

```tsx
fireEvent.click(

screen.getByText("Save")

);
```

---

# 217. User Event

```tsx
await userEvent.type(

input,

"Pankaj"

);
```

---

# 218. Mock API

```tsx
jest.mock(

"./api"

);
```

---

# 219. Wait For Async

```tsx
await waitFor(()=>{

expect(

screen.getByText(

"Users"

)

).toBeInTheDocument();

});
```

---

# 220. Test Form

```tsx
userEvent.type(

input,

"John"

);

userEvent.click(

button

);

expect(

submit

).toHaveBeenCalled();
```

---

# ⭐ Senior Scenario Questions (Very Common)

## Q221. How would you implement login in a React + .NET application?

**Expected Answer**

Architecture:

```text
React Login Page
        │
        ▼
Axios POST /login
        │
        ▼
.NET Web API
        │
        ▼
JWT + Refresh Token
        │
        ▼
React Stores Token
        │
        ▼
Axios Interceptor Adds Authorization Header
        │
        ▼
Protected APIs
```

Mention:

* Access Token
* Refresh Token
* Protected Routes
* Role-based Authorization
* Auto Logout
* Axios Interceptors

---

## Q222. How do you handle API errors globally?

```tsx
axios.interceptors.response.use(
    response => response,
    error => {
        if (error.response?.status === 401) {
            // redirect to login or refresh token
        }

        return Promise.reject(error);
    }
);
```

---

## Q223. How do you organize a large React application?

```text
src/
│
├── components/
├── pages/
├── layouts/
├── hooks/
├── services/
├── api/
├── redux/
├── context/
├── routes/
├── types/
├── utils/
├── assets/
└── tests/
```

---

## Q224. What would you improve in an enterprise React application?

* Use TypeScript
* Redux Toolkit for global state
* TanStack Query for server state
* React Hook Form for forms
* Lazy loading and code splitting
* Error Boundaries
* Centralized API service
* Reusable UI components
* Unit and integration tests
* ESLint + Prettier
* Environment-specific configuration
* Performance profiling

---

This completes **Batch 181–220**. The next batch (**221–260**) can focus on **React System Design, advanced performance optimization, SignalR/WebSockets, micro-frontends, accessibility (WCAG), React internals, and real interview scenarios**, which are especially valuable for senior full-stack interviews.
