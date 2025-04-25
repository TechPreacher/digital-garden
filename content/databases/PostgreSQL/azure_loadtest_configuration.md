---
title: Azure LoadTest Configuration
draft: false
date: 2025-04-23
tags:
  - Azure
  - LoadTest
  - Databases
---
## Configuration of Load Test

Basics:
- Name: postgresql load test
- Description: Performance test of replication under load: 60 seconds

Test Plan:
- Load testing framework: JMeter
- Script, data and configuration files:
  - Script: load_test_artifacts/jmeter_script.jmx
  - File: load_test_artifacts/postgresql-42.7.5.jar
  - Managed identity for authentication scenarios:
    - User-assigned identity: {user assigned identity}
    
Parameters:
  - Environment variables:
    - main_threads: 10
    - main_loops: 100
    - main_database: jdbc:postgresql://{postgresql_main}.postgres.database.azure.com:5432/{database}
    - replica_threads: 50
    - replica_loops: 500
    - replica_database: jdbc:postgresql;//{postgresql_replica}.postgres.database.azure.com:5432/{database}
    - main_writes_per_minute: 120
    - replica_reads_per_minute: 480
  - Secrets: 
    - mainpassword: {main_password}
    - replicapassword: {replica_password}
    - mainuser: {main_user}
    - replicauser: {replica_user}
  - Key vault reference identity:
    - Identity type: User-assigned Identity
    - User-assigned identity: {user_assigned_identity}
    
Load:
  - Load configuration:
    - Engine instances: 4
  - Network:
    - Configure test traffic mode: Public
    
Monitoring:
  - Resources:
    - {postgresql_main}
    - {postgresql_replica}
  - Metrics reference identity:
    - Identity type: User-assigned Identity
    - User-assigned identity: {user_assigned_identity}
    
Test criteria:
  - Auto-stop test:
    - Auto-top test: Enable
    - Error percentage: 90
    - Time window (seconds): 60



Change the keyvault networking to allow access from: Allow public access from all networks.

## Sample  Configurations AZ load test

Examples from AI knowledge base:

```bash
az load test create --test-id sample-test-id --load-test-resource sample-alt-resource --resource-group sample-rg --display-name "Sample Name" --autostop-error-rate 80.5 --autostop-time-window 120 

az load test create --test-id sample-test-id --load-test-resource sample-alt-resource --resource-group sample-rg --display-name "Sample Name" --autostop disable 

az load test create --test-id sample-test-id --load-test-resource sample-alt-resource --resource-group sample-rg --display-name "Sample Name" --autostop enable
```

Create a test with custom defined autostop criteria or enable / disable autostop for a test.

```bash
az load test create --test-id sample-test-id --load-test-resource sample-alt-resource --resource-group sample-rg --display-name "Sample Name" --description "Test description" --test-plan sample-jmx.jmx --engine-instances 1 --env rps=2 count=1 --engine-ref-id-type SystemAssigned
```

Create a test with arguments.

```bash
az load test create --test-id sample-test-id --load-test-resource sample-alt-resource --resource-group sample-rg --display-name "Sample Name" --description "Test description" --test-plan sample-jmx.jmx --secret secret_name1=secret_uri1 secret_name2=secret_uri2 --env rps=2 count=1
```

Create a test with secrets and environment variables.