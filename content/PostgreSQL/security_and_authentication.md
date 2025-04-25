---
title: Security and Authentication Options for Kubernetes Solutions with Azure Database for PostgreSQL Flexible Server
draft: false
date: 2025-04-10
tags:
  - PostgreSQL
  - Azure
  - Security
  - Authentication
---
This document explores the security and authentication options available when deploying solutions on Azure Kubernetes Service (AKS) that use Azure Database for PostgreSQL Flexible Server as their data layer. As organizations increasingly adopt containerized architectures, securing the database tier becomes essential to maintain data integrity, confidentiality, and availability.

## Authentication Methods

Azure Database for PostgreSQL Flexible Server offers multiple authentication mechanisms, each providing different security levels and use cases.

### Microsoft Entra ID Authentication

Microsoft Entra ID (formerly Azure AD) authentication is a robust mechanism that lets you connect to PostgreSQL using identities defined in Microsoft Entra ID, eliminating the need to manage separate database credentials[11].

Key benefits include:
- Centralized identity management across Azure services
- Support for token-based authentication for applications
- Management of password policies and rotation in a single place
- Multiple authentication forms supported by Microsoft Entra ID
- Ability to manage database permissions using external groups[7]

In this architecture, your Kubernetes application can request a token from Azure Instance Metadata Service, which is then used for authentication with the database. This process works through these steps:
1. Your application requests a token from the Azure Instance Metadata Service identity endpoint
2. A call is made to Microsoft Entra ID to request an access token
3. A JWT access token is returned by Microsoft Entra ID
4. Your application sends the access token to your Flexible Server
5. The Flexible Server validates the token with Microsoft Entra ID[11]

### Managed Identities

Managed identities provide an automatically managed identity in Microsoft Entra ID for applications running in AKS to connect to PostgreSQL without handling credentials[15].

There are two types:
- **System-assigned**: Tied directly to the lifecycle of an Azure resource
- **User-assigned**: Created independently and can be assigned to multiple resources[15]

Using managed identities eliminates credential management challenges and provides these advantages:
- No need to handle or store credentials in your application code
- Credentials aren't accessible to developers, reducing potential exposure
- Can be used to authenticate to any resource supporting Microsoft Entra authentication
- Available at no additional cost[15]

For Kubernetes deployments, this approach is particularly valuable as it removes sensitive database credentials from pod specifications and configuration files[6].

### Password-Based Authentication

For scenarios requiring traditional password authentication, PostgreSQL Flexible Server supports both local PostgreSQL authentication and hybrid approaches combining Microsoft Entra ID with local authentication[7].

When using password-based authentication, it's strongly recommended to use SCRAM (Salted Challenge Response Authentication Mechanism) with SHA-256 hashing instead of the older MD5 mechanism. SCRAM offers several security benefits:
- Protection against rainbow-table attacks
- Prevention of man-in-the-middle attacks
- Defense against stored password attacks
- Support for multiple hashing algorithms
- Support for passwords containing non-ASCII characters[5]

## Network Security Options

Securing network access to your PostgreSQL instance is a critical layer of defense in your overall security architecture.

### Private Network Access

Azure Database for PostgreSQL Flexible Server allows full private access using Azure Virtual Network (VNet) integration:
- Servers can only be reached through private IP addresses
- Public access is denied, and servers can't be accessed using public endpoints
- Network traffic remains within the Azure network[5][8]

This integration ensures that only resources within your defined network boundary can communicate with your database instance, significantly reducing the attack surface[7].

### Public Access with IP Firewall Rules

For use cases requiring public accessibility, you can configure IP-based firewall rules:
- Define specific IP addresses or ranges allowed to connect
- All other connection attempts are blocked by default
- Easily manage rules through the Azure Portal, CLI, or ARM templates[14]

### Transport Layer Security (TLS/SSL)

To secure data in transit, Azure Database for PostgreSQL Flexible Server enforces TLS encryption:
- TLS version 1.2 is enforced by default
- Client applications can be configured to verify certificates for secure connections
- Specific connection string parameters like `sslmode=verify-full` can enforce certificate verification[14]

For applications requiring certificate verification, you can download the necessary root CA certificates:

```plain
psql "sslmode=verify-full sslrootcert=c:\\ssl\DigiCertGlobalRootCA.crt.pem host=mydemoserver.postgres.database.azure.com dbname=postgres user=myadmin"
```

## Encryption and Data Protection

Azure Database for PostgreSQL Flexible Server provides comprehensive data protection through multiple encryption mechanisms.

### Data-at-Rest Encryption

All data, including backups and temporary files created during query execution, is automatically encrypted:
- Uses FIPS 140-2 validated cryptographic module
- Implements AES 256-bit cipher included in Azure storage
- Keys can be system-managed (default) or customer-managed[8]

### Customer-Managed Keys

