# Data Sources

A data source is a first-class object in TableKit. It is not a string path inside an ETL method. It is a defined, named, reusable component.

---

## The Problem

In most pipelines, the source is invisible:

```python
return spark.read.csv("/mnt/landing/customers/")
```

This line tells you nothing about the source except its location. You do not know what format options are applied, whether there is a schema, or whether this path is used anywhere else. If the path changes, you search the codebase hoping to find every occurrence.

---

## Declarative Sources

TableKit defines sources as structured objects with a name, type, location, and options. A source can be attached to a `TableModel`, making the relationship between a table and its data permanent and visible.

When a source is attached to a table, the ETL layer can read from it automatically. The ETL does not need to know the source type, path, or format. It asks the table, reads the source, and proceeds.

---

## Supported Source Types

**File sources** cover the most common landing zone formats: CSV, JSON, Parquet, ORC, and Avro. Format-specific options like headers, delimiters, and multiline support are part of the source definition.

**Delta sources** read from Delta Lake tables with optional time travel. You can read the current state, a specific version, or the state at a specific timestamp. This is useful for incremental pipelines and historical audits.

**JDBC sources** connect to relational databases. Connection details, table or query, fetch size, and credentials are all part of the source definition. The ETL layer does not see the connection string.

**Auto Loader** is Databricks' incremental file ingestion mechanism. It tracks which files have been processed and picks up new arrivals automatically. Auto Loader sources support schema inference and hints, with schema stored at a configurable location.

---

## Source and Table Together

The most powerful use of sources is binding them to tables:

```
TableModel
├── definition    → the schema
├── source        → where the data comes from
└── column_mapping → how source columns map to target columns
```

This makes the table definition self-contained. Anyone reading it knows the schema, the source, and the mapping. There is no need to trace through ETL code to understand where data originates.

---

## Streaming

Delta and Auto Loader sources support streaming reads. The same source definition works for both batch and streaming pipelines. Auto Loader tracks processed files through a checkpoint, so restarts pick up exactly where they left off.
