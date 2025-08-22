# Conventions
- Keywords in **UPPERCASE**; replace placeholders like `<table>` with your names.
- String literals use single quotes `'text'`. Oracle identifiers can be double-quoted `"CaseSensitive"` (not recommended).
- Oracle sample table for constants: `DUAL`.

---

# Chapter 2 — Data Types (Oracle)
**CHAR**
```sql
CHAR(size)              -- fixed-length, 1–2000 bytes
```
**VARCHAR / VARCHAR2**
```sql
VARCHAR2(size)          -- variable-length, 1–4000 bytes (Oracle preferred)
VARCHAR(size)           -- standard alias in some DBs
```
**NUMBER**
```sql
NUMBER                  -- arbitrary precision
NUMBER(p)               -- precision p
NUMBER(p, s)            -- precision p, scale s
```
**DATE & TIMESTAMP**
```sql
DATE                    -- date + time to seconds
TIMESTAMP[(fraction)]
TIMESTAMP[(fraction)] WITH TIME ZONE
TIMESTAMP[(fraction)] WITH LOCAL TIME ZONE
```
**LOBs**
```sql
BLOB                    -- binary data
CLOB                    -- character data
NCLOB                   -- national character data
```
> Other common types (optional): `NCHAR(n)`, `NVARCHAR2(n)`, `RAW(n)`, `FLOAT[(p)]`.

---

# Chapter 3 — Constraints (Oracle)
**Column‑level (inline)**
```sql
CREATE TABLE emp (
  emp_id     NUMBER       PRIMARY KEY,
  email      VARCHAR2(50) UNIQUE,
  name       VARCHAR2(40) NOT NULL,
  age        NUMBER        CHECK (age >= 18)
);
```
**Table‑level (out of line)**
```sql
CREATE TABLE emp (
  emp_id   NUMBER,
  dept_id  NUMBER,
  email    VARCHAR2(50),
  CONSTRAINT pk_emp PRIMARY KEY (emp_id),
  CONSTRAINT uq_emp_email UNIQUE (email),
  CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id) REFERENCES dept(dept_id),
  CONSTRAINT ck_emp_age CHECK (age >= 18)
);
```
**Foreign key options**
```sql
FOREIGN KEY (dept_id) REFERENCES dept(dept_id) ON DELETE CASCADE
FOREIGN KEY (dept_id) REFERENCES dept(dept_id) ON DELETE SET NULL
```
**Add/Drop constraints**
```sql
ALTER TABLE emp ADD CONSTRAINT ck_age CHECK (age BETWEEN 18 AND 60);
ALTER TABLE emp DROP CONSTRAINT ck_age;
```

---

# Chapter 4 — SQL Statement Families
- **DQL**: `SELECT ...`
- **DDL**: `CREATE | ALTER | RENAME | TRUNCATE | DROP`
- **DML**: `INSERT | UPDATE | DELETE`
- **TCL**: `COMMIT | ROLLBACK | SAVEPOINT`
- **DCL**: `GRANT | REVOKE`

---

# Chapter 6 — DQL (SELECT) Core Forms
```sql
-- Basic
SELECT col_list FROM <table>;
SELECT * FROM <table>;

-- Constant / expression
SELECT 'Mohanraj' AS name FROM DUAL;
SELECT 1 + 2 AS sum FROM DUAL;

-- Full template
SELECT [DISTINCT] col_expr[, ...]
FROM   table_ref [alias][, ...]
[WHERE condition]
[GROUP BY group_expr[, ...]]
[HAVING group_condition]
[ORDER BY sort_expr [ASC|DESC] [NULLS FIRST|LAST][, ...]];
```

---

# Chapter 7 — Projection (columns)
**Syntax**
```sql
SELECT column_name FROM table_name;
SELECT table_alias.column_name FROM table_name table_alias;
SELECT column1, column2 FROM table_name;
SELECT * FROM table_name;  -- all columns
```
**DISTINCT**
```sql
SELECT DISTINCT column_name FROM table_name;
SELECT DISTINCT col1, col2 FROM table_name;
```
**Expressions & Aliases**
```sql
SELECT salary * 12 AS annual_salary FROM emp;
SELECT salary * 12 annual_salary FROM emp;     -- AS optional
SELECT first_name || ' ' || last_name AS full_name FROM emp;  -- Oracle concat
SELECT CONCAT(first_name, last_name) AS full_name FROM emp;   -- portable
```

---

# Chapter 8 — Selection (rows)
**WHERE clause**
```sql
SELECT * FROM table_name WHERE condition;

-- Common predicates
col = value
col <> value      -- or !=
col > value | col >= value | col < value | col <= value
col BETWEEN low AND high
col IN (v1, v2, ...)
col LIKE pattern [ESCAPE '\\']
col IS NULL | col IS NOT NULL
```

---

