# DBMS Lab External --- Complete Revision Notes

**Oracle SQL + PL/SQL \| Practical revision sheet**

> Use this as a last-minute revision guide. The examples use the
> `Employee`, `Department`, and `Person` tables created during
> preparation. Run table-creation and sample-data scripts once before
> practising dependent queries.

------------------------------------------------------------------------

## 0. Sample tables and data

### Create tables

``` sql
CREATE TABLE Department (
    deptno NUMBER(2) PRIMARY KEY,
    dname  VARCHAR2(30) NOT NULL,
    loc    VARCHAR2(30)
);

CREATE TABLE Employee (
    empno  NUMBER(5) PRIMARY KEY,
    ename  VARCHAR2(30) NOT NULL,
    email  VARCHAR2(50) UNIQUE,
    sal    NUMBER(8,2) CHECK (sal > 0),
    deptno NUMBER(2),
    CONSTRAINT emp_dept_fk
        FOREIGN KEY (deptno)
        REFERENCES Department(deptno)
);
```

### Insert sample data

``` sql
INSERT INTO Department VALUES (10, 'CSE', 'Hyderabad');
INSERT INTO Department VALUES (20, 'ECE', 'Hyderabad');
INSERT INTO Department VALUES (30, 'IT', 'Bengaluru');

INSERT INTO Employee VALUES (101, 'Ravi',  'ravi@gmail.com',  30000, 10);
INSERT INTO Employee VALUES (102, 'Anu',   'anu@gmail.com',   45000, 20);
INSERT INTO Employee VALUES (103, 'Kiran', 'kiran@gmail.com', 35000, 10);
INSERT INTO Employee VALUES (104, 'Priya', 'priya@gmail.com', 50000, 30);
INSERT INTO Employee VALUES (105, 'Arjun', 'arjun@gmail.com', 25000, 20);

COMMIT;
```

For self-join/manager examples, add a manager column and values:

``` sql
ALTER TABLE Employee ADD (mgr NUMBER(5));

UPDATE Employee SET mgr = NULL WHERE empno = 101;
UPDATE Employee SET mgr = 101  WHERE empno IN (102, 103);
UPDATE Employee SET mgr = 102  WHERE empno IN (104, 105);

COMMIT;
```

Manager relationships: - Ravi (101) has no manager. - Anu (102) and
Kiran (103) report to Ravi. - Priya (104) and Arjun (105) report to Anu.

------------------------------------------------------------------------

# PART A --- SQL

## 1. SQL command categories

  -----------------------------------------------------------------------
  Category                Purpose                 Commands
  ----------------------- ----------------------- -----------------------
  DDL                     Defines/changes         `CREATE`, `ALTER`,
                          database structures     `DROP`, `TRUNCATE`

  DML                     Changes table data      `INSERT`, `UPDATE`,
                                                  `DELETE`

  DQL                     Retrieves data          `SELECT`

  DCL                     Controls permissions    `GRANT`, `REVOKE`

  TCL                     Controls transactions   `COMMIT`, `ROLLBACK`,
                                                  `SAVEPOINT`
  -----------------------------------------------------------------------

### Common examples

``` sql
-- Add a column
ALTER TABLE Employee ADD (phone VARCHAR2(15));

-- Modify a column
ALTER TABLE Employee MODIFY (ename VARCHAR2(50));

-- Remove a column
ALTER TABLE Employee DROP COLUMN phone;

-- Remove all rows but keep the table structure
TRUNCATE TABLE Employee;

-- Remove the table structure and its data
DROP TABLE Employee;
```

**Remember:** `DELETE` can remove selected rows using `WHERE`;
`TRUNCATE` removes all rows and does not take a `WHERE` condition.

## 2. INSERT, UPDATE, DELETE, SELECT

``` sql
-- Insert one row
INSERT INTO Employee (empno, ename, email, sal, deptno)
VALUES (106, 'Rahul', 'rahul@gmail.com', 28000, 10);

-- Update selected rows
UPDATE Employee
SET sal = 30000
WHERE empno = 101;

-- Delete selected rows
DELETE FROM Employee
WHERE empno = 106;

-- Display all columns
SELECT * FROM Employee;

-- Display selected columns
SELECT ename, sal
FROM Employee;
```

**Exam caution:** Always check whether an `UPDATE` or `DELETE` needs a
`WHERE` clause. Without it, every row is affected.

## 3. Data types

  Type                   Use
  ---------------------- -----------------------------------
  `CHAR(n)`              Fixed-length character data
  `VARCHAR2(n)`          Variable-length character data
  `NCHAR`, `NVARCHAR2`   Unicode character data
  `NUMBER(p)`            Number with precision `p`
  `NUMBER(p,s)`          Precision `p`, scale `s`
  `DATE`                 Date and time to seconds
  `TIMESTAMP`            Date/time with fractional seconds
  `CLOB`                 Large character data
  `BLOB`                 Large binary data

Examples:

``` sql
ename VARCHAR2(30)
sal   NUMBER(8,2)
dob   DATE
```

## 4. Constraints

Constraints enforce rules on table data.

  Constraint      Meaning
  --------------- ----------------------------------------------
  `PRIMARY KEY`   Uniquely identifies each row; cannot be NULL
  `NOT NULL`      Value must be supplied
  `UNIQUE`        Prevents duplicate values
  `CHECK`         Enforces a condition
  `FOREIGN KEY`   References a key in another table

### Examples

``` sql
CREATE TABLE Student (
    sid    NUMBER PRIMARY KEY,
    sname  VARCHAR2(30) NOT NULL,
    email  VARCHAR2(50) UNIQUE,
    age    NUMBER CHECK (age >= 18),
    deptno NUMBER,
    CONSTRAINT student_dept_fk
        FOREIGN KEY (deptno)
        REFERENCES Department(deptno)
);
```

Named constraints can be added later:

``` sql
ALTER TABLE Student
ADD CONSTRAINT student_age_ck CHECK (age >= 18);
```

