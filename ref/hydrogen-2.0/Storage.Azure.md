# Primavera.Hydrogen.Storage.Azure

**Class library that contains types that define generic storage services that use Microsoft Azure storage services.**

## Blob Storage (`AzureBlobStorageService`)

The `AzureBlobStorageService` implements the `IBlobStorageService` using the Microsoft Azure storage services.

The service implementation should be registered using one of the following extension methods for `IServiceCollection`:

```csharp
IServiceCollection services = (...);
services.AddAzureBlobStorage();
```

This will register the service using default configuration options set my a configuration section named `AzureBlobStorageOptions`.

```csharp
IServiceCollection services = (...);
services.AddAzureBlobStorage(
    (options) =>
    {
        // (...)
    });
```

This will register the service using the specified configuration delegate, after reading the default configuration section (if found).

### `AzureBlobStorageOptions`

This options class provides the following configuration options:

- `ConnectionString` (required) (no default value) - the connection string to connect to the Azure storage service.
- `ServerConnectionTimeout` (optional) (default is 5 seconds) - the timeout time when establishing a connection to the storage server.
- `MaximumExecutionTime` (optional) (default is 60 seconds) - the maximum execution time of the storage server requests (including all retries).
- `RetryPolicy` (optional) (default is Exponential) - the retry policy that should be applied to all requests to the storage server.
- `RetryPolicyMaximumAttempts` (optional) (default is 3) - the maximum number of retries for all requests to the storage server.
- `RetryPolicyBackoffTime` (optional) (default is 4 seconds) - the initial back-off time of the retry policy applied to all requests to the storage server.
- `BlobNamingPolicy` (required) (default is `BlobNamingPolicy.CaseInsensitive`) - defines the naming policy to verify blob names.

> Azure Blob Storage requires containers' names to be lower-case but blob names are case-sensitive. However, the original implementation of AzureBlockBlobReference enforced blob names to also be lower-case. Now you can modify that default behavior by setting `BlobNamingPolicy` to `BlobNamingPolicy.CaseSensitive`.

## Multi-model Database (`CosmosDbMultiModelDatabaseService`)

The `CosmosDbMultiModelDatabaseService` implements the `IMultiModelDatabaseService` using the Microsoft Azure CosmosDB.

The service implementation should be registered using one of the following extension methods for `IServiceCollection`:

```csharp
IServiceCollection services = (...);
services.AddCosmosDbMultiModelDatabase();
```

This will register the service using default configuration options set my a configuration section named `CosmosDbMultiModelDatabaseOptions`.

```csharp
IServiceCollection services = (...);
services.AddCosmosDbMultiModelDatabase(
    (options) =>
    {
        // (...)
    });
```

This will register the service using the specified configuration delegate, after reading the default configuration section (if found).

### `CosmosDbMultiModelDatabaseOptions`

This options class provides the following configuration options:

- `ConnectionString` (required) (no default value) - the connection string to connect to the CosmosDB service.
- `RequestTimeout` (optional) (no default value) - the maximum time to wait for responses from the server.
- `RateLimitedMaximumRetries` (optional) (no default value) - the number of times that requests should be retried on throttled.
- `RateLimitedMaximumWaitTime` (optional) (no default value) - the maximum time that a throttled request can be retried.
- `RetryPolicy` (optional) (default is Exponential) - the retry policy that should be applied to all requests to the server.
- `RetryPolicyMaximumAttempts` (optional) (default is 3) - the maximum number of retries for all requests to the server.
- `RetryPolicyBackoffTime` (optional) (default is 500 milliseconds) - the initial back-off time of the retry policy applied to all requests to the server.
- `RetryPolicyMaximumDelay` (optional) (default is 30 seconds) - the maximum delay of the retry policy applied to all requests to the server.

## Table Storage (`AzureTableStorageService`)

The `AzureTableStorageService` implements the `ITableStorageService` using the Microsoft Azure storage services.

The service implementation should be registered using one of the following extension methods for `IServiceCollection`:

```csharp
IServiceCollection services = (...);
services.AddAzureTableStorage();
```

This will register the service using default configuration options set my a configuration section named `AzureTableStorageOptions`.

```csharp
IServiceCollection services = (...);
services.AddAzureTableStorage(
    (options) =>
    {
        // (...)
    });
```

This will register the service using the specified configuration delegate, after reading the default configuration section (if found).

### `AzureTableStorageOptions`

This options class provides the following configuration options:

- `ConnectionString` (required) (no default value) - the connection string to connect to the Azure storage service.
- `ServerConnectionTimeout` (optional) (default is 5 seconds) - the timeout time when establishing a connection to the storage server.
- `MaximumExecutionTime` (optional) (default is 60 seconds) - the maximum execution time of the storage server requests (including all retries).
- `RetryPolicy` (optional) (default is Exponential) - the retry policy that should be applied to all requests to the storage server.
- `RetryPolicyMaximumAttempts` (optional) (default is 3) - the maximum number of retries for all requests to the storage server.
- `RetryPolicyBackoffTime` (optional) (default is 500 milliseconds) - the initial back-off time of the retry policy applied to all requests to the storage server.

### Specific Behaviors of the Service

The following behaviors of `ITableStorageService` are specific to this implementation.

#### Querying Data

To create instance of `ITableQuery` to retrieve records using a query you should use `AzureTableQuery` and `AzureTableQueryCondition`.

To create a simple filter on a record property:

```csharp
ITableQuery query = new AzureTableQuery()
    .Where(
        AzureTableQueryCondition("Column1", TableQueryComparison.Equal, "10"));
```

You can combine conditions with logical operations. Here is an example:

```csharp
ITableQuery query = new AzureTableQuery()
    .Where(
        AzureTableQueryCondition.Or(
            AzureTableQueryCondition.And(
                AzureTableQueryCondition.IsEqual("Column1", 1),
                AzureTableQueryCondition.IsEqual("Column2", 1)),
            AzureTableQueryCondition.Or(
                AzureTableQueryCondition.And(
                    AzureTableQueryCondition.IsEqual("Column1", 2),
                    AzureTableQueryCondition.IsEqual("Column2", 2)),
                AzureTableQueryCondition.And(
                    AzureTableQueryCondition.IsEqual("Column1", 3),
                    AzureTableQueryCondition.IsEqual("Column2", 3)))));
```

You can also select the columns returned in the result:

```csharp
ITableQuery query = new AzureTableQuery()
    .Where(...)
    .Select("Column1", "Column2");
```

Or limit the number of records returned:

```csharp
ITableQuery query = new AzureTableQuery()
    .Where(...)
    .Take(10);
```

#### Known Limitations

- The Azure Storage service limits the values that can be stored on `DateTime` or `DateTimeOffset` properties. The minimum value is `12:00 midnight, January 1, 1601 A.D. (C.E.), UTC`.
- `DateTimeOffset` values are always converted to UTC before being stored.