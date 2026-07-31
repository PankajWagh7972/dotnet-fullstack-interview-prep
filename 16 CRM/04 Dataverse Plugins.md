In **Dynamics 365 / Dataverse Plugins**, Secure and Unsecure configuration values are passed **once when the plugin instance is created**, not on every execution. The Plugin Registration Tool stores these values and passes them to the plugin constructor.

---

# Scenario

Suppose your plugin calls an external payment API.

* **Secure Configuration**

  * API Key
  * Client Secret
  * Connection String

* **Unsecure Configuration**

  * API Base URL
  * Retry Count
  * Timeout
  * Environment Name

---

# Step 1. Create Plugin

```csharp
using Microsoft.Xrm.Sdk;
using Newtonsoft.Json;

public class AccountPlugin : IPlugin
{
    private readonly PluginConfiguration _config;

    public AccountPlugin(string unsecureConfig, string secureConfig)
    {
        _config = new PluginConfiguration();

        // Read Unsecure Configuration
        if (!string.IsNullOrWhiteSpace(unsecureConfig))
        {
            _config = JsonConvert.DeserializeObject<PluginConfiguration>(unsecureConfig);
        }

        // Read Secure Configuration
        if (!string.IsNullOrWhiteSpace(secureConfig))
        {
            _config.ApiKey = secureConfig;
        }
    }

    public void Execute(IServiceProvider serviceProvider)
    {
        var tracing =
            (ITracingService)serviceProvider.GetService(typeof(ITracingService));

        tracing.Trace($"API Url : {_config.ApiUrl}");
        tracing.Trace($"Retry Count : {_config.RetryCount}");

        // Never trace API keys in production
        tracing.Trace($"API Key Exists : {!string.IsNullOrEmpty(_config.ApiKey)}");

        // Example call
        ExternalApi.Call(
            _config.ApiUrl,
            _config.ApiKey);
    }
}
```

---

# Step 2. Configuration Class

```csharp
public class PluginConfiguration
{
    public string ApiUrl { get; set; }

    public int RetryCount { get; set; }

    public string ApiKey { get; set; }
}
```

---

# Step 3. Register Plugin

When registering using Plugin Registration Tool

### Unsecure Configuration

```json
{
  "ApiUrl":"https://api.company.com",
  "RetryCount":3
}
```

### Secure Configuration

```
1234567890abcdef-secret-api-key
```

---

# Step 4. Execute

Whenever CRM executes the plugin

```
Plugin Registration Tool
      │
      │
      ▼
Constructor
(
   unsecureConfig,
   secureConfig
)

      │
      ▼

Store values in fields

      │
      ▼

Execute()

Uses configuration
```

---

# Example with External API

```csharp
public static class ExternalApi
{
    public static void Call(string url, string apiKey)
    {
        using var client = new HttpClient();

        client.DefaultRequestHeaders.Add("x-api-key", apiKey);

        var response = client.GetAsync(url).Result;
    }
}
```

---

# Real Project Example

Suppose you integrate with SAP.

### Unsecure

```json
{
    "BaseUrl":"https://sap.company.com/api",
    "Timeout":30,
    "RetryCount":5
}
```

### Secure

```
Bearer eyJhbGciOi...
```

Plugin:

```csharp
public void Execute(IServiceProvider serviceProvider)
{
    HttpClient client = new HttpClient();

    client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Bearer", _config.ApiKey);

    client.BaseAddress = new Uri(_config.BaseUrl);

    var response = client.GetAsync("customers").Result;
}
```

---

# Another Common Approach (Key-Value Format)

Instead of JSON, some teams use simple key-value pairs.

### Unsecure Configuration

```
ApiUrl=https://api.company.com;
RetryCount=3;
Timeout=60
```

Parser:

```csharp
public static Dictionary<string, string> Parse(string config)
{
    var dict = new Dictionary<string, string>();

    foreach (var item in config.Split(';'))
    {
        if (string.IsNullOrWhiteSpace(item))
            continue;

        var parts = item.Split('=');

        if (parts.Length == 2)
            dict[parts[0]] = parts[1];
    }

    return dict;
}
```

Usage:

```csharp
public AccountPlugin(string unsecure, string secure)
{
    var values = Parse(unsecure);

    _apiUrl = values["ApiUrl"];
    _retryCount = int.Parse(values["RetryCount"]);
    _timeout = int.Parse(values["Timeout"]);

    _apiKey = secure;
}
```

---

# Best Practices

| Store In                   | Examples                                                       | Why                                                                 |
| -------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Secure Configuration**   | API Keys, Client Secret, OAuth Token, Connection String        | Encrypted and accessible only to users with appropriate privileges. |
| **Unsecure Configuration** | API URL, Timeout, Retry Count, Feature Flags, Environment Name | Easy to update and safe for non-sensitive values.                   |
| **Avoid**                  | Hardcoding secrets in code                                     | Requires recompilation and exposes credentials.                     |
| **Avoid**                  | Logging secure values                                          | Secrets may appear in Plugin Trace Logs.                            |
| **Prefer**                 | JSON for complex settings                                      | Easier to extend and maintain than custom-delimited strings.        |

