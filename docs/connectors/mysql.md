# MySQL / MariaDB

Connect Anything Graph to MySQL or MariaDB using the `mysql` adapter. Binding YAML uses the same shape as PostgreSQL; the runtime compiles SQL with MySQL dialect automatically.

## Quick connect

```bash
anythinggraph source add
```

Choose **MySQL / MariaDB (`mysql`)**, pick a profile name (for example `analytics_mysql`), and enter your connection string:

```
mysql://username:password@localhost:3306/yourdb
```

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  analytics_mysql:
    adapter: mysql
    dsn: env:AG_ANALYTICS_MYSQL_DSN
```

**Secrets** (`.env`):

```bash
AG_ANALYTICS_MYSQL_DSN=mysql://user:pass@localhost:3306/yourdb
```

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key |
| `entities.*.from` | SQL **table name** |
| `entities.*.id` | Primary key column |
| `entities.*.fields` | Playbook field → column name |
| `relationships.*.link_column` | Foreign-key column on the object table |

Binding file suffix: `.mysql.yaml`

Example:

```yaml
source_id: analytics_mysql

entities:
  crm_user:
    from: users
    id: user_id
    fields: [full_name]

relationships:
  owns_order:
    object: crm_order
    link_column: owner_user_id
```

**Do not put** DSN, raw SQL, or `adapter` in binding files.

## Introspect (MCP)

```json
{ "source_id": "analytics_mysql", "schema_name": "yourdb" }
```

`schema_name` is the MySQL database name used for catalog discovery.

## See also

- [PostgreSQL](postgresql.md) — same binding pattern, different adapter key
- [Connectors overview](index.md)
