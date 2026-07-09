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

Perfect. Let's ignore the code for now and focus only on **understanding the complete authentication process**. Once you understand the flow, writing the code becomes straightforward.

---

# Scenario

Suppose you have:

* **Frontend:** React
* **Backend:** ASP.NET Core API
* **Authentication Provider:** Microsoft Entra ID (Azure AD)

Example:

```text
Employee Portal

Frontend  → React

Backend   → ASP.NET Core API

Login      → Microsoft Entra ID
```

---

# Step 1 - User Opens React App

```text
Browser

↓

http://localhost:3000
```

React loads.

No user is logged in.

React shows

```text
Login with Microsoft
```

button.

---

# Step 2 - User Clicks Login

```text
React

↓

MSAL Library

↓

Microsoft Entra ID
```

MSAL redirects (or opens a popup) to Microsoft Entra ID.

Think of MSAL as:

> **A bridge between your React app and Microsoft Entra ID.**

---

# Step 3 - Microsoft Entra ID Authenticates User

Microsoft asks

```text
Email

Password

MFA (if enabled)
```

Example

```text
pankaj@company.com

Password

OTP
```

---

Microsoft verifies

```text
✔ User exists

✔ Password correct

✔ MFA completed
```

---

# Step 4 - Microsoft Creates JWT Token

After successful login

Microsoft creates

```text
JWT Access Token
```

Example

```text
eyJhbGciOi...
```

This token contains information like

```text
Name

Email

Roles

Permissions (Scopes)

Expiry

Tenant ID
```

Think of it as an **Employee ID Card**.

It proves:

> "Microsoft has authenticated this user."

---

# Step 5 - React Receives the Token

Now React has

```text
Access Token
```

React **does not verify** it.

It simply stores it (typically managed by MSAL).

```text
React

↓

Access Token
```

---

# Step 6 - User Clicks "Employees"

React wants employee data.

So React calls

```text
GET

/api/employees
```

But this API is protected.

React sends

```http
Authorization

Bearer eyJhbGciOi...
```

This is like showing your employee ID card at the security gate.

---

# Step 7 - API Receives the Request

ASP.NET Core receives

```text
GET

/api/employees
```

along with

```text
Bearer Token
```

---

# Step 8 - Authentication Middleware Executes

Before your controller runs

ASP.NET Core executes

```text
UseAuthentication()
```

Middleware extracts

```text
Bearer Token
```

---

# Step 9 - Token Validation

ASP.NET Core asks

```text
Is Signature Valid?

↓

YES

Is Token Expired?

↓

NO

Correct Tenant?

↓

YES

Correct Audience?

↓

YES

Correct Issuer?

↓

YES
```

If any answer is **No**

↓

```text
401 Unauthorized
```

Controller never executes.

---

# Step 10 - Create User

If everything is valid

ASP.NET Core creates

```text
HttpContext.User
```

Inside it

```text
Name

Email

Roles

Claims

Permissions
```

Now API knows

```text
Current User

=

Pankaj
```

---

# Step 11 - Authorization

Now

```text
UseAuthorization()
```

checks

Suppose controller

```text
[Authorize]
```

API asks

```text
Is user authenticated?

↓

YES
```

Continue.

---

Suppose

```text
[Authorize(Roles="Admin")]
```

Now API asks

```text
Is role Admin?

↓

YES
```

Continue.

Otherwise

```text
403 Forbidden
```

---

# Step 12 - Controller Executes

Finally

```text
EmployeeController

↓

Get Employees

↓

Return JSON
```

Response

```json
[
 {
   "id":1,
   "name":"John"
 }
]
```

---

# Whole Flow

```text
                USER

                  │

                  ▼

        React Application

                  │

        Click Login Button

                  │

                  ▼

      Microsoft Entra ID

                  │

      Verify User Credentials

                  │

                  ▼

       Generate JWT Token

                  │

                  ▼

       React receives Token

                  │

                  ▼

 Call ASP.NET Core API

 Authorization: Bearer Token

                  │

                  ▼

     Authentication Middleware

                  │

        Validate JWT

                  │

        Signature

        Expiry

        Audience

        Issuer

                  │

                  ▼

     Create HttpContext.User

                  │

                  ▼

      Authorization Middleware

                  │

      [Authorize] succeeds

                  │

                  ▼

      EmployeeController

                  │

                  ▼

         SQL Server

                  │

                  ▼

        Return Response
```

---

# What Happens if the Token is Expired?

Suppose

```text
Token Lifetime

1 hour
```

After 1 hour

```text
React

↓

Calls API

↓

API says

401 Unauthorized
```

Then MSAL silently requests a **new access token** (if the user's session with Entra ID is still valid).

The user usually doesn't notice this process.

---

# Simple Analogy

Imagine an office building.

```text
You

↓

Reception

↓

Show Company ID Card

↓

Reception verifies ID

↓

Gate Opens

↓

Enter Office
```

Mapping:

| Office Example   | Entra ID Example                       |
| ---------------- | -------------------------------------- |
| Employee ID Card | JWT Access Token                       |
| Reception        | ASP.NET Core Authentication Middleware |
| Security Check   | JWT Validation                         |
| Office Gate      | `[Authorize]`                          |
| Office           | Controller/API                         |

---

# Interview Answer (2 Minutes)

If an interviewer asks **"Explain Azure AD (Microsoft Entra ID) authentication flow."**, you can answer:

> "The React application uses the MSAL library to authenticate the user with Microsoft Entra ID. The user signs in using Microsoft credentials, and Entra ID issues a JWT access token containing claims such as the user's identity, roles, and scopes. React includes this token in the `Authorization: Bearer` header when calling the ASP.NET Core API. The API's authentication middleware validates the token by checking its signature, issuer, audience, and expiration. If the token is valid, ASP.NET Core creates a `ClaimsPrincipal` and stores it in `HttpContext.User`. The authorization middleware then evaluates attributes like `[Authorize]` or role-based policies. If authorization succeeds, the request reaches the controller; otherwise, the API returns `401 Unauthorized` or `403 Forbidden` depending on whether authentication or authorization failed."

This explanation is the one most interviewers expect from a developer with around **5+ years of .NET experience** because it demonstrates that you understand the **end-to-end authentication flow**, not just the code.