Composite primary key:

``` sql
CREATE TABLE Enrolment (
    student_id NUMBER,
    course_id  NUMBER,
    CONSTRAINT enrolment_pk PRIMARY KEY (student_id, course_id)
);
```

### Important age/DOB distinction

If a table stores an `AGE` column:

``` sql
age NUMBER CHECK (age >= 18)
```

If a table stores `DOB` and the rule is "must be at least 18 years old
today", a normal Oracle `CHECK` constraint cannot use `SYSDATE` because
it is non-deterministic. Use a trigger to enforce the rule dynamically.
Do not write a `SYSDATE`-based CHECK constraint.

## 5. SELECT, WHERE and filtering

``` sql
-- All employees
SELECT * FROM Employee;

-- Salary greater than 30000
SELECT ename, sal
FROM Employee
WHERE sal > 30000;

-- Employees in departments 10 and 20
SELECT *
FROM Employee
WHERE deptno IN (10, 20);

-- Salary from 25000 to 45000 (inclusive)
SELECT *
FROM Employee
WHERE sal BETWEEN 25000 AND 45000;

-- Names beginning with A
SELECT *
FROM Employee
WHERE ename LIKE 'A%';

-- Names with A as the second character
SELECT *
FROM Employee
WHERE ename LIKE '_a%';

-- Rows where email is missing
SELECT *
FROM Employee
WHERE email IS NULL;

-- Unique department numbers
SELECT DISTINCT deptno
FROM Employee;

-- Highest salary first
SELECT ename, sal
FROM Employee
ORDER BY sal DESC;
```

### Operators to remember

-   Comparisons: `=`, `<>`, `!=`, `>`, `<`, `>=`, `<=`
-   Logical: `AND`, `OR`, `NOT`
-   Range: `BETWEEN ... AND ...` (inclusive)
-   Set membership: `IN`, `NOT IN`
-   Pattern matching: `LIKE`
-   NULL tests: `IS NULL`, `IS NOT NULL`

**NULL rule:** Do not write `column = NULL`. Use `IS NULL`.

`LIKE` wildcards: - `%` = zero or more characters. - `_` = exactly one
character.

## 6. SQL functions

### Character functions

``` sql
SELECT UPPER(ename) FROM Employee;
SELECT LOWER(ename) FROM Employee;
SELECT INITCAP(ename) FROM Employee;
SELECT LENGTH(ename) FROM Employee;
SELECT SUBSTR(ename, 1, 3) FROM Employee;
SELECT INSTR(ename, 'a') FROM Employee;
SELECT CONCAT(ename, ' Employee') FROM Employee;

-- Concatenation operator
SELECT ename || ' earns ' || sal FROM Employee;
```

### Numeric functions

``` sql
SELECT ROUND(456.789, 2) FROM DUAL;  -- 456.79
SELECT TRUNC(456.789, 2) FROM DUAL;  -- 456.78
SELECT MOD(10, 2) FROM DUAL;         -- 0
SELECT ABS(-25) FROM DUAL;           -- 25
SELECT CEIL(4.2) FROM DUAL;          -- 5
SELECT FLOOR(4.8) FROM DUAL;         -- 4
```

`DUAL` is Oracle's one-row table, commonly used to evaluate expressions.

### Date functions

``` sql
SELECT SYSDATE FROM DUAL;
SELECT ADD_MONTHS(SYSDATE, 6) FROM DUAL;
SELECT MONTHS_BETWEEN(SYSDATE, DATE '2005-06-15') FROM DUAL;
SELECT LAST_DAY(SYSDATE) FROM DUAL;
SELECT NEXT_DAY(SYSDATE, 'MONDAY') FROM DUAL;
SELECT EXTRACT(YEAR FROM SYSDATE) FROM DUAL;
```

Insert a date safely:

``` sql
INSERT INTO Person (pid, pname, dob)
VALUES (1, 'Ravi', TO_DATE('15-06-2005', 'DD-MM-YYYY'));
```

Oracle date literal format:

``` sql
DATE '2005-06-15'
```

### Date conversion and formatting

``` sql
-- Text to DATE
SELECT TO_DATE('15-06-2005', 'DD-MM-YYYY') FROM DUAL;

-- DATE to formatted text
SELECT TO_CHAR(SYSDATE, 'DD-MON-YYYY') FROM DUAL;

-- Text to number
SELECT TO_NUMBER('25000') FROM DUAL;
```

`TO_DATE` converts text into a date. `TO_CHAR` converts a date/number
into formatted text.

### Age calculation from DOB

``` sql
SELECT ename,
       TRUNC(MONTHS_BETWEEN(SYSDATE, dob) / 12) AS age
FROM Person;
```

For a precise 18th-birthday cutoff:

``` sql
SELECT pname, dob
FROM Person
WHERE dob <= ADD_MONTHS(TRUNC(SYSDATE), -216);
```

### NULL-handling function

``` sql
SELECT ename, NVL(email, 'No email')
FROM Employee;
```

`NVL(value, replacement)` returns the replacement when the first value
is NULL.

## 7. Aggregate functions

Aggregate functions calculate one result from multiple rows.

``` sql
SELECT COUNT(*) FROM Employee;
SELECT SUM(sal) FROM Employee;
SELECT AVG(sal) FROM Employee;
SELECT MAX(sal) FROM Employee;
SELECT MIN(sal) FROM Employee;
```

-   `COUNT(*)` counts rows.
-   `COUNT(column)` counts non-NULL values in that column.
-   `SUM`, `AVG`, `MAX`, `MIN` ignore NULL values.

With the sample employees: - Total salary = 185000 - Average salary =
37000 - Maximum salary = 50000 - Minimum salary = 25000

## 8. GROUP BY and HAVING

`GROUP BY` creates groups. `HAVING` filters groups.

