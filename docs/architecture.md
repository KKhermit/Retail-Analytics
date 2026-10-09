# Architecture

## Current implementation

The repository scaffold and an initial raw-data profile exist. `data/raw/` contains nine Olist CSV files covering customers, orders, items, payments, reviews, products, sellers, geolocation, and category translations. The Python packages, test directory, processed-data directory, and SQL directory remain placeholders. There is no running pipeline, database, analytics model, or BI integration.

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

Testing and data-quality checks will be introduced as pipeline behavior is implemented. Docker and Docker Compose are planned for reproducible local services. Later work may move parts of the pipeline to Azure and add GitHub Actions for CI.

All pipeline components above are future plans. The initial dataset is now selected, but final table schemas, transformations, deployment details, and cloud services remain undecided. The dataset requires multiple related tables; repeated geolocation postal prefixes and review identifiers need careful key and join handling, as documented in the data dictionary. Each stage will be designed and implemented separately.
