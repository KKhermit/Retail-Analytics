# Architecture

## Current implementation

Only the repository scaffold exists. The Python packages, test directory, data directories, and SQL directory are placeholders. There is no running pipeline, database, analytics model, or BI integration.

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

All components above are future plans. Dataset selection, table schemas, transformations, deployment details, and cloud services remain undecided. Each stage will be designed and implemented separately.