``` sql
-- Average salary in each department
SELECT deptno, AVG(sal)
FROM Employee
GROUP BY deptno;

-- Departments whose average salary is above 30000
SELECT deptno, AVG(sal)
FROM Employee
GROUP BY deptno
HAVING AVG(sal) > 30000;

-- Count employees in each department
SELECT deptno, COUNT(*)
FROM Employee
GROUP BY deptno;

-- Departments having more than one employee
SELECT deptno, COUNT(*)
FROM Employee
GROUP BY deptno
HAVING COUNT(*) > 1;

-- Filter individual employees first, then group
SELECT deptno, AVG(sal)
FROM Employee
WHERE sal > 25000
GROUP BY deptno
HAVING AVG(sal) > 30000;
```

### WHERE vs HAVING

-   `WHERE` filters individual rows before grouping.
-   `HAVING` filters groups after `GROUP BY`.
-   Use `WHERE` for conditions on ordinary rows.
-   Use `HAVING` for conditions involving aggregate results such as
    `COUNT(*)`, `AVG(sal)`.

**Important:** A selected non-aggregate column should normally appear in
the `GROUP BY` clause.

Logical processing order to remember:

`FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`

## 9. JOINS --- important practice section

A JOIN combines related rows from two or more tables.

### The most important skill: writing the ON condition

Ask: 1. Which tables are involved? 2. Which column connects them? 3. Is
the connecting column a foreign key and referenced primary key?

For Employee and Department, the relationship is:

`Employee.deptno` → `Department.deptno`

Therefore:

``` sql
ON E.deptno = D.deptno
```

### INNER JOIN

Returns only matching rows.

``` sql
SELECT E.ename, D.dname
FROM Employee E
INNER JOIN Department D
ON E.deptno = D.deptno;
```

### LEFT JOIN

Returns all rows from the left table and matching rows from the right
table. Non-matches on the right appear as NULL.

``` sql
SELECT E.ename, D.dname
FROM Employee E
LEFT JOIN Department D
ON E.deptno = D.deptno;
```

### RIGHT JOIN

Returns all rows from the right table and matching rows from the left
table.

``` sql
SELECT E.ename, D.dname
FROM Employee E
RIGHT JOIN Department D
ON E.deptno = D.deptno;
```

### FULL OUTER JOIN

Returns matched rows and unmatched rows from both tables.

``` sql
SELECT E.ename, D.dname
FROM Employee E
FULL OUTER JOIN Department D
ON E.deptno = D.deptno;
```

### CROSS JOIN

Returns every possible combination of rows. No `ON` condition is used.

``` sql
SELECT E.ename, D.dname
FROM Employee E
CROSS JOIN Department D;
```

If the first table has `m` rows and the second has `n` rows, CROSS JOIN
returns `m × n` rows. With 5 employees and 3 departments, it returns 15
rows.

### SELF JOIN

A table is joined with itself. Use aliases to distinguish the two
copies.

Employee name and manager name:

``` sql
SELECT E.ename AS Employee,
       M.ename AS Manager
FROM Employee E
LEFT JOIN Employee M
ON E.mgr = M.empno;
```

Manager and subordinate:

``` sql
SELECT E.ename AS Manager,
       M.ename AS Subordinate
FROM Employee E
LEFT JOIN Employee M
ON M.mgr = E.empno;
```

**Do not confuse these conditions:**

``` sql
-- Employee's manager
ON E.mgr = M.empno

-- Employees reporting to a manager
ON M.mgr = E.empno
```

They are different relationships.

### Join with a range condition

If a `Salgrade` table has `losal` and `hisal`:

``` sql
SELECT E.ename, E.sal, S.grade
FROM Employee E
JOIN Salgrade S
ON E.sal BETWEEN S.losal AND S.hisal;
```

### Join writing template

``` sql
SELECT columns
FROM Table1 A
JOIN Table2 B
ON A.common_column = B.common_column;
```

For Employee--Department, remember `E.deptno = D.deptno`.

## 10. Set operations

Set operators combine the results of two SELECT statements.

Both SELECT statements must return: - The same number of columns. -
Corresponding columns with compatible data types. - Columns in the same
logical order.

``` sql
-- Combines results and removes duplicates
SELECT deptno FROM Employee
UNION
SELECT deptno FROM Department;

-- Keeps duplicates
SELECT deptno FROM Employee
UNION ALL
SELECT deptno FROM Department;

-- Returns common values
SELECT deptno FROM Employee
INTERSECT
SELECT deptno FROM Department;

-- Values in first query but not in second
SELECT deptno FROM Department
MINUS
SELECT deptno FROM Employee;
```

Remember: - `UNION` = combine, remove duplicates. - `UNION ALL` =
combine, keep duplicates. - `INTERSECT` = common rows. - `MINUS` = first
result minus second result; order matters.

With the sample data, Employee department values have 5 rows and
Department has 3 rows. `UNION ALL` returns 8 rows.

## 11. SUBQUERIES --- important practice section

A subquery is a query inside another query.

### Single-row subquery

Returns one value. Use operators such as `=`, `>`, `<`, `>=`, `<=`.

Employees earning more than the company average:

``` sql
SELECT ename, sal
FROM Employee
WHERE sal > (
    SELECT AVG(sal)
    FROM Employee
);
```

Employees earning more than the minimum salary:

``` sql
SELECT ename
FROM Employee
WHERE sal > (
    SELECT MIN(sal)
    FROM Employee
);
```

### Multi-row subquery using IN

Employees working in departments located in Hyderabad:

``` sql
SELECT ename
FROM Employee
WHERE deptno IN (
    SELECT deptno
    FROM Department
    WHERE loc = 'Hyderabad'
);
```

Employees working in departments where at least one employee earns more
than 40000:

``` sql
SELECT ename
FROM Employee
WHERE deptno IN (
    SELECT deptno
    FROM Employee
    WHERE sal > 40000
);
```

### ANY and ALL

Suppose the subquery returns salaries 25000 and 45000.

-   `> ANY` means greater than at least one returned value.
-   `> ALL` means greater than every returned value.
-   `< ANY` means less than at least one returned value.
-   `< ALL` means less than every returned value.

