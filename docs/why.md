# Why TableKit

## Schema as Code

Most data teams treat schemas as a side effect of the pipeline. The schema is inferred, or it lives in a YAML file no one updates, or it's scattered across thirty `StructField` calls with no comments.

TableKit treats the schema as code. Not configuration, not documentation — code. That means:

- **Version controlled.** Schema changes go through code review.
- **Tested.** You can write tests against your schema definitions.
- **Discoverable.** IDE autocomplete works. You know what columns exist without running a query.
- **Self-documenting.** Comments are part of the schema, not a separate document.

---

## Composition Over Repetition

Every data team ends up writing the same patterns over and over:

```
created_at TIMESTAMP
updated_at TIMESTAMP
created_by STRING
updated_by STRING
```

These appear in every table. Without a framework, they get copy-pasted. One day someone adds a column in one table and forgets the others. Six months later, the schemas are different across tables and no one knows why.

TableKit solves this with composition:

```mermaid
graph TD
    A[AUDIT_COLUMNS\ndefined once] --> T1[dim_customers]
    A --> T2[dim_products]
    A --> T3[fact_orders]
    A --> T4[every other table]
```

Change `AUDIT_COLUMNS` in one place. Every table picks up the change.

The same applies to SCD2 columns, address structs, contact schemas — anything that appears in more than one table.

---

## Explicit Over Implicit

The two biggest sources of bugs in data pipelines are implicit assumptions and hidden state.

**Implicit source paths:**

```python
# Where does this data come from?
def extract(self):
    return self.spark.read.csv("/data/customers/")
```

**Explicit source binding:**

```mermaid
graph LR
    FS[FileSource\npath: /data/customers\nformat: csv\nheader: true] --> TM[TableModel\ndim_customers]
```

The table knows its source. Anyone reading the table definition knows where the data comes from without reading the ETL code.

**Implicit environment handling:**

```python
# Which environment is this?
df.write.saveAsTable("main.customers.users")
```

**Explicit environment awareness:**

```mermaid
graph LR
    TM[TableModel\nenv: dev] --> FN1[main.dev_customers.users]
    TM2[TableModel\nenv: prod] --> FN2[main.customers.users]
```

The environment is set once. Every table name derives from it automatically.

---

## Separation of Concerns

In a typical notebook pipeline, schema definition, source reading, transformation logic, and write mode are all mixed together. Changing one thing requires understanding everything.

TableKit splits these into separate layers:

```mermaid
graph TD
    A[What does the table look like?\nTableDefinition] -.->|separate| B
    B[Where does the data come from?\nDataSource] -.->|separate| C
    C[How do we convert raw data?\nTransformations] -.->|separate| D
    D[How do we write to the target?\nBaseETL write mode]
```

Each question has one answer in one place. When the source changes, you change the source. When the schema changes, you change the definition. When the transformation changes, you change the transformation.

---

## Testability

Pipelines built with TableKit are testable at every level:

```mermaid
graph TD
    U1[Unit: ColumnSchema\nvalidate column properties]
    U2[Unit: TableDefinition\nvalidate composition and schema]
    U3[Unit: DataSource\nvalidate source configuration]
    U4[Unit: Transformations\npure functions, easy to test]
    I1[Integration: ETL\nmock source, test full run]
    I2[Integration: Pipeline\ntest multi-table orchestration]

    U1 --> U2
    U2 --> U4
    U3 --> I1
    U4 --> I1
    I1 --> I2
```

Transformations are pure functions — DataFrame in, DataFrame out. Sources are injectable. The ETL contract is defined by an abstract class. Every part can be tested independently.

---

## Get TableKit

| Edition | What's included | Price |
|---------|----------------|-------|
| [**TableKit Core**](https://sagnelli.gumroad.com/l/tablekit-core) | Core library, schemas, sources, ETL framework, 43 tests, full docs | €49 |
| [**TableKit Pro**](https://sagnelli.gumroad.com/l/tablekit-pro) | Everything in Core + Databricks Asset Bundle, pipeline definitions, deployment notebooks | €99 |
