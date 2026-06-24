# PostgreSQL

Connect Anything Graph to PostgreSQL using the `sql` adapter (sqlx).

## Quick connect

Make sure the stack is running (`anythinggraph start`), then:

```bash
anythinggraph source add
```

Choose **PostgreSQL (`sql`)**, pick a profile name (for example `warehouse_pg`), and enter your connection string:

```
postgres://username:password@localhost:5432/yourdb
```

The CLI validates the connection, writes `profiles/local.yaml`, and stores the DSN in `.env`.

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  warehouse_pg:
    adapter: sql
    dsn: env:AG_WAREHOUSE_PG_DSN
```

**Secrets** (`.env`):

```bash
AG_WAREHOUSE_PG_DSN=postgres://user:pass@localhost:5432/yourdb
```

Env var names are derived from your `source_id`. The demo profile uses `AG_SQL_DSN` for `warehouse_pg` when configured manually from `.env.example`.

Restart or reload after editing:

```bash
anythinggraph stop && anythinggraph start
```

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key, e.g. `warehouse_pg` |
| `entities.*.from` | SQL **table name** |
| `entities.*.id` | Primary key column (or logical id column) |
| `entities.*.fields` | Playbook field → column name |
| `relationships.*.link_column` | Foreign-key column on the object table |

Binding file suffix: `.postgres.yaml` (for adapter `sql`).

Example:

```yaml
source_id: warehouse_pg

entities:
  crm_user:
    from: users
    id: user_id
    fields: [full_name]

  crm_account:
    from: accounts
    id: account_name
    fields: [industry]

relationships:
  owns_account:
    object: crm_account
    link_column: owner_user_id
```

**Do not put** DSN, raw SQL, or `adapter` in binding files — credentials stay in the profile and `.env` only.

## Introspect (MCP)

Use the `introspect_source` MCP tool to list tables, columns, and foreign keys:

```json
{ "source_id": "warehouse_pg", "schema_name": "public" }
```

`schema_name` is the Postgres schema (default `public`). It is used for introspection only — not a binding YAML field.

## See also

- [Connect your data](../getting-started/connect-your-data.md) — CLI wizard and listing sources
- [Connectors overview](index.md) — all adapters and binding patterns
