# SQL Databases

Postgres is more than just a server for storing vanilla datatypes and querying them. Instead, it’s a powerful data management engine that can reformat output data, store weird datatypes such as arrays, execute logic, and provide enough power to rewrite incoming queries.

| Term         | Definition                                                                                                                                                                                                                       |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Columns      | A domain of values of a certain type, sometimes called an _attribute_                                                                                                                                                            |
| Rows         | An object comprised of a set of column values, sometimes called a _tuple_                                                                                                                                                        |
| Tables       | A set of rows with the same columns, sometimes called a _relation_                                                                                                                                                               |
| Primary Key  | The unique value that pinpoints a specific row                                                                                                                                                                                   |
| Foreign Key  | A data constraint that ensures that each entry in a column in one table uniquely corresponds to a row in another table (or even the same table)                                                                                  |
| CRUD         | **C**reate, **R**ead, **U**pdate, **D**elete                                                                                                                                                                                     |
| SQL          | **S**tructured **Q**uery **L**anguage, the _lingua franca_ of a relational database                                                                                                                                              |
| Join         | Combining two tables into one by some matching columns                                                                                                                                                                           |
| Left join    | Combining two tables into one by some matching columns or `NULL` if nothing matches the left table                                                                                                                               |
| Index        | A data structure to optimize selection of a specific set of columns                                                                                                                                                              |
| B-Tree index | A good standard index; values are stored as a balanced tree data structure; very flexible; B-tree indexes are the default in Postgres                                                                                            |
| Hash index   | Another good standard index in which each index value is unique; hash indexes tend to offer better performance for comparison operations than B-tree indexes but are less flexible and don’t allow for things like range queries |

## Datatypes

| Datatype   | Meaning                                        |
| ---------- | ---------------------------------------------- |
| text       | a string of any length                         |
| varchar(n) | a string of variable length up to n characters |
| char(n)    | a string of exactly n characters               |
| SERIAL     | an auto incrementing integer                   |
| integer    | an integer                                     |

## Creat a new table

Creating a table involves giving the table a name and a list of columns with types and (optional) constraint information. Each table should also nominate a unique identifier column to pinpoint specific rows. That identifier is called a PRIMARY KEY.  
We start by creating a countries table.

```sql
CREATE TABLE countries (
    country_code char(2) PRIMARY KEY,
    country_name text UNIQUE
);
```

This table will store a set of rows, where each is identified by a two character code and the name is unique. These columns both have constraints. The PRIMARY KEY constrains the country_code column to disallow duplicate country codes. Only one us and one gb may exist. We explicitly gave country_name a similar unique constraint, although it is not a primary key.  
Now we add a cities table. To ensure any inserted country_code also exists in our countries table, we add the REFERENCES keyword. Because the country_code column references another table’s key, it’s known as the foreign key constraint.

```sql
CREATE TABLE cities (
    name text NOT NULL,
    postal_code varchar(9) CHECK (postal_code <> ''),
    country_code char(2) REFERENCES countries,
    PRIMARY KEY (country_code, postal_code)
);
```

This time, we constrained the name in cities by disallowing NULL values. We constrained postal_code by checking that no values are empty strings (<> means not equal). Furthermore, because a PRIMARY KEY uniquely identifies a row, we created a compound key: country_code + postal_code. Together, they uniquely define a row.  
A venue exists in both a postal code and a specific country. The foreign key must be two columns that reference both cities primary key columns. (MATCH FULL is a constraint that ensures either both values exist or both are NULL.)

```sql
CREATE TABLE venues (
    venue_id SERIAL PRIMARY KEY,
    name varchar(255),
    street_address text,
    type char(7) CHECK ( type in ('public','private') ) DEFAULT 'public',
    postal_code varchar(9),
    country_code char(2),
    FOREIGN KEY (country_code, postal_code)
        REFERENCES cities (country_code, postal_code) MATCH FULL
);
```

This venue_id column is a common primary key setup: automatically incremented integers (1, 2, 3, 4, and so on). You make this identifier using the SERIAL keyword.  
We’ll create a new table named events. The events table should have these columns: a SERIAL integer event_id, a title, starts and ends (of type timestamp), and a venue_id (foreign key that references venues).

