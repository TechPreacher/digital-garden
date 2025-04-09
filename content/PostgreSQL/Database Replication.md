```
---
title: "Database Replication"
draft: false
date: "2025-04-08"
tags:
  - Azure
  - PostgreSQL
---
```
## AWS (Block Level Replication)

Block-level replication in PostgreSQL on AWS refers to a replication mechanism that operates at the physical level, transferring changes from the source database to replicas using Write-Ahead Logs (WAL). This method is commonly employed in Amazon RDS for PostgreSQL and is technically known as _physical replication_. Here's how it works:

- **Mechanism**: Changes made to the primary database are recorded in WAL files. These logs are continuously sent from the primary instance to the replica(s) using PostgreSQL's native streaming replication protocol. The replica replays these logs to maintain consistency with the primary database[1][2][3].
- **Characteristics**:
    - It replicates all changes at the server level, including data modifications, schema changes, and maintenance operations like `VACUUM`. Operations recorded in WAL are replayed on replicas, ensuring they mirror the primary instance[2].
    - Replicas are read-only and can be used for scaling read workloads without affecting the primary database's performance[3].
- **Use Cases**:
    - Ensuring high availability and disaster recovery.
    - Scaling read-heavy applications by offloading queries to replicas.
    - Supporting cross-region replication for geographical redundancy[1][3].

It is distinct from logical replication, which operates at a higher level, replicating specific tables or subsets of data based on their replication identity (e.g., primary key)[4].