For a non-empty set: - `X > ANY(set)` is equivalent to `X > MIN(set)`. -
`X > ALL(set)` is equivalent to `X > MAX(set)`. - `X < ANY(set)` is
equivalent to `X < MAX(set)`. - `X < ALL(set)` is equivalent to
`X < MIN(set)`.

Examples:

``` sql
-- Salary greater than at least one salary in department 20
SELECT ename, sal
FROM Employee
WHERE sal > ANY (
    SELECT sal
    FROM Employee
    WHERE deptno = 20
);

-- Salary greater than every salary in department 20
SELECT ename, sal
FROM Employee
WHERE sal > ALL (
    SELECT sal
    FROM Employee
    WHERE deptno = 20
);
```

### EXISTS and NOT EXISTS

`EXISTS` checks whether the subquery returns at least one row.
`NOT EXISTS` checks that it returns no rows.

Departments that have at least one employee:

``` sql
SELECT D.dname
FROM Department D
WHERE EXISTS (
    SELECT 1
    FROM Employee E
    WHERE E.deptno = D.deptno
);
```

Departments that have no employees:

``` sql
SELECT D.dname
FROM Department D
WHERE NOT EXISTS (
    SELECT 1
    FROM Employee E
    WHERE E.deptno = D.deptno
);
```

`SELECT 1` is commonly used because EXISTS only checks whether a row
exists; it does not need the selected column's value.

### Correlated subquery --- the difficult one

A correlated subquery refers to a column from the outer query. The inner
query is evaluated in relation to the current outer row.

**Question:** Display employees whose salary is greater than the average
salary of their own department.

``` sql
SELECT E.ename, E.sal, E.deptno
FROM Employee E
WHERE E.sal > (
    SELECT AVG(I.sal)
    FROM Employee I
    WHERE I.deptno = E.deptno
);
```

Understand the aliases: - `E` = current employee in the outer query. -
`I` = employees examined by the inner query. - `I.deptno = E.deptno`
means: calculate the average for the department of the current outer
employee.

Walk-through: - Ravi is in department 10. Department 10 average = 32500.
Ravi earns 30000, so he is excluded. - Anu is in department 20.
Department 20 average = 35000. Anu earns 45000, so she is included. -
Kiran is in department 10. Kiran earns 35000, so he is included. - Priya
is in department 30. Department 30 average = 50000. Her salary equals
the average, so she is excluded. - Arjun is in department 20. He earns
25000, so he is excluded.

Expected names: Anu and Kiran.

**This is the key correlated-subquery condition:**

``` sql
WHERE I.deptno = E.deptno
```

Do not replace it with `I.deptno = 10`; the outer alias makes the
condition work for every employee's department.

Another correlated example: employees earning more than every other
employee in their own department:

``` sql
SELECT E.ename, E.sal, E.deptno
FROM Employee E
WHERE E.sal > (
    SELECT MAX(M.sal)
    FROM Employee M
    WHERE M.deptno = E.deptno
      AND M.empno <> E.empno
);
```

The `M.empno <> E.empno` condition excludes the current employee from
the inner comparison. If an employee is the only employee in their
department, the inner query returns NULL and the comparison is not TRUE.

### How to build a subquery from an English question

1.  Identify what value/list must be calculated first.
2.  Write that inner SELECT.
3.  Decide whether it returns one value or multiple values.
4.  Choose the outer comparison: `=`, `>`, `IN`, `ANY`, `ALL`, `EXISTS`.
5.  Write the outer SELECT.

Example: "Employees earning more than the average salary."

-   Inner question: What is the average salary?
-   Inner SQL: `SELECT AVG(sal) FROM Employee`
-   Outer question: Which employees have salary greater than that?
-   Outer SQL: `SELECT ename FROM Employee WHERE sal > (...)`

------------------------------------------------------------------------

# PART B --- PL/SQL

## 12. PL/SQL block structure

PL/SQL combines SQL with programming features such as variables,
conditions, loops and exception handling.

``` sql
DECLARE
    -- Optional declarations: variables, cursors
BEGIN
    -- Mandatory executable statements
EXCEPTION
    -- Optional error handling
END;
/
```

-   `DECLARE` is optional.
-   `BEGIN` and `END` are mandatory.
-   `EXCEPTION` is optional.
-   `/` submits the block in SQL\*Plus-style environments. In online
    IDEs, follow the tool's execution instructions.

### First PL/SQL program

``` sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello DBMS');
END;
/
```

`DBMS_OUTPUT.PUT_LINE` displays a line of output. Make sure the
environment's DBMS Output display is enabled if required.

## 13. Variables and assignment

``` sql
DECLARE
    v_name   VARCHAR2(30) := 'Ravi';
    v_salary NUMBER := 30000;
    v_age    NUMBER(2) := 20;
BEGIN
    DBMS_OUTPUT.PUT_LINE(v_name);
    DBMS_OUTPUT.PUT_LINE(v_salary);
    DBMS_OUTPUT.PUT_LINE(v_age);
END;
/
```

-   `:=` assigns a value.
-   `=` is used for comparison.
-   `NUMBER(2)` allows up to two digits of precision.

### `%TYPE`

A variable can inherit the datatype of a table column:

``` sql
DECLARE
    v_salary Employee.sal%TYPE;
BEGIN
    SELECT sal INTO v_salary
    FROM Employee
    WHERE empno = 101;

    DBMS_OUTPUT.PUT_LINE(v_salary);
END;
/
```

## 14. SELECT INTO

Use `SELECT ... INTO` to retrieve a single row's value into PL/SQL
variable(s).

``` sql
DECLARE
    v_salary Employee.sal%TYPE;
BEGIN
    SELECT sal INTO v_salary
    FROM Employee
    WHERE empno = 102;

    DBMS_OUTPUT.PUT_LINE(v_salary);
END;
/
```

