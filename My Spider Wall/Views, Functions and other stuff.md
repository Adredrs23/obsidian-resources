https://www.mssqltips.com/sqlservertip/7437/sql-stored-procedures-views-functions-examples/
Simple, complex, Inline, Indexed, Partitioned & System Views

1. Views: https://database.guide/what-is-a-view/
- A simple query querying a complex queries. 
- It does not hold the actual data; it holds only the definition of the view in the data dictionary.

The view has primarily two purposes:
- Simplify the complex SQL queries.
- Provide restriction to users from accessing sensitive data.

![[Pasted image 20230826104357.png]]

2. Materialized View
- A view that not only stores the definition but also the data by creating replicas of that data.
Clustered Index & Non Clustered Index
Stored Procedure
Functions
With No Expand  - clustered INdex
![[Pasted image 20230826102431.png]]
Index Views needs to be updated in PostGres using Refresh Command.

3. Partitioned View
   https://www.brentozar.com/archive/2016/09/partitioned-views-guide/
   - Need atleast one clustered index
```sql
CREATE VIEW dbo.Doodad
AS
SELECT columns...
FROM Table1
UNION ALL
SELECT columns...
FROM Table2
UNION ALL
SELECT columns...
FROM Table3

SELECT * FROM dbo.Doodad
```

Clusters
![[Pasted image 20230826105640.png]]![[Pasted image 20230826105145.png]]