For enhanced control over encryption, customer-managed keys allow organizations to:
- Leverage Azure Key Vault capabilities
- Use key-encryption keys (KEKs) to encrypt the data-encryption keys (DEKs)
- Set up segregation-of-duties compliant access to keys and data
- Configure required policies and roles administered against Microsoft Entra ID[1]

This feature is particularly valuable for organizations with strict compliance requirements or those needing complete control over their encryption keys[5].

## Identity and Access Management

Proper identity and access management is essential for maintaining secure database operations.

### Role-Based Access Control (RBAC)

PostgreSQL Flexible Server supports role-based access control to provide fine-grained permission management:
- Microsoft Entra administrators can manage both Microsoft Entra users and local users
- PostgreSQL administrators can create only local password-based users
- Multiple Microsoft Entra administrators can be configured concurrently
- Database permissions for non-admin Microsoft Entra roles are managed similarly to regular roles[11]

### Service Principals and Managed Identities

For application authentication from Kubernetes, you can use:
- Service principals with access tokens
- Managed identities to eliminate the need for credential management

A practical example for using a service principal with access tokens:
```bash
az login --service-principal -u "ID" -p "SECRET" --tenant "TENANT"
export PGPASSWORD=$(az account get-access-token --resource-type oss-rdbms --query "[accessToken]" -o tsv)
psql "host=mydatabase.database.azure.com user=service-principal dbname=db sslmode=require"
```

However, access tokens typically expire after 60 minutes, which can cause connectivity issues after VM restarts. To address this, you can increase token lifetime or implement automated token refresh mechanisms[2].

### Workload Identity Federation

For Kubernetes deployments, Workload Identity Federation enables a passwordless connection model:
- Establishes trusted connectivity between an external identity provider and Microsoft Entra ID
- Allows workloads to use tokens issued by their identity provider
- Exchanges these tokens for valid Microsoft Entra ID access tokens
- Uses these tokens to access Azure services like PostgreSQL Flexible Server[12]

This approach provides a more secure and operationally sound method for connecting Kubernetes applications to Azure resources without managing credentials[12].

## Security Best Practices

To maximize security posture when connecting Kubernetes applications to PostgreSQL Flexible Server, consider these best practices:

### Authentication Recommendations

- Use Microsoft Entra ID authentication when possible
- Implement SCRAM password hashing for local authentication
- Leverage managed identities for application access
- Consider Workload Identity Federation for non-Azure Kubernetes deployments[7][13]

### Network Security

- Deploy PostgreSQL Flexible Server in isolated Azure Virtual Networks
- Use Network Security Groups for network-level access control
- Implement Private Link for secure private connectivity
- Enable TLS 1.2 or higher for all connections[7][13]

### Ongoing Security Management

- Regularly rotate credentials and encryption keys
- Implement Azure Policy for automated security compliance
- Monitor database activity through logs and metrics
- Update to the latest PostgreSQL version with necessary patches[5][7]

### Protecting Against Common Threats

Be aware of specific PostgreSQL security risks when deploying in Kubernetes:
- Avoid "trust" authentication which provides access without passwords
- Be cautious with PostgreSQL user roles like pg_execute_server_program that can execute OS commands
- Monitor for suspicious activity that could indicate cryptojacking attempts
- Run PostgreSQL as a non-root user for defense-in-depth[3][9]

## Conclusion

Azure Database for PostgreSQL Flexible Server provides a robust set of security and authentication options for applications running in Kubernetes. By leveraging Microsoft Entra ID authentication, managed identities, network isolation, and proper encryption practices, organizations can establish a secure foundation for their data layer.

The optimal approach depends on specific organizational requirements, compliance needs, and architectural considerations. For most modern applications, a combination of Microsoft Entra authentication, managed identities, VNet integration, and customer-managed keys provides a comprehensive security posture while maintaining operational efficiency.

By following the security best practices outlined in this report, organizations can protect their PostgreSQL databases from common threats while ensuring their applications running in Kubernetes can securely and efficiently access the data they need.

