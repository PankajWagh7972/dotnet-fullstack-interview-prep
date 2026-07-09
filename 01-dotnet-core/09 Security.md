# How do you secure an ASP.NET Core application?

## Interview Definition

**Securing an ASP.NET Core application means protecting it from unauthorized access, data breaches, and common security vulnerabilities by implementing authentication, authorization, secure communication, input validation, and other security best practices.**

---

# Common Security Techniques

| Security Feature         | Purpose                            |
| ------------------------ | ---------------------------------- |
| Authentication           | Verify who the user is             |
| Authorization            | Control what the user can access   |
| HTTPS                    | Encrypt data in transit            |
| JWT Authentication       | Secure APIs                        |
| Input Validation         | Prevent invalid or malicious input |
| SQL Injection Prevention | Protect the database               |
| XSS Protection           | Prevent malicious scripts          |
| CSRF Protection          | Prevent forged requests            |
| Secure Headers           | Improve browser security           |
| Secrets Management       | Protect sensitive configuration    |
| Logging & Monitoring     | Detect suspicious activity         |

---

# 1. Authentication (Who are you?)

Verify the identity of the user.

Example:

* Username/Password
* JWT Token
* OAuth
* OpenID Connect

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer();
```

Apply:

```csharp
[Authorize]
public IActionResult GetProfile()
{
    return Ok();
}
```

---

# 2. Authorization (What can you access?)

After login, check permissions.

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser()
{
    return Ok();
}
```

Only users with the **Admin** role can access this action.

---

# 3. Always Use HTTPS

Encrypt communication between client and server.

```csharp
app.UseHttpsRedirection();
```

Benefits:

* Prevents data interception.
* Protects passwords and tokens.

---

# 4. Use JWT for Web APIs

For REST APIs, authenticate users using JWT Bearer Tokens.

```http
Authorization: Bearer eyJhbGciOi...
```

Benefits:

* Stateless
* Scalable
* Widely used

---

# 5. Validate User Input

Never trust user input.

```csharp
public class User
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
```

Controller:

```csharp
if (!ModelState.IsValid)
    return BadRequest(ModelState);
```

---

# 6. Prevent SQL Injection

❌ Bad

```csharp
string sql = $"SELECT * FROM Users WHERE Name='{name}'";
```

✔ Good (Parameterized Query)

```csharp
var user = await _context.Users
    .Where(x => x.Name == name)
    .FirstOrDefaultAsync();
```

EF Core automatically parameterizes queries.

---

# 7. Prevent Cross-Site Scripting (XSS)

ASP.NET Core Razor automatically HTML-encodes output.

```html
@Model.Name
```

Never render untrusted HTML directly unless it has been properly sanitized.

---

# 8. Prevent CSRF (Cross-Site Request Forgery)

For MVC/Razor forms:

```csharp
[ValidateAntiForgeryToken]
```

View:

```html
@Html.AntiForgeryToken()
```

**Note:** JWT-protected APIs that don't rely on browser cookies are generally not vulnerable to CSRF in the same way.

---

# 9. Store Secrets Securely

Never hardcode:

```csharp
string password = "123456";
```

Instead use:

* `appsettings.json` (non-sensitive configuration)
* Environment Variables
* User Secrets (Development)
* Azure Key Vault (Production)

---

# 10. Secure HTTP Headers

Examples:

* Content-Security-Policy (CSP)
* X-Content-Type-Options
* X-Frame-Options

These help protect against attacks like clickjacking and XSS.

---

# 11. Logging & Monitoring

Log:

* Login failures
* Unauthorized access
* Exceptions
* Suspicious activity

Use:

* ILogger
* Serilog
* Application Insights

---

# 12. Keep Dependencies Updated

Update:

* .NET SDK
* NuGet packages
* Third-party libraries

Security vulnerabilities are frequently fixed in updates.

---

# Real-World Example

A Banking API

```text
Client
   │
HTTPS
   │
JWT Authentication
   │
Authorization
   │
Input Validation
   │
Business Logic
   │
EF Core (Parameterized Queries)
   │
SQL Server
```

Every layer adds protection.

---

# Common Interview Questions

### **Q: How do you secure a Web API?**

* Use HTTPS
* JWT Authentication
* Role/Policy-based Authorization
* Validate inputs
* Return proper status codes
* Protect secrets
* Log security events

---

### **Q: How does EF Core prevent SQL Injection?**

EF Core generates **parameterized SQL queries**, so user input isn't directly concatenated into SQL statements.

---

### **Q: What is the difference between Authentication and Authorization?**

| Authentication    | Authorization                |
| ----------------- | ---------------------------- |
| Who are you?      | What can you access?         |
| Verifies identity | Verifies permissions         |
| Happens first     | Happens after authentication |

---

# Interview One-Liner

> **To secure an ASP.NET Core application, I use HTTPS, implement authentication and authorization, validate user input, use EF Core or parameterized queries to prevent SQL injection, protect against XSS and CSRF where applicable, store secrets securely, configure security headers, log security events, and keep dependencies up to date.**

---

# Quick Revision

| Security                        | Purpose                  |
| ------------------------------- | ------------------------ |
| Authentication                  | Verify user identity     |
| Authorization                   | Control access           |
| HTTPS                           | Encrypt traffic          |
| JWT                             | Secure APIs              |
| Input Validation                | Prevent invalid data     |
| EF Core / Parameterized Queries | Prevent SQL Injection    |
| Output Encoding                 | Prevent XSS              |
| Anti-Forgery Tokens             | Prevent CSRF (MVC/Razor) |
| Secure Secret Storage           | Protect credentials      |
| Logging                         | Monitor security events  |
| Security Headers                | Harden browser security  |

### **Memory Trick**

Remember:

**A A H J V S X C S L U**

* **A**uthentication
* **A**uthorization
* **H**TTPS
* **J**WT
* **V**alidation
* **S**QL Injection Prevention
* **X**SS Prevention
* **C**SRF Protection
* **S**ecrets Management
* **L**ogging
* **U**pdate Dependencies
