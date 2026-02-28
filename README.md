# TableKit

**Type-safe PySpark table definitions and ETL framework**

TableKit is a framework for defining data schemas and ETL pipelines in Python. Instead of writing raw PySpark schemas by hand, you define tables as structured Python objects with IDE support, validation, and automatic schema generation.

---

## The Problem

Data engineering teams repeat the same patterns over and over: writing schemas by hand with no type checking, copy-pasting audit columns across dozens of tables, hardcoding source paths inside ETL logic, and building pipelines that cannot be tested or reused.

TableKit addresses this with a layered architecture that gives each concern exactly one place to live.

```
┌──────────────────────────────────────────────────────┐
│                    ETL Layer                          │
│  Orchestrates the flow between sources and targets.  │
├──────────────────────────────────────────────────────┤
│                  Tables Layer                        │
│  Definitions: schema per table.                      │
│  Transformations: how to convert raw data.           │
├───────────────────────────┬──────────────────────────┤
│      Schemas Layer        │      Sources Layer        │
│  Reusable column sets.    │  Declarative data        │
│  Complex type templates.  │  source definitions.     │
├───────────────────────────┴──────────────────────────┤
│                   Core Library                       │
│  ColumnSchema · TableDefinition · TableModel         │
└──────────────────────────────────────────────────────┘
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [Core Concepts](docs/concepts.md) | ColumnSchema, TableDefinition, TableModel — what they are and why they work |
| [Architecture](docs/architecture.md) | The four layers, their responsibilities, and how they connect |
| [Data Sources](docs/sources.md) | Declarative source definitions for CSV, JDBC, Delta, Auto Loader, and more |
| [ETL Framework](docs/etl.md) | How the ETL layer uses everything below it — modes, results, incremental loading |
| [Why TableKit](docs/why.md) | The case for schema-as-code and what you get when you buy |

---

## Get TableKit

| Edition | What's included | Price |
|---------|----------------|-------|
| [**TableKit Core**](https://sagnelli.gumroad.com/l/tablekit-core) | Core library, schemas, sources, ETL framework, 43 tests, full docs | €49 |
| [**TableKit Pro**](https://sagnelli.gumroad.com/l/tablekit-pro) | Everything in Core + Databricks Asset Bundle, pipeline definitions, deployment notebooks | €99 |

---

## Built by

Enrico Sagnelli — Data Engineer with experience in banking, insurance, and legal data platforms across Azure and Databricks.

[sagnelli.gumroad.com](https://sagnelli.gumroad.com)
