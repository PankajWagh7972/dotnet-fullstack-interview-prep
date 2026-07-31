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
