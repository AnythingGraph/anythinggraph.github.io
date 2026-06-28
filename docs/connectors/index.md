# Connectors & Bindings

Anything Graph connects your agents to live data without copying it into a new warehouse. Register a source in your profile, introspect its schema through MCP, then author **bindings** that map tables, collections, or API paths to entities in an **ontology playbook** — with credentials kept in `.env`, never in playbooks.

Pick a connector below for step-by-step setup, or start with [Connect your data](../getting-started/connect-your-data.md) to add a source with the CLI.

## Currently supported adapters

The adapters shipped in the repo today are the `adapter-*` crates in the [ag-cli/crates](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates) directory:

| Crate | Profile `adapter` | Typical sources |
|-------|-------------------|-----------------|
| [adapter-sql](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates/adapter-sql) | `sql` | PostgreSQL (via sqlx) |
| [adapter-csv](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates/adapter-csv) | `csv` | Local CSV and flat files |
| [adapter-soql](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates/adapter-soql) | `soql` | Salesforce REST API |
| [adapter-mongodb](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates/adapter-mongodb) | `mongodb` | MongoDB collections |
| [adapter-rest](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates/adapter-rest) | `rest` | HTTP JSON APIs with OpenAPI specs |

Each adapter is a Rust crate implementing the shared [`adapter-core`](https://github.com/AnythingGraph/AnythingGraph/tree/main/Sources/OSS/ag-cli/crates/adapter-core) `DataAdapter` trait. More connectors are on the roadmap — same playbook model, different backends.

## All adapters (including planned)

| Adapter | Profile `adapter` | Typical sources | Introspect (MCP) | Status |
|---------|-------------------|-----------------|------------------|--------|
| **SQL** | `sql` | PostgreSQL (via sqlx) | Tables, columns, foreign keys | Available |
| **CSV** | `csv` | Local CSV and flat files | Column headers from file | Available |
| **SOQL** | `soql` | Salesforce REST API | Describe objects & fields | Available |
| **MySQL** | `mysql` | MySQL, MariaDB | Tables, columns, keys | Available |
| **SQL Server** | `mssql` | Microsoft SQL Server, Azure SQL | Tables, columns, keys | Available |
| **MongoDB** | `mongodb` | MongoDB collections | Collections & field samples | Available |
| **REST / OpenAPI** | `rest` | HTTP JSON APIs with OpenAPI specs | Paths, parameters, response shapes | Available |
| **BigQuery** | `bigquery` | Google BigQuery datasets | Datasets, tables, columns | Planned |
| **Snowflake** | `snowflake` | Snowflake warehouses | Schemas, tables, columns | Planned |
| **Databricks** | `databricks` | Databricks SQL / Unity Catalog | Catalogs, schemas, tables | Planned |
| **Elasticsearch** | `elasticsearch` | Elasticsearch, OpenSearch indices | Index mappings | Planned |
| **S3 / Parquet** | `s3` | Amazon S3, object storage, Parquet files | Prefixes, columns from Parquet | Planned |
| **GraphQL** | `graphql` | GraphQL endpoints | Schema introspection | Planned |
| **Google Sheets** | `google_sheets` | Google Sheets spreadsheets | Sheet tabs & column headers | Planned |
| **HubSpot** | `hubspot` | HubSpot CRM objects | Object properties | Planned |



## Binding patterns

- **`mysql` / `mssql`** — same YAML shape as Postgres (`from`, `fields`, `subject_link_column`); SQL dialect is compiled automatically
- **`mongodb`** — set `from` to a collection name; runtime compiles `find:` / `count:` operations
- **`rest`** — set `from` to an API path (e.g. `/users`); runtime compiles `GET` request templates; profile needs `base_url`

Binding file suffix often matches the source key (`.postgres`, `.mysql`, `.mongodb`, `.rest`, …). The adapter type comes from the profile entry referenced by `source_id`.

## Configure a connector

Register sources in `profiles/local.yaml` and set credentials in `.env`. See [Connect your data](../getting-started/connect-your-data.md) for profile examples and environment variables.
