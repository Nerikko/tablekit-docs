# Architecture

## The Four Layers

TableKit organises data engineering code into four layers. Each layer depends only on the layers below it.

```mermaid
graph BT
    A[Core Library\nColumnSchema · TableDefinition · TableModel]
    B[Schemas Layer\nAUDIT_COLUMNS · SCD2_COLUMNS · Complex Types]
    C[Sources Layer\nFileSource · JDBCSource · DeltaSource · AutoLoaderSource]
    D[Tables Layer\nDefinitions · Transformations]
    E[ETL Layer\nBaseETL · SourceBasedETL · Pipeline]

    A --> B
    A --> C
    B --> D
    C --> D
    D --> E
```

---

## Why Four Layers

Most data pipelines mix everything together. Schema definition, source reading, transformation logic, and write operations are all in the same notebook or class. Changing one thing requires understanding everything around it.

TableKit separates these concerns so each type of change has one place to make it.

---

## Core Library

The foundation. No opinions about your data, no business logic.

Provides: typed column definitions, composable table schemas, environment-aware table identities, schema enforcement, and PII masking utilities.

---

## Schemas Layer

Reusable components that belong to no single table.

```mermaid
graph LR
    A[AUDIT_COLUMNS] --> T1[dim_customers]
    A --> T2[dim_products]
    A --> T3[fact_orders]
    B[SCD2_COLUMNS] --> T4[dim_customers_history]
    B --> T5[dim_products_history]
```

Define audit columns once. Every table that needs them composes them in. One change propagates everywhere.

---

## Sources Layer

Where data comes from is a first-class concern, not an implementation detail.

```mermaid
graph TD
    DS[DataSource]
    DS --> FS[FileSource\nCSV · JSON · Parquet]
    DS --> JD[JDBCSource\nSQL Server · PostgreSQL]
    DS --> DL[DeltaSource\nDelta Lake + time travel]
    DS --> AL[AutoLoaderSource\nDatabricks incremental]
```

Every source has the same interface: `source.read(spark)`. The ETL layer does not care about the type.

---

## Tables Layer

Two folders. Two jobs.

**Definitions** — schema, source, and column mapping for each table. Everything needed to describe it.

**Transformations** — pure functions that convert raw source data to the target schema. A DataFrame goes in, a clean DataFrame comes out.

---

## ETL Layer

Connects sources to targets through transformations.

```mermaid
sequenceDiagram
    participant E as ETL
    participant S as Source
    participant T as Transform
    participant D as Delta Table

    E->>S: extract()
    S-->>E: raw DataFrame
    E->>T: transform(df)
    T-->>E: clean DataFrame
    E->>E: validate schema
    E->>D: load (merge / append / overwrite)
    D-->>E: ETLResult
```

---

## Data Flow End to End

```mermaid
flowchart LR
    LS[Landing Zone] --> SRC[DataSource]
    DB[(Database)] --> SRC
    DL[Delta Bronze] --> SRC
    SRC --> TR[Transformations]
    TR --> SC[Schema Enforcement]
    SC --> DLT[Delta Table]
```

---

## Why This Structure

**One place per change.** Schema change → edit the definition. Source change → edit the source. Transformation change → edit the function.

**Composability.** Columns, definitions, and sources are all composable. Build complex things from simple tested pieces.

**Testability.** Transformations are pure functions. Sources are injectable. Each layer can be tested independently.

**Consistency.** Audit columns and SCD2 patterns are defined once. Every table using them is automatically consistent.

**Environment safety.** Environment-aware naming is built into `TableModel`. You cannot accidentally target the wrong catalog.
