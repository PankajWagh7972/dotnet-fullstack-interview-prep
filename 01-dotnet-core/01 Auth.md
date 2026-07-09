Azure AD (now called **Microsoft Entra ID**) authentication is one of the most frequently asked interview topics for 5+ years of .NET experience.

---

# What is Microsoft Entra ID (Azure AD)?

**Microsoft Entra ID** is Microsoft's cloud-based identity provider that authenticates users and applications.

Instead of storing usernames and passwords in your application, users log in through Microsoft Entra ID, which issues a **JWT access token**. Your ASP.NET Core API validates this token before allowing access.

```text
Client (React/Angular/Postman)
          │
          │ Login
          ▼
Microsoft Entra ID
          │
          │ JWT Access Token
          ▼
ASP.NET Core API
          │
 Validate Token
          │
          ▼
Authorized Response
```

---

# Step 1: Register Your API in Microsoft Entra ID

In the Azure Portal:

1. Go to **Microsoft Entra ID**
2. **App registrations**
3. Click **New registration**
4. Give it a name (e.g., `EmployeeAPI`)
5. Register the application

You'll get:

* **Application (Client) ID**
* **Directory (Tenant) ID**

---

# Step 2: Expose an API

Under **Expose an API**:

Create an Application ID URI, for example:

```text
api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Add a scope, for example:

```text
Employee.Read
```

---

# Step 3: Install NuGet Package

```text
Microsoft.Identity.Web
```

---

# Step 4: Configure `appsettings.json`

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "ClientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "Audience": "api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
}
```

---

# Step 5: Configure Authentication

In `Program.cs`:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;

builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();
```

---

# Step 6: Configure Middleware

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

> **Important:** `UseAuthentication()` must come **before** `UseAuthorization()`.

---

# Step 7: Secure Your Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeeController : ControllerBase
{
    [Authorize]
    [HttpGet]
    public IActionResult GetEmployees()
    {
        return Ok("Authenticated User");
    }
}
```

Now the endpoint requires a valid Entra ID access token.

---

# Step 8: Call the API

Client sends:

```http
GET /api/employee

Authorization: Bearer eyJhbGciOi...
```

ASP.NET Core:

* Reads the JWT
* Validates the signature
* Validates issuer
* Validates audience
* Validates expiration
* Creates the authenticated `ClaimsPrincipal`

If valid:

```http
200 OK
```

Otherwise:

```http
401 Unauthorized
```

---

# Access User Information

```csharp
[Authorize]
[HttpGet]
public IActionResult Profile()
{
    var name = User.Identity?.Name;

    var email = User.FindFirst("preferred_username")?.Value;

    var objectId = User.FindFirst("oid")?.Value;

    return Ok(new
    {
        name,
        email,
        objectId
    });
}
```

---

# How Authentication Works Internally

```text
React / Angular
        │
Login
        ▼
Microsoft Entra ID
        │
JWT Token
        ▼
ASP.NET Core API
        │
UseAuthentication()
        │
JWT Middleware validates token
        │
HttpContext.User populated
        │
UseAuthorization()
        │
[Authorize]
        │
Controller Executes
```

---

# Role-Based Authorization

Suppose Azure defines an **Admin** role.

