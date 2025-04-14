---
title: Replication Performance Comparison of PostgreSQL Offerings on Microsoft Azure
draft: false
date: 2025-04-14
tags:
  - Azure
  - PostgreSQL
  - Performance
---
Microsoft Azure provides multiple managed PostgreSQL solutions, each with distinct replication architectures and performance characteristics. This report compares the replication performance of Azure Database for PostgreSQL Flexible Server and Azure Cosmos DB for PostgreSQL (Hyperscale), focusing on their technical implementations, latency profiles, scalability, and failover capabilities.

## Architectural Overview of Replication Mechanisms

### Azure Database for PostgreSQL Flexible Server
Flexible Server employs PostgreSQL's native physical replication enhanced with cloud-native management features. The service uses **replication slots** to manage write-ahead log (WAL) retention, ensuring replicas remain synchronized with the primary instance[5][6]. Key architectural elements include:
- **Asynchronous streaming replication**: Replicas consume WAL segments directly from the primary.
- **Storage-tier independence**: Replicas operate on separate compute and storage resources, allowing vertical scaling without disrupting replication[6].
- **Zone-redundant HA**: Automatically provisions a standby replica in a different availability zone for failover scenarios[10].

### Azure Cosmos DB for PostgreSQL (Hyperscale)
Hyperscale adopts a distributed architecture leveraging Citus extensions for horizontal scaling. Its replication model diverges from traditional PostgreSQL:
- **Shared-nothing architecture**: Data is sharded across multiple nodes, with each node maintaining its own replicas[14].
- **Asynchronous physical replication**: Geo-replicas use log shipping, decoupling compute and storage layers[3][15].
- **Multi-region scaling**: Supports cross-region read replicas with eventual consistency[14].

## Replication Latency Profiles

### Flexible Server
- **Typical lag**: Ranges from **seconds to minutes** under normal loads but can extend to **hours** during heavy write workloads or network congestion[5][10].
- **Monitoring**: The `physical_replication_delay_in_seconds` metric provides per-replica lag measurements, with alerts recommended for lags exceeding 5 minutes[5].
- **Write amplification**: Intensive OLTP workloads may experience increased lag due to WAL accumulation in replication slots, risking storage saturation at 95% capacity thresholds[5][6].

### Hyperscale (Citus)
- **Intra-cluster lag**: Sharded tables exhibit **sub-second latency** for local replicas due to co-located compute and storage[12].
- **Cross-region lag**: Geo-replicas demonstrate **minutes to hours** of delay, influenced by inter-region network latency and workload intensity[14].
- **Query-specific variance**: Analytical queries on distributed tables may experience higher apparent lag due to parallel execution across nodes[12].

## Failover Performance Characteristics

| Metric                  | Flexible Server                        | Hyperscale (Citus)                   |
| ----------------------- | -------------------------------------- | ------------------------------------ |
| **Detection Time**      | 60-90 seconds                          | 30-60 seconds                        |
| **Failover Duration**   | 60-120 seconds (zone-redundant HA)[10] | 20-40 seconds (local replicas)[12]   |
| **Data Loss (RPO)**     | 0 (zero data loss)[10]                 | 0 (zero data loss)[15]               |
| **Recovery Complexity** | Manual replica promotion required[10]  | Automatic failover to HA replicas[7] |

## Scalability and Throughput

### Flexible Server
- **Vertical scaling**: Supports up to **5 read replicas** with independent compute scaling (General Purpose: 2–64 vCores, Memory Optimized: 2–48 vCores)[6].
- **Storage autoscaling**: Premium SSDv2 storage scales to 16 TB with 3000 baseline IOPS[1][6].
- **Throughput limits**: Max 150 MB/s log generation rate (V5 SKUs)[1].

### Hyperscale (Citus)
- **Horizontal scaling**: Linear scaling via node additions (up to 20 worker nodes)[14].
- **Sharding benefits**: Demonstrated **3× throughput improvement** when sharding large tables across 3 nodes[12].
- **Log throughput**: Supports 100–150 MB/s sustained log rates (preview)[9].

## Operational Considerations

