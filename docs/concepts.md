# Core Concepts

TableKit is built on three core objects that work together to define your data infrastructure as code.

---

## ColumnSchema

The smallest unit in TableKit. A column is not just a name and a type — it carries intent.

Every column knows:
- Its data type
- Whether it allows nulls
- Whether it is a primary key or business key
- Whether it should be used for Delta clustering
- Its description for data catalog documentation

This metadata is not stored in a separate YAML or wiki. It lives with the column definition, version-controlled alongside your code.

**Why it matters:** When schema metadata is scattered across documentation and comments, it goes stale. When it is part of the schema definition itself, it stays accurate automatically.

---

## TableDefinition

A `TableDefinition` is a validated, composable collection of columns.

The key capability is **composition**. Definitions can be combined:

```
customer columns  +  audit columns  =  full table definition
```

This means common column sets — audit timestamps, SCD2 keys, soft-delete flags — are defined once and reused across every table that needs them. When the audit columns change, every table picks up the change automatically.

A `TableDefinition` also knows which columns are primary keys, natural keys, and cluster columns. This information is used by the ETL layer without any additional configuration.

**Why it matters:** Copy-paste is the most common source of schema inconsistency in data platforms. Composition eliminates it.

---

## TableModel

A `TableModel` wraps a `TableDefinition` with the full identity of a table: name, database, catalog, and environment.

The environment awareness is the critical feature. The same table definition produces different fully-qualified names depending on the environment:

| Environment | Full Table Name |
|-------------|----------------|
| dev | `main.dev_sales.dim_customers` |
| staging | `main.staging_sales.dim_customers` |
| prod | `main.sales.dim_customers` |

The environment is set once. Every table name derives from it. There is no string manipulation in pipeline code, no environment variables scattered across notebooks.

A `TableModel` can also hold a reference to its data source and column mapping. The table definition becomes the single source of truth for schema, source, and structure.

**Why it matters:** Environment-related bugs are among the most common and most costly in data engineering. Baking environment awareness into the table definition removes an entire category of errors.