Controller:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteEmployee()
{
    return Ok();
}
```

Only users with the **Admin** role can access it.

---

# Scope-Based Authorization

For APIs, scopes are commonly used.

```csharp
[Authorize]
public IActionResult GetEmployees()
{
    return Ok();
}
```

You can also create authorization policies that require a specific scope (for example, `Employee.Read`) before allowing access.

---

# Common Interview Questions

### **Q1: Why do we use Azure AD (Microsoft Entra ID)?**

* Centralized authentication
* Single Sign-On (SSO)
* Multi-Factor Authentication (MFA)
* Secure JWT token issuance
* Integration with Microsoft 365 and Azure services

---

### **Q2: Why do we need `UseAuthentication()` before `UseAuthorization()`?**

* `UseAuthentication()` validates the token and sets `HttpContext.User`.
* `UseAuthorization()` checks whether that authenticated user has permission.

If the order is reversed, authorization has no authenticated user to evaluate.

---

### **Q3: What is validated in the JWT token?**

ASP.NET Core validates:

* Signature
* Issuer (`iss`)
* Audience (`aud`)
* Expiration (`exp`)
* (Optionally) scopes or roles

---

# Interview One-Liner

> **To implement Microsoft Entra ID (Azure AD) authentication in an ASP.NET Core API, I register the API in Entra ID, configure JWT Bearer authentication using `Microsoft.Identity.Web`, add `UseAuthentication()` and `UseAuthorization()` to the middleware pipeline, and secure endpoints with `[Authorize]`. The API validates the JWT access token and creates the authenticated user from its claims.**

---

# Quick Revision

| Step | Action                                             |
| ---- | -------------------------------------------------- |
| 1    | Register API in Microsoft Entra ID                 |
| 2    | Expose an API and create scopes                    |
| 3    | Install `Microsoft.Identity.Web`                   |
| 4    | Configure `appsettings.json`                       |
| 5    | Configure JWT authentication                       |
| 6    | Add `UseAuthentication()` and `UseAuthorization()` |
| 7    | Decorate endpoints with `[Authorize]`              |
| 8    | Client sends Bearer token                          |

### Memory Trick

Think of the flow as:

**Register → Configure → Authenticate → Authorize → Access**

This is the standard authentication flow used in modern ASP.NET Core APIs integrated with Microsoft Entra ID.


Azure AD (now called **Microsoft Entra ID**) authentication is one of the most frequently asked interview topics for 5+ years of .NET experience.

---

# What is Microsoft Entra ID (Azure AD)?

**Microsoft Entra ID** is Microsoft's cloud-based identity provider that authenticates users and applications.

Instead of storing usernames and passwords in your application, users log in through Microsoft Entra ID, which issues a **JWT access token**. Your ASP.NET Core API validates this token before allowing access.

```text
Client (React/Angular/Postman)
          │
          │ Login
          ▼
Microsoft Entra ID
          │
          │ JWT Access Token
          ▼
ASP.NET Core API
          │
 Validate Token
          │
          ▼
Authorized Response
```

---

# Step 1: Register Your API in Microsoft Entra ID

In the Azure Portal:

1. Go to **Microsoft Entra ID**
2. **App registrations**
3. Click **New registration**
4. Give it a name (e.g., `EmployeeAPI`)
5. Register the application

You'll get:

* **Application (Client) ID**
* **Directory (Tenant) ID**

---

# Step 2: Expose an API

Under **Expose an API**:

Create an Application ID URI, for example:

```text
api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Add a scope, for example:

```text
Employee.Read
```

---

# Step 3: Install NuGet Package

```text
Microsoft.Identity.Web
```

---

# Step 4: Configure `appsettings.json`

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "ClientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "Audience": "api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
}
```

---

# Step 5: Configure Authentication

In `Program.cs`:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;

builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();
```

---

# Step 6: Configure Middleware

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

> **Important:** `UseAuthentication()` must come **before** `UseAuthorization()`.

---

# Step 7: Secure Your Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeeController : ControllerBase
{
    [Authorize]
    [HttpGet]
    public IActionResult GetEmployees()
    {
        return Ok("Authenticated User");
    }
}
```

Now the endpoint requires a valid Entra ID access token.

---

# Step 8: Call the API

Client sends:

```http
GET /api/employee

Authorization: Bearer eyJhbGciOi...
```

ASP.NET Core:

* Reads the JWT
* Validates the signature
* Validates issuer
* Validates audience
* Validates expiration
* Creates the authenticated `ClaimsPrincipal`

If valid:

```http
200 OK
```

Otherwise:

```http
401 Unauthorized
```

---

# Access User Information

```csharp
[Authorize]
[HttpGet]
public IActionResult Profile()
{
    var name = User.Identity?.Name;

    var email = User.FindFirst("preferred_username")?.Value;

    var objectId = User.FindFirst("oid")?.Value;

    return Ok(new
    {
        name,
        email,
        objectId
    });
}
```

---

# How Authentication Works Internally

```text
React / Angular
        │
