# ETL Framework

The ETL layer is where everything comes together. It uses tables, sources, and transformations to move data from source to target.

---

## The BaseETL Contract

Every ETL in TableKit follows the same contract:

```mermaid
flowchart TD
    R[run] --> E[extract\nread from source]
    E --> T[transform\napply business logic]
    T --> V[validate\nenforce schema]
    V --> L[load\nwrite to Delta]
    L --> RS[ETLResult\nrecords read/written\nduration · status]
```

`extract()`, `transform()`, `validate()`, and `load()` are separate methods. Each does one thing. You override only what you need to change.

---

## Two Ways to Use BaseETL

### Option 1: Override extract and transform

Full control over the source and transformation logic:

```
CustomETL (extends BaseETL)
├── extract()    → custom source reading logic
├── transform()  → your business transformation
└── validate()   → optional custom validation
```

Use this when the transformation is complex or the source requires special handling.

### Option 2: Source-based ETL

When a `TableModel` has a linked source and column mapping, `SourceBasedETL` handles everything automatically:

```mermaid
graph LR
    TM[TableModel\n+ source\n+ column_mapping] --> SE[SourceBasedETL]
    SE --> E2[extract\nreads from table.source]
    SE --> T2[transform\napplies column_mapping]
    SE --> V2[validate\nenforces schema]
    SE --> L2[load\nwrites to target]
```

No custom code needed. Define the source and mapping at the table level, and the ETL runs itself.

---

## Write Modes

```mermaid
graph TD
    WM[Write Mode]
    WM --> OW[OVERWRITE\nReplace the entire table.\nUsed for full reloads.]
    WM --> AP[APPEND\nAdd rows without touching existing data.\nUsed for append-only tables.]
    WM --> MG[MERGE\nUpsert by primary key.\nUpdate matching rows, insert new ones.\nUsed for dimension tables.]
```

The write mode is set at the ETL level, not inside `load()`. Changing the mode does not require touching the load logic.

---

## Pipeline Orchestration

Multiple ETLs can be combined into a pipeline:

```mermaid
sequenceDiagram
    participant P as SalesPipeline
    participant C as CustomersETL
    participant O as OrdersETL

    P->>C: run()
    C-->>P: ETLResult SUCCESS
    P->>O: run()
    O-->>P: ETLResult SUCCESS
    P-->>P: PipelineResult\ntotal records · duration · status
```

The pipeline:
- Runs ETLs in order
- Stops on failure if `fail_fast=True`
- Collects all results into a `PipelineResult`
- Reports duration and records per ETL

---

## ETLResult

Every ETL run produces an `ETLResult`:

```
ETLResult
├── table_name       → fully-qualified target table name
├── records_read     → rows read from source
├── records_written  → rows written to target
├── start_time       → run start timestamp
├── end_time         → run end timestamp
├── duration_seconds → derived: end - start
├── status           → SUCCESS or FAILED
└── error_message    → set if status is FAILED
```

This makes pipeline monitoring straightforward. Log the result, alert on failure, track trends over time.

---

## Incremental Loading

For large tables, `IncrementalETL` uses a watermark to only process new records:

```mermaid
flowchart LR
    WM[Watermark\nlast run timestamp] --> SRC[Source\nfiltered by watermark]
    SRC --> RAW[Raw DataFrame\nnew records only]
    RAW --> T[Transform]
    T --> MG[Merge to Delta\nupsert by PK]
    MG --> UWM[Update Watermark\nfor next run]
```

Only new or updated records are processed. The watermark is stored and updated after each successful run.
