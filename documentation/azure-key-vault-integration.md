# Azure Key Vault Integration for Flyway

This document describes how to configure Flyway to use Azure Key Vault for securely managing database credentials and other sensitive configuration values.

## Overview

The Azure Key Vault integration allows Flyway to retrieve secrets (such as database passwords, connection strings, and usernames) directly from Azure Key Vault at runtime, avoiding the need to store sensitive values in configuration files or environment variables.

## Prerequisites

- An Azure subscription with an active Key Vault instance
- Appropriate permissions to read secrets from the Key Vault (e.g., `Key Vault Secrets User` role)
- One of the following authentication methods configured:
  - Azure Managed Identity (recommended for production)
  - Azure Service Principal (client ID + client secret)
  - Azure CLI credentials (recommended for local development)
  - Azure Environment credentials

## Configuration

### Maven / Gradle Plugin

Add the Azure Key Vault resolver dependency to your build file:

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-azure-keyvault</artifactId>
    <version>${flyway.version}</version>
</dependency>
```

### Flyway Configuration

In your `flyway.conf` or application properties, reference Key Vault secrets using the `${akv:secret-name}` placeholder syntax:

```properties
# Azure Key Vault configuration
flyway.azure.keyVault.url=https://my-keyvault.vault.azure.net/

# Reference secrets by name
flyway.url=jdbc:postgresql://myhost:5432/mydb
flyway.user=${akv:db-username}
flyway.password=${akv:db-password}
```

### Authentication

#### Managed Identity (Recommended)

When running on Azure (App Service, AKS, VM, etc.), Managed Identity is automatically used if no other credentials are configured:

```properties
flyway.azure.keyVault.url=https://my-keyvault.vault.azure.net/
```

#### Service Principal

```properties
flyway.azure.keyVault.url=https://my-keyvault.vault.azure.net/
flyway.azure.keyVault.clientId=<your-client-id>
flyway.azure.keyVault.clientSecret=<your-client-secret>
flyway.azure.keyVault.tenantId=<your-tenant-id>
```

#### Environment Variables

Alternatively, set the following environment variables:

```bash
export AZURE_CLIENT_ID=<your-client-id>
export AZURE_CLIENT_SECRET=<your-client-secret>
export AZURE_TENANT_ID=<your-tenant-id>
```

## Secret Versioning

To reference a specific version of a secret, use the `@version` suffix:

```properties
flyway.password=${akv:db-password@abc123def456}
```

If no version is specified, the latest enabled version is used.

## Security Considerations

- **Least Privilege**: Grant only `Key Vault Secrets User` (read) permissions to the Flyway identity — never `Key Vault Administrator`.
- **Network Restrictions**: Consider restricting Key Vault access to specific virtual networks or IP ranges.
- **Audit Logging**: Enable Key Vault diagnostic logging to track secret access.
- **Secret Rotation**: When rotating secrets, update the Key Vault secret value; Flyway will pick up the new value on the next run.

## Troubleshooting

### `SecretNotFoundException`

Verify the secret name matches exactly (case-sensitive) and that the secret is enabled in Key Vault.

### `AuthenticationFailedException`

Ensure the identity running Flyway has been granted the `Key Vault Secrets User` role on the Key Vault resource.

### `KeyVaultErrorException: Forbidden`

The Key Vault firewall may be blocking access. Check the Key Vault network settings and ensure the client IP or VNet is allowed.

## Example: Spring Boot Integration

```yaml
spring:
  flyway:
    url: jdbc:sqlserver://myserver.database.windows.net:1433;database=mydb
    user: ${akv:sql-username}
    password: ${akv:sql-password}
    azure:
      key-vault:
        url: https://my-keyvault.vault.azure.net/
```

## Related

- [Azure Key Vault Documentation](https://docs.microsoft.com/en-us/azure/key-vault/)
- [Azure Identity SDK for Java](https://docs.microsoft.com/en-us/java/api/overview/azure/identity-readme)
- [Flyway Configuration Reference](https://documentation.red-gate.com/flyway/flyway-cli-and-api/configuration/parameters)