```sql
CREATE TABLE events(
	event_id SERIAL PRIMARY KEY,
	title text,
	starts timestamp,
	ends timestamp,
	venue_id integer,
	FOREIGN KEY (venue_id)
		REFERENCES venue (venue_id)
);
```

## CRUD

### Populating a table (C)

```sql
INSERT INTO countries (country_code, country_name)
VALUES ('us','United States'), ('mx','Mexico'), ('au','Australia'),
       ('gb','United Kingdom'), ('de','Germany'), ('dk','Denmark'),
       ('ll','Loompaland');
```

Trying to insert a row with a reference that doesn't exist.

```sql
INSERT INTO cities VALUES ('Toronto','M4C1B5','ca');

ERROR: insert or update on table "cities" violates foreign key constraint "cities_country_code_fkey"
DETAIL: Key (country_code)=(ca) is not present in table "countries".
```

This failure is good! Because country_code REFERENCES countries, the country_code must exist in the countries table. The REFERENCES keyword constrains fields to another table’s primary key. This is called maintaining referential integrity, and it ensures our data is always correct.  
It’s worth noting that NULL is valid for cities.country_code because NULL represents the lack of a value. If you want to disallow a NULL country_code reference, you would define the table cities column like this: `country_code char(2) REFERENCES countries NOT NULL`.

```sql
INSERT INTO cities
VALUES ('Portland','87200','us'), ('Kongens Lyngby','2800','dk');

INSERT INTO venues (name, postal_code, country_code)
VALUES ('Crystal Ballroom', '97206', 'us'), ('My Place','2800','dk');
```

You can optionally request that PostgreSQL return columns after insertion by ending the query with a RETURNING statement.

```sql
INSERT INTO venues (name, postal_code, country_code)
VALUES ('Voodoo Doughnut', '97206', 'us') RETURNING venue_id;

INSERT INTO events (title, starts, ends, venue_id)
VALUES ('Fight Club', '2018-02-15 17:30:00','2018-02-15 19:30:00',2),
       ('April Fools Day', '2018-04-01 00:00:00', '2018-04-01 23:59:00', NULL),
       ('Christmas Day', '2018-02-15 19:30:00', '2018-12-25 23:59:00', NULL);
```

Rather than setting the venue_id explicitly, you can sub-SELECT it using a more human-readable title.

```sql
INSERT INTO events (title, starts, ends, venue_id)
VALUES ('Moby', '2018-02-06 21:00', '2018-02-06 23:00', (SELECT venue_id
                                                         FROM venues
                                                         WHERE name = 'Crystal Ballroom')),
       ('Wedding', '2018-02-26 21:00:00', '2018-02-26 23:00:00', (SELECT venue_id
                                                                  FROM venues
                                                                  WHERE name = 'Voodoo Doughnut')),
       ('Dinner with Mom', '2018-02-26 18:00:00', '2018-02-26 20:30:00', (SELECT venue_id
                                                                          FROM venues
                                                                          WHERE name = 'My Place')),
       ('Valentine''s Day', '2018-02-14 00:00:00', '2018-02-14 23:59:00', null);
```

### Reading from a table (R)

We can validate that the proper rows were inserted by reading them using the SELECT...FROM table command.

```sql
SELECT * FROM countries;
```

| country_code | country_name   |
| ------------ | -------------- |
| us           | United States  |
| mx           | Mexico         |
| au           | Australia      |
| gb           | United Kingdom |
| de           | Germany        |
| ll           | Loompaland     |

### Updating a row (U)

We mistakenly entered a postal_code that doesn’t actually exist in Portland. One postal code that does exist and may just belong to one of the authors is 97206. Rather than delete and reinsert the value, we can update it inline.

```sql
UPDATE cities SET postal_code = '97206' WHERE name = 'Portland';
```

### Deleting from a table (D)

According to any respectable map, Loompaland isn’t a real place, so let’s remove it from the table. You specify which row to remove using the WHERE clause. The row whose country_code equals ll will be removed.

```sql
DELETE FROM countries WHERE country_code = 'll';
```

## Join

