---
title: PostgreSQL
draft: false
date: 2025-04-08
tags:
  - Azure
  - PostgreSQL
  - Databases
---
Here I collect my learnings on running PostgreSQL in Azure.

![[PostgreSQL.png]]
## PostgreSQL

- [[postgresql_on_azure]] - PostgreSQL on Azure
- [[hosting_options]] - Azure Database for PostgreSQL vs. Azure Cosmos DB for PostgreSQL
- [[python_libraries]] - Azure Database for PostgreSQL Python libraries
- [[postgresql_case_sensitivity]] - Azure Databaes for PostgreSQL and Case
- [[security_and_authentication]] - Security and Authentication Options for Kubernetes Solutions with Azure Database for PostgreSQL Flexible Server

## Database Replication

- [[database_replication]] - Database Replication in PostgreSQL
- [[syncronous_replication_postgresql_azure]] - Synchronous Read-Only Replication in Azure PostgreSQL: Possibilities and Limitations
- [[database_replicaton_performance]] - Database Replication Performance Tests
- [[replication-perf-azure-aws]] - Performance Comparison: PostgreSQL Replication on AWS Aurora vs Azure Database Flexible Server
- [[replication_perf_comparison_azure]] - Replication Performance Comparison of PostgreSQL Offerings on Microsoft Azure

## Azure Load Testing

- [[load_test_configuration]] - Azure Load Test Configuration
- [[load_test_1_saturation]] - Load Test 1: Saturating the System
- [[load_test_2_timed]] - Load Test 2: Timed Testing

- [[azure_loadtest_configuration]] - Configuration I used in Azure Load Test

- [[troubleshooting_azure_load_testing]] - Troubleshooting Failed Tests in Azure Load Testing with JMeter

## Code Sample

- [[code_samples]] - Code samples for this learning project

## Mirroring for Azure Database for PostgreSQL in Fabric

- [Announcing Mirroring for Azure Database for PostgreSQL in Microsoft Fabric for Public Preview](https://techcommunity.microsoft.com/blog/adforpostgresql/announcing-mirroring-for-azure-database-for-postgresql-in-microsoft-fabric-for-p/4396750)
- [Mirroring Azure Database for PostgreSQL flexible server](https://learn.microsoft.com/en-us/fabric/database/mirrored-database/azure-database-postgresql)
- [Tutorial: Configure Microsoft Fabric mirrored databases from Azure Database for PostgreSQL flexible server](https://learn.microsoft.com/en-us/fabric/database/mirrored-database/azure-database-postgresql-tutorial)
- [Monitor Fabric mirrored database replication](https://learn.microsoft.com/en-us/fabric/database/mirrored-database/monitor)

![[fabric-mirroring-database-postgresql.png]]

### Limitations

- Currently, Mirroring doesn't support Azure Database for PostgreSQL flexible server behind an Azure Virtual Network or private networking.
- You need to update your Azure Database for PostgreSQL flexible server firewall rules to [Allow public network access](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-networking-servers-deployed-public-access-enable-public-access), and enable the [Allow Azure services](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-networking-public#allow-all-azure-ip-addresses) option to connect to your Azure Database for PostgreSQL flexible server.
- Active transactions continue to hold the write ahead log (WAL) truncation until the transaction commits and the mirrored Azure Database for PostgreSQL flexible server catches up, or the transaction aborts. Long-running transactions might result in the WAL filling up more than usual. WAL on source Azure Database for PostgreSQL flexible server should be monitored so that storage does not fill up.
- The source Azure Database for PostgreSQL flexible server can be either a **General Purpose** or Memory **Optimized** compute tier. **Burstable** compute tier is not supported as source for mirroring.

## Load Testing

- [Load testing databases with Azure Load Testing](https://techcommunity.microsoft.com/blog/appsonazureblog/load-testing-databases-with-azure-load-testing/3846557)
- [Define fail criteria for load tests by using Azure Load Testing](https://learn.microsoft.com/en-us/azure/load-testing/how-to-define-test-criteria?tabs=portal#auto-stop-configuration)
- [Use secrets and environment variables in Azure Load Testing](https://learn.microsoft.com/en-us/azure/load-testing/how-to-parameterize-load-tests?wt.mc_id=azloadtesting_techcomm202206_blog_cnl&tabs=jmeter#secrets)

## Azure Load Test Setup using Terraform

- GitHub Repo: https://github.com/TechPreacher/azure_loadtest_terraform
- Blog Post: https://corti.com/start-load-testing-azure-postgresql-flexible-server-with-read-only-replica-using-azure-load-testing-in-minutes/

## Aurora Replica Lag

See [Replication with Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html#Aurora.Replication.Replicas)
>As a result, all Aurora Replicas return the same data for query results with minimal replica lag. This lag is usually much less than 100 milliseconds after the primary instance has written an update. Replica lag varies depending on the rate of database change. That is, during periods where a large amount of write operations occur for the database, you might see an increase in replica lag.

