# 🕶️ Super Blind

| | |
|---|---|
| **Category** | Web |
| **Points** | 200 |
| **Flag** | `InIt{bl1nd_sql_1nj3ct10n_g0brrrrr!@#}` |

## 📝 Challenge Analysis

The app has a search feature (`/search?q=...`) backed by an internal SQLite database:

```sql
SELECT name, description FROM records WHERE name LIKE '%{q}%'
```

A strict filter blocks `SELECT`, `UNION`, `OR`, `AND`, `CASE`, `WHEN`, `THEN`, `ELSE`, `SLEEP`, comment markers (`--`, `/*`) and semicolons.

## 🛠️ Solution

### 1. Bypassing the filters
SQLite supports string concatenation with `||` and the inline conditional `iif(condition, true_val, false_val)`. Neither is filtered.

### 2. Building a boolean oracle
Inject a payload that concatenates inside the `LIKE` pattern:

```sql
archive' || iif(<condition>, '', 'xyz') || '
```

| Condition | Pattern becomes | Result |
|---|---|---|
| True | `%archive%` | Matches the "archive" record, returns 1 match |
| False | `%archivexyz%` | "No matching records" |

### 3. Leaking the secret
The table has the columns `name`, `description` and `secret`. A **binary search** on `unicode(substr(secret, index, 1))` extracts the 37-character flag one character at a time.

## 🏁 Flag

```text
InIt{bl1nd_sql_1nj3ct10n_g0brrrrr!@#}
```

## 🧰 Tools
Python (requests) · Burp Suite