What sets relational databases like PostgreSQL apart is their ability to join tables together when reading them. Joining, in essence, is an operation taking two separate tables and combining them in some way to return a single table.

### Inner join

The basic form of a join is the inner join. In the simplest form, you specify two columns (one from each table) to match by, using the ON keyword.

```sql
SELECT cities.*, country_name
FROM cities INNER JOIN countries/*or FROM cities JOIN countries*/
    ON cities.country_code = countries.country_code;
```

The join returns a single table, sharing all columns’ values of the cities table plus the matching country_name value from the countries table.  
You can also join a table like cities that has a compound primary key.  
Joining the venues table with the cities table requires both foreign key columns. To save on typing, you can alias the table names by following the real table name directly with an alias, with an optional AS between (for example, venues v or venues AS v).

```sql
SELECT v.venue_id, v.name, c.name
FROM venues v INNER JOIN cities c
    ON v.postal_code=c.postal_code AND v.country_code=c.country_code;
```

### Outer join

In addition to inner joins, PostgreSQL can also perform outer joins. Outer joins are a way of merging two tables when the results of one table must always be returned, whether or not any matching column values exist on the other table.  
Let’s first craft a query that returns an event title and venue name as an inner join (the word INNER from INNER JOIN is not required, so leave it off here).

```sql
SELECT e.title, v.name FROM events e JOIN venues v ON e.venue_id = v.venue_id;
```

| title      | name            |
| ---------- | --------------- |
| Fight Club | Voodoo Doughnut |

INNER JOIN will return a row only if the column values match. Because we can’t have NULL venues.venue_id, the two NULL events.venue_ids refer to nothing. Retrieving all of the events, whether or not they have a venue, requires a LEFT OUTER JOIN (shortened to LEFT JOIN).

```sql
SELECT e.title, v.name FROM events e LEFT JOIN venues v ON e.venue_id = v.venue_id;
```

| title           | name            |
| --------------- | --------------- |
| Fight Club      | Voodoo Doughnut |
| April Fools Day |                 |
| Christmas Day   |                 |

If you require the inverse, all venues and only matching events, use a RIGHT JOIN. Finally, there’s the FULL JOIN, which is the union of LEFT and RIGHT; you’re guaranteed all values from each table, joined wherever columns match.

## Indexing

The speed of PostgreSQL (and any other RDBMS) lies in its efficient management of blocks of data, reduced disk reads, query optimization, and other techniques. But those only go so far in fetching results quickly. If we select the title of Christmas Day from the events table, the algorithm must scan every row for a match to return. Without an index, each row must be read from disk to know whether a query should return it.  
An index is a special data structure built to avoid a full table scan when performing a query.  
PostgreSQL automatically creates an index on the primary key—in particular a B-tree index—where the key is the primary key value and where the value points to a row on disk. Using the UNIQUE keyword is another way to force an index on a table column.  
You can explicitly add a hash index using the CREATE INDEX command, where
each value must be unique (like a hashtable or a map).

```sql
CREATE INDEX events_title ON events USING hash (title);
```

For less-than/greater-than/equals-to matches, we want an index more flexible than a simple hash, like a B-tree, which can match on ranged queries.

Consider a query to find all events that are on or after April 1.

```sql
SELECT * FROM events WHERE starts >= '2018-04-01';
```

For this, a tree is the perfect data structure. To index the starts column with a B-tree, use this:

```sql
CREATE INDEX events_starts ON events USING btree (starts);
```

Now our query over a range of dates will avoid a full table scan. It makes a huge difference when scanning millions or billions of rows.  
It’s worth noting that when you set a FOREIGN KEY constraint, PostgreSQL will not automatically create an index on the targeted column(s). You’ll need to create an index on the targeted column(s) yourself. Even if you don’t like using database constraints, you will often find yourself creating indexes on columns you plan to join against in order to help speed up foreign key joins.

## Aggregate Functions

An aggregate query groups results from several rows by some common criteria. It can be as simple as counting the number of rows in a table or calculating the average of some numerical column.  
The simplest aggregate function is count(), which is fairly self-explanatory. Counting all titles that contain the word Day (note: % is a wildcard on LIKE searches).

```sql
SELECT count(title) FROM events WHERE title LIKE '%Day%';
```

