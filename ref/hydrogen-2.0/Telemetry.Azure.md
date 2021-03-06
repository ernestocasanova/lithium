# Primavera.Hydrogen.Telemetry.Azure

**Class library that contains types that provide telemetry services that use Microsoft Azure Application Insights.**

## `AzureInsightsTelemetryOptions`

`AzureInsightsTelemetryOptions` provides the configuration options that control both the behavior of ITelemetryClient and of the server-based telemetry
components when configured to use Application Insights.

The following options are available:

- `InstrumentationKey` - the instrumentation key of the Azure Application Insights service. If null, telemetry recording will be disabled.
- `Enabled` (default is true) - a value indicating whether telemetry recording is enabled.
- `IgnoreErrors` (default is true) - a value indicating whether any exception raised while invoking Azure Application Insights should be ignored.
- `DeveloperMode` (default is false) - if true, then the telemetry records will be expedited faster to Application Insights.
- `EnableAdaptiveSampling` (default is false) - adaptive sampling ensures that only a portion of the telemetry records will actually be recorded by Application Insights.
- `DefaultProperties` (optional) - a dictionary containing key/value pairs that will appended to all telemetry records.
- `RequestTracking` (optional) - a set of properties specific to requests tracking (see `RequestTrackingOptions)`.
- `DependencyTracking` (optional) - a set of properties specific to dependencies tracking (see `DependencyTrackingOptions)`.
- `TelemetryChannel` (optional) - the telemetry channel that should be used by Application Insights (useful only for unit testing).

### RequestTrackingOptions

For the specific case of tracking requests to the server, the following options are available:

- `TrackRequestHeadersPaths` (default is empty) - when set, all the request headers for any request that matches one of the specified paths will be appended to request telemetry record in the custom properties (named `RequestHeader-*`).
- `TrackResponseHeadersPaths` (default is empty) - when set, all the response headers for any request that matches one of the specified paths will be appended to request telemetry record in the custom properties (named `ResponseHeader-*`).
- `IgnoreNotFoundPaths` (default is empty) - when set, any request that returns 404 (NotFound) and matches one of the specified paths will not be tracked by telemetry.
- `IgnoreAvailabilityTestsRequests` (default is false), when true, any requests resulting from availability tests will not be collected.

> Availability tests typically cause noise in the telemetry records that make it more difficult to diagnose problems because of the quantity of records produced. Simply ignoring those requests is usually a good idea.

> `IgnoreAvailabilityTestsRequests` has effect only on requests telemetry. It as no effect on dependencies, traces, metrics, etc.

### DependencyTrackingOptions

For the specific case of tracking dependencies from the server, the following options are available:

- `IgnoreNotFoundCommands` (default is empty) - when set, any dependency that returns 404 (NotFound) and matches one of the specified commands will not be tracked by telemetry.

### Configuration

`AzureInsightsTelemetryOptions` follows the .NET Core conventions, so that means that it can be setup in the application settings files (e.g. `appsettings.json`) or with a configure delegate or with a combination of both (see the extension methods on `IServiceCollection` described bellow for more information).

Here is an example of the configuration section in the settings file:

```json
"AzureInsightsTelemetryOptions": {
    "InstrumentationKey": "d54d583f-4ee3-487a-bfcc-9ddc17036774",
    "Enabled": true,
    "IgnoreErrors": true,
    "DeveloperMode": true,
    "EnableAdaptiveSampling": true,
    "DefaultProperties": {
        "CustomProperty1": "CustomValue1"
    },
    "RequestTracking": {
        "TrackRequestHeadersPaths": [
            "/api/*"
        ],
        "TrackResponseHeadersPaths": [
            "/api/*"
        ],
        "IgnoreNotFoundPaths": [
            "/api/*"
        ],
        "IgnoreAvailabilityTestsRequests": true
    },
    "RequestTracking": {
        "IgnoreNotFoundCommands": [
            "*.table.core.windows.net*"
        ]
    }
}
```

## Telemetry Client (`AzureInsightsTelemetryClientService`)

The `AzureInsightsTelemetryClientService` implements the `ITelemetryClientService` using the Microsoft Azure Application Insights.

This allows any application to record telemetry explicitly.

The service implementation should be registered using one of the following extension methods for `IServiceCollection`:

```csharp
IServiceCollection services = (...);
services.AddAzureInsightsTelemetryClient();
```

This will register the service using default configuration options set by a configuration section named `AzureInsightsTelemetryOptions`.

```csharp
IServiceCollection services = (...);
services.AddAzureInsightsTelemetryClient(
    (options) =>
    {
        // (...)
    });
```

This will register the service using the specified configuration delegate, after reading the default configuration section (if found).

## Server-based Telemetry (`AzureInsightsTelemetryServiceCollectionExtensions`)

The following extension methods for `IServiceCollection` are available to setup Azure Application Insights on a hosted application:

```csharp
IServiceCollection services = (...);
services.AddAzureInsightsTelemetry();
```

This will register all the Application Insights services using default configuration options (see `AzureInsightsTelemetryOptions`).

```csharp
IServiceCollection services = (...);
services.AddAzureInsightsTelemetry(
    (options) =>
    {
        // (...)
    });
```

This will register all the Application Insights services using the specified configuration delegate, after reading the default configuration section (if found).

### Requests Tracking

After calling the previous methods at application startup, Application Insights will automatically track all incoming requests to the server.

You can customize how that tracking takes place by configuration (see `RequestTrackingOptions`). You can also use the extensibility
mechanisms provided by Application Insights (see Telemetry Initializers).

### Dependencies Tracking

Dependency tracking is also automatically initialized after initializing the service.

You can customize how that tracking takes place by configuration (see `DependencyTrackingOptions`). You can also use the extensibility
mechanisms provided by Application Insights (see Telemetry Initializers).

### Logging

After initializing the services, logging through Application Insights (via trace records) will also be initialized but the concrete behavior will depend, as
you might expect, on how logging is actually configured in the application.

Azure Application Insights adheres to the logging conventions and you can setup the behavior of the provider using a configuration section named `ApplicationInsights` like in the following example:

```json
"Logging": {
    "LogLevel": {
        "Default": "Debug"
    },
    "ApplicationInsights": {
        "LogLevel": {
            "Default": "Information"
        }
    }
}
```

### Exceptions Tracking

Unhandled exceptions raised on the server will also be tracked through the logging provider, depending also on how logging is configured.

## Other Features

### Telemetry Initializers

Application Insights provides the notion of a telemetry initializer (`ITelemetryInitializer`) that allows complementing telemetry items when they occur based
on some custom logic.

The following telemetry initializers are available (and are used in specific ways when server-based telemetry is added to the application).

Either way, you can customize that behavior before adding server-based telemetry or you can add it to any application that configures those services without
using the extension methods described before.

> Telemetry initializers should be added to the service collection BEFORE the telemetry service is configured.

#### `RequestHeadersRequestTelemetryInitializer`

`RequestHeadersRequestTelemetryInitializer` allows adding the request headers to request telemetry records as custom properties named `RequestHeader-*`.

It allows you to specify which headers for which requests you want tracked.

If you want to track all request headers for all requests, you can simply rely on the default behavior of `AzureInsightsTelemetryServiceCollectionExtensions.AddAzureInsightsTelemetry` and set `RequestTrackingOptions.TrackRequestHeadersPaths`.

This is the same as adding the initializer yourself like this:

```csharp
services.AddSingleton<ITelemetryInitializer, RequestHeadersRequestTelemetryInitializer>();

services.AddAzureInsightsTelemetry();
```

If you want either to track specific headers and/or track them for specific requests, you need to add the service yourself before calling `AzureInsightsTelemetryServiceCollectionExtensions.AddAzureInsightsTelemetry`:

```csharp
services.AddSingleton<ITelemetryInitializer>(
    (provider) =>
    {
        return new RequestHeadersRequestTelemetryInitializer(provider)
        {
            Requests = RequestTelemetryInitializerBehavior.Specified,
            RequestPaths = new List<string>()
            {
                "/api/*",
                "*/mycontroller/*"
            },
            Headers = HeadersRequestTelemetryInitializerBehavior.Specified,
            HeaderNames = new List<string>()
            {
                "Header1",
                "Header2"
            }
        };
    });

services.AddAzureInsightsTelemetry();
```

#### `ResponseHeadersRequestTelemetryInitializer`

`ResponseHeadersRequestTelemetryInitializer` allows adding the response headers to request telemetry records as custom properties named `ResponseHeader-*`.

It allows you to specify which headers for which requests you want tracked.

If you want to track all response headers for all requests, you can simply rely on the default behavior of `AzureInsightsTelemetryServiceCollectionExtensions.AddAzureInsightsTelemetry` and set `RequestTrackingOptions.TrackResponseHeadersPaths`.

This is the same as adding the initializer yourself like this:

```csharp
services.AddSingleton<ITelemetryInitializer, ResponseHeadersRequestTelemetryInitializer>();

services.AddAzureInsightsTelemetry();
```

If you want either to track specific headers and/or track them for specific requests, you need to add the service yourself before calling `AzureInsightsTelemetryServiceCollectionExtensions.AddAzureInsightsTelemetry`:

```csharp
services.AddSingleton<ITelemetryInitializer>(
    (provider) =>
    {
        return new ResponseHeadersRequestTelemetryInitializer(provider)
        {
            Requests = RequestTelemetryInitializerBehavior.Specified,
            RequestPaths = new List<string>()
            {
                "/api/*",
                "*/mycontroller/*"
            },
            Headers = HeadersRequestTelemetryInitializerBehavior.Specified,
            HeaderNames = new List<string>()
            {
                "Header1",
                "Header2"
            }
        };
    });

services.AddAzureInsightsTelemetry();
```

#### `StatusCodeRequestTelemetryInitializer`

`StatusCodeRequestTelemetryInitializer` allows making App Insights treat a given status code response as success instead of error. This is useful, for example, for 404 return code, which are treated by default as errors but are normal responses for Web API endpoints.

It is possible to handle all responses, regardless of the request path:

```csharp
services.AddSingleton<ITelemetryInitializer, StatusCodeRequestTelemetryInitializer>();
```

> `HttpStatusCode.NotFound` is the default value for the `StatusCode` property of the initializer.

Or handle specific status codes and request paths:

```csharp
services.AddSingleton<ITelemetryInitializer>(
    (provider) =>
    {
        return new StatusCodeRequestTelemetryInitializer(provider)
         {
             Requests = RequestTelemetryInitializerBehavior.Specified,
             RequestPaths = new List<string>()
             {
                "/api/*",
            },
            StatusCode = HttpStatusCode.NotImplemented
        };
    });
```

### Telemetry Processors

Telemetry processors are executed in a pipeline by the telemetry engine, one at a time. They provide extensibility for that engine and allow implementing things like filtering telemetry records based on their properties (after all initializers run).

> Telemetry initializers should be added to the service collection AFTER the telemetry service is configured.

#### `IgnoreAvailabilityRequestTelemetryProcessor`

`IgnoreAvailabilityRequestTelemetryProcessor` allows simply ignoring (not tracking) any request telemetry record that is the result of an availability test (e.g. tests automated by Azure Insights, requests on the probe endpoints, etc.).

Those kind of requests are identified by inspecting the `SyntheticSource` property of the request telemetry. When it has a value it means that the request originated from an availability test.

If you want to all requests like these, you can simply rely on the default behavior of `AzureInsightsTelemetryServiceCollectionExtensions.AddAzureInsightsTelemetry` and set the `RequestTrackingOptions.IgnoreAvailabilityTestsRequests` to true.

This is the same as adding the initializer yourself like this:

```csharp
services.AddAzureInsightsTelemetry();

services.AddApplicationInsightsTelemetryProcessor<IgnoreAvailabilityRequestTelemetryProcessor>();
```

If you want to have that behavior only on specified requests routes, you should add the processor, like this, after invoking `AzureInsightsTelemetryServiceCollectionExtensions.AddAzureInsightsTelemetry`:

```csharp
services.AddAzureInsightsTelemetry();

services.AddSingleton<ITelemetryProcessorFactory>(
    (provider) =>
    {
        return new TelemetryProcessorFactory<IgnoreAvailabilityRequestTelemetryProcessor>(
        {
            (next) =>
            {
                return new IgnoreAvailabilityRequestTelemetryProcessor(next)
                {
                    Requests = RequestTelemetryProcessorBehavior.Specified,
                    RequestPaths = new List<string>()
                    {
                        "*/probe",
                    }
                }
            }
        };
    });
```