**Important:** A `SELECT INTO` must return exactly one row: - Zero rows
→ `NO_DATA_FOUND`. - More than one row → `TOO_MANY_ROWS`.

For multiple rows, use a cursor.

## 15. IF, ELSIF, ELSE

``` sql
DECLARE
    v_sal NUMBER := 30000;
BEGIN
    IF v_sal > 25000 THEN
        DBMS_OUTPUT.PUT_LINE('High Salary');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Low Salary');
    END IF;
END;
/
```

Multiple conditions:

``` sql
IF condition1 THEN
    statements;
ELSIF condition2 THEN
    statements;
ELSE
    statements;
END IF;
```

**Remember:** The keyword is `ELSIF`, not `ELSE IF`.

Pass/fail example:

``` sql
DECLARE
    v_marks NUMBER := 75;
BEGIN
    IF v_marks >= 40 THEN
        DBMS_OUTPUT.PUT_LINE('PASS');
    ELSE
        DBMS_OUTPUT.PUT_LINE('FAIL');
    END IF;
END;
/
```

## 16. Loops

### FOR LOOP

``` sql
BEGIN
    FOR i IN 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
END;
/
```

Reverse:

``` sql
BEGIN
    FOR i IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
END;
/
```

Even numbers from 1 to 10 --- this is the practice program:

``` sql
BEGIN
    FOR i IN 1..10 LOOP
        IF MOD(i, 2) = 0 THEN
            DBMS_OUTPUT.PUT_LINE(i);
        END IF;
    END LOOP;
END;
/
```

Basic PL/SQL numeric FOR loops do not use `BY 2`; use `MOD` or another
condition to select even numbers.

### WHILE LOOP

``` sql
DECLARE
    i NUMBER := 1;
BEGIN
    WHILE i <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
        i := i + 1;
    END LOOP;
END;
/
```

### Simple LOOP

``` sql
DECLARE
    i NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE(i);
        i := i + 1;

        EXIT WHEN i > 5;
    END LOOP;
END;
/
```

  Loop          Use
  ------------- -----------------------------------------
  FOR           Known range
  WHILE         Continue while a condition is true
  Simple LOOP   Continue until an EXIT condition is met

## 17. Exception handling

An exception handles runtime errors.

Common predefined exceptions:

  Exception         Meaning
  ----------------- ------------------------------------
  `NO_DATA_FOUND`   SELECT INTO returned no rows
  `TOO_MANY_ROWS`   SELECT INTO returned multiple rows
  `ZERO_DIVIDE`     Division by zero
  `OTHERS`          Handles other exceptions

Example:

``` sql
DECLARE
    v_num    NUMBER := 10;
    v_result NUMBER;
BEGIN
    v_result := v_num / 0;
    DBMS_OUTPUT.PUT_LINE(v_result);

EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Cannot divide by zero');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Some error occurred');
END;
/
```

General form:

``` sql
EXCEPTION
    WHEN exception_name THEN
        statements;
    WHEN OTHERS THEN
        statements;
```

`WHEN OTHERS` is usually placed last.

## 18. Procedures

A procedure is a named PL/SQL program stored in the database. It can be
executed when needed.

### Basic procedure

``` sql
CREATE OR REPLACE PROCEDURE greet
IS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello Student');
END;
/
```

Execute:

``` sql
EXECUTE greet;
```

Or:

``` sql
BEGIN
    greet;
END;
/
```

### Procedure with an IN parameter

``` sql
CREATE OR REPLACE PROCEDURE show_salary (
    p_empno IN NUMBER
)
IS
    v_salary NUMBER;
BEGIN
    SELECT sal INTO v_salary
    FROM Employee
    WHERE empno = p_empno;

    DBMS_OUTPUT.PUT_LINE(v_salary);
END;
/
```

Execute:

``` sql
EXECUTE show_salary(101);
```

Parameter modes: - `IN`: sends a value into the procedure. - `OUT`:
sends a value back through a parameter. - `IN OUT`: accepts a value and
can return its modified value.

### IS vs AS

In Oracle stored procedure and function declarations, `IS` and `AS` are
generally interchangeable.

``` sql
CREATE OR REPLACE PROCEDURE greet
IS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello');
END;
/
```

`AS` can be used in place of `IS`.

## 19. Functions

A function is a named PL/SQL program that **must return a value**.

``` sql
CREATE OR REPLACE FUNCTION find_square (
    n IN NUMBER
)
RETURN NUMBER
IS
BEGIN
    RETURN n * n;
END;
/
```

Call it:

``` sql
SELECT find_square(6) FROM DUAL;
```

Expected result: `36`.

### Procedure vs Function

  -----------------------------------------------------------------------
  Procedure                           Function
  ----------------------------------- -----------------------------------
  Performs an operation               Computes/returns a value

  Does not require a RETURN value     Must return a value

  Can use OUT parameters              Uses `RETURN`

  Called as a procedure               Can be used in an expression or
                                      eligible SQL statement
  -----------------------------------------------------------------------

## 20. Cursors

A cursor is a pointer/handle to a SQL statement's result set. It allows
query results to be processed.

Two main types: 1. Implicit cursor --- Oracle manages it automatically.
2. Explicit cursor --- Programmer declares and controls it.

### Implicit cursor

Oracle automatically creates implicit cursors for SQL statements such as
INSERT, UPDATE, DELETE and single-row SELECT INTO.

``` sql
BEGIN
    UPDATE Employee
    SET sal = sal + 1000
    WHERE deptno = 10;

    DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' rows updated');
END;
/
```

Implicit cursor attributes:

  -----------------------------------------------------------------------
  Attribute                           Meaning
  ----------------------------------- -----------------------------------
  `SQL%FOUND`                         TRUE if the most recent SQL
                                      statement affected/fetched at least
                                      one row

  `SQL%NOTFOUND`                      TRUE if it affected/fetched no rows

  `SQL%ROWCOUNT`                      Number of rows affected/fetched

  `SQL%ISOPEN`                        Always FALSE after an implicit SQL
                                      statement completes
  -----------------------------------------------------------------------

