# Architecture

## The Four Layers

TableKit organises data engineering code into four layers. Each layer has one job. No layer reaches up into the layers above it.

```
┌──────────────────────────────────────────────────────┐
│                    ETL Layer                          │
│  Orchestrates the flow between sources and targets.  │
│  Runs extract, transform, validate, and load.        │
├──────────────────────────────────────────────────────┤
│                  Tables Layer                        │
│  Definitions: the schema for each table.             │
│  Transformations: how to convert raw data.           │
├───────────────────────────┬──────────────────────────┤
│      Schemas Layer        │      Sources Layer        │
│  Reusable column sets.    │  Declarative data        │
│  Complex type templates.  │  source definitions.     │
├───────────────────────────┴──────────────────────────┤
│                   Core Library                       │
│  ColumnSchema · TableDefinition · TableModel         │
│  Schema enforcement · PII masking · Utilities        │
└──────────────────────────────────────────────────────┘
```

---

## Why Four Layers

Most data pipelines mix everything together. The schema is defined inside the ETL. The source path is hardcoded in the extract method. The transformation logic is buried between schema setup and write operations.

This works until it doesn't. When a schema changes, you search for every place that schema appears. When a source path changes, you hope you find all the ETLs that use it. When an audit column is added, you update it in thirty places and miss three.

TableKit separates these concerns so that each type of change has exactly one place to make it.

---

## Core Library

The foundation. No opinions about your data. No business logic.

Provides the building blocks that everything else uses: typed column definitions, composable table schemas, environment-aware table identities, and utilities for schema enforcement and data masking.

This layer is stable. It changes only when the fundamental abstractions need to evolve.

---

## Schemas Layer

Reusable components that belong to no single table.

Audit columns appear in every table. SCD2 keys appear in every slowly-changing dimension. Address structures appear wherever addresses are stored. These are defined once in the schemas layer and composed into tables that need them.

---

## Sources Layer

Where data comes from is a first-class concern, not an implementation detail buried in ETL code.

Supported source types:

| Source | Use case |
|--------|----------|
| File (CSV, JSON, Parquet, ORC) | Data lake landing zones |
| Delta Lake | Bronze / silver / gold layers, with time travel |
| JDBC | Relational databases (SQL Server, PostgreSQL, etc.) |
| Auto Loader | Databricks incremental file ingestion |
| Catalog Table | Any table registered in the Spark catalog |

A source is defined once and attached to a table. The ETL layer reads from it without needing to know the source type.

---

## Tables Layer

Two folders. Two jobs.

**Definitions** — what each table looks like. Schema, source, column mapping. The complete identity of the table.

**Transformations** — how raw data becomes target data. Pure functions: a DataFrame goes in, a clean DataFrame comes out. No side effects. Easy to test.

Keeping these separate means you can test a transformation without running a full pipeline, and you can change a schema without touching transformation logic.

---

## ETL Layer

The orchestration layer. It reads from sources, applies transformations, validates the result, and writes to the target.

The flow is always the same:

```
Extract → Transform → Validate → Load → Result
```

What changes per table is the extract logic and the transformation. The validate step enforces the target schema. The load step handles OVERWRITE, APPEND, or MERGE based on configuration.

Every ETL run produces a result with records read, records written, duration, and status. A pipeline collects these results across multiple ETLs and reports the full picture.
