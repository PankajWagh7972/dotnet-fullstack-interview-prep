# SQL, Entity Framework Core & React Essentials

*For Basic to Intermediate .NET Full‑Stack Developers (1–4 Years Experience)*

This section provides focused, practical knowledge for working with SQL databases, Entity Framework Core, and building React front‑ends that communicate with .NET back‑ends.

---

## Table of Contents

1. [SQL Fundamentals for .NET Developers](#sql-fundamentals-for-net-developers)
2. [Entity Framework Core – Beyond the Basics](#entity-framework-core--beyond-the-basics)
3. [React for .NET Developers](#react-for-net-developers)

---

## SQL Fundamentals for .NET Developers

### 1. What are the essential SQL statements every developer should know?

**Answer:**  
You must be comfortable with `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and `JOIN`.

```sql
-- SELECT: retrieve data
SELECT Id, Name, Price FROM Products WHERE Price > 100;

-- INSERT: add new row
INSERT INTO Products (Name, Price, CategoryId) VALUES ('Keyboard', 49.99, 2);

-- UPDATE: modify existing row
UPDATE Products SET Price = 45.99 WHERE Id = 5;

-- DELETE: remove row
DELETE FROM Products WHERE Id = 10;

-- JOIN: combine tables
SELECT p.Name, c.Name AS CategoryName
FROM Products p
INNER JOIN Categories c ON p.CategoryId = c.Id;
```

---

### 2. Explain `INNER JOIN`, `LEFT JOIN`, and `RIGHT JOIN` with examples.

**Answer:**

- **INNER JOIN** – returns only rows where there is a match in **both** tables.
- **LEFT JOIN** – returns **all rows** from the left table, plus matched rows from the right (or NULL if no match).
- **RIGHT JOIN** – returns all rows from the right table, plus matched rows from the left.

```sql
-- Tables:
-- Categories: (1, 'Electronics'), (2, 'Books')
-- Products: (1, 'Laptop', 1), (2, 'Mouse', 1), (3, 'Notebook', NULL)

-- INNER JOIN: only products with a category
SELECT p.Name, c.Name FROM Products p
INNER JOIN Categories c ON p.CategoryId = c.Id;
-- Result: Laptop - Electronics, Mouse - Electronics

-- LEFT JOIN: all products, even those without category
SELECT p.Name, c.Name FROM Products p
LEFT JOIN Categories c ON p.CategoryId = c.Id;
-- Result: Laptop - Electronics, Mouse - Electronics, Notebook - NULL

-- RIGHT JOIN: all categories, even those without products
SELECT p.Name, c.Name FROM Products p
RIGHT JOIN Categories c ON p.CategoryId = c.Id;
-- Result: Laptop - Electronics, Mouse - Electronics, NULL - Books
```

---

### 3. How do you use `GROUP BY` and aggregate functions?

**Answer:**  
`GROUP BY` groups rows that have the same values in specified columns. You then apply aggregate functions like `COUNT`, `SUM`, `AVG`, `MAX`, `MIN` to each group.

```sql
-- Count products per category
SELECT CategoryId, COUNT(*) AS ProductCount
FROM Products
GROUP BY CategoryId;

-- Total sales per customer, only showing those with more than 1 order
SELECT CustomerId, SUM(TotalAmount) AS TotalSpent
FROM Orders
GROUP BY CustomerId
HAVING COUNT(Id) > 1;
```

---

### 4. What are indexes and how do they improve query performance?

**Answer:**  
An index is a database structure that speeds up data retrieval. It works like a book's index – instead of scanning every page, you jump directly to the relevant entries.

**Create an index:**
```sql
CREATE INDEX IX_Products_Name ON Products(Name);
```

**When to index:**
- Columns used frequently in `WHERE` clauses.
- Columns used in `JOIN` conditions.
- Columns used in `ORDER BY`.

**Composite index (multiple columns):**
```sql
CREATE INDEX IX_Orders_CustomerId_OrderDate ON Orders(CustomerId, OrderDate);
```

**Trade‑offs:**
- ✅ Faster `SELECT` queries.
- ❌ Slower `INSERT`, `UPDATE`, `DELETE` (indexes must be maintained).
- ❌ Consumes additional disk space.

---

### 5. What is a stored procedure and when would you use one?

**Answer:**  
A stored procedure is a pre‑compiled collection of SQL statements saved on the database server. It can accept parameters and return results.

**Creating a stored procedure:**
```sql
CREATE PROCEDURE GetProductsByCategory
    @CategoryId INT
AS
BEGIN
    SELECT Id, Name, Price
    FROM Products
    WHERE CategoryId = @CategoryId;
END;
```

**Executing it:**
```sql
EXEC GetProductsByCategory @CategoryId = 1;
```

**Advantages:**
- **Performance** – Pre‑compiled and cached execution plan.
- **Security** – Users can be granted permission to execute without direct table access.
- **Reusability** – Encapsulates complex logic.

**Calling from C# (ADO.NET):**
```csharp
using var cmd = new SqlCommand("GetProductsByCategory", connection);
cmd.CommandType = CommandType.StoredProcedure;
cmd.Parameters.AddWithValue("@CategoryId", 1);
```

---

### 6. Explain transactions and the ACID properties.

**Answer:**  
A **transaction** is a unit of work that must either complete entirely or have no effect at all. ACID ensures reliability:

- **Atomicity** – All operations succeed or none do.
- **Consistency** – Data integrity constraints are maintained.
- **Isolation** – Concurrent transactions do not interfere.
- **Durability** – Once committed, changes persist even after a failure.

**Example:**
```sql
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

-- If both succeed:
COMMIT;
-- If any fails:
ROLLBACK;
```

---

## Entity Framework Core – Beyond the Basics

### 7. How do you configure one‑to‑many and many‑to‑many relationships in EF Core?

**Answer:**

**One‑to‑Many:**
```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Product> Products { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; }
}
```
EF Core automatically configures the relationship via the `CategoryId` foreign key.

**Many‑to‑Many (EF Core 5+):**
```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Course> Courses { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    public ICollection<Student> Students { get; set; }
}
```
EF Core creates an automatic join table (`StudentCourse`). You can customise it using `OnModelCreating`:
```csharp
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students)
    .UsingEntity(j => j.ToTable("StudentCourses"));
```

---

### 8. What is the difference between `Add`, `Update`, and `Attach` methods?

| Method   | Behaviour                                                                               |
|----------|-----------------------------------------------------------------------------------------|
| `Add`    | Marks the entity as `Added` (new row to be inserted).                                    |
| `Update` | Marks the entire entity as `Modified` (all properties will be updated).                  |
| `Attach` | Attaches an existing entity to the context with `Unchanged` state. Useful for updates when you only want to modify specific properties. |

**Example – Updating only specific fields:**
```csharp
var product = new Product { Id = 1, Price = 29.99m }; // Only Id and Price set
dbContext.Products.Attach(product);
dbContext.Entry(product).Property(p => p.Price).IsModified = true;
await dbContext.SaveChangesAsync();
```

---

### 9. How do you execute raw SQL queries in EF Core?

**Answer:**  
Use `FromSqlRaw` or `FromSqlInterpolated` for querying, and `ExecuteSqlRaw` or `ExecuteSqlInterpolated` for non‑query commands.

**Query:**
```csharp
var products = await dbContext.Products
    .FromSqlInterpolated($"SELECT * FROM Products WHERE Price > {minPrice}")
    .ToListAsync();
```

**Non‑query (UPDATE/DELETE):**
```csharp
int rowsAffected = await dbContext.Database
    .ExecuteSqlInterpolatedAsync($"UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = {categoryId}");
```

**Stored procedure:**
```csharp
var products = await dbContext.Products
    .FromSqlInterpolated($"EXEC GetProductsByCategory @CategoryId = {catId}")
    .ToListAsync();
```

---

### 10. What are common performance pitfalls in EF Core and how to avoid them?

| Pitfall                                     | Solution                                                                 |
|---------------------------------------------|--------------------------------------------------------------------------|
| **N+1 query problem**                       | Use `.Include()` / `.ThenInclude()` for eager loading of related data.    |
| **Selecting all columns when only a few needed** | Use `.Select()` to project only required fields.                      |
| **Client‑side evaluation**                  | Keep queries as `IQueryable` until final execution. Avoid `AsEnumerable()` too early. |
| **Tracking overhead for read‑only queries** | Use `.AsNoTracking()`.                                                    |
| **Missing indexes**                         | Analyse slow queries and add appropriate database indexes.                |

**Example of N+1 problem and fix:**
```csharp
// BAD: N+1 queries – one for orders, then one per order to get customer
var orders = await dbContext.Orders.ToListAsync();
foreach (var order in orders)
    Console.WriteLine(order.Customer.Name);

// GOOD: Eager loading – single query with JOIN
var orders = await dbContext.Orders.Include(o => o.Customer).ToListAsync();
```

---

### 11. How do you seed initial data in EF Core?

**Answer:**  
Use the `HasData` method in `OnModelCreating`.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Category>().HasData(
        new Category { Id = 1, Name = "Electronics" },
        new Category { Id = 2, Name = "Books" }
    );

    modelBuilder.Entity<Product>().HasData(
        new Product { Id = 1, Name = "Laptop", Price = 999.99m, CategoryId = 1 },
        new Product { Id = 2, Name = "C# Programming", Price = 39.99m, CategoryId = 2 }
    );
}
```
The seed data is applied when you run `dotnet ef database update` or `dbContext.Database.Migrate()`.

---

## React for .NET Developers

### 12. What are components, props, and state in React?

**Answer:**

- **Component** – A reusable piece of UI (function or class) that returns JSX.
- **Props** – Read‑only data passed from parent to child component.
- **State** – Mutable data managed within a component that triggers re‑renders when changed.

**Example – Functional Component with props and state:**
```jsx
import React, { useState } from 'react';

function Counter({ initialCount }) {  // initialCount is a prop
  const [count, setCount] = useState(initialCount); // count is state

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Usage: <Counter initialCount={5} />
```

---

### 13. Explain the `useState` and `useEffect` hooks.

**Answer:**

**`useState`** – Adds state to a functional component.
```jsx
const [value, setValue] = useState(initialValue);
```

**`useEffect`** – Performs side effects (data fetching, subscriptions, manual DOM changes). It runs after render.
```jsx
useEffect(() => {
  // Code to run after render
  console.log('Component mounted or updated');

  // Optional cleanup function
  return () => console.log('Component will unmount');
}, [dependencies]); // Empty array [] = run once on mount
```

**Example – Fetch data when component mounts:**
```jsx
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('https://localhost:5001/api/products')
      .then(res => res.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      });
  }, []); // Empty dependency array = runs once

  if (loading) return <div>Loading...</div>;
  return <ul>{products.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

---

### 14. How do you handle forms and user input in React?

**Answer:**  
Use **controlled components** where form elements' values are bound to state.

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Logging in with:', email, password);
    // Call API...
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        required
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

### 15. How do you make API calls from React to an ASP.NET Core back‑end?

**Answer:**  
Use the browser's `fetch` API or libraries like `axios`. Handle loading and error states.

**Example with `fetch` and async/await:**
```jsx
function ProductCreate() {
  const [name, setName] = useState('');
  const [price, setPrice] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      const response = await fetch('https://localhost:5001/api/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, price: parseFloat(price) })
      });
      if (!response.ok) throw new Error('Failed to create');
      const data = await response.json();
      console.log('Created:', data);
      // Redirect or show success message
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={e => setName(e.target.value)} placeholder="Name" />
      <input value={price} onChange={e => setPrice(e.target.value)} placeholder="Price" type="number" step="0.01" />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Create Product'}
      </button>
    </form>
  );
}
```

**Enabling CORS in ASP.NET Core:**
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReact", policy =>
    {
        policy.WithOrigins("http://localhost:3000")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});
app.UseCors("AllowReact");
```