### Explicit cursor --- manual OPEN, FETCH, CLOSE

Four steps: 1. DECLARE cursor. 2. OPEN cursor. 3. FETCH rows. 4. CLOSE
cursor.

Example: Display all employee names.

``` sql
DECLARE
    CURSOR c_emp IS
        SELECT ename FROM Employee;

    v_name Employee.ename%TYPE;
BEGIN
    OPEN c_emp;

    LOOP
        FETCH c_emp INTO v_name;

        EXIT WHEN c_emp%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(v_name);
    END LOOP;

    CLOSE c_emp;
END;
/
```

Example: Display names and salaries.

``` sql
DECLARE
    CURSOR c_emp IS
        SELECT ename, sal FROM Employee;

    v_name Employee.ename%TYPE;
    v_sal  Employee.sal%TYPE;
BEGIN
    OPEN c_emp;

    LOOP
        FETCH c_emp INTO v_name, v_sal;

        EXIT WHEN c_emp%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(v_name || ' ' || v_sal);
    END LOOP;

    CLOSE c_emp;
END;
/
```

**Important:** Put `EXIT WHEN c_emp%NOTFOUND` immediately after FETCH
and before printing. Otherwise, the previous variable value may be
printed again after the final unsuccessful fetch.

Explicit cursor attributes:

  Attribute          Meaning
  ------------------ -------------------------------
  `c_emp%FOUND`      Last FETCH returned a row
  `c_emp%NOTFOUND`   Last FETCH returned no row
  `c_emp%ROWCOUNT`   Number of rows fetched so far
  `c_emp%ISOPEN`     Whether the cursor is open

### Cursor FOR LOOP

Oracle automatically opens, fetches and closes the cursor.

``` sql
DECLARE
    CURSOR c_emp IS
        SELECT ename FROM Employee;
BEGIN
    FOR emp IN c_emp LOOP
        DBMS_OUTPUT.PUT_LINE(emp.ename);
    END LOOP;
END;
/
```

`emp` is a record for the current row. Access its column with
`emp.ename`.

A cursor FOR LOOP using a declared cursor is still an explicit cursor.
For a question that asks you to demonstrate manual cursor operations,
use OPEN/FETCH/CLOSE.

## 21. Triggers --- high priority

A trigger is a stored PL/SQL block that automatically executes when a
specified database event occurs.

### Trigger timing

-   `BEFORE`: executes before the event.
-   `AFTER`: executes after the event.

### Trigger events

-   `INSERT`
-   `UPDATE`
-   `DELETE`

### Trigger level

-   Statement-level: executes once for the entire SQL statement.
-   Row-level: executes once for every affected row; indicated by
    `FOR EACH ROW`.

### General syntax

``` sql
CREATE OR REPLACE TRIGGER trigger_name
BEFORE INSERT ON table_name
FOR EACH ROW
BEGIN
    -- Trigger statements
END;
/
```

### :NEW and :OLD

  Operation   `:NEW`                       `:OLD`
  ----------- ---------------------------- -----------------------------
  INSERT      New row values               Not available as an old row
  UPDATE      New values                   Previous values
  DELETE      Not available as a new row   Deleted row values

Salary update example: - Old salary = 25000. - New salary = 35000. -
`:OLD.sal` = 25000. - `:NEW.sal` = 35000.

### Trigger: prevent salary below 10000

``` sql
CREATE OR REPLACE TRIGGER check_salary
BEFORE INSERT ON Employee
FOR EACH ROW
BEGIN
    IF :NEW.sal < 10000 THEN
        RAISE_APPLICATION_ERROR(
            -20001,
            'Salary must be at least 10000'
        );
    END IF;
END;
/
```

If the condition is true, the custom error stops the insert.

### Trigger: age must be at least 18 when DOB is stored

Assume:

``` sql
CREATE TABLE Person (
    pid   NUMBER PRIMARY KEY,
    pname VARCHAR2(30),
    dob   DATE
);
```

Trigger:

``` sql
CREATE OR REPLACE TRIGGER check_age
BEFORE INSERT OR UPDATE OF dob ON Person
FOR EACH ROW
BEGIN
    IF :NEW.dob IS NULL THEN
        RAISE_APPLICATION_ERROR(-20001, 'Date of birth is required');

    ELSIF :NEW.dob > ADD_MONTHS(TRUNC(SYSDATE), -216) THEN
        RAISE_APPLICATION_ERROR(-20002, 'Age must be at least 18');
    END IF;
END;
/
```

Explanation: - 216 months = 18 years. -
`ADD_MONTHS(TRUNC(SYSDATE), -216)` calculates the date 18 years ago. - A
DOB later than that cutoff means the person is under 18. - The trigger
checks both INSERT and UPDATE of DOB.

### Trigger: before deleting an employee

``` sql
CREATE OR REPLACE TRIGGER before_delete
BEFORE DELETE ON Employee
FOR EACH ROW
BEGIN
    DBMS_OUTPUT.PUT_LINE('Employee is being deleted');
END;
/
```

### Trigger: after salary update

``` sql
CREATE OR REPLACE TRIGGER after_salary_update
AFTER UPDATE OF sal ON Employee
FOR EACH ROW
BEGIN
    DBMS_OUTPUT.PUT_LINE(
        'Salary changed from ' || :OLD.sal ||
        ' to ' || :NEW.sal
    );
END;
/
```

### CHECK constraint vs trigger --- do not confuse

If the table stores an AGE column:

``` sql
age NUMBER CHECK (age >= 18)
```

If the table stores DOB and age must be at least 18 relative to today's
date, use a trigger. Oracle does not allow `SYSDATE` in a CHECK
constraint.

## 22. SQL Injection

SQL injection is a vulnerability where untrusted user input changes the
meaning of a dynamically constructed SQL statement.

It usually happens when an application concatenates raw user input into
SQL.