Login
        ▼
Microsoft Entra ID
        │
JWT Token
        ▼
ASP.NET Core API
        │
UseAuthentication()
        │
JWT Middleware validates token
        │
HttpContext.User populated
        │
UseAuthorization()
        │
[Authorize]
        │
Controller Executes
```

---

# Role-Based Authorization

Suppose Azure defines an **Admin** role.

Controller:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteEmployee()
{
    return Ok();
}
```

Only users with the **Admin** role can access it.

---

# Scope-Based Authorization

For APIs, scopes are commonly used.

```csharp
[Authorize]
public IActionResult GetEmployees()
{
    return Ok();
}
```

You can also create authorization policies that require a specific scope (for example, `Employee.Read`) before allowing access.

---

# Common Interview Questions

### **Q1: Why do we use Azure AD (Microsoft Entra ID)?**

* Centralized authentication
* Single Sign-On (SSO)
* Multi-Factor Authentication (MFA)
* Secure JWT token issuance
* Integration with Microsoft 365 and Azure services

---

### **Q2: Why do we need `UseAuthentication()` before `UseAuthorization()`?**

* `UseAuthentication()` validates the token and sets `HttpContext.User`.
* `UseAuthorization()` checks whether that authenticated user has permission.

If the order is reversed, authorization has no authenticated user to evaluate.

---

### **Q3: What is validated in the JWT token?**

ASP.NET Core validates:

* Signature
* Issuer (`iss`)
* Audience (`aud`)
* Expiration (`exp`)
* (Optionally) scopes or roles

---

# Interview One-Liner

> **To implement Microsoft Entra ID (Azure AD) authentication in an ASP.NET Core API, I register the API in Entra ID, configure JWT Bearer authentication using `Microsoft.Identity.Web`, add `UseAuthentication()` and `UseAuthorization()` to the middleware pipeline, and secure endpoints with `[Authorize]`. The API validates the JWT access token and creates the authenticated user from its claims.**

---

# Quick Revision

| Step | Action                                             |
| ---- | -------------------------------------------------- |
| 1    | Register API in Microsoft Entra ID                 |
| 2    | Expose an API and create scopes                    |
| 3    | Install `Microsoft.Identity.Web`                   |
| 4    | Configure `appsettings.json`                       |
| 5    | Configure JWT authentication                       |
| 6    | Add `UseAuthentication()` and `UseAuthorization()` |
| 7    | Decorate endpoints with `[Authorize]`              |
| 8    | Client sends Bearer token                          |

### Memory Trick

Think of the flow as:

**Register → Configure → Authenticate → Authorize → Access**

This is the standard authentication flow used in modern ASP.NET Core APIs integrated with Microsoft Entra ID.


Azure AD (now called **Microsoft Entra ID**) authentication is one of the most frequently asked interview topics for 5+ years of .NET experience.

---

# What is Microsoft Entra ID (Azure AD)?

**Microsoft Entra ID** is Microsoft's cloud-based identity provider that authenticates users and applications.

Instead of storing usernames and passwords in your application, users log in through Microsoft Entra ID, which issues a **JWT access token**. Your ASP.NET Core API validates this token before allowing access.

```text
Client (React/Angular/Postman)
          │
          │ Login
          ▼
Microsoft Entra ID
          │
          │ JWT Access Token
          ▼
ASP.NET Core API
          │
 Validate Token
          │
          ▼
Authorized Response
```

---

# Step 1: Register Your API in Microsoft Entra ID

In the Azure Portal:

1. Go to **Microsoft Entra ID**
2. **App registrations**
3. Click **New registration**
4. Give it a name (e.g., `EmployeeAPI`)
5. Register the application

You'll get:

* **Application (Client) ID**
* **Directory (Tenant) ID**

---

# Step 2: Expose an API

Under **Expose an API**:

Create an Application ID URI, for example:

```text
api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Add a scope, for example:

```text
Employee.Read
```

---

# Step 3: Install NuGet Package

```text
Microsoft.Identity.Web
```

---

# Step 4: Configure `appsettings.json`

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "ClientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "Audience": "api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
}
```