---

### 16. What is React Router and how do you set up routing?

**Answer:**  
React Router enables navigation between different components/views without full page reloads.

**Installation:**
```
npm install react-router-dom
```

**Basic setup:**
```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/products">Products</Link>
      </nav>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/products" element={<ProductList />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Accessing URL parameters in a component:**
```jsx
import { useParams } from 'react-router-dom';

function ProductDetail() {
  const { id } = useParams();
  const [product, setProduct] = useState(null);

  useEffect(() => {
    fetch(`https://localhost:5001/api/products/${id}`)
      .then(res => res.json())
      .then(data => setProduct(data));
  }, [id]);

  if (!product) return <div>Loading...</div>;
  return <h1>{product.name} - ${product.price}</h1>;
}
```

---

### 17. How do you manage global state in React without external libraries?

**Answer:**  
Use the **Context API** combined with `useReducer` or `useState` for simpler global state management.

**Creating a context:**
```jsx
// AuthContext.js
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

**Using the context in components:**
```jsx
// App.js
import { AuthProvider } from './AuthContext';

function App() {
  return (
    <AuthProvider>
      <Navbar />
      <Routes>...</Routes>
    </AuthProvider>
  );
}

// Navbar.js
import { useAuth } from './AuthContext';

function Navbar() {
  const { user, logout } = useAuth();
  return (
    <div>
      {user ? (
        <>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </div>
  );
}
```

