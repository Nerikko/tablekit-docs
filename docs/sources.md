# Data Sources

A `DataSource` defines where data comes from. It is a first-class object that can be attached to a `TableModel`, passed to an ETL, or used directly.

---

## The Problem with Hardcoded Sources

In most pipelines, the source path is buried inside the ETL code:

```python
def extract(self):
    return self.spark.read.csv("/some/path/customers/")
```

This means:
- The source location is invisible from the table definition
- Changing the source requires editing the ETL class
- You cannot reuse the same source across multiple ETLs without duplication
- Testing requires mocking at the class level

---

## Declarative Sources

A `DataSource` separates the source from the logic:

```
DataSource
├── name         → identifier for this source
├── source_type  → CSV, JSON, Parquet, Delta, JDBC, AutoLoader, etc.
├── path         → file path or table location
├── options      → format-specific options (header, delimiter, etc.)
└── schema       → optional explicit StructType
```

Define it once, attach it to a table, and the ETL layer reads from it automatically.

---

## Source Types

```mermaid
graph TD
    DS[DataSource]
    DS --> FS[FileSource\nCSV · JSON · Parquet · ORC · Avro]
    DS --> JS[JDBCSource\nSQL Server · PostgreSQL · MySQL · Oracle]
    DS --> DLS[DeltaSource\nDelta Lake with optional time travel]
    DS --> ALS[AutoLoaderSource\nDatabricks cloudFiles incremental ingestion]
    DS --> TS[TableSource\nSpark catalog table]
```

---

## Source-Table Binding

Attaching a source to a `TableModel` makes the relationship explicit:

```mermaid
graph LR
    SRC[FileSource\npath: /landing/customers\nformat: csv] --> TBL
    MAP[column_mapping\ncustomer_id → id\nemail → user_email] --> TBL
    TBL[TableModel\ndim_customers] --> ETL[SourceBasedETL]
    ETL --> DLT[Delta Table]
```

The `TableModel` knows:
- Where its data comes from
- How source columns map to target columns

The ETL does not need to know either. It asks the table, reads the source, applies the mapping, and loads.

---

## Time Travel with Delta

`DeltaSource` supports Delta Lake time travel for historical reads:

```mermaid
graph LR
    T0[Version 0\n2024-01-01] --> T1[Version 1\n2024-06-01]
    T1 --> T2[Version 2\n2024-12-01]
    T2 --> T3[Version 3\ncurrent]

    Q1[DeltaSource\nversion: 1] -.reads.-> T1
    Q2[DeltaSource\ntimestamp: 2024-12-01] -.reads.-> T2
    Q3[DeltaSource\nno options] -.reads.-> T3
```

This is useful for incremental pipelines, auditing, and debugging production issues without needing backups.

---

## Streaming Sources

`DeltaSource` and `AutoLoaderSource` support streaming reads via `source.read_stream(spark)`:

```mermaid
flowchart LR
    LS[Landing Zone\nnew files arrive] --> AL[AutoLoaderSource\ncloudFiles format]
    AL --> SD[Streaming DataFrame]
    SD --> T[Transformations]
    T --> DW[Delta Table\nstreaming write]
    DW --> CP[Checkpoint\noffset tracking]
    CP --> AL
```

Auto Loader tracks which files have been processed. New files are picked up automatically. No manual watermarks.