Sources
[1] New Features for Azure Database for PostgreSQL Flexible Server https://www.infoq.com/news/2022/11/azure-database-postgresql/
[2] Azure PostgreSQL Flexible Server enabled with Azure AD - How to give application access after restarts https://stackoverflow.com/questions/76109854/azure-postgresql-flexible-server-enabled-with-azure-ad-how-to-give-application
[3] Securing PostgreSQL from Cryptojacking Campaigns | CSA https://cloudsecurityalliance.org/blog/2023/04/12/securing-postgresql-from-cryptojacking-campaigns-in-kubernetes
[4] Connecting Azure Kubernetes Service and Azure Database for PostgreSQL using Azure Key Vault https://stackoverflow.com/questions/64004272/connecting-azure-kubernetes-service-and-azure-database-for-postgresql-using-azur
[5] Security - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-security
[6] Connect With Managed Identity - Azure Database for PostgreSQL ... https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-connect-with-managed-identity
[7] Security Hardening Best Practices for PostgreSQL Flexible Server https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/security-hardening-best-practices-for-postgresql-flexible-server/ba-p/3843404
[8] Overview - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-gb/azure/postgresql/flexible-server/overview
[9] Securing PostgreSQL from Cryptojacking Campaigns in Kubernetes https://www.crowdstrike.com/en-us/blog/securing-postgresql-from-cryptojacking-campaigns-in-kubernetes/
[10] Overview of deploying a highly available PostgreSQL database on AKS with Azure CLI - Azure Kubernetes Service https://learn.microsoft.com/en-us/azure/aks/postgresql-ha-overview
[11] Microsoft Entra authentication with Azure Database for PostgreSQL ... https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-azure-ad-authentication
[12] Connect your Kubernetes application to your database without any ... https://alexisplantin.fr/workload-identity-federation/
[13] Security Hardening Best Practices for PostgreSQL Flexible Server https://www.linkedin.com/posts/varundhawan_security-hardening-best-practices-for-postgresql-activity-7074036452975652865-zmDH
[14] Encrypted connectivity using TLS/SSL - Azure Database for ... https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-connect-tls-ssl
[15] Managed identities - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-identity
[16] Active Directory authentication - Azure Database for PostgreSQL - Single Server https://learn.microsoft.com/en-us/azure/postgresql/single-server/concepts-azure-ad-authentication
[17] Networking overview with public access (allowed IP addresses) - Azure Database for PostgreSQL - Flexible Server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-networking-public
[18] How to Run a Postgres Database in Azure Kubernetes Service and Integrate it with a Node.js Express Application https://www.freecodecamp.org/news/how-to-run-postgres-in-kubernetes/
[19] Configuring Azure AD with PostgreSQL in TSB | Service Bridge https://docs.tetrate.io/service-bridge/operations/postgresql/azure-credentials
[20] Azure Database for PostgreSQL Flexible Server Privilege Escalation and Remote Code Execution https://msrc.microsoft.com/blog/2022/04/azure-database-for-postgresql-flexible-server-privilege-escalation-and-remote-code-execution/
[21] Connecting to an Azure Database for PostgreSQL flexible server from an Azure Function serverless https://stackoverflow.com/questions/76371555/connecting-to-an-azure-database-for-postgresql-flexible-server-from-an-azure-fun
[22] Connect to Azure Kubernetes Service - Azure Database for PostgreSQL - Single Server https://learn.microsoft.com/en-us/azure/postgresql/single-server/concepts-aks
[23] Integrate Azure Database for PostgreSQL with Service Connector https://learn.microsoft.com/en-us/azure/service-connector/how-to-integrate-postgres
[24] Enhance your Azure PostgreSQL Flexible Server security posture with Azure Defender https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/enhance-your-azure-postgresql-flexible-server-security-posture/ba-p/3999093
[25] Create Azure Database for PostgreSQL Flexible Server | Using Azure Portal https://www.youtube.com/watch?v=AeQi195rg_4
[26] Deploy a highly available PostgreSQL database on AKS with Azure CLI - Azure Kubernetes Service https://learn.microsoft.com/en-us/azure/aks/deploy-postgresql-ha
[27] How to enable Azure Active Directory Authentication for PostgreSQL? https://stackoverflow.com/questions/75072194/how-to-enable-azure-active-directory-authentication-for-postgresql
[28] Azure security baseline for Azure Database for PostgreSQL https://learn.microsoft.com/en-us/security/benchmark/azure/baselines/azure-database-for-postgresql-flexible-server-security-baseline
[29] Azure Database for PostgreSQL flexible server | Microsoft Learn https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-networking-ssl-tls
[30] Deploy Django app on AKS with Azure Database for PostgreSQL ... https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/tutorial-django-aks-database
[31] Microsoft Entra authentication with Azure Database for PostgreSQL ... https://docs.azure.cn/en-us/postgresql/flexible-server/concepts-azure-ad-authentication
[32] Azure Database for PostgreSQL flexible server - Learn Microsoft https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-configure-sign-in-azure-ad-authentication
[33] PostgreSQL best practices - Trend Micro https://www.trendmicro.com/cloudoneconformity/knowledge-base/azure/PostgreSQL/
[34] Connections and Authentication / SSL server parameters https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/server-parameters-table-connections-and-authentication-ssl
[35] Consuming Postgres Flexible Server from ACA with Managed Identity https://github.com/microsoft/azure-container-apps/discussions/1227
[36] Security Hardening Best Practices for PostgreSQL Flexible Server https://azureaggregator.wordpress.com/2023/06/11/security-hardening-best-practices-for-postgresql-flexible-server/
[37] Azure PostgreSQL flexible Server SSL enforcement disabled https://orca.security/resources/blog/azure-postgresql-flexible-server-ssl-enforcement-disabled/
