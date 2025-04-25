---
title: "Azure Database for PostgreSQL Case Sensitivity"
draft: false
date: "2025-04-08"
tags:
  - Azure
  - PostgreSQL
---
## Azure Databaes for PostgreSQL and Case

Azure Database for PostgreSQL may encounter issues when databases contain multiple tables with names that differ only in case. This occurs because of PostgreSQL's specific identifier handling:

1. **Default Behavior**: PostgreSQL automatically converts unquoted identifiers (such as table and column names) to lowercase. For example, `TableName`, `TABLENAME`, and `tablename` are all treated as identical unless explicitly quoted[2][4][8].
2. **Case-Sensitive Identifiers**: When you create tables with quoted names (like `"TableName"` and `"tablename"`), PostgreSQL treats them as separate objects. However, you must then use double quotes consistently in all queries referencing these tables—otherwise, you'll encounter "table does not exist" errors[3][8].
3. **Recommendation**: To prevent confusion and conflicts:
    - Use lowercase names exclusively for tables and columns.
    - Avoid relying on case-sensitive naming conventions unless absolutely necessary.
    - If case sensitivity is required, ensure all queries consistently use double quotes around identifiers[2][5][7].

This behavior is fundamental to PostgreSQL and applies to Azure Database for PostgreSQL as well. Since improper handling of case-sensitive names can create operational issues, it's crucial to plan your naming conventions carefully.

Sources  
[1] Manage case-insensitive data in PostgreSQL | AWS Database Blog  
[https://aws.amazon.com/blogs/database/manage-case-insensitive-data-in-postgresql/](https://aws.amazon.com/blogs/database/manage-case-insensitive-data-in-postgresql/)  
[2] PostgreSQL — column names of a table are case-sensitive  
[https://deeplearning.lipingyang.org/2017/01/07/postgresql-column-names-of-a-table-are-case-sensitive/](https://deeplearning.lipingyang.org/2017/01/07/postgresql-column-names-of-a-table-are-case-sensitive/)  
[3] Case sensitivity differences for ANSI SQL - AWS Documentation  
[https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.sql.casesensitivity.html](https://docs.aws.amazon.com/dms/latest/sql-server-to-aurora-postgresql-migration-playbook/chap-sql-server-aurora-pg.sql.casesensitivity.html)  
[4] Don't get bit by PostgreSQL case sensitivity - DEV Community  
[https://dev.to/lefebvre/dont-get-bit-by-postgresql-case-sensitivity--457i](https://dev.to/lefebvre/dont-get-bit-by-postgresql-case-sensitivity--457i)  
[5] Avoid capital letters in Postgres names : r/PostgreSQL - Reddit  
[https://www.reddit.com/r/PostgreSQL/comments/1gizgha/avoid_capital_letters_in_postgres_names/](https://www.reddit.com/r/PostgreSQL/comments/1gizgha/avoid_capital_letters_in_postgres_names/)  
[6] PostgreSQL Case Insensitivity  
[https://docs.oracle.com/cd/E19501-01/819-3659/gcnto/index.html](https://docs.oracle.com/cd/E19501-01/819-3659/gcnto/index.html)  
[7] Preview data of Azure Database for PostgreSQL - Microsoft Q&A  
[https://learn.microsoft.com/en-us/answers/questions/2102009/preview-data-of-azure-database-for-postgresql](https://learn.microsoft.com/en-us/answers/questions/2102009/preview-data-of-azure-database-for-postgresql)  
[8] Are PostgreSQL column names case-sensitive? - Stack Overflow  
[https://stackoverflow.com/questions/20878932/are-postgresql-column-names-case-sensitive](https://stackoverflow.com/questions/20878932/are-postgresql-column-names-case-sensitive)