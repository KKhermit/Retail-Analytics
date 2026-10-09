# Architecture

## Current implementation

The repository scaffold and an initial raw-data profile exist. `data/raw/` contains nine Olist CSV files covering customers, orders, items, payments, reviews, products, sellers, geolocation, and category translations. Docker Compose now defines a local PostgreSQL 17 service with environment-based credentials, a localhost port, a readiness check, and a persistent named volume. `sql/schema/001_create_raw_schema.sql` creates an empty `raw` namespace on first initialization; it can also be applied manually as documented in the README.

The Python packages, test directory, and processed-data directory remain placeholders. No dataset tables, ingestion pipeline, analytics model, or BI integration have been implemented. Raw CSV files are not loaded into PostgreSQL yet.

See [the data dictionary](data_dictionary.md) for observed columns, row counts, candidate keys, relationships, data-quality concerns, and a proposed relational schema. The initial ingestion source is these local CSV files. API ingestion remains a possible future extension.

## Planned architecture

The intended flow is:

```text
Raw retail/e-commerce data or API
             |
             v
      Python ingestion
             |
             v
         PostgreSQL
             |
             v
   SQL/dbt transformations
             |
             v
Analytics warehouse / dimensional models
             |
             v
          Power BI
```

Python will ingest source data. PostgreSQL will store it, SQL and later dbt will transform it, and dimensional models will support business analysis in Power BI.

Testing and data-quality checks will be introduced as pipeline behavior is implemented. Docker Compose currently manages the local database; containerizing Python ingestion remains future work. Later work may move parts of the pipeline to Azure and add GitHub Actions for CI.

The local database environment is implemented; all other pipeline components above remain future plans. The initial dataset is now selected, but final table definitions, transformations, deployment details, and cloud services remain undecided. The dataset requires multiple related tables; repeated geolocation postal prefixes and review identifiers need careful key and join handling, as documented in the data dictionary. Each stage will be designed and implemented separately.
