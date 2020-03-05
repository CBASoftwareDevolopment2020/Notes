# [RDB Management and Optimization](https://datsoftlyngby.github.io/soft2020spring/DB/week-08/#3-rdb-management-and-optimization)

20-02-2020

## Security

## Transactions management

A transaction is a logical unit of work containing one or more operations involving one or more shared resources

-   Examples
    -   select, CRUD operations, stored procedures, functions, etc
-   Need to identify high-level functionality of the transactions, such as:
    -   attributes that are updated
    -   search criteria used in a query

If a transaction includes multiple data modifications, they are processed all together - atomicity  
If the transaction is successful, all of the multiple data modifications made during
this transaction are committed and become a permanent part of the database  
If it encounters errors and is not able to complete, all of the multiple data
modifications are rolled back, and the original data recovered, no matter when it
happens

Example:

-   Bank transfer from account A to account B
    -   first, withdraw money from Alice, and second add them to Bob
-   If it succeeds, it remains
    -   transaction is committed
-   If it fails after the first step, withdrawal is restored
    -   money do not just disappear
    -   transaction is rolled back

All transactions have the ACID properties:

-   Atomicity: Either all the work done in the transaction must be performed (commit), or none of it must be performed (rollback), doing part of a transaction is not allowed
-   Consistency: When a transaction is completed, the system must be in a stable and consistent condition
-   Isolation: Different transactions must be isolated from each other and each process in a multi-user system can be programmed as if it was the only process accessing the system, partial work done in one transaction is not visible to other transactions until the transaction is committed
-   Durability: The changes made during a transaction are made persistent when it is committed and committed transaction can not be rolled back

Modifications made by different users may affect each other in a concurrency

-   Transaction management is a programming technique for insuring successful transaction processing and controlling the concurrency
-   Transaction management
    -   protects the data and the data integrity from failure of software, hardware, or power
    -   provides access to multiple users at the same time
    -   prevent simultaneous read and write of the same data by different users

There are three transaction control statements

-   begin transaction
    -   alerts SQL Server that a transaction is beginning
-   rollback transaction
    -   undoes the changes that has already been made
    -   rolls back either to the beginning of the transaction or a named save point within
    -   execution continues with the next statement after the transaction
-   commit transaction
    -   completes the transaction and saves the changes to the database

```sql
BEGIN;
INSERT INTO accounts(name, balance) VALUES('Jack',0);
INSERT INTO accounts(name,balance) VALUES('Alice',10000);
INSERT INTO accounts(name, balance) VALUES(‘Bob’,9000);
COMMIT;
BEGIN;
UPDATE accounts SET balance = balance + 1500
WHERE id = 3;
ROLLBACK;

BEGIN;
UPDATE accounts SET balance = balance - 100.00 WHERE
name = 'Alice’;
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00 WHERE
name = 'Bob’;
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00 WHERE
name = 'Wally’;
COMMIT;
```

### Savepoint

-   Developer can set a savepoint, or a marker inside a transaction to preserve some of the work already done
    -   it is given a name
    -   marks end of a relatively independent portion of the work that has been done
    -   so the transaction can return to it safely, if necessary
-   After a rollback, execution continues with the statement immediately following the rollback

## Data locking

-   Two user sessions make changes on the same page, but only one of them remains - LOST UPDATES
-   One transaction is allowed to read data from a row that has been modified by another running transaction, but not yet committed - DIRTY READS
-   One transaction reads repeatedly an object, for example, in a loop, but meanwhile between two interactions another transaction makes changes to it - NONREPEATABLE READS
-   One transaction makes updates to a group of rows, but meanwhile another transaction inserts a new row in the same group, which will be missed by the first one - PHANTOM READS
-   To ensures that simultaneous transactions do not interfere with each other, pages are locked, while reads or writes are performed
-   Locking is a job of the system Lock Manager
-   Developer can specify how restrictive a lock should be and which database objects should be locked

