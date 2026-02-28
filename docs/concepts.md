# Core Concepts

TableKit is built on three core objects. Everything else builds on top of them.

---

## ColumnSchema

The smallest unit — a single column with all its metadata.

```mermaid
graph TD
    CS[ColumnSchema]
    CS --> N[name]
    CS --> T[type\nStringType · IntegerType · etc]
    CS --> NL[nullable]
    CS --> C[comment\nfor data catalog]
    CS --> PK[is_primary_key]
    CS --> NK[is_natural_key\nbusiness key for merges]
    CS --> CL[is_clustered_by\nDelta optimisation]
    CS --> ID[always_as_identity\nauto-increment]
```

Metadata is not stored in a separate YAML or wiki. It lives with the column definition, version-controlled alongside your code.

**Why it matters:** When schema metadata is scattered across documentation and comments, it goes stale. When it is part of the schema definition itself, it stays accurate automatically.

---

## TableDefinition

A validated, composable collection of `ColumnSchema` objects.

The key capability is **composition**. Definitions combine with `+`:

```mermaid
graph LR
    A[Business Columns\ncustomer_id · email · name] -->|+| C[Full Definition]
    B[AUDIT_COLUMNS\ncreated_at · updated_at] -->|+| C
    C --> D[TableModel]
```

Define audit columns once and reuse them everywhere. When they change, every table picks up the change.

A `TableDefinition` also knows which columns are primary keys, natural keys, and cluster columns — used automatically by the ETL layer.

**Why it matters:** Copy-paste is the most common source of schema inconsistency. Composition eliminates it.

---

## TableModel

Wraps a `TableDefinition` with the full identity of a table: name, database, catalog, and environment.

```mermaid
graph TD
    TM[TableModel] --> TN[table_name]
    TM --> DB[database]
    TM --> CT[catalog]
    TM --> ENV[environment\ndev · staging · prod]
    TM --> DEF[definition]
    TM --> SRC[source\noptional DataSource]
    TM --> MAP[column_mapping\noptional]
    ENV --> FN[full_name\nmain.dev_sales.dim_customers]
```

The environment is set once. Every table name derives from it automatically.

**Why it matters:** Environment-related bugs are among the most common in data engineering. Baking environment awareness into the table definition removes an entire category of errors.