Example of a normal query:

``` sql
SELECT *
FROM Users
WHERE username = 'Ravi'
AND password = '1234';
```

A malicious input such as:

``` text
' OR '1'='1
```

can alter the condition if the application inserts it directly into a
dynamically constructed query. The exact resulting query depends on how
the application builds the SQL.

**Important:** SQL injection is an application-level vulnerability. SQL
does not automatically inject itself. The application must construct and
execute an unsafe query using untrusted input.

### Prevention: bind variables / parameterized queries

An Oracle application should bind values separately instead of
concatenating them into SQL.

``` sql
SELECT *
FROM Users
WHERE username = :username
AND password = :password;
```

`:username` and `:password` are bind-variable placeholders. The
application must bind actual values separately before executing the
statement.

Prevention points: 1. Use bind variables / parameterized queries. 2.
Validate user input. 3. Apply least-privilege permissions. 4. Avoid
concatenating untrusted input into dynamic SQL. 5. Avoid exposing
detailed database errors to users.

------------------------------------------------------------------------

# PART C --- WINDOW FUNCTIONS AND RANKING

## 23. What are window functions?

Window functions calculate a value across a set of related rows while
**keeping each original row** in the output.

Unlike `GROUP BY`, a window function does not collapse all rows in a
group into one row.

General syntax:

``` sql
function_name(expression)
OVER (
    PARTITION BY column
    ORDER BY column
)
```

-   `OVER` defines the window.
-   `PARTITION BY` divides rows into groups; it is optional.
-   `ORDER BY` defines ordering within the window; it depends on the
    function/task.

### GROUP BY vs window function

GROUP BY:

``` sql
SELECT deptno, AVG(sal)
FROM Employee
GROUP BY deptno;
```

Returns one row per department.

Window function:

``` sql
SELECT ename, deptno, sal,
       AVG(sal) OVER (PARTITION BY deptno) AS dept_avg
FROM Employee;
```

Returns every employee row, with their department average shown
alongside it.

## 24. ROW_NUMBER, RANK, DENSE_RANK

These assign ranking numbers to rows.

``` sql
SELECT ename, deptno, sal,
       ROW_NUMBER() OVER (
           PARTITION BY deptno
           ORDER BY sal DESC
       ) AS row_num,
       RANK() OVER (
           PARTITION BY deptno
           ORDER BY sal DESC
       ) AS salary_rank,
       DENSE_RANK() OVER (
           PARTITION BY deptno
           ORDER BY sal DESC
       ) AS dense_salary_rank
FROM Employee;
```

Differences:

  Function         What happens with ties?
  ---------------- --------------------------------------------
  `ROW_NUMBER()`   Gives every row a unique sequential number
  `RANK()`         Tied rows share rank; next rank has a gap
  `DENSE_RANK()`   Tied rows share rank; next rank has no gap

Example salaries sorted descending: 50000, 45000, 45000, 30000.

    Salary   ROW_NUMBER   RANK   DENSE_RANK
  -------- ------------ ------ ------------
     50000            1      1            1
     45000            2      2            2
     45000            3      2            2
     30000            4      4            3

### Rank employees department-wise

``` sql
SELECT empno, ename, deptno, sal,
       RANK() OVER (
           PARTITION BY deptno
           ORDER BY sal DESC
       ) AS salary_rank
FROM Employee;
```

-   `PARTITION BY deptno` restarts the ranking for each department.
-   `ORDER BY sal DESC` gives the highest salary rank 1.

### Top salary employee in each department

Use a subquery because window functions are not directly filtered in the
same SELECT's WHERE clause:

``` sql
SELECT empno, ename, deptno, sal
FROM (
    SELECT E.*,
           RANK() OVER (
               PARTITION BY deptno
               ORDER BY sal DESC
           ) AS rnk
    FROM Employee E
)
WHERE rnk = 1;
```

This returns all tied top-salary employees in each department.

If the question asks for exactly one row per department, use
`ROW_NUMBER()` with a deterministic tie-breaker:

``` sql
SELECT empno, ename, deptno, sal
FROM (
    SELECT E.*,
           ROW_NUMBER() OVER (
               PARTITION BY deptno
               ORDER BY sal DESC, empno
           ) AS rn
    FROM Employee E
)
WHERE rn = 1;
```

## 25. LEAD and LAG

These access a value from a following or preceding row in the ordered
window.

### LAG --- previous row's value

``` sql
SELECT ename, sal,
       LAG(sal) OVER (ORDER BY sal) AS previous_salary
FROM Employee;
```

### LEAD --- next row's value

``` sql
SELECT ename, sal,
       LEAD(sal) OVER (ORDER BY sal) AS next_salary
FROM Employee;
```

With partitioning:

``` sql
SELECT ename, deptno, sal,
       LAG(sal) OVER (
           PARTITION BY deptno
           ORDER BY sal
       ) AS previous_salary_in_dept
FROM Employee;
```

-   `LAG` looks backward.
-   `LEAD` looks forward.
-   The first `LAG` value or last `LEAD` value is NULL if no default is
    specified.

Optional default:

``` sql
LAG(sal, 1, 0) OVER (ORDER BY sal)
```

This returns 0 when there is no previous row.

## 26. FIRST_VALUE and LAST_VALUE

These return the first or last value in a window frame.

``` sql
SELECT ename, deptno, sal,
       FIRST_VALUE(sal) OVER (
           PARTITION BY deptno
           ORDER BY sal DESC
       ) AS highest_salary
FROM Employee;
```

`LAST_VALUE` needs care because the default window frame often ends at
the current row. To get the last value across the full partition,
specify the frame:

``` sql
SELECT ename, deptno, sal,
       LAST_VALUE(sal) OVER (
           PARTITION BY deptno
           ORDER BY sal
           ROWS BETWEEN UNBOUNDED PRECEDING
                    AND UNBOUNDED FOLLOWING
       ) AS highest_salary
FROM Employee;
```