[Demo](https://pgdash.io/blog/postgres-transactions.html)

### Locks

-   Locks prevent update conflicts – users can not access data that other users are in process of changing
-   Locks make serialization possible
    -   example: booking one flight ticket can not be done by two users at the same time, as they can not have the same seat
-   Locks control concurrent transactions to enable multiple users access the database at the same time
    -   high concurrency – more users – lower response time

### Concurrency control

-   Pessimistic concurrency control
    -   locks data that has been read and stays in preparation for update, so no one can use it at the same time
    -   enables serialization of transactions
-   Optimistic concurrency control
    -   do not lock the data, but make a check after the update - if the original data has been changed wrong during the transaction, fire an error to occurs and roll the transaction back – the user should resubmit it
-   DB servers provides wide range of concurrency control tools
-   Developers specify the type they need by selecting the transaction isolation level

### Isolation level

-   An isolation level protects a specified transaction from other transactions
-   Setting transaction isolation levels allows programmers to provide greater concurrent access to data, with increased risk of integrity problems
-   Assigned for the whole session

-   The ANSI standard defines four levels of isolation
    -   Level 0: READ UNCOMMITTED - allows dirty reads
    -   Level 1: READ COMMITTED - prevents dirty reads
    -   Level 2: REPEATABLE READ - prevents non-repeatable reads
    -   Level 3: SERIALIZABLE - prevents phantom reads
-   The higher the isolation level, higher the consistency, lower the concurrency (parallelism)
-   The default isolation level for
    -   PostgreSQL is 1
    -   ANSI-92 standard is 3
    -   MySQL – level 2
    -   Oracle – level 1

| Isolation level  | Phenomena allowed                             |
| ---------------- | --------------------------------------------- |
| Read uncommitted | Dirty Read<br>Non Repeatable Read<br>Phantoms |
| Read committed   | Non Repeatable Read<br>Phantoms               |
| Repeatable read  | Phantoms                                      |
| Serializable     |                                               |

### Lockable Resources

| Item            | Description                       |
| --------------- | --------------------------------- |
| RID             | Row identifier                    |
| KEY             | Row lock within an index          |
| PAGE            | Data pageor index page            |
| EXTENT          | Group of pages                    |
| TABLE           | Entire table                      |
| HOBT            | A heap or B-tree                  |
| FILE            | A database file                   |
| APPLICATION     | An application-specified resource |
| METADATA        | Metadata locks                    |
| ALLOCATION_UNIT | An allocation unit                |
| DATABASE        | Entire database                   |

### Granularity

-   Refers to the amount of data that can be locked at one time
-   Can range from a single storage page to an entire database
-   The locking mechanism protects, but also reduces availability of data
    -   as the lock granularity increases, the processing required to obtain a lock decreases, but it also degrades performance
    -   as lock granularity decreases, the amount of processing required to maintain and coordinate the locks increases

### Lock comparison

#### Page lock

-   Pages are locked as they are accessed
-   Page lock is placed only on a single page and therefore on a small subset of data
-   Page locks are less restrictive than table locks
-   Page locks are used preferably whenever it is possible

#### Table lock

-   Table lock covers the entire table
    -   the most restrictive lock
-   A table lock is implemented via means called escalation
    -   if a user gets access to an entire table, e.g. an update with no WHERE clause, the server escalates the page lock to a table lock
    -   by default, if any SQL statement accumulates 200 page locks, it is also escalated to a table lock
-   A developer can create table level locks by an option at FROM clause of SELECT statement

| Option                                                                                   | Description                                                                                                                              |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| HOLDLOCK<br>SERIALIZABLE<br>REPEATABLEREAD<br>READCOMMITTED<br>READUNCOMMITTED<br>NOLOCK | Control locking behavior for a table and override the locks that would be used to enforce the isolation level of the current transaction |
| ROWLOCK<br>PAGLOCK<br>TABLOCK<br>TABLOCKX                                                | Specify the size and type of the locks to be used for a table                                                                            |
| READPAST                                                                                 | Skip locked rows                                                                                                                         |
| UPDLOCK                                                                                  | Use update locks instead of shared locks                                                                                                 |

#### Holdlock

-   holdlock/noholdlock is a clause of select statement that overrides the isolation level set for the duration of this statement
-   holdlock is equivalent to and enforces SERIALIZABLE isolation level (3)
-   Use holdlock only if strictly necessary
-   Use noholdlock option only if you want SQL Server to release any lock regardless of isolation level

#### Deadlock

Ending deadlock

-   DB server ends a deadlock by automatically terminating one of the transactions
    1.  Rolls back the transaction of the deadlock victim
    2.  The winner is the transaction that has been processing the longest
    3.  Notifies the deadlock victim's application with message number 1205
    4.  Cancels the deadlock victim's current request
    5.  Allows the other transaction to continue
-   Clients should listen regularly for a message 1205
-   If a deadlock occurs, the victim application should resubmit the transaction

Avoiding deadlock

-   Have all transaction access the tables in the same order
-   Avoid long running transactions; make transactions small and commit as soon as possible
-   Avoid numerous simultaneous executions of DML commands like insert, update, delete
-   Wherever possible use stored procedures to perform transactions to ensure consistent access order to tables
-   Avoid user input while you have a holdlock on a table
-   Check for error 1205, reconnect and re-submit the victim transaction

### Transaction log

-   Every transaction is recorded in a transaction log
    -   to maintain database consistency
    -   to aid in recovery
-   The log is a storage area that automatically tracks all changes to a database
    -   there are some exceptions of non-logged operations
-   Modifications are recorded in the log on disk as they are executed, before they are written in the database
-   Based on the transaction log, DB server can recover data automatically in event of a power loss, system software failure, client problem, or a transaction cancellation request
-   In case of failure, the server takes care to recover a stable state
    -   all committed transactions are rolled forward to the database
    -   all non committed transactions are rolled back to the previous check point
