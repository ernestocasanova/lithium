# Primavera.Hydrogen.Rest.Client

**Class library that contains types that allow communication with REST services.**

## ServiceClient

`ServiceClient<T>` is the base class that should be used by any service client - the entry point of a client library to communicate with the REST service.

This base class ensures multiple standard behaviors for the client library, like:

- The shape of the service actions (operations).
- Serialization of requests and deserialization of responses.
- Authentication and authorization of client applications.
- Localization.
- Specific request headers.
- Throttling.
- Etc.

## ServiceClientActions

`ServiceClientActions<T>` is used by `ServiceClient<T>` to perform the actual requests to the REST service and ensures the correct handling of every kind of action, including serialization and deserialization of responses.

The following methods are available:

- `ExecuteGetAsync<TResult>()` - GET without request body and with result body.
- `ExecuteGetNoResultAsync()` - GET without request body or result body.
- `ExecutePostAsync<TRequestBody, TResult>()` - POST with request body and result body.
- `ExecutePostAsync<TResult>()` - POST without request body and with result body.
- `ExecutePostNoResultAsync<TRequestBody>()` - POST with request body and without result body.
- `ExecutePostNoResultAsync()` - POST without request body or result body.
- `ExecuteDeleteNoResultAsync<TRequestBody>()` - DELETE with request body and without result body.
- `ExecuteDeleteNoResultAsync()`  DELETE without request body or result body.
- `ExecutePutAsync<TRequestBody, TResult>()` - PUT with request body and with result body.
- `ExecutePutAsync<TResult>()` - PUT without request body and with result body.
- `ExecutePutNoResultAsync<TRequestBody>()` - PUT with request body and without result body.
- `ExecutePutNoResultAsync()` - PUT without request body or result body.

## ServiceOperationResult

Every operation (action) on a service client should always return a `ServiceOperationResult` or a `ServiceOperationResult<T>` (if the result has a body).
This type will contain information both about the HTTP request and the HTTP response.

```csharp
Task<ServiceOperationResult<EmployeeData>> GetEmployeeAsync(string employeeId, CancellationToken cancellationToken = default)

Task<ServiceOperationResult> DeleteDepartmentAsync(string departmentId, CancellationToken cancellationToken = default)
```

## ServiceException

When a service operation does not return a valid response - that is, returns an unexpected status code - a `ServiceException` will be raised, including also
information about the HTTP request and the HTTP response and the `ServiceError` returned by the operation.

This means that using the service client to call a service operation requires a specific pattern to handle the possible exceptions:

```csharp
try
{
    ServiceOperationResult<EmployeeData> result = await client.GetEmployeeAsync("id").ConfigureAwait(false);
    EmployeeData employee = result.Body;
    // (...)
}
catch (ServiceException ex)
{
    ServiceError error = ex.Body;
    // (...)
}
```

## Authentication

All service clients should be integrated with Identity Server and thus require credentials to authenticate the client application (the application using the service client).

Typically these credentials are defined by an instance of `ServiceClientCredentials` and provided to the service client in the constructor.

This library provides a set of different credentials that can be used.

### AccessTokenCredentials

`AccessTokenCredentials` allows authenticating the client application by providing the access token, previously obtained from the authority server:

```csharp
string accessToken = (...)
using (MyServiceClient client = new MyServiceClient(baseUri, new AccessTokenCredentials(accessToken))
{
    // (...)
}
```

### ClientCredentials

`ClientCredentials` allows authenticating the client application using the client credentials flow by providing the client identifier, the client secret and an optional scope. The service client will use that identity to obtain the correct access token when needed.

```csharp
using (MyServiceClient client = new MyServiceClient(
    baseUri,
    ClientCredentials
        .ForScope(
            authorityServerUri,
            clientId,
            clientSecret,
            scope))
{
    // (...)
}
```

### AuthenticationCallbackCredentials

AuthenticationCallbackCredentials allows providing a callback that will be invoked every time the service client needs to obtain the access token. Then the client application provides that access token from that callback using the arguments received (that include the address of the authority server).

This form of specifying the service client credentials has the benefit of not requiring the client application to know the address of the authority server in advance (and, depending on how the REST service is configure, not even knowing the scopes required to use each specific service operation).

```csharp
using (MyServiceClient client = new MyServiceClient(
    baseUri,
    new AuthenticationCallbackCredentials(
        async (args) =>
        {
            string accessToken = ClientCredentials
                .ForAllScopes(
                    args.Authority,
                    clientId,
                    clientSecret);
            return accessToken;
        }))
{
    // (...)
}
```

### NoCredentials

On some scenarios it may be useful to pass no credentials to a service client. This can be achieved by the `NoCredentials` service client credentials, like in the following example:

```csharp
using (MyServiceClient client = new MyServiceClient(baseUri, ServiceClientCredentials.NoCredentials)
{
    // (...)
}
```

## Serialization

Request body and response bodies in a REST service are serialized to JSON. The current version of Hydrogen uses the `System.Text.Json` libraries
to perform serialization and deserialization and those operations are controlled by the options set in the `ServiceClient<T>` (`SerializationOptions` and `DeserializationOptions`).

The default behavior for serialization is implemented in `HttpRequestMessageExtensions.AddJsonContent()`.

The default behavior for deserialization is implemented in `ServiceClientActions.DeserializeResponseAsync()`.

> The current version of `System.Text.Json` does not support `TimeSpan` values by default. That is why the default `SerializationOptions` and `DeserializationOptions` add two custom converters (`Iso8601TimeSpanConverter` and `Iso8601TimeSpanNullableConverter`). These converter also need to be configured on the REST service server, using `RestApiMvcBuilderExtensions.AddJsonOptionsForWebApi()`.

## Transient Faults

On a Web server it is common that some transient errors may occur. Transient errors are those that may occur in a point in time but will repeat if the request is retried.

ServiceClient supports the notion of a retry policy to allow the client application to specify how these transient errors should be handler.

### SetRetryPolicy()

`ServiceClient<T>.SetRetryPolicy()` allows the client application to specify which retry policy is applied for all operations.

By default, the service client will be configured to use `ExponentialBackoffRetryStrategy` retry strategy with the `HttpStatusCodeErrorDetectionStrategy` error detection strategy.

For more information on the retry strategies and error detection strategies available, see the documentation on `Primavera.Hydrogen.Policies`.

## Localization

`ServiceClient<T>` supports requests and responses localization via the `Accept-Language` request header.

```csharp
using (MyServiceClient client = new MyServiceClient(...))
{
    client.AcceptLanguage = "pt";
    // (...)
}
```