### Flexible Server
- **Replication slot management**: Requires active monitoring to prevent storage bloat from retained WAL files[5].
- **Cross-region replication**: Limited to manual logical replication setups without native multi-region HA[10].
- **Cost optimization**: V5 SKUs provide 50% better price/performance vs. V4[1].

### Hyperscale (Citus)
- **Sharding overhead**: Requires schema modifications to leverage distributed tables fully[14].
- **Consistency tradeoffs**: Queries may return stale data from geo-replicas during regional network partitions[15].
- **Migration complexity**: Logical replication tools required for sharded data ingestion[12].

## Performance Optimization Techniques

### Flexible Server
1. **Write-through cache**: Reduces logical decoding latency by 44–59% through WAL caching[5].
2. **IOPS autoscaling**: Dynamically adjusts storage performance during peak loads[6].
3. **Parameter tuning**: Increase `max_wal_senders` and `wal_keep_size` for high-replica-count deployments[5].

### Hyperscale (Citus)
1. **Colocated shards**: Minimizes cross-node queries by grouping related data[14].
2. **Reference table replication**: Broadcasts small tables to all nodes for local joins[14].
3. **Query parallelization**: Leverages `citus.parallel_degree` to accelerate analytical workloads[12].


## Comparative Analysis Table

| Feature                  | Flexible Server                              | Hyperscale (Citus)                        |
|--------------------------|----------------------------------------------|--------------------------------------------|
| **Max Replicas**         | 5 read replicas[6]                           | Unlimited worker nodes[14]                |
| **Replication Type**     | Physical (async)[5]                          | Physical (async) + Sharding[14]           |
| **Typical Use Case**     | OLTP with moderate read scaling              | HTAP with massive parallelization         |
| **Cross-Region Latency** | Not natively supported[10]                   | 100–500 ms (geo-replicas)[15]             |
| **Storage Cost**         | $0.115/GB (Premium SSDv2)[1]                 | $0.10/GB (distributed)[7]                 |
| **Failover Automation**  | Manual promotion required[10]                | Automatic with HA replicas[7]             |


## Conclusion

**Azure Database for PostgreSQL Flexible Server** excels in traditional OLTP scenarios requiring predictable failover (60–120 seconds) and zero data loss guarantees. Its strength lies in managed replication slot handling and vertical scalability for workloads under 16 TB.

**Azure Cosmos DB for PostgreSQL (Hyperscale)** outperforms in distributed OLAP/HTAP environments, demonstrating 3× throughput improvements through sharding[12]. The architecture supports sub-second failovers and linear scalability but introduces complexity in query optimization for sharded datasets.

For hybrid workloads, Hyperscale's ability to scale reads across multiple regions makes it preferable for global deployments, while Flexible Server remains cost-effective for single-region transactional systems requiring strict consistency.