---

# Step 5: Configure Authentication

In `Program.cs`:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;

builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();
```

---

# Step 6: Configure Middleware

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

> **Important:** `UseAuthentication()` must come **before** `UseAuthorization()`.

---

# Step 7: Secure Your Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeeController : ControllerBase
{
    [Authorize]
    [HttpGet]
    public IActionResult GetEmployees()
    {
        return Ok("Authenticated User");
    }
}
```

Now the endpoint requires a valid Entra ID access token.

---

# Step 8: Call the API

Client sends:

```http
GET /api/employee

Authorization: Bearer eyJhbGciOi...
```

ASP.NET Core:

* Reads the JWT
* Validates the signature
* Validates issuer
* Validates audience
* Validates expiration
* Creates the authenticated `ClaimsPrincipal`

If valid:

```http
200 OK
```

Otherwise:

```http
401 Unauthorized
```

---

# Access User Information

```csharp
[Authorize]
[HttpGet]
public IActionResult Profile()
{
    var name = User.Identity?.Name;

    var email = User.FindFirst("preferred_username")?.Value;

    var objectId = User.FindFirst("oid")?.Value;

    return Ok(new
    {
        name,
        email,
        objectId
    });
}
```

---

# How Authentication Works Internally

```text
React / Angular
        │
Login
        ▼
Microsoft Entra ID
        │
JWT Token
        ▼
ASP.NET Core API
        │
UseAuthentication()
        │
JWT Middleware validates token
        │
HttpContext.User populated
        │
UseAuthorization()
        │
[Authorize]
        │
Controller Executes
```

---

# Role-Based Authorization

Suppose Azure defines an **Admin** role.

Controller:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteEmployee()
{
    return Ok();
}
```

Only users with the **Admin** role can access it.

---

# Scope-Based Authorization

For APIs, scopes are commonly used.

```csharp
[Authorize]
public IActionResult GetEmployees()
{
    return Ok();
}
```

You can also create authorization policies that require a specific scope (for example, `Employee.Read`) before allowing access.

---

# Common Interview Questions

### **Q1: Why do we use Azure AD (Microsoft Entra ID)?**

* Centralized authentication
* Single Sign-On (SSO)
* Multi-Factor Authentication (MFA)
* Secure JWT token issuance
* Integration with Microsoft 365 and Azure services

---

### **Q2: Why do we need `UseAuthentication()` before `UseAuthorization()`?**

* `UseAuthentication()` validates the token and sets `HttpContext.User`.
* `UseAuthorization()` checks whether that authenticated user has permission.

If the order is reversed, authorization has no authenticated user to evaluate.

---

### **Q3: What is validated in the JWT token?**

ASP.NET Core validates:

* Signature
* Issuer (`iss`)
* Audience (`aud`)
* Expiration (`exp`)
* (Optionally) scopes or roles

---

# Interview One-Liner

> **To implement Microsoft Entra ID (Azure AD) authentication in an ASP.NET Core API, I register the API in Entra ID, configure JWT Bearer authentication using `Microsoft.Identity.Web`, add `UseAuthentication()` and `UseAuthorization()` to the middleware pipeline, and secure endpoints with `[Authorize]`. The API validates the JWT access token and creates the authenticated user from its claims.**

---

# Quick Revision

| Step | Action                                             |
| ---- | -------------------------------------------------- |
| 1    | Register API in Microsoft Entra ID                 |
| 2    | Expose an API and create scopes                    |
| 3    | Install `Microsoft.Identity.Web`                   |
| 4    | Configure `appsettings.json`                       |
| 5    | Configure JWT authentication                       |
| 6    | Add `UseAuthentication()` and `UseAuthorization()` |
| 7    | Decorate endpoints with `[Authorize]`              |
| 8    | Client sends Bearer token                          |

### Memory Trick

Think of the flow as:

**Register → Configure → Authenticate → Authorize → Access**

This is the standard authentication flow used in modern ASP.NET Core APIs integrated with Microsoft Entra ID.