---

### 18. What are some common patterns for organising a React project?

**Answer:**  
A typical folder structure for a medium‑sized React app:

```
src/
├── components/          # Reusable UI components (Button, Card, Modal)
├── pages/               # Page‑level components (HomePage, ProductsPage)
├── hooks/               # Custom hooks (useFetch, useLocalStorage)
├── services/            # API call functions (productService.js, authService.js)
├── context/             # Context providers (AuthContext.js, ThemeContext.js)
├── utils/               # Helper functions (formatDate.js, constants.js)
├── App.js
└── index.js
```

**Example API service (`services/productService.js`):**
```javascript
const API_URL = 'https://localhost:5001/api/products';

export const getProducts = async () => {
  const response = await fetch(API_URL);
  if (!response.ok) throw new Error('Failed to fetch');
  return response.json();
};

export const createProduct = async (product) => {
  const response = await fetch(API_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(product)
  });
  if (!response.ok) throw new Error('Failed to create');
  return response.json();
};
```

---

### 19. How do you handle loading and error states gracefully in React?

**Answer:**  
Use state variables to track loading and error status.

```jsx
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchProducts();
  }, []);

  async function fetchProducts() {
    try {
      setLoading(true);
      setError(null);
      const data = await getProducts();
      setProducts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  return (
    <ul>
      {products.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

---

### 20. What is the purpose of `key` prop in lists?

**Answer:**  
React uses the `key` prop to identify which items have changed, been added, or removed. It helps React optimise re‑rendering by reusing existing DOM elements.

**Always use a unique, stable identifier:**
```jsx
{products.map(product => (
  <ProductCard key={product.id} product={product} />
))}
```

**Avoid using array index as key** if the list can change order or items can be added/removed, as it can cause rendering bugs and performance issues.

---

*These notes cover the essential SQL, EF Core, and React concepts you'll use daily as a .NET full‑stack developer. Build small projects that combine these technologies to reinforce your learning.*
