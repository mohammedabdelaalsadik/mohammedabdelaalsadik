# Centralized Logging in .NET Using Serilog, Elasticsearch, and Kibana

## 1. Introduction

Modern applications need more than simple text logs. In distributed systems, APIs, microservices, and SaaS platforms, logs must be searchable, structured, and easy to analyze.

In this article, we will build a centralized logging architecture using:

- **Serilog** for structured logging in .NET
- **Elasticsearch** for storing and indexing logs
- **Kibana** for searching, filtering, and visualizing logs

## 2. Why Centralized Logging?

Centralized logging helps teams:

- Track API requests and responses
- Analyze errors and exceptions
- Monitor performance issues
- Search logs across multiple services
- Detect production problems faster
- Improve support and debugging

## 3. High-Level Architecture

```text
.NET Application
      |
      | Serilog
      v
Elasticsearch
      |
      v
Kibana Dashboard
```

## 4. Serilog Role

Serilog captures logs as structured JSON instead of plain text.

Example log data:

```json
{
  "Timestamp": "2026-05-21T10:30:00",
  "Level": "Error",
  "Message": "Failed to get customer data",
  "UserId": 120,
  "RequestPath": "/api/customers/120",
  "Exception": "Database timeout"
}
```

This makes logs searchable by fields like:

- `UserId`
- `RequestPath`
- `StatusCode`
- `Exception`
- `TenantId`
- `CorrelationId`

## 5. Elasticsearch Role

Elasticsearch stores logs as indexed documents.

It allows fast searching by:

- Error level
- API endpoint
- User ID
- Tenant ID
- Date range
- Exception type
- Request duration

## 6. Kibana Role

Kibana is the UI layer used to explore logs.

With Kibana, we can:

- Search production errors
- Create dashboards
- Monitor API failures
- Filter logs by service
- Analyze request performance
- Track exceptions over time

## 7. .NET Serilog Configuration

Install packages:

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Elasticsearch
dotnet add package Serilog.Enrichers.Environment
dotnet add package Serilog.Enrichers.Thread
```

Example configuration:

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithThreadId()
    .WriteTo.Console()
    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
    {
        AutoRegisterTemplate = true,
        IndexFormat = "app-logs-{0:yyyy.MM.dd}"
    })
    .CreateLogger();

builder.Host.UseSerilog();
```

## 8. Add Correlation ID

Correlation ID is very important for tracing one request across services.

```csharp
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault()
                        ?? Guid.NewGuid().ToString();

    context.Response.Headers["X-Correlation-ID"] = correlationId;

    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});
```

## 9. Logging API Requests

Example:

```csharp
Log.Information("Request completed {@RequestInfo}", new
{
    Path = context.Request.Path,
    Method = context.Request.Method,
    StatusCode = context.Response.StatusCode,
    UserId = userId,
    TenantId = tenantId
});
```

## 10. Recommended Log Fields

- `ApplicationName`
- `Environment`
- `MachineName`
- `RequestPath`
- `HttpMethod`
- `StatusCode`
- `UserId`
- `TenantId`
- `CorrelationId`
- `ElapsedMilliseconds`
- `Exception`
- `SourceContext`

## 11. Kibana Searches

Find all errors:

```text
Level: Error
```

Find errors for a specific API:

```text
RequestPath: "/api/orders"
```

Find logs by correlation ID:

```text
CorrelationId: "abc-123"
```

Find slow requests:

```text
ElapsedMilliseconds > 3000
```

## 12. Best Practices

- Do not log passwords or tokens
- Use structured logging, not string-only logs
- Add correlation ID
- Separate logs by environment
- Use daily indexes
- Log request duration
- Log business identifiers like `TenantId` and `UserId`
- Create Kibana dashboards for errors and performance

## 13. Conclusion

Using Serilog with Elasticsearch and Kibana gives your .NET application a powerful centralized logging system.

It helps developers, DevOps, QA, and support teams quickly understand what happened, where it happened, and why it happened.

This setup is very useful for enterprise systems, SaaS platforms, microservices, and production APIs.
