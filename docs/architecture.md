# Architecture

## The Layered Model

TableKit organises code into four layers. Each layer depends only on the layers below it. Nothing reaches up.

```mermaid
graph TD
    ETL[ETL Layer\nBaseETL · SourceBasedETL\nPipeline orchestration]
    Tables[Tables Layer\nDefinitions · Transformations]
    Mid[Schemas Layer\nAUDIT_COLUMNS · SCD2_COLUMNS\nComplex types]
    Sources[Sources Layer\nFileSource · JDBCSource\nDeltaSource · AutoLoaderSource]
    Core[Core Library\nColumnSchema · TableDefinition\nTableModel · Utils]

    ETL --> Tables
    ETL --> Sources
    Tables --> Mid
    Tables --> Core
    Mid --> Core
    Sources --> Core
```

---

## Layer Responsibilities

### Core Library

The foundation. No business logic. No opinions about your data.

- `ColumnSchema` — a column with its type and metadata
- `TableDefinition` — a validated, composable collection of columns
- `TableModel` — full table identity with environment awareness
- `enforce_schema()` — cast DataFrames to target schemas
- `mask_pii_columns()` — type-aware PII masking

This layer is stable. It changes only when the core abstractions need to change.

### Schemas Layer

Reusable column sets and complex type definitions that belong to no single table.

```mermaid
graph LR
    AUDIT[AUDIT_COLUMNS\ncreated_at\nupdated_at\ncreated_by\nupdated_by]
    SCD2[SCD2_COLUMNS\nsk surrogate key\nvalid_from\nvalid_to\nis_current]
    TYPES[Complex Types\nADDRESS_SCHEMA\nCONTACT_SCHEMA\nMONEY_SCHEMA]
```

These are defined once and composed into any table that needs them. The alternative — copy-pasting — guarantees inconsistency over time.

### Sources Layer

Declarative data source definitions. A source is defined once and can be attached to a table or used directly in an ETL.

```mermaid
graph TD
    DS[DataSource]
    DS --> CSV[FileSource\nCSV / JSON / Parquet]
    DS --> DB[JDBCSource\nSQL Server / PostgreSQL]
    DS --> DL[DeltaSource\nDelta Lake + time travel]
    DS --> AL[AutoLoaderSource\nDatabricks incremental]
```

Every source has the same interface: `source.read(spark)`. The ETL layer does not care where the data comes from.

### Tables Layer

Where your actual tables live.

**Definitions** — the schema for each table, composed from core components and schemas.

**Transformations** — functions that convert raw source data to the target schema. Pure functions: DataFrame in, DataFrame out.

```mermaid
graph LR
    SRC[Source Data\nraw columns] -->|transform_fn| TGT[Target Schema\nclean columns]
```

Keeping transformations separate from definitions means you can test them independently, reuse them, and reason about them in isolation.

### ETL Layer

The orchestration layer. It connects sources to tables through transformations.

```mermaid
sequenceDiagram
    participant Pipeline
    participant ETL
    participant Source
    participant Transform
    participant Delta

    Pipeline->>ETL: run()
    ETL->>Source: extract()
    Source-->>ETL: raw DataFrame
    ETL->>Transform: transform(df)
    Transform-->>ETL: clean DataFrame
    ETL->>ETL: validate(df)
    ETL->>Delta: load(df)
    Delta-->>ETL: records written
    ETL-->>Pipeline: ETLResult
```

`BaseETL` defines the contract. Subclasses implement `extract()` and `transform()`. Everything else — validation, loading, result tracking — is handled by the base class.

---

## Data Flow

A full pipeline run from source to Delta table:

```mermaid
flowchart LR
    LS[Landing Zone\nCSV / JSON / Parquet] --> SRC
    DB[(Database\nJDBC)] --> SRC
    DL[Delta Lake\nbronze layer] --> SRC

    SRC[DataSource\n.read spark] --> RAW[Raw DataFrame]
    RAW --> TRANS[Transformations\ncolumn mapping\nbusiness logic]
    TRANS --> SCHEMA[Schema Enforcement\nenforce_schema]
    SCHEMA --> LOAD{Write Mode}

    LOAD -->|OVERWRITE| OW[Delta Table\nfull replace]
    LOAD -->|APPEND| AP[Delta Table\nappend rows]
    LOAD -->|MERGE| MG[Delta Table\nupsert by PK]
```

---

## Why This Structure

**Single source of truth.** A table's schema, source, and column mapping live in one place. Not scattered across notebooks and YAML files.

**Composability.** Columns, definitions, and sources are all composable. Build complex things from simple, tested pieces.

**Testability.** Each layer can be tested independently. Transformations are pure functions. Sources can be mocked. The ETL contract is defined by `BaseETL`.

**Consistency.** Audit columns, SCD2 patterns, and complex types are defined once. Every table that uses them is automatically consistent.

**Environment safety.** Environment-aware naming is built into `TableModel`. You cannot accidentally write to prod from a dev pipeline if the environment is set correctly.
