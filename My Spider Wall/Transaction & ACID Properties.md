
Referenced from: https://www.mongodb.com/basics/acid-transactions
https://www.mongodb.com/databases/types/transactional-databases
## Key-points: 

1. 
	A **transaction** is a group of database read and write operations that only succeeds if all the operations within succeed. Transactions can impact a single record or multiple records.

2. 
	 When doing a transaction, it keeps track of every update that it makes along the way. If the connection breaks before the transaction is complete, or if any command in the transaction fails, then the database rolls back (undoes) all of the changes it had written in the course of the transaction.
   
   Downside: locks - involved resources - prevent concurrent writes - introduces waiting clients - poor latency 

3. 
	A - commands that make up a transaction are a single unit that succeed or fail together.
    C - consistent with database constraints, no illegal state
    I - Isolated env, transactions don't interfere
    D - Persisted and failsafe. Resistant to crashes and power outage
 

