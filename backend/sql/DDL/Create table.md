
https://www.programiz.com/sql/create-table
https://www.tutorialspoint.com/sql/sql-create-table.htm

# SQL CREATE TABLE

The SQL `CREATE TABLE` statement is used to create a database table. We use this table to store records (data). For example,

### Example

```
-- create a table named Companies with different columns

CREATE TABLE Companies (
  id int,
  name varchar(50),
  address text,
  email varchar(50),
  phone varchar(10)
);
```

[Run Code](https://www.programiz.com/sql/online-compiler)

Here, the SQL command creates a database named Companies with the columns: id, name, address, email and phone.

---

## SQL CREATE TABLE Syntax

```
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    column3 datatype,
   ....
);
```

Here,

- **table_name** is name of the table you want to create
- **column** is the name of a column in the table
- **datatype** is the type of data that the column can hold (e.g., integer, varchar, date)

---

## Example: SQL CREATE TABLE

```
-- create a table Students with different columns
CREATE TABLE Students(
  id int,
  name varchar(50),
  address text,
  grades  varchar(50),
  phone varchar(10)
);
```

[Run Code](https://www.programiz.com/sql/online-compiler)

Here, we created a table named Students with **five** columns.

![Create Table in SQL](https://www.programiz.com/sites/tutorial2program/files/sql-create-table-example.png "Create Table in SQL")

Create Table in SQL

The table we created will not contain any data as we have not inserted anything into the table

Here, `int`, `varchar(50)`, and `text` specify types of data that could be stored in the respective columns.

**Note:** We must provide data types for each column while creating a table. To learn more, visit [SQL Data Types](https://www.programiz.com/sql/data-types).

---

## CREATE TABLE IF NOT EXISTS

If we try to create a table that already exists, we get an error message `'Error: table already exists'`.

To fix this issue, we can add the optional `IF NOT EXISTS` command while creating a table.

Let's look at an example.

```
-- create a Companies table if it does not exist

CREATE TABLE IF NOT EXISTS Companies (
  id int,
  name varchar(50),
  address text,
  email varchar(50),
  phone varchar(10)
);
```

[Run Code](https://www.programiz.com/sql/online-compiler)

Here, the SQL command checks if a table named Companies exists, and if not, it creates a table with specified columns.

---

## Create Table Using Another Existing Table

In SQL, we can create a new table by duplicating an existing table's structure.

Let's look at an example.

```
-- create a backup table from the existing table Customers

CREATE TABLE CustomersBackup 
    AS 
    SELECT * 
    FROM Customers;
```

[Run Code](https://www.programiz.com/sql/online-compiler)

This SQL command creates the new table named CustomersBackup, duplicating the structure of the Customers table.

**Note**: You can choose to copy all or specific columns.