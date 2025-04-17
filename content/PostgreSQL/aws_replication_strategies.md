---
title: AWS Replication Strategies
draft: false
date: 2025-04-17
tags:
  - AWS
  - PostgreSQL
  - Replication
---
"Block Level Replication" in the context of AWS Aurora for PostgreSQL refers to the way Aurora's storage subsystem works: it replicates data at the storage (block) level rather than at the SQL or row level. This is a foundational feature of Aurora and underpins several of its high-availability and durability capabilities.

## Where "Block Level Replication" Fits

- **Aurora's Storage Layer:** Aurora continuously replicates each 10GB block of your database volume six times across three Availability Zones using physical synchronous replication. This is not something you configure separately—it's an inherent part of how Aurora manages data durability and availability[1][4].
- **Not a User-Configurable Replication Strategy:** Unlike logical replication (table/row level) or Aurora global database (cross-region cluster replication), block level replication is always on and transparent to the user. It is the mechanism that allows Aurora Replicas to share the same underlying storage as the primary instance, resulting in very low replica lag and fast failover[1][2][4].

## How It Relates to Other Replication Strategies

| Replication Strategy         | Level        | User-Configurable | Primary Use Case                       |
|-----------------------------|--------------|-------------------|----------------------------------------|
| Block Level Replication     | Storage/Block| No                | High durability, fast failover, HA     |
| Aurora Replicas             | Cluster      | Yes               | Read scaling, high availability        |
| Aurora Global Database      | Cross-Region | Yes               | Disaster recovery, global reads        |
| Logical Replication         | Table/Row    | Yes               | Migration, selective data replication  |

## Summary

- **Block level replication** is the core technology that ensures all Aurora PostgreSQL clusters are highly durable and available by synchronously replicating data blocks across multiple Availability Zones[1][4].
- It is not a replication "strategy" you choose, but the underlying mechanism that supports Aurora Replicas and enables the high-availability features of Aurora PostgreSQL[1][2][4].
- Other replication strategies (Aurora Replicas, Global Database, logical replication) operate at higher levels and are built on top of this foundational block-level replication.

In short, block level replication is always active in Aurora PostgreSQL and is the reason for its high durability and fast failover, but it is not a user-facing replication option like logical or global database replication[1][4].

Sources
[1] Understand replication capabilities in Amazon Aurora PostgreSQL https://aws.amazon.com/blogs/database/understand-replication-capabilities-in-amazon-aurora-postgresql/
[2] Replication with Amazon Aurora PostgreSQL https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Replication.html
[3] Replication with Amazon Aurora - AWS Documentation https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html
[4] Cloud Vendor Deep-Dive: PostgreSQL on AWS Aurora - Severalnines https://severalnines.com/blog/cloud-vendor-deep-dive-postgresql-aws-aurora/
[5] Use logical replication for tables in RDS for PostgreSQL | AWS re:Post https://repost.aws/knowledge-center/rds-postgresql-use-logical-replication
[6] Is there a difference between Aurora Global DB and Cross-region ... https://www.reddit.com/r/aws/comments/emoi2b/is_there_a_difference_between_aurora_global_db/
[7] Cloud Vendor Deep-Dive: PostgreSQL on AWS Aurora - Severalnines https://severalnines.com/blog/cloud-vendor-deep-dive-postgresql-aws-aurora/
[8] Architecting for Zero Data Loss Disaster Recovery using Amazon ... https://community.aws/content/2iClj0zWGGE4JVyBkVyHkJpH54J/zero-dataloss-dr-using-rds?lang=en
[9] Building Multi-Region Resiliency with Amazon RDS and Amazon ... https://developer.gs.com/blog/posts/building-multi-region-resiliency-with-amazon-rds-and-amazon-aurora
[10] Is Amazon Aurora RDS (without Multi-AZ) supported yet? https://forum.camunda.io/t/is-amazon-aurora-rds-without-multi-az-supported-yet/6291
[11] Overview of PostgreSQL logical replication with Aurora https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Replication.Logical.html
[12] Amazon Aurora Cheat Sheet - Tutorials Dojo https://tutorialsdojo.com/amazon-aurora/
[13] Deep dive on Amazon Aurora and Amazon RDS for PostgreSQL ... https://aws.amazon.com/blogs/database/is-amazon-rds-for-postgresql-or-amazon-aurora-postgresql-a-better-choice-for-me/
[14] RDS - Postgres to Aurora high level question re replica's https://repost.aws/questions/QUdJ1QHfiWRzyn1qFuR_4uMg/rds-postgres-to-aurora-high-level-question-re-replica-s
