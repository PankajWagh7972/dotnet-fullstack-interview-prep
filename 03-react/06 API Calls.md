In React, the two most common ways to call a Web API are:

1. **Fetch API** (built into the browser)
2. **Axios** (third-party library with more features)

---

# 1. Using Fetch API

## GET Request

```javascript
import { useEffect, useState } from "react";

function Users() {
    const [users, setUsers] = useState([]);

    useEffect(() => {
        fetch("https://jsonplaceholder.typicode.com/users")
            .then((response) => {
                if (!response.ok) {
                    throw new Error("Failed to fetch users");
                }
                return response.json();
            })
            .then((data) => setUsers(data))
            .catch((error) => console.error(error));
    }, []);

    return (
        <div>
            <h2>Users</h2>

            {users.map(user => (
                <p key={user.id}>{user.name}</p>
            ))}
        </div>
    );
}

export default Users;
```

---

## GET using async/await

```javascript
import { useEffect, useState } from "react";

function Users() {

    const [users, setUsers] = useState([]);

    useEffect(() => {
        getUsers();
    }, []);

    const getUsers = async () => {
        try {
            const response = await fetch(
                "https://jsonplaceholder.typicode.com/users"
            );

            if (!response.ok) {
                throw new Error("API Error");
            }

            const data = await response.json();

            setUsers(data);

        } catch (error) {
            console.error(error);
        }
    };

    return (
        <>
            {users.map(user =>
                <div key={user.id}>{user.name}</div>
            )}
        </>
    );
}
```

---

## POST Request

```javascript
const saveUser = async () => {

    const response = await fetch(
        "https://jsonplaceholder.typicode.com/users",
        {
            method: "POST",

            headers: {
                "Content-Type": "application/json"
            },

            body: JSON.stringify({
                name: "Pankaj",
                city: "Pune"
            })
        });

    const result = await response.json();

    console.log(result);
};
```

---

# 2. Using Axios

Install

```bash
npm install axios
```

---

## GET Request

```javascript
import axios from "axios";
import { useEffect, useState } from "react";

function Users() {

    const [users, setUsers] = useState([]);

    useEffect(() => {

        getUsers();

    }, []);

    const getUsers = async () => {

        try {

            const response = await axios.get(
                "https://jsonplaceholder.typicode.com/users"
            );

            setUsers(response.data);

        } catch (error) {

            console.log(error);

        }
    };

    return (
        <>
            {users.map(user =>
                <div key={user.id}>
                    {user.name}
                </div>
            )}
        </>
    );
}

export default Users;
```

---

## POST Request

```javascript
const saveUser = async () => {

    try {

        const response = await axios.post(
            "https://jsonplaceholder.typicode.com/users",
            {
                name: "Pankaj",
                city: "Pune"
            });

        console.log(response.data);

    } catch (error) {

        console.log(error);

    }
};
```

---

# Calling Your ASP.NET Core Web API

Suppose your API is

```text
GET https://localhost:5001/api/employees
```

Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeesController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new[]
        {
            new { Id = 1, Name = "Pankaj" },
            new { Id = 2, Name = "Rahul" }
        });
    }
}
```

React

```javascript
const getEmployees = async () => {

    const response = await axios.get(
        "https://localhost:5001/api/employees"
    );

    setEmployees(response.data);
};
```

---

# Creating a Reusable Axios Instance (Production)

```javascript
import axios from "axios";

const api = axios.create({

    baseURL: "https://localhost:5001/api",

    timeout: 10000,

    headers: {
        "Content-Type": "application/json"
    }
});

export default api;
```

Use it like this:

```javascript
import api from "./api";

const response = await api.get("/employees");
```

---

# Adding JWT Token

```javascript
const token = localStorage.getItem("token");

const response = await axios.get(
    "https://localhost:5001/api/employees",
    {
        headers: {
            Authorization: `Bearer ${token}`
        }
    });
```

Or configure it once:

```javascript
const api = axios.create({
    baseURL: "https://localhost:5001/api"
});

api.interceptors.request.use(config => {

    const token = localStorage.getItem("token");

    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
});
```

Now every request automatically includes the token.

---

# Fetch vs Axios

| Feature                       | Fetch                        | Axios                          |
| ----------------------------- | ---------------------------- | ------------------------------ |
| Built into browser            | ✅ Yes                        | ❌ No (install required)        |
| Automatic JSON parsing        | ❌ No (`response.json()`)     | ✅ Yes (`response.data`)        |
| HTTP error handling           | ❌ Must check `response.ok`   | ✅ Throws for non-2xx responses |
| Request/Response interceptors | ❌ No                         | ✅ Yes                          |
| Timeout support               | ❌ Requires `AbortController` | ✅ Built-in                     |
| Request cancellation          | ✅ `AbortController`          | ✅ Supported                    |
| Upload/download progress      | Limited                      | ✅ Easy                         |
| Default headers & base URL    | Manual                       | ✅ Easy                         |

---

# Which should you use?

* **Use Fetch** if you want to avoid extra dependencies and only need basic HTTP requests.
* **Use Axios** in most enterprise React applications because it provides:

  * Interceptors for JWT authentication
  * Global error handling
  * Automatic JSON transformation
  * Easier configuration with a base URL
  * Built-in timeout support
  * Cleaner API

### Interview Answer

> **Fetch** is the native browser API for making HTTP requests. It's lightweight and requires manually checking `response.ok` and parsing JSON with `response.json()`. **Axios** is a popular HTTP client library that automatically parses JSON, throws errors for unsuccessful responses, supports interceptors, request cancellation, timeouts, and global configuration. In enterprise React applications, Axios is commonly preferred because it simplifies authentication, logging, and centralized error handling.