# Chapter 9 — Operators
**Concatenation**
```sql
expr1 || expr2            -- Oracle
CONCAT(expr1, expr2)      -- ANSI/others
```
**Logical**
```sql
AND    OR    NOT
```
**Special**
```sql
col IN (..)
col NOT IN (..)
col BETWEEN low AND high
col NOT BETWEEN low AND high
col IS NULL | IS NOT NULL
col LIKE 'A%'
col NOT LIKE 'A%'
```

---

# Chapter 10 — Functions
**Single‑row (return one value per row)**
```sql
function_name(expr[, ...])
```
**Multi‑row / Aggregate**
```sql
COUNT(*)
COUNT(expr)
SUM(expr)
AVG(expr)
MAX(expr)
MIN(expr)
```
**Aggregate with GROUP BY**
```sql
SELECT dept_id, COUNT(*)
FROM   emp
GROUP BY dept_id;
```

---

# Chapter 11 — GROUP BY
```sql
SELECT group_expr, aggregate_func(expr)
FROM   table
[WHERE condition]
GROUP BY group_expr[, ...];
```
Rules: every selected non‑aggregated column must appear in `GROUP BY`.

---

# Chapter 12 — HAVING
```sql
SELECT dept_id, COUNT(*) AS cnt
FROM   emp
GROUP BY dept_id
HAVING COUNT(*) > 5;
```
**Difference**: `WHERE` filters rows **before** grouping; `HAVING` filters groups **after** grouping.

---

# Chapter 13 — ORDER BY
```sql
SELECT * FROM emp ORDER BY salary DESC;
SELECT * FROM emp ORDER BY dept_id ASC, salary DESC;
SELECT * FROM emp ORDER BY 1, 2;             -- by select-list position
SELECT * FROM emp ORDER BY salary DESC NULLS LAST;  -- Oracle
```

---

# Chapter 14 — Subqueries
**Basic subquery**
```sql
SELECT *
FROM   emp
WHERE  dept_id = (SELECT dept_id FROM dept WHERE name = 'IT');   -- single‑row
```
**Multi‑row operators**
```sql
WHERE col IN  (SELECT ...)
WHERE col NOT IN (SELECT ...)
WHERE col = ANY (SELECT ...)
WHERE col > ALL (SELECT ...)
WHERE EXISTS     (SELECT 1 FROM ... WHERE ...)
WHERE NOT EXISTS (SELECT 1 FROM ... WHERE ...)
```
**Scalar subquery**
```sql
SELECT emp_id, (SELECT MAX(salary) FROM emp) AS max_sal FROM emp;
```
**Nested subquery**
```sql
SELECT *
FROM   emp
WHERE  dept_id IN (
  SELECT dept_id
  FROM   dept
  WHERE  location IN (
    SELECT location FROM office WHERE region = 'SOUTH'
  )
);
```
**MAX & MIN via subquery**
```sql
SELECT * FROM emp WHERE salary = (SELECT MAX(salary) FROM emp);
SELECT * FROM emp WHERE salary = (SELECT MIN(salary) FROM emp);
```
**EMP–MGR relationship (self reference)**
```sql
-- employees with their manager names
SELECT e.emp_id, e.name AS emp_name, m.name AS mgr_name
FROM   emp e
LEFT JOIN emp m ON e.mgr_id = m.emp_id;   -- mgr_id NULL means top manager
```

---

# Chapter 15 — Joins
**CROSS (Cartesian)**
```sql
SELECT * FROM emp CROSS JOIN dept;
-- Old style (avoid): SELECT * FROM emp, dept;
```
**INNER / EQUI JOIN**
```sql
SELECT e.*, d.*
FROM   emp e
JOIN   dept d ON e.dept_id = d.dept_id;      -- ANSI
```
**NATURAL JOIN** (joins by columns with the same name; use carefully)
```sql
SELECT * FROM emp NATURAL JOIN dept;
```
**OUTER JOINs**
```sql
SELECT e.name, d.name
FROM   emp e
LEFT  JOIN dept d ON e.dept_id = d.dept_id;   -- unmatched e kept

SELECT e.name, d.name
FROM   emp e
RIGHT JOIN dept d ON e.dept_id = d.dept_id;   -- unmatched d kept

SELECT e.name, d.name
FROM   emp e
FULL  JOIN dept d ON e.dept_id = d.dept_id;   -- keep all
```
**SELF JOIN**
```sql
SELECT e.name AS emp_name, m.name AS mgr_name
FROM   emp e
JOIN   emp m ON e.mgr_id = m.emp_id;
```
**Multiple conditions on JOIN**
```sql
SELECT *
FROM   sales s
JOIN   product p
  ON s.prod_id = p.prod_id
 AND s.txn_date BETWEEN p.start_date AND p.end_date;
```

---

