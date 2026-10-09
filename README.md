# Retail Analytics Platform

A learning-focused portfolio project for data engineering, data analytics, and data science internship applications.

The goal is to build an end-to-end retail/e-commerce analytics platform: ingest raw data with Python, store and transform it, model it for analytics, and eventually present business insights through BI dashboards.

## Current status

The repository contains the initial project structure, a profile of the Olist raw CSV dataset, and a local PostgreSQL development environment managed by Docker Compose. A SQL script creates an empty `raw` schema (a namespace for future tables). No dataset tables, ingestion pipeline, transformations, or dashboards have been implemented, and no Python dependencies are required yet.

The project will grow in small, validated steps. pytest, dbt, Power BI, Azure, and CI remain future plans.

## Repository layout

- `docs/`: planned architecture and architectural decisions.
- `src/`: Python source code for future ingestion and processing.
- `tests/`: future automated tests and data-quality checks.
- `data/raw/`: local source data, excluded from Git.
- `data/processed/`: local generated data, excluded from Git.
- `sql/schema/`: numbered SQL schema scripts, applied in filename order on first database initialization.
- `docker-compose.yml`: local PostgreSQL service and persistent data volume.
- `requirements.txt`: Python dependencies; none at this stage.
- `.env.example`: safe example values for local database configuration.

## Local PostgreSQL setup

Install Docker Desktop (which includes Docker Compose) and start its engine. Run the following commands from the repository root.

1. Create local configuration:

   ```sh
   cp .env.example .env
   ```

   Edit `.env` and replace `POSTGRES_PASSWORD` with a local development password. `POSTGRES_DB` and `POSTGRES_USER` select the initial database and role. `POSTGRES_PORT` selects the host port; change it if port 5432 is already occupied. The development role is a PostgreSQL superuser, as configured by the official image; a production environment will need a separate access design.

2. Start PostgreSQL:

   ```sh
   docker compose up -d --wait
   ```

   Plain `docker compose up` also works, showing logs in the terminal. On the first startup, the image creates the configured database and runs the SQL files mounted from `sql/schema/`. This currently creates only the `raw` namespace.

3. Verify readiness, the selected database, and the schema:

   ```sh
   docker compose ps
   docker compose exec postgres sh -c 'pg_isready -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
   docker compose exec postgres sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -v ON_ERROR_STOP=1 -c "SELECT current_database();" -c "SELECT schema_name FROM information_schema.schemata WHERE schema_name = '\''raw'\'';"'
   ```

   Expect a healthy service, an accepting-connections response, your configured database name (`retail_analytics` by default), and one schema named `raw`.

Connect from a host database client at `127.0.0.1`, using the port, database, username, and password from `.env`. The published port is bound to localhost. To open an interactive SQL session without installing a host client:

```sh
docker compose exec postgres sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
```

### Applying schema changes

The official image runs initialization scripts only when its data directory is empty. Existing volumes are preserved: restarting the container does not rerun scripts, and editing database names or credentials in `.env` does not change an already initialized database.

To apply or reapply the current script manually:

```sh
docker compose exec -T postgres sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -v ON_ERROR_STOP=1' < sql/schema/001_create_raw_schema.sql
```

This script is safe to reapply because it uses `CREATE SCHEMA IF NOT EXISTS`. Apply future scripts explicitly in filename order and review their behavior before rerunning them; they are not automatically migrations or safe to repeat.

### Stopping and persistence

```sh
docker compose down
```

This removes the service container and network but retains the named `postgres_data` volume. Starting again reuses the stored database. `docker compose down -v` also deletes the database volume and its contents; use it only when intentionally resetting local data. Do not change the PostgreSQL major version against an existing volume without a planned database upgrade.

Image and initialization behavior follow the [official PostgreSQL image documentation](https://hub.docker.com/_/postgres). Compose reads configuration values from `.env` through [variable interpolation](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/); explicitly exported shell variables take precedence.

## Configuration and data

Keep credentials in the ignored local `.env` file. Commit only safe example values in `.env.example`. Never commit credentials, secrets, or connection strings. Local environment files and datasets are excluded by `.gitignore`.

See [the planned architecture](docs/architecture.md) and [the decision log](docs/decisions.md) for project direction.