Sources  
[1] Working with read replicas for Amazon RDS for PostgreSQL  
[https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html)  
[2] Do AWS replicas of Postgres RDS instances require Vacuum and ...  
[https://serverfault.com/questions/1117978/do-aws-replicas-of-postgres-rds-instances-require-vacuum-and-analyze-independent](https://serverfault.com/questions/1117978/do-aws-replicas-of-postgres-rds-instances-require-vacuum-and-analyze-independent)  
[3] Troubleshoot replication lags in RDS for PostgreSQL | AWS re:Post  
[https://repost.aws/knowledge-center/rds-postgresql-replication-lag](https://repost.aws/knowledge-center/rds-postgresql-replication-lag)  
[4] Documentation: 17: Chapter 29. Logical Replication - PostgreSQL  
[https://www.postgresql.org/docs/current/logical-replication.html](https://www.postgresql.org/docs/current/logical-replication.html)  
[5] Understand replication capabilities in Amazon Aurora PostgreSQL  
[https://aws.amazon.com/blogs/database/understand-replication-capabilities-in-amazon-aurora-postgresql/](https://aws.amazon.com/blogs/database/understand-replication-capabilities-in-amazon-aurora-postgresql/)  
[6] Best practices for Amazon RDS PostgreSQL replication - AWS  
[https://aws.amazon.com/blogs/database/best-practices-for-amazon-rds-postgresql-replication/](https://aws.amazon.com/blogs/database/best-practices-for-amazon-rds-postgresql-replication/)  
[7] Use logical replication for tables in RDS for PostgreSQL | AWS re:Post  
[https://repost.aws/knowledge-center/rds-postgresql-use-logical-replication](https://repost.aws/knowledge-center/rds-postgresql-use-logical-replication)  
[8] Disaster recovery options in the cloud - AWS Documentation  
[[d]]

## Azure (Read Replicas)

The closest equivalent to block-level replication in Azure Database for PostgreSQL Flexible Server is **read replicas**. This feature uses PostgreSQL's native physical replication technology, specifically asynchronous streaming replication with replication slots, complemented by file-based log shipping when needed. Here's a detailed breakdown:

- **Replication Mechanism**:
    - Read replicas are updated asynchronously using PostgreSQL's physical replication, which operates at the block level by transferring changes recorded in Write-Ahead Logs (WAL) from the primary server to replicas[1][6].
- **Characteristics**:
    - Replicas are read-only and can be used for scaling read workloads.
    - Up to five replicas can be created per primary server, either within the same region or across different Azure regions (geo-replication)[1][5].
    - Replica creation involves a snapshot approach for same-region replicas or base backups for geo-replicas[1].
- **Use Cases**:
    - Scaling read-heavy applications.
    - Ensuring high availability and disaster recovery.
    - Supporting cross-region redundancy.

This approach mirrors AWS's block-level replication in functionality, leveraging physical replication for efficient data synchronization. However, Azure also offers logical replication for more granular control over data subsets[2].

Sources  
[1] Read replicas - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas)  
[2] Azure Database for PostgreSQL flexible server | Microsoft Learn  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical)  
[3] Using Microsoft Azure Database for PostgreSQL flexible server as a ...  
[https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.AzureDBPostgreSQL.html](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.AzureDBPostgreSQL.html)  
[4] Is it possible to have a read replicas of azure flexible postgresql ...  
[https://stackoverflow.com/questions/70081184/is-it-possible-to-have-a-read-replicas-of-azure-flexible-postgresql-server-with](https://stackoverflow.com/questions/70081184/is-it-possible-to-have-a-read-replicas-of-azure-flexible-postgresql-server-with)  
[5] Geo-replication - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas-geo](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas-geo)  
[6] Creating and Monitoring Read Replicas in Azure Postgres Flexible ...  
[https://techcommunity.microsoft.com/blog/adforpostgresql/creating-and-monitoring-read-replicas-in-azure-postgres-flexible-server/3903826](https://techcommunity.microsoft.com/blog/adforpostgresql/creating-and-monitoring-read-replicas-in-azure-postgres-flexible-server/3903826)  
[7] azure-databases-docs/articles/postgresql/flexible-server/how-to ...  
[https://github.com/MicrosoftDocs/azure-databases-docs/blob/main/articles/postgresql/flexible-server/how-to-read-replicas-portal.md](https://github.com/MicrosoftDocs/azure-databases-docs/blob/main/articles/postgresql/flexible-server/how-to-read-replicas-portal.md)  
[8] Azure Database for PostgreSQL flexible server - Microsoft Learn  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/overview](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/overview)

## Record Level Replication for Azure Database for PostgreSQL

To implement **record-level replication** in Azure Database for PostgreSQL, you can use **logical replication**. This feature provides fine-grained control over replicated data, allowing you to replicate specific tables or data subsets (such as selected rows) through publications and subscriptions. Here's how to set it up:

---

### **Steps to Implement Record-Level Replication**

### **1. Enable Logical Replication on the Server**

- Navigate to your Azure Database for PostgreSQL Flexible Server in the Azure portal.
- Update the following server parameters:
    - `wal_level` to `logical`
    - `max_replication_slots` to at least 1
    - `max_worker_processes` to a sufficient number (e.g., 16)
- Save the changes and restart the server.

### **2. Prepare the Source Database**

- Connect to the source database (publisher).
- Create a table if it doesn't exist:
    
    ```SQL
    CREATE TABLE basic (id INTEGER NOT NULL PRIMARY KEY, a TEXT);
    INSERT INTO basic VALUES (1, 'apple'), (2, 'banana');
    ```
    
- Create a publication for the table:
    
    ```SQL
    CREATE PUBLICATION pub FOR TABLE basic;
    ```
    

### **3. Prepare the Target Database**

- Connect to the target database (subscriber).
- Create a table with the same schema as the source database:
    
    ```SQL
    CREATE TABLE basic (id INTEGER NOT NULL PRIMARY KEY, a TEXT);
    ```
    

### **4. Set Up a Subscription**

- On the target database, create a subscription to connect to the source database:
    
    ```SQL
    CREATE SUBSCRIPTION sub
    CONNECTION 'host=<source_server>.postgres.database.azure.com user=<rep_user> dbname=<source_dbname> password=<password>'
    PUBLICATION pub;
    ```
    

### **5. Verify Replication**

- Insert a new record into the source database:
    
    ```SQL
    INSERT INTO basic VALUES (3, 'mango');
    ```
    
- Query the target database to confirm the replication:
    
    ```SQL
    SELECT * FROM basic;
    ```
    

---

### **Filtering Rows for Record-Level Replication**

While logical replication doesn't natively filter rows, you can achieve row-level filtering by using **Row-Level Security (RLS)** or by implementing triggers and views.

### Example Using RLS:

1. Define an RLS policy on the source table to control which rows are visible during replication:
    
    ```SQL
    ALTER TABLE basic ENABLE ROW LEVEL SECURITY;
    CREATE POLICY replicate_only_apples ON basic
    USING (a = 'apple');
    ```
    
2. Only rows matching the policy will be visible and replicated.

---

### **Key Considerations**

- Logical replication doesn't replicate DDL changes—schema changes must be applied manually on both databases.
- Ensure network connectivity between source and target databases.
- Use separate roles for replication with appropriate permissions:
    
    ```SQL
    ALTER ROLE <rep_user> WITH REPLICATION;
    ```
    

By using logical replication and optionally combining it with RLS or custom logic, you can implement record-level replication in Azure Database for PostgreSQL.

Sources  
[1] Creating Replicas in Azure Database for PostgreSQL - SQLShack  
[https://www.sqlshack.com/creating-replicas-in-azure-database-for-postgresql/](https://www.sqlshack.com/creating-replicas-in-azure-database-for-postgresql/)  
[2] Security - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-security](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-security)  
[3] Azure Database for PostgreSQL flexible server | Microsoft Learn  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical)  
[4] Does Azure Postgresql support table view replication?  
[https://stackoverflow.com/questions/58953506/does-azure-postgresql-support-table-view-replication](https://stackoverflow.com/questions/58953506/does-azure-postgresql-support-table-view-replication)  
[5] Azure Database for PostgreSQL : Logical Replication  
[https://azureaggregator.wordpress.com/2023/04/19/azure-database-for-postgresql-logical-replication/](https://azureaggregator.wordpress.com/2023/04/19/azure-database-for-postgresql-logical-replication/)  
[6] Geo-disaster recovery - Azure Database for PostgreSQL - Flexible ...  
[https://docs.azure.cn/en-us/postgresql/flexible-server/concepts-geo-disaster-recovery](https://docs.azure.cn/en-us/postgresql/flexible-server/concepts-geo-disaster-recovery)  
[7] Read replicas - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas)  
[8] Geo-replication - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas-geo](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas-geo)

## Azure Database for PostgreSQL, Primary Key, Replication

Primary keys (or unique indexes) are required for all tables using logical replication in Azure Database for PostgreSQL. The system uses a table's replica identity—typically the primary key—to track rows during replication.

### Key Details:

1. **Replica Identity**:
    - Logical replication uses a table's primary key as its replica identity for row identification.
    - Tables without primary keys must use `FULL` replica identity (including all columns), though this is less efficient and can affect performance.
2. Example of setting replica identity:
    
    ```SQL
    ALTER TABLE your_table REPLICA IDENTITY FULL;
    ```
    
3. **Best Practice**:
    - Use primary keys on all replicated tables for optimal performance and reliability.
    - Replication may be unreliable without primary keys, making it difficult to track changes.
4. **Documentation Reference**: Microsoft specifies that logical replication uses a **replication identity**, typically implemented as a primary key[1][7][9].

Using primary keys on replicated tables ensures smoother configuration and operation of the replication process.

Sources  
[1] Azure Database for PostgreSQL flexible server | Microsoft Learn  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical)  
[2] High availability (Reliability) in Azure Database for PostgreSQL  
[https://learn.microsoft.com/en-us/azure/reliability/reliability-postgresql-flexible-server](https://learn.microsoft.com/en-us/azure/reliability/reliability-postgresql-flexible-server)  
[3] Read replicas - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas)  
[4] Using Microsoft Azure Database for PostgreSQL as a target - Qlik Help  
[https://help.qlik.com/en-US/replicate/November2024/Content/Replicate/Main/Microsoft_Azure_PostgreSQL/ms_postgresql.htm](https://help.qlik.com/en-US/replicate/November2024/Content/Replicate/Main/Microsoft_Azure_PostgreSQL/ms_postgresql.htm)  
[5] Creating Replicas in Azure Database for PostgreSQL - SQLShack  
[https://www.sqlshack.com/creating-replicas-in-azure-database-for-postgresql/](https://www.sqlshack.com/creating-replicas-in-azure-database-for-postgresql/)  
[6] Replicate PostgrSQL on-premise to Azure Database for PostgreSQL  
[https://stackoverflow.com/questions/67002541/replicate-postgrsql-on-premise-to-azure-database-for-postgresql](https://stackoverflow.com/questions/67002541/replicate-postgrsql-on-premise-to-azure-database-for-postgresql)  
[7] Tuning logical replication on Azure Database for PostgreSQL  
[https://techcommunity.microsoft.com/blog/adforpostgresql/tuning-logical-replication-on-azure-database-for-postgresql-–-flexible-server/4162840](https://techcommunity.microsoft.com/blog/adforpostgresql/tuning-logical-replication-on-azure-database-for-postgresql-%E2%80%93-flexible-server/4162840)  
[8] Azure Database for PostgreSQL flexible server - Learn Microsoft  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/overview](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/overview)  
[9] Azure Database for PostgreSQL : Logical Replication  
[https://techcommunity.microsoft.com/blog/adforpostgresql/azure-database-for-postgresql--logical-replication/3799509](https://techcommunity.microsoft.com/blog/adforpostgresql/azure-database-for-postgresql--logical-replication/3799509)

## Creating Replication User

Here's how to assign the `REPLICATION` attribute to a database user in Azure Database for PostgreSQL:

1. **Connect to the Database**:  
    Connect to your PostgreSQL server using  
    `psql` or the Azure portal.
2. **Create or Modify a User**:
    - For a new replication user:  
        Replace  
        `secure_password` with a strong password.
        
        ```SQL
        CREATE ROLE replication_user WITH LOGIN REPLICATION PASSWORD 'secure_password';
        ```
        
    - For an existing user:  
        Replace  
        `existing_user` with the target role name.
        
        ```SQL
        ALTER ROLE existing_user WITH REPLICATION;
        ```
        
3. **Grant Necessary Privileges**:  
    Grant the required privileges for replication:  
    
    ```SQL
    GRANT azure_pg_admin TO replication_user;
    ```
    
4. **Configure Server Parameters**:
    - Set `wal_level` to `logical` for logical replication.
    - Include necessary extensions like `pglogical` in `shared_preload_libraries` if needed.
5. **Update Connection Settings**:  
    Add this line to your  
    `pg_hba.conf` file:
    
    ```Plain
    host replication replication_user your_ip/32 md5
    ```
    
    - Replace `your_ip/32` with your resource's IP address or subnet.
6. **Reload Configuration**:  
    Apply the changes without a server restart:  
    
    ```SQL
    SELECT pg_reload_conf();
    ```
    
7. These steps configure your user for replication tasks in Azure Database for PostgreSQL[1][2][8].
8. Sources:  
    [1] Azure Database for PostgreSQL flexible server | Microsoft Learn  
    [https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-logical)  
    [2] How can you grant replication permissions to a user in PostgreSQL?  
    [https://www.dragonflydb.io/faq/postgres-grant-replication-to-user](https://www.dragonflydb.io/faq/postgres-grant-replication-to-user)  
    [3] Security - Azure Database for PostgreSQL - Flexible Server  
    [https://docs.azure.cn/en-us/postgresql/flexible-server/concepts-security](https://docs.azure.cn/en-us/postgresql/flexible-server/concepts-security)  
    [4] Azure Database for PostgreSQL : Logical Replication  
    [https://azureaggregator.wordpress.com/2023/04/19/azure-database-for-postgresql-logical-replication/](https://azureaggregator.wordpress.com/2023/04/19/azure-database-for-postgresql-logical-replication/)  
    [5] Managing roles & attributes with PostgreSQL - Prisma  
    [https://www.prisma.io/dataguide/postgresql/authentication-and-authorization/role-management](https://www.prisma.io/dataguide/postgresql/authentication-and-authorization/role-management)  
    [6] Manage read replicas - Azure portal, REST API - Flexible Server  
    [https://docs.azure.cn/en-us/postgresql/flexible-server/how-to-read-replicas-portal](https://docs.azure.cn/en-us/postgresql/flexible-server/how-to-read-replicas-portal)  
    [7] Read replicas - Azure Database for PostgreSQL flexible server  
    [https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas)  
    [8] Postgres replication | dlt Docs - dltHub  
    [https://dlthub.com/docs/dlt-ecosystem/verified-sources/pg_replication](https://dlthub.com/docs/dlt-ecosystem/verified-sources/pg_replication)