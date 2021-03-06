# PRIMAVERA Hydrogen Release Notes

> These release notes include only the most important releases.

### <a name="2.0.7.350"></a>Version 2.0.7.350

**Primavera.Hydrogen.Security.Azure**

- Replaced the Azure KeyVault external package to use `Azure.Security.KeyVault.Certificates` (version 4.1.0), `Azure.Security.KeyVault.Keys` (version 4.1.0) and `Azure.Security.KeyVault.Secrets` (version 4.1.0).
- **[Breaking]** Added property `TenantId` to `AzureKeyVaultSecretsStorageOptions` to set the client application client credentials (KeyVault service clients no longer accept authentication callbacks to identify the authorization server and the tenant identifier).
- **[Breaking]** Renamed property `AutomaticAuthenticationEnabled` in `AzureKeyVaultSecretsStorageOptions` to `ManagedIdentityEnabled`.
- Renamed member `Automatic` in `AzureKeyVaultAuthenticationKind` to `ManagedIdentity` and added `Developer`.
- **[Breaking]** `ISecretsStorageService.DeleteSecretAsync()` now returns a `Task`, not a `Task<Secret>`.
- **[Breaking]** Properties `CreatedOn`, `UpdatedOn`, `NotBefore`, `Expires` of `Attributes` are now `DateTimeOffset?`, not `DateTime?`.
- **[Breaking]** `Certificate.SecretIdentifier` is now a `Uri`, not a `SecretIdentifier`.
- **[Breaking]** `Certificate.KeyIdentifier` is now a `Uri`, not a `KeyIdentifier`.
- **[Breaking]** Properties `Identifier` and `Storage` in `CertificateIdentifier`, `KeyIdentifier` and `SecretIdentifier` are now `Uri`, not `string`.
- `IAzureTokenService` and the whole `Primavera.Security.Azure.AppAuthentication` namespace were removed.

### <a name="2.0.7.348"></a>Version 2.0.7.348

**Primavera.Hydrogen.Search.Azure**

- Replaced the Azure Search external package to use `Azure.Search.Documents` (version 11.1.1).
- Added `Search<T>` and `SearchAsync<T>` to `ISearchDocumentsOperations`.

### <a name="2.0.7.347"></a>Version 2.0.7.347

**General**

- Updated Primavera.Hydrogen.DesignTime.Configuration to 2.0.0.48.
- **[Breaking]** Removed all obsolete types (e.g. `TestServerFixture`) and obsolete members (e.g. `SmartGuard.IsEmail()`).

**Primavera.Hydrogen.Storage.Azure**

- Replaced the Azure Blob Storage external package to use `Azure.Storage.Blobs` (version 12.7.0).
- **[Breaking]** Removed the `IBlockBlobReference.UploadFromStreamAsync()` overload that accepted `length`.
- **[Breaking]** Removed `mode` from `IBlockBlobReference.DownloadToFileAsync()`.
- **[Breaking]** Removed `IBlobReference.Properties` `IBlobReference.BlobType`. Use `IBlobReference.GetPropertiesAsync()` instead.