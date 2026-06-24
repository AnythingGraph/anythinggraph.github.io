# MongoDB

Connect Anything Graph to MongoDB collections using the `mongodb` adapter. Bindings reference collection names; the runtime compiles `find:` and `count:` operations automatically.

## Quick connect

```bash
anythinggraph source add
```

Choose **MongoDB (`mongodb`)**, pick a profile name (for example `mongo_main`), then enter:

- **Connection URI:** `mongodb://localhost:27017` (or your Atlas connection string)
- **Database name:** e.g. `anythinggraph`

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  mongo_main:
    adapter: mongodb
    dsn: env:AG_MONGO_MAIN_DSN
    database: env:AG_MONGO_MAIN_DATABASE
```

**Secrets** (`.env`):

```bash
AG_MONGO_MAIN_DSN=mongodb://localhost:27017
AG_MONGO_MAIN_DATABASE=mydb
```

The demo profile in the repo uses `AG_MONGODB_DSN` and `AG_MONGODB_DATABASE` for the `mongo_main` source.

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key |
| `entities.*.from` | **Collection name** |
| `entities.*.fields` | Playbook field → document field path |

Binding file suffix: `.mongodb.yaml`

Example:

```yaml
source_id: mongo_main

entities:
  crm_user:
    from: users
    id: _id
    fields:
      user_id: user_id
      full_name: full_name

  crm_transaction:
    from: transactions
    id: _id
    fields:
      amount: amount

relationships:
  user_transactions:
    object: crm_transaction
    link_column: user_id
```

**Do not author** raw `find:` or `count:` operations in binding YAML — the compiler generates them.

## Introspect (MCP)

```json
{ "source_id": "mongo_main", "schema_name": "mydb" }
```

`schema_name` is the MongoDB database name (or use the `database` field from the profile).

## See also

- [Connectors overview](index.md)
- [Connect your data](../getting-started/connect-your-data.md)
