---
name: gentic-data
description: Guides the data management workflow using Gentic MCP tools. Activates when users want to create tables, import CSVs, query data, sync records, or manage their database.
license: MIT
metadata:
  author: gentic
  version: "1.0.0"
---

# Gentic Data

Workflow guide for managing a cloud database using Gentic's MCP tools. Users can create tables from CSVs, query data with SQL, insert records, and sync tables — all through natural language.

## When to apply

- User wants to create a table or import data from a CSV
- User wants to query or analyze their data
- User wants to see what tables they have
- User wants to preview a table's structure or sample data
- User wants to update, sync, or append data from a CSV
- User wants to insert one or more records into a table
- User asks about their "database", "tables", or "data"

## Tools reference

| Tool | Purpose | Cost |
|------|---------|------|
| `list_database_tables` | List all tables with row counts | Free |
| `create_table_from_csv` | Create a new table from any CSV URL | Free |
| `sample_table` | Preview sample records and column info | Free |
| `get_table_schema` | Get detailed column schema (names, types, nullable) | Free |
| `query_data` | Execute SQL SELECT queries for analysis | Free |
| `update_table_from_csv` | Update a table from CSV (replace, append, or upsert) | Free |
| `sync_table_from_csv` | Sync table: update existing + insert new records | Free |
| `batch_update_table_from_csv` | Batch update only existing records from CSV | Free |
| `insert_record` | Insert a single record with duplicate prevention | Free |
| `batch_insert_records` | Insert multiple records in one batch (max 1000) | Free |

## Workflow

### 1. Check existing tables

Use `list_database_tables` to see what the user already has. This is the starting point for most interactions.

### 2. Create tables from data

Use `create_table_from_csv` to import data. Supports multiple URL formats:
- **HTTPS**: `https://example.com/data.csv`
- **S3**: `https://bucket.s3.amazonaws.com/file.csv`
- **Google Sheets**: `https://docs.google.com/spreadsheets/d/{ID}/edit` (auto-converted to CSV)
- **Google Drive**: `https://drive.google.com/file/d/{ID}/view` (auto-converted to download link)

The file must be publicly accessible ("Anyone with the link"). Table names can only contain letters, numbers, and underscores.

### 3. Explore table structure

Before querying, use `sample_table` or `get_table_schema` to understand the data:
- `sample_table` — returns sample records + column names/types. Best for getting a quick feel for the data.
- `get_table_schema` — returns detailed column info (names, types, nullable). Best when you need to know exact schema before inserting or querying.

### 4. Query data

Use `query_data` to run SQL analysis. Key rules:
- **Only SELECT and WITH (CTEs) are allowed** — no INSERT, UPDATE, DELETE, DROP
- **File-reading functions are blocked** — no `read_csv_auto()`, `read_parquet()`, `glob()`, etc.
- Users query their own tables only

Example queries:
```sql
SELECT * FROM sales WHERE date > '2024-01-01' ORDER BY amount DESC LIMIT 10
SELECT category, SUM(revenue) as total FROM sales GROUP BY category
WITH monthly AS (SELECT DATE_TRUNC('month', date) as month, SUM(amount) as total FROM orders GROUP BY 1) SELECT * FROM monthly ORDER BY month
```

Always use `sample_table` first to see the available columns before writing queries.

### 5. Update existing tables

Three tools for different update patterns:

**`update_table_from_csv`** — general-purpose update with three modes:
- `replace` (default): Drop and recreate the table with new data
- `append`: Add all CSV rows to the table (may create duplicates)
- `upsert`: Add only rows where `unique_column` doesn't already exist

**`sync_table_from_csv`** — the best tool when users say "sync", "update", or "refresh":
- Updates existing records that match on `unique_column`
- Inserts new records that don't exist yet
- Single operation, no duplicates

**`batch_update_table_from_csv`** — updates only existing records, ignores new ones:
- Use when the CSV contains corrections to existing data
- Matches by `unique_column`, updates all other columns

### 6. Insert records

**`insert_record`** — insert a single record:
- Requires `unique_column` for duplicate prevention
- Rejects the insert if the unique value already exists
- Always call `sample_table` first to see required columns

**`batch_insert_records`** — insert multiple records in one batch:
- All records must have the same columns
- Maximum 1000 records per call
- Duplicates (matching `unique_column`) are silently skipped

## Typical workflows

### "Import this spreadsheet into my database"
1. `create_table_from_csv` with the Google Sheets or CSV URL
2. `sample_table` to verify the import looks correct

### "What data do I have?"
1. `list_database_tables` to show all tables
2. `sample_table` on tables the user is interested in

### "Analyze my sales data"
1. `sample_table` on the relevant table to see columns
2. `query_data` with the appropriate SQL query
3. Present results in a clear format (tables, summaries, insights)

### "Update my table with new data from this sheet"
1. `sample_table` to see current columns and identify the unique key
2. Ask the user which column to match records on
3. `sync_table_from_csv` to update existing + add new records

### "Add these records to my table"
1. `sample_table` to see required columns
2. `insert_record` or `batch_insert_records` with the data
3. Confirm the insert and show updated row count

### "Replace all data in my table"
1. Confirm with the user that they want to overwrite all existing data
2. `update_table_from_csv` with `mode: "replace"`

## Important notes

- All tools are org-scoped. Users only see their own database and tables.
- All tools are free — no credits charged.
- Table names and column names can only contain letters, numbers, and underscores.
- `query_data` is read-only — it cannot modify data. Use the insert/update tools for writes.
- CSV files are capped at 100 MB per import.
- When a user asks to "add data" or "update", always ask which column contains unique identifiers to prevent duplicates — don't default to append mode.
- If a user asks to "create a database" or "start fresh", they don't need to — their database is created automatically on first use.