To get the first start time and last end time of all events at the Crystal Ballroom, use min() (return the smallest value) and max() (return the largest value).

```sql
SELECT min(starts), max(ends)
FROM events INNER JOIN venues
    ON events.venue_id = venues.venue_id
WHERE venues.name = 'Crystal Ballroom';
```

Aggregate functions are useful but limited on their own. If we wanted to count all events at each venue, we could write the following for each venue ID:

```sql
SELECT count(*) FROM events WHERE venue_id = 1;
SELECT count(*) FROM events WHERE venue_id = 2;
SELECT count(*) FROM events WHERE venue_id = 3;
SELECT count(*) FROM events WHERE venue_id IS NULL;
```

This would be tedious as the number of venues grows. This is where the GROUP BY command comes in handy.

## Grouping

GROUP BY is a shortcut for running the previous queries all at once. With GROUP BY, you tell Postgres to place the rows into groups and then perform some aggregate function (such as count()) on those groups.

```sql
SELECT venue_id, count(*) FROM events GROUP BY venue_id;
```

The GROUP BY condition has its own filter keyword: HAVING. HAVING is like the WHERE clause, except it can filter by aggregate functions (whereas WHERE cannot).  
The following query SELECTs the most popular venues, those with two or more events:

```sql
SELECT venue_id FROM events GROUP BY venue_id HAVING count(*) >= 2 AND venue_id IS NOT NULL;
```

You can use GROUP BY without any aggregate functions. If you call SELECT... FROM...GROUP BY on one column, you get only unique values.  
This kind of grouping is so common that SQL has a shortcut for it in the DISTINCT keyword.

```sql
SELECT venue_id FROM events GROUP BY venue_id;
SELECT DISTINCT venue_id FROM events;
```

The results of both queries will be identical.

## Window Functions

Window functions are similar to GROUP BY queries in that they allow you to run aggregate functions across multiple rows. The difference is that they allow you to use built-in aggregate functions without requiring every single field to be grouped to a single row.  
If we attempt to select the title column without grouping by it, we can expect an error.

```sql
SELECT title, venue_id, count(*) FROM events GROUP BY venue_id;

ERROR: column "events.title" must appear in the GROUP BY clause or \ be used in an aggregate function
```

We are counting up the rows by venue_id, and in the case of Fight Club and Wedding, we have two titles for a single venue_id. Postgres doesn’t know which title to display.  
Whereas a GROUP BY clause will return one record per matching group value, a window function, which does not collapse the results per group, can return a separate record for each row.  
Window functions return all matches and replicate the results of any aggregate function.

```sql
SELECT title, count(*) OVER (PARTITION BY venue_id) FROM events;
```

We like to think of PARTITION BY as akin to GROUP BY, but rather than grouping the results outside of the SELECT attribute list (and thus combining the results into fewer rows), it returns grouped values as any other field (calculating on the grouped variable but otherwise just another attribute). Or in SQL parlance, it returns the results of an aggregate function OVER a PARTITION of the result set.

## Transactions

Transactions are the bulwark of relational database consistency. All or nothing, that’s the transaction motto. Transactions ensure that every command of a set is executed. If anything fails along the way, all of the commands are rolled back as if they never happened.  
PostgreSQL transactions follow ACID compliance, which stands for:

-   **A**tomic (either all operations succeed or none do)
-   **C**onsistent (the data will always be in a good state and never in an inconsistent state)
-   **I**solated (transactions don’t interfere with one another)
-   **D**urable (a committed transaction is safe, even after a server crash)

We can wrap any transaction within a BEGIN TRANSACTION block. To verify atomicity, we’ll kill the transaction with the ROLLBACK command.

```sql
BEGIN TRANSACTION;
DELETE FROM events;
ROLLBACK;
SELECT * FROM events;
```

The events all remain. Transactions are useful when you’re modifying two tables that you don’t want out of sync. The classic example is a debit/credit system for a bank, where money is moved from one account to another:

```sql
BEGIN TRANSACTION;
    UPDATE account SET total=total+5000.0 WHERE account_id=1337;
    UPDATE account SET total=total-5000.0 WHERE account_id=45887;
END;
```

