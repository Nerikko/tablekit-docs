# ETL Framework

The ETL layer connects everything. It reads from sources, applies transformations, validates results, and writes to Delta tables.

---

## A Consistent Contract

Every ETL in TableKit follows the same four-step flow:

```
Extract → Transform → Validate → Load
```

This consistency matters. When you read an ETL built with TableKit, you already know its structure. When you debug a failure, you know exactly which step to look at. When you onboard a new team member, they understand every pipeline after reading one.

---

## Two Modes

**Custom ETL** — you implement extract and transform. The base class handles validation and loading. Use this when the transformation is complex or the source requires special handling.

**Source-based ETL** — the table definition already has a source and column mapping. The ETL reads the source, applies the mapping, enforces the schema, and loads. No custom code required.

The second mode is the goal. When a table definition is complete — schema, source, mapping — the ETL writes itself.

---

## Write Modes

Three write modes cover the full range of data loading patterns:

**OVERWRITE** replaces the entire table. Used for full reloads when upstream data can change retroactively or when partition pruning is not worth the complexity.

**APPEND** adds rows without touching existing data. Used for append-only tables like event logs and audit trails.

**MERGE** upserts by primary key. Matching rows are updated. New rows are inserted. This is the standard mode for dimension tables.

The write mode is a configuration choice, not a code change. Switching from APPEND to MERGE on a table does not require editing load logic.

---

## Results and Observability

Every ETL run produces a structured result:

- Table name
- Records read from source
- Records written to target
- Run duration
- Status (success or failure)
- Error message if failed

A pipeline that runs multiple ETLs collects all results and reports them together. Total records written, which ETLs succeeded, which failed, and why. This output is designed to be logged, monitored, and alerted on.

---

## Incremental Loading

For large tables that grow continuously, loading everything on every run is wasteful. TableKit's incremental pattern processes only records created or modified since the last successful run.

The watermark is the timestamp of the last run. The source is filtered to records after the watermark. After a successful load, the watermark is updated. On failure, the watermark stays at its previous value so the next run retries the same window.

---

## Pipelines

Multiple ETLs compose into a pipeline. The pipeline runs them in order, collects their results, and stops on failure if configured to do so.

A pipeline is not a scheduler. It does not know about cron expressions or Databricks job clusters. It is an ordered sequence of ETL runs with shared configuration — environment, write mode, fail-fast behaviour — and a combined result.

Scheduling and infrastructure are handled by the deployment layer. In the Pro edition, this is a Databricks Asset Bundle with job definitions, cluster configuration, and retry logic.