In this ascending order, the last salary in the full department
partition is the highest.

### Window-function vocabulary

-   `PARTITION BY`: divides rows into groups.
-   `ORDER BY`: sorts rows inside each group/window.
-   `ROWS BETWEEN`: defines the window frame.
-   `UNBOUNDED PRECEDING`: start at the first row in the partition.
-   `UNBOUNDED FOLLOWING`: end at the last row in the partition.
-   `CURRENT ROW`: current row.

------------------------------------------------------------------------

# PART D --- LAST-MINUTE PRACTICAL TEMPLATES

## 27. SQL templates to memorize

### Join two related tables

``` sql
SELECT columns
FROM Table1 A
JOIN Table2 B
ON A.foreign_key = B.primary_key;
```

Employee--Department:

``` sql
SELECT E.ename, D.dname
FROM Employee E
JOIN Department D
ON E.deptno = D.deptno;
```

### Aggregate by department

``` sql
SELECT deptno, COUNT(*)
FROM Employee
GROUP BY deptno
HAVING COUNT(*) > 1;
```

### Single-row subquery

``` sql
SELECT columns
FROM table_name
WHERE column_name > (
    SELECT aggregate_function(column_name)
    FROM table_name
);
```

### Multi-row subquery

``` sql
SELECT columns
FROM table_name
WHERE column_name IN (
    SELECT column_name
    FROM another_table
    WHERE condition
);
```

### Correlated subquery

``` sql
SELECT A.column_name
FROM table_name A
WHERE A.value > (
    SELECT AVG(B.value)
    FROM table_name B
    WHERE B.group_column = A.group_column
);
```

### Explicit cursor

``` sql
DECLARE
    CURSOR c1 IS SELECT column_name FROM table_name;
    v_value table_name.column_name%TYPE;
BEGIN
    OPEN c1;
    LOOP
        FETCH c1 INTO v_value;
        EXIT WHEN c1%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_value);
    END LOOP;
    CLOSE c1;
END;
/
```

### Procedure

``` sql
CREATE OR REPLACE PROCEDURE procedure_name
IS
BEGIN
    statements;
END;
/
```

### Function

``` sql
CREATE OR REPLACE FUNCTION function_name
RETURN NUMBER
IS
BEGIN
    RETURN value;
END;
/
```

### Trigger

``` sql
CREATE OR REPLACE TRIGGER trigger_name
BEFORE INSERT OR UPDATE ON table_name
FOR EACH ROW
BEGIN
    IF condition THEN
        RAISE_APPLICATION_ERROR(-20001, 'Error message');
    END IF;
END;
/
```

------------------------------------------------------------------------

# PART E --- COMMON MISTAKES AND QUICK CHECKS

## 28. Mistakes to avoid

1.  `SELECT INTO` is for exactly one row. Use a cursor for multiple
    rows.
2.  Use `:=` for assignment in PL/SQL; use `=` for comparison.
3.  Write `ELSIF`, not `ELSE IF`.
4.  In a cursor loop, check `%NOTFOUND` immediately after FETCH.
5.  Close a manually managed explicit cursor.
6.  `FOR EACH ROW` makes a trigger row-level.
7.  `:OLD` means previous values; `:NEW` means new values.
8.  `BEFORE` and `AFTER` describe trigger timing.
9.  `WHERE` filters rows; `HAVING` filters groups.
10. Do not use aggregate conditions in `WHERE`; use `HAVING` after
    grouping.
11. In a join, identify the real relationship before writing `ON`.
12. In a self join, aliases represent two roles played by the same
    table.
13. `UNION` removes duplicates; `UNION ALL` keeps them.
14. `MINUS` is directional: first query minus second query.
15. `IN` is useful when a subquery returns multiple values.
16. `EXISTS` checks whether at least one row exists.
17. In a correlated subquery, connect the inner alias to the current
    outer row.
18. `RANK` leaves gaps after ties; `DENSE_RANK` does not.
19. `ROW_NUMBER` gives each row a unique sequence number.
20. Oracle CHECK constraints cannot use `SYSDATE` to enforce a dynamic
    DOB age rule.

## 29. Important concepts --- one-line revision

  -----------------------------------------------------------------------
  Concept                             Remember
  ----------------------------------- -----------------------------------
  `DUAL`                              One-row Oracle table for testing
                                      expressions

  `%TYPE`                             Inherits a column's datatype

  `DBMS_OUTPUT.PUT_LINE`              Displays output

  `NO_DATA_FOUND`                     SELECT INTO returned no rows

  `TOO_MANY_ROWS`                     SELECT INTO returned more than one
                                      row

  `SQL%ROWCOUNT`                      Number of rows affected by the most
                                      recent implicit SQL statement

  `MOD(n,2)`                          Remainder after division by 2; 0
                                      means even

  `ADD_MONTHS(date,-216)`             Date 18 years earlier

  `RAISE_APPLICATION_ERROR`           Raises a custom application error

  `IN`                                Matches any value in a
                                      list/subquery

  `ANY`                               At least one comparison must be
                                      true

  `ALL`                               Every comparison must be true

  `EXISTS`                            Subquery returns at least one row

  `PARTITION BY`                      Splits rows into groups for a
                                      window function
  -----------------------------------------------------------------------

## 30. Suggested final revision order

1.  Write one trigger from memory, especially the DOB age trigger.
2.  Write one explicit cursor using OPEN/FETCH/CLOSE.
3.  Write one procedure and one function.
4.  Practise Employee--Department JOIN and both SELF JOIN directions.
5.  Practise one `IN` subquery and one correlated subquery.
6.  Revise GROUP BY/HAVING and set operators.
7.  Read the ranking/window function differences and practise the
    `RANK()` query.
8.  Review constraints, SQL functions, exceptions and common mistakes.

**Final exam strategy:** Read the question carefully, identify the
required tables and relationships, write the smallest correct
query/program, and check semicolons, aliases, matching datatypes, and
the required `END; /` structure.
