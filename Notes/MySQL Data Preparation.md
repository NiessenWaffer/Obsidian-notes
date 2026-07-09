# MySQL Concepts to Understand Before Creating Data

## Data Types

| Type | What it stores | Example |
|------|---------------|---------|
| `INT` / `BIGINT` | Whole numbers | `1`, `100`, `9999` |
| `VARCHAR(n)` | Text with max length n | `VARCHAR(50)` for names |
| `TEXT` | Long text | Paragraphs, descriptions |
| `DATE` | Date only (YYYY-MM-DD) | `2026-07-07` |
| `DATETIME` / `TIMESTAMP` | Date + time | `2026-07-07 14:30:00` |
| `BOOLEAN` / `TINYINT(1)` | True/False | `1` or `0` |
| `ENUM('a','b','c')` | One value from a fixed list | `ENUM('active','inactive')` |
| `DECIMAL(p,d)` | Precise numbers (money) | `DECIMAL(10,2)` |

### VARCHAR
- Use when text length varies but has a reasonable limit
- Always set a max length (e.g., `VARCHAR(100)`)
- Don't use VARCHAR(255) by default — pick based on actual data

## Constraints

| Constraint | What it does |
|------------|-------------|
| `PRIMARY KEY` | Uniquely identifies each row. Only one per table. Auto-increment usually. |
| `UNIQUE` | Ensures all values in a column (or combination) are different |
| `NOT NULL` | Column must have a value — cannot be empty |
| `DEFAULT` | Sets a fallback value if none is provided |
| `FOREIGN KEY` | Links to a column in another table (enforces referential integrity) |
| `INDEX` | Speeds up searches for that column (not a constraint, but related) |

### UNIQUE
- Use when no two rows should have the same value in that column (e.g., employee email)
- Can combine multiple columns: `UNIQUE(employee_id, coverage_date)`
- Differs from PRIMARY KEY: a table can have multiple UNIQUE constraints

### NULL vs NOT NULL
- `NULL` = unknown / missing (not the same as zero or empty string)
- Use `NOT NULL` when the value is always required
- Use `NULL` for optional fields (e.g., cancellation_date)

## AUTO_INCREMENT
- Automatically generates sequential numbers (1, 2, 3...)
- Commonly used for `id` columns
- MySQL handles it — you don't insert a value

## Sample Syntax

```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    status ENUM('active', 'inactive') DEFAULT 'active',
    date_hired DATE,
    salary DECIMAL(10,2)
);
```

## Before Creating Data, Ask Yourself
- [x] What is the primary key?
- [x] Which columns must be unique?
- [x] Which columns can be NULL?
- [x] What is the max length for each text field?
- [x] Does this column reference another table?
- [x] What is the default value (if any)?
- [x] What ENUM values are possible?
