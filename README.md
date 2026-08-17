# 📘 SQL Theory & Quick Revision
---

# PART 1 — QUICK SQL REVISION

## 1. SELECT & Column Aliases

**What it is:** Retrieves columns/expressions from a table.
**Syntax:**
```sql
SELECT column1, column2 AS alias_name
FROM table_name;
```
**Key points:**
- `AS` is optional (`salary AS sal` == `salary sal`) but always use it — clearer in interviews.
- Aliases with spaces need double quotes in Oracle: `salary AS "Monthly Salary"`.
- Column aliases **cannot** be used in `WHERE` (alias doesn't exist yet when WHERE runs).
- Column aliases **can** be used in `ORDER BY` (runs after SELECT).
- `SELECT *` is fine for exploration, never use in production/joins (ambiguous columns, performance).

**When to use:** Every query — it's the entry point.
**Example:**
```sql
SELECT employee_id, salary AS monthly_salary FROM employees;
```
**Common mistake:** Using the alias inside `WHERE` in the same query — throws "invalid identifier" in Oracle.
**Interview tip:** Be ready to explain *why* an alias fails in WHERE — it's a favorite trick question (logical execution order).

---

## 2. WHERE & Comparison/Logical Operators

**What it is:** Filters **rows** before grouping.
**Syntax:**
```sql
SELECT * FROM employees
WHERE salary > 50000 AND department_id = 10;
```
**Key points:**
- Operators: `=, !=` (or `<>`), `<, >, <=, >=`.
- `AND` — all conditions true; `OR` — any condition true; `NOT` — negates.
- `AND` has higher precedence than `OR` — always parenthesize mixed conditions.
- `WHERE` cannot contain aggregate functions (`WHERE COUNT(*) > 5` is illegal).
- `WHERE` executes **before** `GROUP BY`, so it filters raw rows, not groups.

**When to use:** Any time you need to filter individual rows.
**Example:**
```sql
SELECT * FROM employees WHERE salary > 40000 AND (department_id = 10 OR department_id = 20);
```
**Common mistake:** Forgetting parentheses with mixed `AND`/`OR`, silently changing logic.
**Interview tip:** Explain `!=` vs `<>` — both work in Oracle, `<>` is ANSI standard.

---

## 3. IN / NOT IN

**What it is:** Shorthand for multiple `OR` equality checks.
**Syntax:**
```sql
SELECT * FROM employees WHERE department_id IN (10, 20, 30);
```
**Key points:**
- `IN` = `= ANY`. `NOT IN` = `<> ALL`.
- `NOT IN` with a list/subquery containing **NULL** returns **zero rows** — classic trap.
- Works with subqueries: `WHERE dept_id IN (SELECT department_id FROM departments WHERE location='Delhi')`.
- More readable than chained `OR`.
- Prefer `NOT EXISTS` over `NOT IN` when the subquery can contain NULLs.

**When to use:** Matching against a known/derived list of values.
**Example:**
```sql
SELECT * FROM products WHERE category NOT IN ('Electronics','Furniture');
```
**Common mistake:** Using `NOT IN` with a subquery that can return NULL — silently returns nothing.
**Interview tip:** This NULL trap is one of the most commonly asked SQL "gotcha" questions.

---

## 4. BETWEEN

**What it is:** Inclusive range filter.
**Syntax:**
```sql
SELECT * FROM employees WHERE salary BETWEEN 30000 AND 60000;
```
**Key points:**
- Inclusive on both ends (`>= low AND <= high`).
- Works on numbers, dates, and strings (alphabetical range).
- Always list the **lower** bound first — `BETWEEN 60000 AND 30000` returns nothing.
- Can be combined with `NOT BETWEEN`.
- For dates, be careful with time components (`BETWEEN '2024-01-01' AND '2024-01-31'` may exclude timestamps late on the 31st).

**When to use:** Range checks (salary bands, date ranges).
**Example:**
```sql
SELECT * FROM orders WHERE order_date BETWEEN DATE '2024-01-01' AND DATE '2024-03-31';
```
**Common mistake:** Reversing bounds, or forgetting BETWEEN is inclusive (off-by-one logic errors).
**Interview tip:** Know that `BETWEEN` is equivalent to two conditions — interviewers ask you to rewrite it that way.

---

## 5. LIKE (Pattern Matching)

**What it is:** Wildcard-based string matching.
**Syntax:**
```sql
SELECT * FROM employees WHERE first_name LIKE 'A%';
```
**Key points:**
- `%` = zero or more characters; `_` = exactly one character.
- Case-sensitive in Oracle by default.
- `NOT LIKE` negates the pattern.
- Use `ESCAPE` to search for literal `%` or `_`.
- Leading-wildcard patterns (`'%son'`) prevent index usage — performance concern.

**When to use:** Partial text search (names, codes, emails).
**Example:**
```sql
SELECT * FROM customers WHERE email LIKE '%@gmail.com';
```
**Common mistake:** Forgetting LIKE is case-sensitive and missing matches due to casing.
**Interview tip:** Be ready to write a pattern for "second letter is 'a'" → `'_a%'`.

---

## 6. IS NULL / IS NOT NULL

**What it is:** The only correct way to test for NULL.
**Syntax:**
```sql
SELECT * FROM employees WHERE manager_id IS NULL;
```
**Key points:**
- `NULL` means "unknown" — `= NULL` and `!= NULL` both evaluate to UNKNOWN, never TRUE.
- NULL is excluded from most aggregate calculations (except `COUNT(*)`).
- NULL in arithmetic makes the whole expression NULL (`salary + NULL = NULL`).
- Use `NVL(column, default)` in Oracle to substitute a value for NULL.
- `IN`/`NOT IN` lists behave unexpectedly when NULL is present (see topic 3).

**When to use:** Finding missing data — employees with no manager, no department, etc.
**Example:**
```sql
SELECT * FROM employees WHERE department_id IS NULL;
```
**Common mistake:** Writing `WHERE column = NULL` — always returns no rows, no error thrown.
**Interview tip:** Explain three-valued logic (TRUE/FALSE/UNKNOWN) — a common conceptual question.

---

## 7. Table Aliases

**What it is:** Short names for tables, essential in joins/subqueries.
**Syntax:**
```sql
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```
**Key points:**
- Mandatory once you self-join (need two names for the same table).
- Improves readability and reduces typing in multi-table queries.
- Once aliased, you **must** use the alias everywhere in that query (can't mix `employees.col` and `e.col`).
- Doesn't require `AS` in Oracle (`employees e` is valid).
- Scoped only to the query it's defined in.

**When to use:** Any join, especially self-joins and correlated subqueries.
**Example:**
```sql
SELECT a.employee_id, b.employee_id
FROM employees a, employees b
WHERE a.manager_id = b.employee_id;
```
**Common mistake:** Referring to the original table name after aliasing it — causes an "invalid identifier" error.
**Interview tip:** In self-joins, always explain in words what each alias represents (e.g., "e = employee, m = manager").

---

## 8. DISTINCT

**What it is:** Removes duplicate rows from the result set.
**Syntax:**
```sql
SELECT DISTINCT department_id FROM employees;
```
**Key points:**
- Applies to the **entire selected row combination**, not a single column, when multiple columns are listed.
- `COUNT(DISTINCT column)` counts unique non-null values.
- DISTINCT runs late in logical order — after SELECT, before ORDER BY.
- Can be expensive on large datasets (requires sorting/hashing internally).
- Often a query "needs DISTINCT" because of a join producing duplicate rows — fixing the join is usually better than papering over it with DISTINCT.

**When to use:** Removing duplicate values or duplicate rows caused by joins.
**Example:**
```sql
SELECT DISTINCT category FROM products;
```
**Common mistake:** Using DISTINCT to "hide" a join bug that's genuinely duplicating rows, instead of fixing the join logic.
**Interview tip:** Be ready to say when DISTINCT is a band-aid vs. a genuine requirement.

---

## 9. Aggregate Functions — COUNT, SUM, AVG, MIN, MAX

**What it is:** Functions that compute a single value from multiple rows.
**Syntax:**
```sql
SELECT COUNT(*), SUM(salary), AVG(salary), MIN(salary), MAX(salary) FROM employees;
```
**Key points:**
- `COUNT(*)` counts all rows including NULLs; `COUNT(column)` counts only non-NULL values.
- `SUM`/`AVG` ignore NULLs automatically (don't treat them as 0).
- Aggregates cannot be nested directly (`SUM(AVG(x))` is illegal without a subquery).
- Without `GROUP BY`, an aggregate collapses the whole table into one row.
- Mixing aggregate and non-aggregate columns in `SELECT` without `GROUP BY` is illegal (except constants).

**When to use:** Summarizing data — totals, counts, averages per group or overall.
**Example:**
```sql
SELECT department_id, AVG(salary) AS avg_sal FROM employees GROUP BY department_id;
```
**Common mistake:** Assuming `AVG` divides by total row count instead of non-NULL count.
**Interview tip:** Know the exact difference between `COUNT(*)`, `COUNT(1)`, and `COUNT(column)` cold.

---

## 10. GROUP BY

**What it is:** Groups rows sharing the same value(s) so aggregates can be computed per group.
**Syntax:**
```sql
SELECT department_id, COUNT(*) FROM employees GROUP BY department_id;
```
**Key points:**
- Every non-aggregated column in `SELECT` must appear in `GROUP BY`.
- Can group by multiple columns (composite groups).
- Runs **after** `WHERE`, **before** `HAVING`.
- Can group by an expression (`GROUP BY TRUNC(order_date, 'MM')`).
- `GROUP BY` without aggregate functions behaves like `DISTINCT` on those columns.

**When to use:** Per-category summaries (per department, per customer, per product).
**Example:**
```sql
SELECT department_id, MAX(salary) FROM employees GROUP BY department_id;
```
**Common mistake:** Adding a column to SELECT and forgetting to add it to GROUP BY → Oracle throws "not a GROUP BY expression".
**Interview tip:** Yes — `GROUP BY` can be used with zero aggregate functions, purely to deduplicate combinations.

---

## 11. HAVING

**What it is:** Filters **groups** after aggregation.
**Syntax:**
```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;
```
**Key points:**
- Can reference aggregate functions directly (unlike WHERE).
- Runs **after** GROUP BY, so it sees already-aggregated results.
- Can also filter on non-aggregated grouped columns (`HAVING department_id > 10`), though that's usually better in WHERE for efficiency.
- Doesn't require the aggregate to appear in SELECT.
- Cannot use column aliases defined in SELECT in Oracle's HAVING (dialect-dependent — safer to repeat the expression).

**When to use:** Any condition that depends on an aggregated value.
**Example:**
```sql
SELECT department_id, COUNT(*) FROM employees GROUP BY department_id HAVING COUNT(*) > 3;
```
**Common mistake:** Putting an aggregate condition in WHERE instead of HAVING — causes an error, not a wrong answer.
**Interview tip:** Always be ready to explain the difference from topic 12 in one sentence each.

---

## 12. WHERE vs HAVING (Explicit Comparison)

| Aspect | WHERE | HAVING |
|---|---|---|
| Filters | Individual **rows** | **Groups** (post-aggregation) |
| Executes | Before `GROUP BY` | After `GROUP BY` |
| Aggregate functions | ❌ Not allowed | ✅ Allowed |
| Can use column aliases | ❌ No | ⚠️ Dialect-dependent, avoid relying on it |
| Typical use | Restrict rows entering the group | Restrict which groups appear in output |
| Can exist without the other | ✅ Yes | ✅ Yes (works without GROUP BY too — treats whole table as one group) |

**Why this matters:** `WHERE` reduces the dataset *before* Oracle bothers grouping it — cheaper, and it decides which rows are even eligible to be aggregated. `HAVING` looks at the *result* of aggregation and decides which groups survive. Using them together lets you filter twice: once at the row level, once at the group level — e.g., "consider only employees hired after 2020 (`WHERE`), then show departments with more than 5 such employees (`HAVING`)."
**Interview tip:** A great one-liner to memorize: *"WHERE filters before grouping; HAVING filters after grouping."*

---

## 13. ORDER BY

**What it is:** Sorts the final result set.
**Syntax:**
```sql
SELECT employee_id, salary FROM employees ORDER BY salary DESC, employee_id ASC;
```
**Key points:**
- Executes **last** in logical order — after SELECT, DISTINCT.
- Default direction is `ASC`.
- Can sort by column name, alias, or column position number (`ORDER BY 2`).
- NULLs sort last by default in Oracle ascending order (`NULLS LAST` is default for ASC; use `NULLS FIRST`/`NULLS LAST` to override).
- Multiple sort keys are applied left to right.

**When to use:** Whenever result order matters (top-N queries, reports).
**Example:**
```sql
SELECT * FROM employees ORDER BY department_id, salary DESC;
```
**Common mistake:** Assuming query results are ordered by default without `ORDER BY` — Oracle gives no ordering guarantee.
**Interview tip:** Know how to combine `ORDER BY` with `FETCH FIRST n ROWS ONLY` (Oracle 12c+) for top-N / Nth-highest queries.

---

## 14. JOINs (All Types)

**What it is:** Combines rows from two or more tables based on a related column.

### INNER JOIN
```sql
SELECT e.first_name, d.department_name
FROM employees e INNER JOIN departments d ON e.department_id = d.department_id;
```
Returns only rows with a match in **both** tables.

### LEFT (OUTER) JOIN
```sql
SELECT e.first_name, d.department_name
FROM employees e LEFT JOIN departments d ON e.department_id = d.department_id;
```
Returns **all** left-table rows; unmatched right-side columns become NULL.

### RIGHT (OUTER) JOIN
```sql
SELECT e.first_name, d.department_name
FROM employees e RIGHT JOIN departments d ON e.department_id = d.department_id;
```
Returns **all** right-table rows; unmatched left-side columns become NULL.

### FULL OUTER JOIN
```sql
SELECT e.first_name, d.department_name
FROM employees e FULL OUTER JOIN departments d ON e.department_id = d.department_id;
```
Returns all rows from both sides; unmatched columns from either side become NULL. (All four join types are natively supported in Oracle.)

### CROSS JOIN
```sql
SELECT p.product_name, c.category_name FROM products p CROSS JOIN categories c;
```
Cartesian product — every row of A with every row of B, no condition needed.

### SELF JOIN
```sql
SELECT e.first_name AS employee, m.first_name AS manager
FROM employees e LEFT JOIN employees m ON e.manager_id = m.employee_id;
```
A table joined to itself — needs two aliases. Used for hierarchies (employee-manager).

**Key points (general):**
- Putting a right-table filter in `WHERE` instead of the `ON` clause silently converts a LEFT JOIN into an INNER JOIN.
- JOINs can multiply row counts if the join key isn't unique on one side (duplicate rows).
- Oracle also supports old-style `(+)` outer join syntax — avoid it in modern code, but recognize it.
- `NATURAL JOIN` and `USING` exist but hide join columns — avoid in interviews, be explicit with `ON`.
- Order of tables in a LEFT/RIGHT JOIN matters; INNER/CROSS/FULL are order-independent in result (not in duplicate ordering).

**Common mistake:** `LEFT JOIN ... WHERE d.location = 'Delhi'` — this silently drops the NULL rows a LEFT JOIN was meant to preserve, because `d.location = 'Delhi'` is UNKNOWN for NULL rows. Fix: move the condition into the `ON` clause.
**Interview tip:** This LEFT JOIN → INNER JOIN trap is one of the top 3 most-asked SQL debugging questions.

---

## 15. Subqueries (All Types)

**What it is:** A query nested inside another query.

| Type | Description | Example clause |
|---|---|---|
| Single-row | Returns exactly one row/value | `WHERE salary > (SELECT AVG(salary) FROM employees)` |
| Multi-row | Returns multiple rows, used with `IN`/`ANY`/`ALL` | `WHERE department_id IN (SELECT department_id FROM departments WHERE location='Delhi')` |
| Correlated | References the outer query, re-evaluated per outer row | `WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.department_id = e1.department_id)` |
| `IN` subquery | Match against a list from a subquery | see above |
| `EXISTS` subquery | Tests row existence, stops at first match | `WHERE EXISTS (SELECT 1 FROM employee_project ep WHERE ep.employee_id = e.employee_id)` |
| `ANY` | TRUE if condition holds for **at least one** returned value | `WHERE salary > ANY (SELECT salary FROM employees WHERE department_id = 20)` |
| `ALL` | TRUE if condition holds for **every** returned value | `WHERE salary > ALL (SELECT salary FROM employees WHERE department_id = 20)` |
| In `WHERE` | Filters rows using a derived value/list | most examples above |
| In `FROM` | Acts as a temporary derived table (inline view) | `FROM (SELECT department_id, AVG(salary) avg_sal FROM employees GROUP BY department_id) dept_avg` |
| In `SELECT` | Returns a scalar value per outer row | `SELECT name, (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.customer_id) AS order_count FROM customers c` |

**Key points:**
- A single-row subquery used with `=`, `>`, `<` etc. **must** return exactly one row, or Oracle throws "single-row subquery returns more than one row".
- `EXISTS` only cares about row existence, not values — often faster than `IN` on large/correlated data and NULL-safe.
- `NOT EXISTS` is the NULL-safe alternative to `NOT IN`.
- Subqueries in `FROM` (inline views) must have an alias in Oracle.
- Correlated subqueries run once **per outer row** — can be slow on large tables; often rewritable as a JOIN.

**Common mistake:** Using `NOT IN` on a subquery that can return NULL (returns zero rows silently) instead of `NOT EXISTS`.
**Interview tip:** Be ready to convert between a correlated subquery and an equivalent JOIN — interviewers love asking "rewrite this without a subquery."

---

## 16. Logical Order of SQL Query Execution

```text
1. FROM        -- identify source tables
2. JOIN        -- combine tables, apply ON conditions
3. WHERE       -- filter individual rows
4. GROUP BY    -- form groups
5. HAVING      -- filter groups
6. SELECT      -- compute output columns/aliases
7. DISTINCT    -- remove duplicate rows
8. ORDER BY    -- sort final result
9. FETCH/LIMIT -- restrict row count (Oracle: FETCH FIRST n ROWS ONLY)
```

**Why it matters:**
- Explains why `WHERE` can't use `SELECT` aliases (SELECT hasn't run yet) but `ORDER BY` can (it runs last).
- Explains why `WHERE` can't use aggregates (aggregation hasn't happened yet) but `HAVING` can.
- Explains why filtering in `WHERE` is generally cheaper than filtering in `HAVING` — fewer rows reach the expensive GROUP BY step.
- Explains the LEFT JOIN → INNER JOIN trap: a `WHERE` condition on the right table runs *after* the join has already produced NULL rows, and then eliminates them.
- This mental model is the #1 tool for solving "genuinely difficult" combined queries — always ask "what does the row set look like at each stage?"


---