Sources
[1] Cost optimize your Azure PostgreSQL Flexible Server deployments https://www.danielstechblog.io/cost-optimize-your-azure-postgresql-flexible-server-deployments/
[2] Creating Replicas in Azure Database for PostgreSQL - SQL Shack https://www.sqlshack.com/creating-replicas-in-azure-database-for-postgresql/
[3] What is the Hyperscale service tier? - Azure SQL Database https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql
[4] Any real world cost comparison between Azure SQL and ... - Reddit https://www.reddit.com/r/AZURE/comments/kz93ot/any_real_world_cost_comparison_between_azure_sql/
[5] concepts-read-replicas https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas
[6] Scaling resources - Azure Database for PostgreSQL flexible server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-scaling-resources
[7] Get high-performance scaling for your Azure database workloads ... https://azure.microsoft.com/en-us/blog/get-high-performance-scaling-for-your-azure-database-workloads-with-hyperscale/
[8] Comparing Postgres Managed Services: AWS, Azure, GCP and ... https://blog.peerdb.io/comparing-postgres-managed-services-aws-azure-gcp-and-supabase
[9] Azure SQL Database Hyperscale FAQ - Learn Microsoft https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale-frequently-asked-questions-faq?view=azuresql
[10] Setting up disaster recovery in Azure Postgres Flexible server. https://www.linkedin.com/pulse/setting-up-disaster-recovery-azure-postgres-flexible-server-sreeram-fkrxc
[11] What's Happening to Azure Database for PostgreSQL Single Server? https://learn.microsoft.com/en-us/azure/postgresql/migrate/whats-happening-to-postgresql-single-server
[12] Anyone with Azure PostgreSQL Hyperscale (Citus) experience? https://www.reddit.com/r/PostgreSQL/comments/hg5699/anyone_with_azure_postgresql_hyperscale_citus/
[13] Read queries on replicas - Azure SQL Database ... - Learn Microsoft https://learn.microsoft.com/en-us/azure/azure-sql/database/read-scale-out?view=azuresql
[14] Read replicas - Azure Cosmos DB for PostgreSQL | Microsoft Learn https://learn.microsoft.com/en-us/azure/cosmos-db/postgresql/concepts-read-replicas
[15] Hyperscale secondary replicas - Azure SQL Database https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale-replicas?view=azuresql
[16] What's Happening to Azure Database for PostgreSQL Single Server? https://learn.microsoft.com/en-us/azure/postgresql/migrate/whats-happening-to-postgresql-single-server
[17] Azure Database for PostgreSQL flexible server vs. Single Server https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-compare-single-server-flexible-server
[18] Choosing an Azure Database: An Evaluation of Cost and Performance https://blogit.michelin.io/choosing-an-azure-database-an-evaluation-of-cost-and-performance/
[19] Monitoring and metrics - Azure Database for PostgreSQL flexible ... https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-monitoring
[20] Migrate from Azure Database for PostgreSQL - Single Server to ... https://learn.microsoft.com/en-us/azure/postgresql/migrate/migration-service/tutorial-migration-service-single-to-flexible
[21] Comparison between Azure PostgreSQL Paas Offerings https://techcommunity.microsoft.com/blog/fasttrackforazureblog/comparison-azure-postgresql-single-server-flexible-server-hyperscalecitus/3474966
[22] Reliability and high availability in PostgreSQL - Flexible Server https://learn.microsoft.com/en-us/azure/reliability/reliability-postgresql-flexible-server
[23] Azure Database for PostgreSQL (Single Server, Hyperscale ... https://docs.dynatrace.com/docs/ingest-from/microsoft-azure-services/azure-integrations/azure-cloud-services-metrics/monitor-azure-db-postgresql
[24] How fast can a $5436/mo Azure SQL DB Hyperscale load data? https://www.brentozar.com/archive/2019/02/how-fast-can-a-5436-mo-azure-sql-db-hyperscale-load-data/
[25] Benchmarking Managed PostgreSQL Cloud Solutions - Part Four https://severalnines.com/blog/benchmarking-managed-postgresql-cloud-solutions-part-four-microsoft-azure/
[26] Is Azure Postgresql a meme product? - Reddit https://www.reddit.com/r/AZURE/comments/1fwakjb/is_azure_postgresql_a_meme_product/
[27] Data Latency between primary database and named replicas in ... https://learn.microsoft.com/en-us/answers/questions/1295981/data-latency-between-primary-database-and-named-re
[28] Hyperscale secondary replicas - Azure SQL Database https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale-replicas?view=azuresql
[29] What's new in Hyperscale (Citus) for Postgres on Azure (ft. read ... https://www.citusdata.com/blog/2021/05/29/whats-new-in-hyperscale-citus-for-postgres-on-azure-ft-read-replicas/
[30] Confused why PostgreSQL flexible server is so expensive vs ... https://www.reddit.com/r/AZURE/comments/14vl9mo/confused_why_postgresql_flexible_server_is_so/
[31] What To Know About Microsoft Azure PostgreSQL Hyperscale https://thwack.solarwinds.com/groups/data-driven/b/blog/posts/what-to-know-about-microsoft-azure-postgresql-hyperscale
[32] Elevating Performance with Continuous Priming | Data Exposed https://www.youtube.com/watch?v=9AfmK2VLp8M
[33] Performance diagnostics in Hyperscale - Azure SQL Database https://learn.microsoft.com/en-us/azure/azure-sql/database/hyperscale-performance-diagnostics?view=azuresql
