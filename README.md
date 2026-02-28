# TableKit

**Type-safe PySpark table definitions and ETL framework**

TableKit is a framework for defining data schemas and ETL pipelines in Python. Instead of writing raw PySpark schemas by hand, you define tables as structured Python objects — with IDE support, validation, and automatic schema generation.

---

## The Problem

Data engineering teams repeat the same patterns over and over:

- Writing `StructType` schemas by hand with no type checking
- Copy-pasting audit columns across dozens of table definitions
- Hardcoding source paths inside ETL logic
- Building one-off pipelines that can't be tested or reused
- No standard structure across projects — every pipeline looks different

The result is fragile code, inconsistent schemas, and pipelines that break silently.

---

## The Solution

TableKit introduces a layered architecture that separates concerns clearly:

```
┌─────────────────────────────────────────────┐
│                 ETL Layer                    │
│   Orchestrates the flow. Extract, transform, │
│   validate, load. Reusable across tables.    │
├─────────────────────────────────────────────┤
│               Tables Layer                  │
│  Definitions: what the table looks like.    │
│  Transformations: how to convert raw data.  │
├─────────────────────────────────────────────┤
│   Schemas Layer    │    Sources Layer        │
│   Reusable column  │  Declarative data      │
│   sets and types.  │  source definitions.   │
├─────────────────────────────────────────────┤
│               Core Library                  │
│   ColumnSchema, TableDefinition,            │
│   TableModel, utilities.                    │
└─────────────────────────────────────────────┘
```

Each layer has one job. Nothing bleeds into another.

---

## Documentation

| Document | Description |
|----------|-------------|
| [Core Concepts](docs/concepts.md) | ColumnSchema, TableDefinition, TableModel explained |
| [Architecture](docs/architecture.md) | Why this structure works and how the layers connect |
| [Data Sources](docs/sources.md) | Declarative source definitions for CSV, JDBC, Delta, and more |
| [ETL Framework](docs/etl.md) | How the ETL layer uses everything below it |
| [Why TableKit](docs/why.md) | The case for schema-as-code in data engineering |

---

## Get TableKit

| Edition | What's included | Price |
|---------|----------------|-------|
| [**TableKit Core**](https://sagnelli.gumroad.com/l/tablekit-core) | Core library, schemas, examples, ETL framework, 43 tests, full docs | €49 |
| [**TableKit Pro**](https://sagnelli.gumroad.com/l/tablekit-pro) | Everything in Core + Databricks Asset Bundle, pipeline definitions, deployment notebooks | €99 |

---

## Built by

Enrico Sagnelli — Data Engineer with experience in banking, insurance, and legal data platforms across Azure and Databricks.

[sagnelli.gumroad.com](https://sagnelli.gumroad.com)
