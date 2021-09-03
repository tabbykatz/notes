# PSQL

### PostgreSQL client application commands

| terminal command|  psql equivalent|
|--|--|
| `creatdb`  |   CREATE DATABASE|
| `dropdb` |  DROP DATABASE|
| `psql ` | to enter psql shell |
| `psql -d db_name` |  connect to db as you enter|

### some quick psql metacommands

| metacommand | action |
|--|--|
|`\p`|show contents of the query buffer |
| `\r`| reset the query buffer  |
|`\l`| list all databases |
| `\c database_name` |  connect|
| `\dt` | list tables in current db by schema |
|`\d table_name`|describe a table's schema|
|  `\q`| quit |
| `\s`| show your command history!|

 ### CREATE TABLE syntax

```sql
CREATE TABLE table_name (
    column_1_name column_1_data_type [constraints, ...],
    column_2_name column_2_data_type [constraints, ...],
    .
    .
    .
    constraints
);

```
| command word | meaning |datatype| constraint or default value|
|--|--|--|--|
|`CREATE TABLE name (` | make a table named 'name'| | |
| `id serial UNIQUE NOT NULL,` | column called id | serial| UNIQUE: prevents any duplicate values from being entered into that column.
| | | |NOT NULL: a value MUST be specified for this column; it cannot be left empty|
| `username char (25),` | col called username | char(25)||
| `enabled boolean DEFAULT TRUE);` | col called enabled |boolean | DEFAULT: TRUE|

### data types

|Data Type|Description|
|--|--|
|serial*|This data type is used to create identifier columns for a PostgreSQL database. These identifiers are integers, auto-incrementing, and cannot contain a null value.|
| char(N) | This data type specifies that information stored in a column can contain strings of up to N characters in length. If a string less than length N is stored, then the remaining string length is filled with space characters. |
|varchar(N)|This data type specifies that information stored in a column can contain strings of up to N characters in length. If a string less than length N is stored, then the remaining string length isn't used.
|boolean|This is a data type that can only contain two values "true" or "false". In PostgreSQL, boolean values are often displayed in a shorthand format, t or f
|integer or INT|An integer is simply a "whole number." An example might be 1 or 50, -50, or 792197 depending on what storage type is used.
|decimal(precision, scale)|The decimal type takes two arguments, one being the total number of digits in the entire number on both sides of the decimal point (the precision), the second is the number of the digits in the fractional part of the number to the right of the decimal point (the scale).
|timestamp|The timestamp type contains both a simple date and time in YYYY-MM-DD HH:MM:SS format.
|date|The date type contains a date but no time.
|*The use of `serial` is [no longer recommended](https://wiki.postgresql.org/wiki/Don%27t_Do_This#Don.27t_use_serial) for new (production) applications.

###  ALTER TABLE* Syntax
```psql
ALTER TABLE table_to_change
    stuff_to_change_goes_here
    additional_arguments
```
*to alter schema

|command  | action |
|--|--|
| `ALTER TABLE table_name RENAME TO new_name;` | rename the table |
`ALTER TABLE table_name RENAME COLUMN col_name TO new_col_name;`| rename a column | 
|`ALTER TABLE table_name ALTER COLUMN col_name TYPE new_data_type;`| change a column's data type|
|`ALTER TABLE table_name ALTER COLUMN column_name SET NOT NULL;` | add `NOT NULL` constraint|
|`ALTER TABLE table_name ADD CONSTRAINT [constraint_name] constraint_clause;`| Add literally any other constraint|
|||
|||
|`ALTER CONSTRAINT`| change certain aspects of Foreign Key constraints|
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNTg3NDA3MjQsLTEzMjgwODY5NjgsLT
E1MTM1NDAwNDYsMTg1MjQxMjU3Niw3NjAwNzg5MzYsMTg3OTkz
OTU0NiwtMTI1NjM4OTM3MSwxMzAxNDY4NDY4LC0xOTI2NzUwNT
A0XX0=
-->