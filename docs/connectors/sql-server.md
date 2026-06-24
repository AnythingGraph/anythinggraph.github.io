# SQL Server

Connect Anything Graph to Microsoft SQL Server or Azure SQL using the `mssql` adapter. Binding YAML matches PostgreSQL and MySQL; the runtime compiles T-SQL automatically.

## Quick connect

```bash
anythinggraph source add
```

Choose **SQL Server (`mssql`)**, pick a profile name (for example `erp_mssql`), and enter a **JDBC-style** connection string:

```
jdbc:sqlserver://localhost:1433;databaseName=yourdb;user=sa;password=YourPassword
```

For Azure SQL, use your server hostname and enable encryption as required by your instance.

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  erp_mssql:
    adapter: mssql
    dsn: env:AG_ERP_MSSQL_DSN
```

**Secrets** (`.env`):

```bash
AG_ERP_MSSQL_DSN=jdbc:sqlserver://localhost:1433;databaseName=yourdb;user=sa;password=YourPassword
```

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key |
| `entities.*.from` | SQL **table name** |
| `entities.*.id` | Primary key column |
| `entities.*.fields` | Playbook field → column name |
| `relationships.*.link_column` | Foreign-key column on the object table |

Binding file suffix: `.mssql.yaml`

Example:

```yaml
source_id: erp_mssql

entities:
  crm_user:
    from: Users
    id: UserId
    fields: [FullName]

relationships:
  owns_account:
    object: crm_account
    link_column: OwnerUserId
```

**Do not put** DSN, raw SQL, or `adapter` in binding files.

## Introspect (MCP)

```json
{ "source_id": "erp_mssql", "schema_name": "dbo" }
```

`schema_name` is the SQL Server schema (default `dbo`).

## See also

- [PostgreSQL](postgresql.md) — same binding pattern for SQL-family sources
- [Connectors overview](index.md)
