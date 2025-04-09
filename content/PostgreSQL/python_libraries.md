---
title: "Azure Database for PostgreSQL Python libraries"
draft: false
date: "2025-04-08"
tags:
  - Azure
  - PostgreSQL
---
There are several Python libraries available for accessing Azure Database for PostgreSQL, each suited for different purposes:

### **Azure SDK for Management**

The Azure Python SDK enables programmatic management of Azure Database for PostgreSQL Flexible Servers, handling tasks like provisioning and resource management:

- Install required libraries:
    
    ```Shell
    pip install azure-mgmt-resource azure-identity azure-mgmt-postgresqlflexibleservers
    ```
    
- Example usage for creating a database:
    
    ```Python
    from azure.identity import DefaultAzureCredential
    from azure.mgmt.postgresqlflexibleservers import PostgreSQLManagementClient
    
    client = PostgreSQLManagementClient(
        credential=DefaultAzureCredential(),
        subscription_id="<subscription-id>"
    )
    
    response = client.databases.begin_create(
        resource_group_name="<resource-group-name>",
        server_name="<server-name>",
        database_name="<database-name>",
        parameters={"properties": {"charset": "utf8", "collation": "en_US.utf8"}}
    ).result()
    
    print(response)
    ```
    

This library specializes in resource management operations such as creating, scaling, or deleting servers and databases[1][8].

---

### **Database Connectivity Libraries**

For database queries and interactions, you can use these recommended libraries:

### **psycopg2**

- A widely adopted PostgreSQL adapter for Python.
- Example usage:
    
    ```Python
    import psycopg2
    
    conn = psycopg2.connect(
        host="YOUR_SERVER_NAME.postgres.database.azure.com",
        database="YOUR_DB_NAME",
        user="YOUR_USERNAME",
        password="YOUR_PASSWORD"
    )
    
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM your_table")
    rows = cursor.fetchall()
    
    for row in rows:
        print(row)
    
    conn.close()
    ```
    

This library integrates seamlessly with Azure Database for PostgreSQL and has strong community support[3][7].

### **pyodbc**

- A practical choice for ODBC driver connections.
- Example usage:
    
    ```Python
    import pyodbc
    
    cnxn = pyodbc.connect(
        'DRIVER={PostgreSQL ODBC Driver};PORT=5432;SERVER=YOUR_SERVER_NAME.postgres.database.azure.com;' +
        'DATABASE=YOUR_DB_NAME;UID=YOUR_USERNAME;PWD=YOUR_PASSWORD'
    )
    
    cursor = cnxn.cursor()
    cursor.execute("SELECT * FROM SALES")
    
    for row in cursor.fetchall():
        print(row)
    
    cnxn.close()
    ```
    

This library is the go-to choice when working with ODBC drivers[10].

---

### **Choosing the Right Library**

- Use **Azure SDK** for resource management and configuration tasks.
- Use **psycopg2** or **pyodbc** for direct database operations.

For typical Python applications, **psycopg2** is the recommended choice, thanks to its straightforward usage and strong PostgreSQL integration[3][7].

Sources  
[1] Quickstart: Create With Azure Libraries (SDK) for Python  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/quickstart-create-server-python-sdk](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/quickstart-create-server-python-sdk)  
[2] Why does Azure recommend Postgresql for Python app in ... - Reddit  
[https://www.reddit.com/r/AZURE/comments/15f4vay/why_does_azure_recommend_postgresql_for_python/](https://www.reddit.com/r/AZURE/comments/15f4vay/why_does_azure_recommend_postgresql_for_python/)  
[3] Build AI Apps with Azure Database for PostgreSQL - LinkedIn  
[https://www.linkedin.com/pulse/build-ai-apps-azure-database-postgresql-kalai-shakrapani-nsr3c](https://www.linkedin.com/pulse/build-ai-apps-azure-database-postgresql-kalai-shakrapani-nsr3c)  
[4] Connecting Python application to Azure Database for PostgreSQL  
[https://lnx.azurewebsites.net/connecting-python-application-to-azure-database-for-postgresql/](https://lnx.azurewebsites.net/connecting-python-application-to-azure-database-for-postgresql/)  
[5] Azure-Samples/azure-postgres-pgvector-python - GitHub  
[https://github.com/Azure-Samples/azure-postgres-pgvector-python](https://github.com/Azure-Samples/azure-postgres-pgvector-python)  
[6] python - Connection to a Azure SQL DB in many modules needed  
[https://stackoverflow.com/questions/60352147/connection-to-a-azure-sql-db-in-many-modules-needed-how-to](https://stackoverflow.com/questions/60352147/connection-to-a-azure-sql-db-in-many-modules-needed-how-to)  
[7] azure-python-labs/01-postgres/README.md at main - GitHub  
[https://github.com/Azure-Samples/azure-python-labs/blob/main/01-postgres/README.md](https://github.com/Azure-Samples/azure-python-labs/blob/main/01-postgres/README.md)  
[8] Quickstart: Create With Azure Libraries (SDK) for Python  
[https://docs.azure.cn/en-us/postgresql/flexible-server/quickstart-create-server-python-sdk](https://docs.azure.cn/en-us/postgresql/flexible-server/quickstart-create-server-python-sdk)  
[9] Connect using Python - Azure Database for PostgreSQL flexible server  
[https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/connect-python](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/connect-python)  
[10] Azure MySQL/PostgreSQL libraries for Python | Microsoft Learn  
[https://learn.microsoft.com/en-us/python/api/overview/azure/database-for-mysql-postgres?view=azure-python](https://learn.microsoft.com/en-us/python/api/overview/azure/database-for-mysql-postgres?view=azure-python)