If something happened between the two updates, this bank just lost five grand. But when wrapped in a transaction block, the initial update is rolled back, even if the server explodes.

## Stored Procedures

Sometimes the database doesn‘t give us everything we need natively and we need to run some code to fill in the gaps. At that point, though, you need to decide where the code is going to run. Should it run in Postgres or should it run on the application side? If you decide you want the database to do the heavy lifting, Postgres offers stored procedures. Stored procedures are extremely powerful and can be used to do an enormous range of tasks, from performing complex mathematical operations that aren’t supported in SQL to triggering cascading series of events to pre-validating data before it‘s written to tables and far beyond. On the one hand, stored procedures can offer huge performance advantages. But the architectural costs can be high. You may avoid streaming thousands of rows to a client application, but you have also bound your application code to this database. And so the decision to use stored procedures should not be made lightly.  
We create a procedure (or FUNCTION) that simplifies INSERTing a new event at a venue without needing the venue_id. Here‘s what the procedure will accomplish: if the venue doesn’t exist, it will be created first and then referenced in the new event. The procedure will also return a Boolean indicating whether a new venue was added as a helpful bonus.

```sql
CREATE OR REPLACE FUNCTION add_event(
    title text,
    starts timestamp,
    ends timestamp,
    venue text,
    postal varchar(9),
    country char(2))
RETURNS boolean AS $$
DECLARE
    did_insert boolean := false;
    found_count integer;
    the_venue_id integer;
BEGIN
    SELECT venue_id INTO the_venue_id
    FROM venues v
    WHERE v.postal_code=postal AND v.country_code=country AND v.name ILIKE venue
    LIMIT 1;
    IF the_venue_id IS NULL THEN
        INSERT INTO venues (name, postal_code, country_code)
        VALUES (venue, postal, country)
        RETURNING venue_id INTO the_venue_id;
        did_insert := true;
    END IF;
    -- Note: this is a notice, not an error as in some programming languages
    RAISE NOTICE 'Venue found %', the_venue_id;
    INSERT INTO events (title, starts, ends, venue_id)
    VALUES (title, starts, ends, the_venue_id);
    RETURN did_insert;
END;
$$ LANGUAGE plpgsql;
```

This stored procedure is run as a SELECT statement.

```sql
SELECT add_event('House Party', '2018-05-03 23:00', '2018-05-04 02:00', 'Run''s House', '97206', 'us');
```

This saves a client two round-trip SQL commands to the database (a SELECT and then an INSERT) and instead performs only one.  
The language we used in the procedure we wrote is PL/pgSQL (which stands for Procedural Language/PostgreSQL).  
In addition to PL/pgSQL, Postgres supports three more core languages for writing procedures: Tcl (PL/Tcl), Perl (PL/Perl), and Python (PL/Python).

Try this shell command:

```shell
createlang 7dbs --list
```

It will list the languages installed in your database. The createlang command is also used to add new languages, which you can find online.

## Triggers

Triggers automatically fire stored procedures when some event happens, such as an insert or update. They allow the database to enforce some required behavior in response to changing data.  
We create a new PL/pgSQL function that logs whenever an event is updated (we want to be sure no one changes an event and tries to deny it later). First, create a logs table to store event changes. A primary key isn’t necessary here because it’s just a log.

```sql
CREATE TABLE logs (
    event_id integer,
    old_title varchar(255),
    old_starts timestamp,
    old_ends timestamp,
    logged_at timestamp DEFAULT current_timestamp
);
```

Next, we build a function to insert old data into the log. The OLD variable represents the row about to be changed (NEW represents an incoming row, which we’ll see in action soon enough). Output a notice to the console with the event_id before returning.

```sql
CREATE OR REPLACE FUNCTION log_event() RETURNS trigger AS $$
DECLARE
BEGIN
    INSERT INTO logs (event_id, old_title, old_starts, old_ends)
    VALUES (OLD.event_id, OLD.title, OLD.starts, OLD.ends);
    RAISE NOTICE 'Someone just changed event #%', OLD.event_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

Finally, we create our trigger to log changes after any row is updated.

```sql
CREATE TRIGGER log_events
    AFTER UPDATE ON events
    FOR EACH ROW EXECUTE PROCEDURE log_event();
