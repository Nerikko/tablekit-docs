# Why TableKit

## The Real Cost of Schema Sprawl

A data platform with ten tables probably has twenty places where schema is defined or assumed. The `StructType` in the notebook. The column list in the SQL query. The field names in the transformation. The documentation in the wiki that nobody updates.

When a schema changes — and it always does — each of those places needs to change. Some of them will be missed. The pipeline will fail in production, or worse, it will succeed and produce wrong data.

TableKit exists because this is a solved problem in software engineering. Application developers do not repeat type definitions across twenty files. They define a model once and use it everywhere. Data engineers should do the same.

---

## Schema as the Source of Truth

When a table definition is the single source of truth, a schema change has exactly one place to make it. The ETL reads from the definition. The tests validate against the definition. The documentation generates from the definition.

This is not a new idea. It is how well-engineered systems work. TableKit brings it to PySpark.

---

## What Composition Eliminates

Every data team has audit columns. Every data team has SCD2 tables. Every data team has address or contact structures that appear across multiple tables.

Without composition, these are copy-pasted. Over time, copies diverge. One table has `created_at` as a timestamp. Another has it as a string. One table has four SCD2 columns. Another has three and uses different names.

With composition, these patterns are defined once. A table either uses them or it does not. There is no third option where it uses a slightly different version.

---

## Explicit Beats Implicit

Two questions that should have obvious answers in any data platform:

**Where does this table's data come from?**
In most pipelines, you trace through ETL code, read configuration files, and check environment variables to answer this. In TableKit, the source is part of the table definition. The answer is one line.

**Which environment does this pipeline target?**
In most platforms, this is a string concatenated somewhere in the code. In TableKit, the environment is set once and every table name derives from it. You cannot accidentally hardcode the wrong catalog.

---

## What You Cannot See Here

This documentation shows what TableKit does and why the design decisions are sound. It does not show how to build it.

The implementation — the column validation logic, the composition rules, the merge condition generation, the schema enforcement utilities, the source abstraction — is what you get when you buy TableKit.

What you buy is not just code. It is months of iteration on patterns used in production data platforms across banking, insurance, and legal industries. It is a starting point that works, rather than a starting point you build yourself.

---

## Get TableKit

| Edition | What's included | Price |
|---------|----------------|-------|
| [**TableKit Core**](https://sagnelli.gumroad.com/l/tablekit-core) | Core library, schemas, sources, ETL framework, 43 tests, full docs | €49 |
| [**TableKit Pro**](https://sagnelli.gumroad.com/l/tablekit-pro) | Everything in Core + Databricks Asset Bundle, pipeline definitions, deployment notebooks | €99 |

[View the documentation repository](https://github.com/Nerikko/tablekit-docs) · [Gumroad store](https://sagnelli.gumroad.com)
