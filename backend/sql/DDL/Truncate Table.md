https://www.tutorialspoint.com/sql/sql-truncate-table.htm

The `TRUNCATE TABLE` statement is a Data Definition Language (DDL) operation that is used to mark the extents of a table for deallocation (empty for reuse). The result of this operation quickly removes all data from a table, typically bypassing a number of integrity enforcing mechanisms intended to protect data (like triggers).

It effectively eliminates all records in a table, but not the table itself. Unlike the `DELETE` statement, `TRUNCATE TABLE` does not generate individual row delete statements, so the usual overhead for logging or locking does not apply.


SQL provides command to TRUNCATE a table completely in one go instead of deleting table records one by one which will be very time consuming and cumbersome process.

## The SQL TRUNCATE TABLE Statement

The SQL **TRUNCATE TABLE** command is used to empty a table. This command is a sequence of DROP TABLE and CREATE TABLE statements and requires the DROP privilege.

You can also use DROP TABLE command to [delete a table](https://www.tutorialspoint.com/sql/sql-delete-table.htm "delete a table") but it will remove the complete table structure from the database and you would need to re-create this table once again if you wish you store some data again.

### Syntax

The basic syntax of a **TRUNCATE TABLE** command is as follows.

TRUNCATE TABLE table_name;

### Example

First let's create a table **CUSTOMERS** which can store the personal details of customers including their name, age, address and salary etc. as shown below −

Open Compiler

CREATE TABLE CUSTOMERS (
   ID INT NOT NULL,
   NAME VARCHAR (20) NOT NULL,
   AGE INT NOT NULL,
   ADDRESS CHAR (25),
   SALARY DECIMAL (18, 2),       
   PRIMARY KEY (ID)
);

Now insert values into this table using the INSERT statement as follows −

Open Compiler

INSERT INTO CUSTOMERS VALUES 
(1, 'Ramesh', 32, 'Ahmedabad', 2000.00 ),
(2, 'Khilan', 25, 'Delhi', 1500.00 ),
(3, 'Kaushik', 23, 'Kota', 2000.00 ),
(4, 'Chaitali', 25, 'Mumbai', 6500.00 ),
(5, 'Hardik', 27, 'Bhopal', 8500.00 ),
(6, 'Komal', 22, 'Hyderabad', 4500.00 ),
(7, 'Muffy', 24, 'Indore', 10000.00 );

The table will be created as follows −

|ID|NAME|AGE|ADDRESS|SALARY|
|---|---|---|---|---|
|1|Ramesh|32|Ahmedabad|2000.00|
|2|Khilan|25|Delhi|1500.00|
|3|Kaushik|23|Kota|2000.00|
|4|Chaitali|25|Mumbai|6500.00|
|5|Hardik|27|Bhopal|8500.00|
|6|Komal|22|Hyderabad|4500.00|
|7|Muffy|24|Indore|10000.00|

Following SQL **TRUNCATE TABLE CUSTOMER** statement will remove all the records of the CUSTOMERS table −

TRUNCATE TABLE CUSTOMERS;

### Verification

Now, the CUSTOMERS table is truncated and the output from SELECT statement will be as shown in the code block below −

Open Compiler

SELECT * FROM CUSTOMERS;

Following will be the output −

Empty set (0.00 sec)

## TRUNCATE vs DELETE

Even though the TRUNCATE and DELETE commands work similar logically, there are some major differences that exist between them. They are detailed in the table below.

|DELETE|TRUNCATE|
|---|---|
|The [DELETE command](https://www.tutorialspoint.com/sql/sql-delete-table.htm "delete command") in SQL removes one or more rows from a table based on the conditions specified in a WHERE Clause.|SQL's TRUNCATE command is used to remove all of the rows from a table, regardless of whether or not any conditions are met.|
|It is a DML(Data Manipulation Language) command.|It is a DDL(Data Definition Language) command.|
|There is a need to make a manual COMMIT after making changes to the DELETE command, for the modifications to be committed.|When you use the TRUNCATE command, the modifications made to the table are committed automatically.|
|It deletes rows one at a time and applies same criteria to each deletion.|It removes all of the information in one go.|
|The WHERE clause serves as the condition in this case.|The WHERE Clause is not available.|
|All rows are locked after deletion.|TRUNCATE utilizes a table lock, which locks the pages so they cannot be deleted.|
|It makes a record of each and every transaction in the log file.|The only activity recorded is the deallocation of the pages on which the data is stored.|
|It consumes a greater amount of transaction space compared to TRUNCATE command.|It takes comparatively less amount of transaction space.|
|If there is an identity column, the table identity is not reset to the value it had when the table was created.|It returns the table identity to a value it was given as a seed.|
|It requires authorization to delete.|It requires table alter permission.|
|When it comes to large databases, it is much slower.|It is much faster.|

## TRUNCATE vs DROP

Unlike TRUNCATE that resets the table structure, [DROP command](https://www.tutorialspoint.com/sql/sql-drop-table.htm "drop command") completely frees the table space from the memory. They are both Data Definition Language (DDL) operations as they interact with the definitions of database objects; which allows the database to automatically commit once these commands are executed with no chance to roll back.

However, there are still some differences exist between these two commands, which have been summarized in the following table −

|DROP|TRUNCATE|
|---|---|
|The DROP command in SQL removes an entire table from a database including its definition, indexes, constraints, data etc.|The TRUNCATE command is used to remove all of the rows from a table, regardless of whether or not any conditions are met and resets the table definition.|
|It is a DDL(Data Definition Language) command.|It is also a DDL(Data Definition Language) command.|
|The table space is completely freed from the memory.|The table still exists in the memory.|
|All the integrity constraints are removed.|The integrity constraints still exist in the table.|
|Requires ALTER and CONTROL permissions on the table schema and table respectively, to be able to perform this command.|Only requires the ALTER permissions to truncate the table.|
|DROP command is much slower than TRUNCATE but faster than DELETE.|TRUNCATE command is faster than both DROP and DELETE commands.|