```

So, it turns out our party at Run’s House has to end earlier than we hoped. We change the event.

```sql
UPDATE events
SET ends='2018-05-04 01:00:00'
WHERE title='House Party';

NOTICE: Someone just changed event #9
```

And the old end time was logged.  
Triggers can also be created before updates and before or after inserts.

## Views

Wouldn’t it be great if you could use the results of a complex query just like any other table? Well, that’s exactly what VIEWs are for. Unlike stored procedures, these aren’t functions being executed but rather aliased queries. Let’s say that we wanted to see only holidays that contain the word Day and have no venue. We could create a VIEW for that like this:

```sql
CREATE VIEW holidays AS
    SELECT event_id AS holiday_id, title AS name, starts AS date
    FROM events
    WHERE title LIKE '%Day%' AND venue_id IS NULL;
```

Creating a view really is as simple as writing a query and prefixing it with CREATE VIEW some_view_name AS. Now you can query holidays like any other table. Under the covers it’s the plain old events table.  
Views are powerful tools for opening up complex queried data in a simple way. The query may be a roiling sea of complexity underneath, but all you see is a table.  
If you want to add a new column to the holidays view, it will have to come from the underlying table.  
Views cannot be updated directly.

## Rules

A RULE is a description of how to alter the parsed query tree. Every time Postgres runs an SQL statement, it parses the statement into a query tree (generally called an abstract syntax tree).  
Operators and values become branches and leaves in the tree, and the tree is walked, pruned, and in other ways edited before execution. This tree is optionally rewritten by Postgres rules, before being sent on to the query planner (which also rewrites the tree to run optimally), and sends this final command to be executed.  
So, to allow updates against our holidays view, we need to craft a RULE that tells Postgres what to do with an UPDATE. Our rule will capture updates to the holidays view and instead run the update on events, pulling values from the pseudorelations NEW and OLD. NEW functionally acts as the relation containing the values we’re setting, while OLD contains the values we query by.

```sql
CREATE RULE update_holidays AS ON UPDATE TO holidays DO INSTEAD
    UPDATE events
    SET title = NEW.name,
        starts = NEW.date,
        colors = NEW.colors
    WHERE title = OLD.name;
```

With this rule in place, now we can update holidays directly.

## Crosstab

We’re going to build a monthly calendar of events, where each month in the calendar year counts the number of events in that month. This kind of operation is commonly done by a pivot table. These constructs “pivot” grouped data around some other output, in our case a list of months. We’ll build our pivot table using the crosstab() function.  
Start by crafting a query to count the number of events per month each year. PostgreSQL provides an extract() function that returns some subfield from a date or timestamp, which aids in our grouping.

```sql
SELECT extract(year from starts) as year,
    extract(month from starts) as month, count(*)
FROM events
GROUP BY year, month
ORDER BY year, month;
```

To use crosstab(), the query must return three columns: rowid, category, and value. We’ll be using the year as an ID, which means the other fields are category (the month) and value (the count).  
The crosstab() function needs another set of values to represent months. This is how the function knows how many columns we need. These are the values that become the columns (the table to pivot against). So let’s create a table to store a list of numbers. Because we‘ll only need the table for a few operations, we‘ll create an ephemeral table, which lasts only as long as the current Postgres session, using the CREATE TEMPORARY TABLE command.

```sql
CREATE TEMPORARY TABLE month_count(month INT);
INSERT INTO month_count VALUES (1),(2),(3),(4),(5),(6),(7),(8),(9),(10),(11),(12);
```

Now we’re ready to call crosstab() with our two queries.  
Remember, the pivot table is using our months as categories, but those months are just integers. So, we define them like this:

```sql
SELECT * FROM crosstab(
    'SELECT extract(year from starts) as year,
        extract(month from starts) as month, count(*)
    FROM events
    GROUP BY year, month
    ORDER BY year, month',
    'SELECT * FROM month_count'
) AS (
    year int,
    jan int, feb int, mar int, apr int, may int, jun int,
    jul int, aug int, sep int, oct int, nov int, dec int
) ORDER BY YEAR;
```