> **Interview Tip:** Secure and unsecure configuration values are injected into the plug-in **constructor** during instantiation. The constructor typically stores them in private fields, and the `Execute` method uses those cached values. They are **not** retrieved from `IServiceProvider` during each execution.


A **ServiceEndpoint** in Dynamics 365/Dataverse represents an Azure messaging endpoint (Azure Service Bus Queue, Topic, or Event Hub). Instead of performing a long-running operation inside the plugin, you send the execution context to Azure, where another application processes it asynchronously.

## Architecture

```text
User Creates Account
        │
        ▼
Dynamics 365
        │
        ▼
Plugin Executes
        │
        ▼
Azure Service Bus Queue / Topic
        │
        ▼
Azure Function / .NET Worker / WebJob
        │
        ▼
SAP / ERP / CRM / Email Service / Any External System
```

---

# Real-World Scenario

Suppose your company wants to sync every new customer created in Dynamics 365 to SAP.

❌ **Bad approach**

```
User Creates Account

↓

Plugin Calls SAP API

↓

SAP Takes 10 Seconds

↓

CRM Form Waits

↓

Possible Timeout
```

The user experiences delays, and synchronous plugins have execution time limits.

---

✅ **Better approach using ServiceEndpoint**

```
User Creates Account

↓

Plugin

↓

Azure Service Bus Queue

↓

Plugin Finishes Immediately

↓

Azure Function Reads Queue

↓

Calls SAP

↓

Updates SAP
```

The user doesn't wait for SAP.

---

# Step 1 - Create Azure Service Bus

```
Namespace
    ↓
CustomerQueue
```

---

# Step 2 - Register Service Endpoint

Using the **Plugin Registration Tool**

```
Register

↓

New Service Endpoint

↓

Azure Service Bus Queue

↓

Connection String

↓

Queue Name
```

It creates a **ServiceEndpoint** record in Dataverse.

---

# Step 3 - Register Plugin Step

Instead of:

```
Create of Account

↓

Plugin
```

Register:

```
Create of Account

↓

Azure Aware Plugin
```

or associate the ServiceEndpoint with the plugin registration, depending on your integration pattern.

---

# What Gets Sent?

Dynamics sends a serialized **RemoteExecutionContext**.

It contains almost everything a normal plugin receives:

* Message Name
* Primary Entity
* Target Entity
* User Id
* Correlation Id
* Input Parameters
* Images
* Shared Variables
* Organization Id

Example:

```json
{
    "MessageName":"Create",
    "PrimaryEntityName":"account",
    "BusinessUnitId":"...",
    "UserId":"...",
    "InputParameters":
    {
        "Target":
        {
            "name":"ABC Ltd",
            "telephone1":"9876543210"
        }
    }
}
```

---

# Azure Function Example

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

public class CustomerSync
{
    [FunctionName("CustomerSync")]
    public void Run(
        [ServiceBusTrigger("customerqueue",
        Connection = "ServiceBusConnection")]
        string message,
        ILogger log)
    {
        log.LogInformation(message);

        // Deserialize RemoteExecutionContext

        // Read Account

        // Send to SAP

        // Save response
    }
}
```

---

# Azure Worker Service Example

```csharp
await foreach (var message in processor.ReceiveMessagesAsync())
{
    Console.WriteLine(message.Body);

    // Deserialize

    // Process

    // Call ERP

    // Complete Message
}
```

---

# Complete Flow

```text
Create Account

↓

Plugin Fires

↓

RemoteExecutionContext Created

↓

Serialized

↓

Azure Service Bus Queue

↓

Azure Function Trigger

↓

Deserialize Context

↓

Extract Account

↓

Call SAP REST API

↓

Store Success/Failure
```

---

# When Should You Use ServiceEndpoint?

Use it when:

* Integrating with SAP, Oracle, Salesforce, or other external systems.
* Sending CRM events to microservices.
* Triggering Azure Functions asynchronously.
* Processing large workloads outside Dataverse.
* Building event-driven architectures.

Avoid it for:

* Immediate validation that must block the transaction.
* Operations where the user needs an instant response before the record is saved.

---

# Interview Answer (2–3 Minutes)

> A **ServiceEndpoint** is a Dataverse entity that represents an Azure messaging endpoint such as an Azure Service Bus Queue, Topic, or Event Hub. It enables Dynamics 365 to publish the `RemoteExecutionContext` to Azure instead of performing lengthy processing inside the plugin. This is commonly used for asynchronous integrations with systems like SAP or external microservices. For example, when an Account is created, the plugin posts the execution context to an Azure Service Bus queue. An Azure Function subscribed to that queue processes the message, calls the external API, handles retries or failures independently, and updates the external system without delaying the user's transaction. This approach improves scalability, resilience, and overall user experience.
