---
title: "Synchronous Read-Only Replication in Azure PostgreSQL: Possibilities and Limitations"
draft: false
date: 2025-04-14
tags:
  - PostgreSQL
  - Azure
  - Performance
---
Quote from: https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas

*Read replicas are primarily designed for scenarios where offloading queries is beneficial, and a slight lag is manageable. They're optimized to provide near real time updates from the primary for most workloads, making them an excellent solution for read-heavy scenarios. However, it's important to note that they aren't intended for synchronous replication scenarios requiring up-to-the-minute data accuracy. While the data on the replica eventually becomes consistent with the primary, there might be a delay, which typically ranges from a few seconds to minutes, and in some heavy workload or high-latency scenarios, this delay could extend to hours.*

Azure Database for PostgreSQL's replication capabilities are designed primarily for scaling read workloads and providing disaster recovery options. Achieving truly synchronous read-only replication with guaranteed up-to-the-minute accuracy under heavy load presents significant challenges due to both architectural limitations and performance considerations.

## The Challenge of Synchronous Replication in Azure PostgreSQL

### Current Azure PostgreSQL Replication Architecture

Azure Database for PostgreSQL flexible server supports read replicas using **asynchronous** replication technology. These replicas are updated using PostgreSQL's native physical replication[2][8]. While this approach efficiently scales read workloads, it comes with an important limitation:

> "The feature is meant for scenarios where replication lag is acceptable, and is meant for offloading queries. It isn't meant for synchronous replication scenarios where replica data is expected to be up to date."[2]

The delay between primary and replica can range from seconds to minutes or even hours under heavy workloads[2]. This inherent lag means that read replicas cannot guarantee up-to-the-minute accuracy for read operations.

### Configuration Limitations in Azure PostgreSQL

While PostgreSQL itself does support synchronous commit options through the `synchronous_commit` parameter, there appear to be significant limitations when using Azure's managed service:

1. Users have reported being unable to set `synchronous_commit` to "remote_apply" in Azure PostgreSQL Flexible Server[10], which would be the necessary setting for ensuring that read replicas have applied all transactions before they become visible.

2. A Microsoft representative confirmed this limitation, noting: "Seems like it is not possible to change to parameters to 'remote_apply'. It is not supported yet in Azure PostgreSQL Flexible."[10]

## Understanding PostgreSQL Synchronous Commit Options

Standard PostgreSQL offers several synchronous commit options that provide different levels of durability guarantees:

- `synchronous_commit = off`: Transactions don't wait for writes to be confirmed
- `synchronous_commit = local`: Waits for transaction to be flushed to local disk
- `synchronous_commit = on`: Waits for transaction to be confirmed as written to disk on standby servers
- `synchronous_commit = remote_write`: Waits for transaction to be written to the standby's operating system
- `synchronous_commit = remote_apply`: Waits for transaction to be applied to the database on the standby[4]

The last option, `remote_apply`, would theoretically ensure that read replicas always have up-to-date data. However, this setting appears unavailable in Azure PostgreSQL Flexible Server.

## Performance Implications of Synchronous Replication

Even if technically possible, synchronous replication comes with significant performance costs:

> "With synchronous replication, the replication delay directly affects the elapsed time of transactions on the master."[4]

This means that under heavy load, write performance would be severely impacted as each transaction would need to wait for confirmation from replica servers before completing.

## Potential Alternatives and Workarounds

### 1. Monitor and Manage Replication Lag

Azure provides metrics for monitoring replication lag[8]. Applications could potentially:
- Check the lag metric before critical read operations
- Direct critical reads to the primary when fresh data is essential
- Use replicas only for operations that can tolerate some staleness

### 2. Application-Level Solutions

Implement application-level caching or consistency strategies:
- Write-through caching for critical data
- Session affinity to direct related operations to the same server
- Conditional reads based on timestamps or version numbers

### 3. Logical Replication

Azure Database for PostgreSQL flexible server supports logical replication[5], which might offer more granular control over replicated objects, though it remains asynchronous.

### 4. Load Balancing Solutions

Tools like pgpool (mentioned in the search results[10]) can help with load balancing between primary and replica servers, but they don't solve the fundamental synchronicity issue.

## Conclusion

Currently, achieving true synchronous read-only replication with guaranteed up-to-the-minute accuracy in Azure PostgreSQL is not directly supported through the platform's built-in capabilities. The `synchronous_commit` parameter with `remote_apply` setting, which would provide this functionality, appears to be unavailable in Azure PostgreSQL Flexible Server at this time[10].

For applications requiring absolute read consistency, the safest approach is to direct those specific queries to the primary server, while using read replicas for queries that can tolerate some data staleness. This hybrid approach allows you to balance performance with data accuracy requirements.