# Chapter 16 — Correlated Subquery
**Definition**: inner query references outer row; runs per row.
```sql
SELECT e.*
FROM   emp e
WHERE  salary > (
  SELECT AVG(salary)
  FROM   emp
  WHERE  dept_id = e.dept_id  -- correlation
);
```
**EXISTS / NOT EXISTS**
```sql
SELECT d.*
FROM   dept d
WHERE  EXISTS (SELECT 1 FROM emp e WHERE e.dept_id = d.dept_id);

SELECT d.*
FROM   dept d
WHERE  NOT EXISTS (SELECT 1 FROM emp e WHERE e.dept_id = d.dept_id);
```
**Nth MAX / MIN (two standard patterns)**
```sql
-- (A) Analytic function (Oracle)
SELECT * FROM (
  SELECT e.*, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM   emp e
) WHERE rnk = :N;   -- Nth highest; use ASC for Nth lowest

-- (B) Correlated count (portable)
SELECT e.*
FROM   emp e
WHERE  :N - 1 = (
  SELECT COUNT(DISTINCT salary)
  FROM   emp
  WHERE  salary > e.salary   -- use < for Nth MIN
);
```

---

# Chapter 17 — DDL
**CREATE TABLE**
```sql
CREATE TABLE table_name (
  column_name data_type [DEFAULT expr] [constraint ...],
  ...,
  [table_constraint ...]
);
```
**RENAME**
```sql
RENAME old_name TO new_name;                  -- object rename
```
**ALTER TABLE**
```sql
ALTER TABLE t ADD    (col_name data_type);
ALTER TABLE t MODIFY (col_name data_type);
ALTER TABLE t DROP   COLUMN col_name;
ALTER TABLE t RENAME COLUMN old_col TO new_col;
ALTER TABLE t ADD CONSTRAINT constraint_name ...;
ALTER TABLE t DROP CONSTRAINT constraint_name;
```
**TRUNCATE**
```sql
TRUNCATE TABLE table_name;
```
**DROP**
```sql
DROP TABLE table_name [PURGE];
```

---

# Chapter 18 — DML
**INSERT**
```sql
INSERT INTO emp (emp_id, name, salary) VALUES (1, 'Alex', 50000);
INSERT INTO emp VALUES (1, 'Alex', 50000, NULL);
INSERT INTO emp (emp_id, name, salary)
SELECT emp_id, name, salary FROM emp_history WHERE year = 2024;
```
**UPDATE**
```sql
UPDATE emp SET salary = 60000 WHERE emp_id = 1;
UPDATE emp e
SET    dept_id = (
  SELECT d.dept_id FROM dept d WHERE d.name = 'IT'
) WHERE e.emp_id = 1;
```
**DELETE**
```sql
DELETE FROM emp WHERE emp_id = 1;
DELETE FROM emp;   -- all rows
```

---

# Chapter 19 — TCL
```sql
COMMIT;                          -- make changes permanent
SAVEPOINT sp1;                   -- mark a point
ROLLBACK TO SAVEPOINT sp1;       -- undo to savepoint
ROLLBACK;                        -- undo entire transaction
```

---

# Chapter 20 — DCL
**Object privileges**
```sql
GRANT SELECT, INSERT ON schema.table TO user;
REVOKE SELECT, INSERT ON schema.table FROM user;
```
**System privileges / roles**
```sql
GRANT CREATE SESSION TO user;
GRANT role_name TO user;
REVOKE role_name FROM user;

-- Option
GRANT SELECT ON schema.table TO user WITH GRANT OPTION;
```

---

# Chapter 21 — Single‑Row Functions (common)
**Character**
```sql
LENGTH(expr)                   -- Oracle: characters; LENGTHB for bytes
UPPER(expr)
LOWER(expr)
INITCAP(expr)                  -- Oracle title case
SUBSTR(expr, start [, len])    -- Oracle; SUBSTRING in other DBs
INSTR(expr, sub [, start [, occurrence]])
REPLACE(expr, search [, repl])
CONCAT(expr1, expr2)           -- or expr1 || expr2 (Oracle)
-- REVERSE(expr)               -- MySQL/SQL Server; Oracle has no built‑in REVERSE
```
**Numeric**
```sql
MOD(n, m)
ROUND(n [, decimals])
TRUNC(n [, decimals])
```
**Date/Time (Oracle)**
```sql
MONTHS_BETWEEN(date1, date2)
LAST_DAY(date)
TRUNC(date [, 'fmt'])          -- e.g., 'MM', 'YYYY'
```
**Conversion**
```sql
TO_CHAR(expr [, 'format' [, 'nls']])   -- number/date to string (Oracle)
TO_NUMBER(char [, 'format'])
TO_DATE(char [, 'format'])
NVL(expr1, expr2)                       -- Oracle null handling
-- COALESCE(expr1, expr2, ...)         -- portable alternative
```

---

# Quick SELECT Possibilities (as requested)
```sql
SELECT col_name FROM emp;
SELECT * FROM emp;
SELECT 'Mohanraj' AS name FROM DUAL;
```

---

# Notes
- Prefer ANSI JOIN syntax over old Oracle `(+)` outer-join operator.
- Use bind variables (`:N`) in tools to parameterize.
- Quoted identifiers are case‑sensitive; avoid unless required.

