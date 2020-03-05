# [Programming RDB](https://datsoftlyngby.github.io/soft2020spring/DB/week-07/#2-programming-rdb)

13-02-2020

## Declarations

```sql
vStaffNo VARCHAR2(5);
vRent NUMBER(6, 2) NOT NULL :5 600;
MAX_PROPERTIES CONSTANT NUMBER :5 100;
vStaffNo Staff.staffNo%TYPE;
vStaffNo1 vStaffNo%TYPE;
vStaffRec Staff%ROWTYPE;
```

## Assigments

```sql
vStaffNo :5 'SG14';
vRent :5 500;
SELECT COUNT(*) INTO x FROM PropertyForRent WHERE staffNo 5 vStaffNo;
SET vStaffNo 5 'SG14'
```

## Control Statements

### If

```sql
IF (condition) THEN
    <SQL statement list>
[ELSIF (condition) THEN <SQL statement list>]
[ELSE <SQL statement list>]
END IF;

/*example*/
IF (position 5 'Manager') THEN
    salary :5 salary*1.05;
ELSE
    salary :5 salary*1.03;
END IF;
```

### Case

```sql
CASE (operand)
[WHEN (whenOperandList) | WHEN (searchCondition)
    THEN <SQL statement list>]
[ELSE <SQL statement list>]
END CASE;

/*examples*/
CASE lowercase(x)
    WHEN 'a'        THEN x:=1;
    WHEN 'b'        THEN x:=2;
                    THEN y:=0;
    WHEN 'default'  THEN x:=3;
END CASE;

UPDATE Staff
SET salary = CASE
    WHEN position = 'Manager'
        THEN salary * 1.05
    ELSE
        THEN salary * 1.02
    END;
```

### Loop

```sql
[labelName:]
LOOP
    <SQL statement list>
    EXIT [labelName] [WHEN (condition)]
END LOOP [labelName];

/*example*/
x:51;
myLoop:
LOOP
    x :5 x11;
    IF (x . 3) THEN
        EXIT myLoop; --- exit loop immediately
END LOOP myLoop;
--- control resumes here
y := 2;
```

### While

```sql
/*PL/SQL*/
WHILE (condition) LOOP
    <SQL statement list>
END LOOP[labelName];

/*SQL*/
WHILE (condition) DO
    <SQL statement list>
END WHILE[labelName];
```

### Repeat

```sql
REPEAT
    <SQL statement list>
UNTIL (condition)
END REPEAT [labelName];
```

### For

```sql
/*PL/SQL*/
FOR indexVariable
    IN lowerbound .. upperbound LOOP
    <SQL statement list>
END LOOP [labelName];

/*example*/
DECLARE numberOfStaff NUMBER;
SELECT COUNT(*) INTO numberOfStaff FROM PropertyForRent
    WHERE staffNo 5 'SG14';myLoop1:
FOR iStaff IN 1 .. numberOfStaff LOOP
    .....
END LOOP myLoop1;

/*SQL*/
FOR indexVariable
    AS querySpecification DO
    <SQL statement list>
END FOR [labelName];

/*example*/
myLoop1:
FOR iStaff AS SELECT COUNT(*) FROM PropertyForRent
              WHERE staffNo 5 'SG14' DO
    .....
END FOR myLoop1;
```

## Exceptions

```sql
DECLARE
    vpCount NUMBER;
    vStaffNo PropertyForRent.staffNo%TYPE := 'SG14'
-- define an exception for the enterprise constraint that prevents a member of staff
-- managing more than 100 properties
    e_too_many_properties EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_too_many_properties,-20000);
BEGIN
    SELECT COUNT(*) INTO vpCount
    FROM PropertyForRent
    WHERE staffNo = vStaffNo;
    IF vpCount = 100
-- raise an exception for the general constraint
        RAISE e_too_many_properties;
    END IF;
EXCEPTION
-- handle the exception for the general constraint
    WHEN e_too_many_properties THEN
        dbms_output.put_line('Member of staff'||staffNo||'already managing 100 properties');
END;
```

## Cursor

```sql
DECLARE
    vPropertyNo PropertyForRent.propertyNo%TYPE
    vStreet     PropertyForRent.street%TYPE
    vCity       PropertyForRent.city%TYPE
    vPostcode   PropertyForRent.postcode%TYPE
    CURSOR propertyCursor IS
        SELECTpropertyNo,street,city,postcode
        FROM PropertyForRent
        WHERE staffNo = 'SG14'
        ORDER BY propertyNo;
BEGIN
-- open the cursor to start of selection, then loop to fetch each row of the result table
    OPEN propertyCursor
    LOOP
-- fetch next row of the result table
        FETCH propertyCursor
            INTO vPropertyNo,vStreet,vCity,vPostcode;
        EXIT WHEN propertyCursor%NOTFOUND;
-- display data
        dbms_output.put_line('Property number:'||vPropertyNo);
        dbms_output.put_line('Street:'||vStreet);
        dbms_output.put_line('City:'||vCity);
        IF postcode IS NOT NULL THEN
            dbms_output.put_line('Post Code:'||vPostcode);
        ELSE
            dbms_output.put_line('Post Code:NULL');
        END IF;
    END LOOP;
    IF propertyCursor%ISOPEN THEN CLOSE propertyCursor END IF;
-- error condition - print out error
EXCEPTION
    WHEN OTHERS THAN
        dbms_output.put_line('Error detected');
    IF propertyCursor%ISOPEN THEN CLOSE propertyCursor END IF;
END;
```

## Subprograms

Subprograms are named PL/SQL blocks that can take parameters and be invoked. PL/SQL has two types of subprogram called (stored) procedures and functions. Procedures and functions can take a set of parameters given to them by the calling program and perform a set of actions. Both can modify and return data passed to them as a parameter. The difference between a procedure and a function is that a function will always return a single value to the caller, whereas a procedure does not. Usually, procedures are used unless only one return value is needed.  
Procedures and functions are very similar to those found in most high-level programming languages, and have the same advantages: they provide modularity and extensibility, they promote reusability and maintainability, and they aid abstraction. A parameter has a specified name and data type but can also be designated as:

-   IN
    -   parameter is used as an input value only.
-   OUT
    -   parameter is used as an output value only.
-   IN OUT
    -   parameter is used as both an input and an output value.

```sql
CREATE OR REPLACE PROCEDURE PropertiesForStaff
    (IN vStaffNo VARCHAR2)
AS . . .
```

## Trigger

```sql
CREATE TRIGGER TriggerName
    BEFORE | AFTER | INSTEAD OF
    INSERT | DELETE | UPDATE [OF TriggerColumnList]
    ON TableName
    [REFERENCING {OLD | NEW} AS {OldName | NewName}]
    [FOR EACH {ROW | STATEMENT}]
    [WHEN Condition]
    <trigger action>
```

## Recursion

```sql
WITH RECURSIVE
AllManagers (staffNo, managerStaffNo) AS
    (SELECT staffNo, managerStaffNo
    FROM Staff
    UNION
    SELECT in.staffNo, out.managerStaffNo
    FROM AllManagers in, Staff out
    WHERE in.managerStaffNo 5 out.staffNo);
SELECT * FROM AllManagers
ORDER BY staffNo, managerStaffNo;
```