For future developments, it's worth monitoring whether Azure adds support for the `synchronous_commit = remote_apply` parameter, as this would enable truly synchronous read-replicas, albeit with a performance trade-off for write operations.

Sources
[1] Automated Continuous PostgreSQL Replication to Azure SQL https://www.cdata.com/kb/tech/postgresql-sync-azure.rst
[2] Read replicas - Azure Cosmos DB for PostgreSQL | Microsoft Learn https://learn.microsoft.com/en-us/azure/cosmos-db/postgresql/concepts-read-replicas
[3] Read Replicas for Azure Database for PostgreSQL now in preview https://azure.microsoft.com/en-us/blog/read-replicas-for-azure-database-for-postgresql-now-in-preview/
[4] Evolution of Fault Tolerance in PostgreSQL: Synchronous Commit https://www.enterprisedb.com/blog/evolution-fault-tolerance-postgresql-synchronous-commit
[5] Azure Database for PostgreSQL flexible server | Microsoft Learn https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical
[6] Enhancing Data Resilience in Azure Database for PostgreSQL ... https://www.linkedin.com/pulse/enhancing-data-resilience-azure-database-postgresql-flexible-michel-3os4f
[7] PostgreSQL Replication – High Availability and Load Balancing https://www.youtube.com/watch?v=hW4k5WV2pec
[8] Read replicas - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas
[9] Creating and Monitoring Read Replicas in Azure Postgres Flexible ... https://techcommunity.microsoft.com/blog/adforpostgresql/creating-and-monitoring-read-replicas-in-azure-postgres-flexible-server/3903826
[10] Unable to set "synchronous_commit" the parameter for Azure ... https://learn.microsoft.com/en-in/answers/questions/2028429/unable-to-set-synchronous-commit-the-parameter-for
[11] Can the replica be safely used for read-only operations in the ... https://learn.microsoft.com/en-us/answers/questions/2148006/can-the-replica-be-safely-used-for-read-only-opera
[12] Chapter 26. High Availability, Load Balancing, and Replication https://www.postgresql.org/docs/current/high-availability.html
[13] Creating Replicas in Azure Database for PostgreSQL - SQL Shack https://www.sqlshack.com/creating-replicas-in-azure-database-for-postgresql/
[14] Postgres Replication Fundamentals You Need to Know - BryteFlow https://bryteflow.com/postgresql-replication-fundamentals-you-need-to-know/
[15] Step-by-Step Guide to Postgres Replication with Azure Event Hubs https://risingwave.com/blog/step-by-step-guide-to-postgres-replication-with-azure-event-hubs/
[16] Working with read replicas for Amazon RDS for PostgreSQL https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html
[17] High availability (Reliability) in Azure Database for PostgreSQL https://learn.microsoft.com/en-us/azure/reliability/reliability-postgresql-flexible-server
[18] Postgres server switching to read only mode during COPY on azure? https://www.reddit.com/r/PostgreSQL/comments/1asr5p8/postgres_server_switching_to_read_only_mode/
[19] Geo-replication - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas-geo
[20] Documentation: 17: 19.6. Replication - PostgreSQL https://www.postgresql.org/docs/current/runtime-config-replication.html
[21] How to Sync Azure Postgres from one instance to other instance https://learn.microsoft.com/en-us/answers/questions/2151984/how-to-sync-azure-postgres-from-one-instance-to-ot
[22] concepts-compare-single-server-flexible-server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-compare-single-server-flexible-server
[23] Overview of business continuity - Azure Database for PostgreSQL ... https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-business-continuity
[24] Read queries on replicas - Azure SQL Database ... - Learn Microsoft https://learn.microsoft.com/en-us/azure/azure-sql/database/read-scale-out?view=azuresql
[25] Azure Database for PostgreSQL flexible server | Microsoft Learn https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical
[26] Server parameters in Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-server-parameters
[27] Promote read replicas - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas-promote
[28] Increased resource utilization on Azure PostgreSQL replica https://stackoverflow.com/questions/69053656/increased-resource-utilization-on-azure-postgresql-replica
[29] Is it possible to have a read replicas of azure flexible postgresql ... https://stackoverflow.com/questions/70081184/is-it-possible-to-have-a-read-replicas-of-azure-flexible-postgresql-server-with
[30] [Answered] How do you scale Azure PostgreSQL? - Dragonfly https://www.dragonflydb.io/faq/azure-postgresql-scaling
[31] Is it possible to have a read/write replicas of azure postgresql https://stackoverflow.com/questions/68635152/is-it-possible-to-have-a-read-write-replicas-of-azure-postgresql
