# Explore your connected data

Once your agent is connected to Anything Graph MCP, browse live schema and sample rows **before** authoring a playbook. Credentials stay in your profile; the agent only sees structure and read-only previews.

## Prerequisites

- [Anything Graph is running](index.md) (`anythinggraph start`)
- [Data sources are connected](connect-your-data.md)
- [Your agent is connected to MCP](connect-your-own-agent.md)
- **Admin bearer token** in MCP config (authoring and introspection tools require admin)

## What to ask your agent

Start with a discovery prompt:

```
Using anythinggraph-cli MCP:
1. list_sources — show my connected data sources
2. For each source, introspect_source to list tables, collections, or objects
3. sample_source on 2–3 interesting resources (limit 5 rows each)
```

Replace source names with yours (e.g. `warehouse_pg`, `payroll_csv`, `mongo_main`).

## MCP tools for exploration

| Tool | Purpose |
|------|---------|
| `list_sources` | Profile keys and adapter types — no secrets returned |
| `introspect_source` | Tables, columns, foreign keys, or API shapes (read-only) |
| `sample_source` | A few raw rows from one table, collection, or object |
| `get_adapter_guide` | How to map this adapter in bindings (call before authoring) |

### Example: Postgres

```
introspect_source with source_id warehouse_pg and schema_name public
```

Then sample a table:

```
sample_source with source_id warehouse_pg, resource users, limit 5
```

### Example: CSV

```
introspect_source with source_id payroll_csv
```

Column headers come from the file — no `schema_name` needed.

### Example: MongoDB

```
introspect_source with source_id mongo_main and schema_name mydb
```

## What you are looking for

As you explore, note:

| Question | Why it matters |
|----------|----------------|
| Which tables or files represent **people** or **accounts**? | Candidate entities |
| What columns are **identifiers** vs display fields? | `identifier` and `attributes` in the playbook |
| Which **foreign keys** or shared columns link records? | Relationships in the playbook + `link_column` in bindings |
| Which sources should sit in **one playbook** vs separate playbooks? | Federated `sources` map |

You do not need a playbook to explore. Playbooks come next when you turn this schema into business vocabulary.

## Read-only by design

Exploration tools never insert, update, or delete data. `sample_source` and `introspect_source` are catalog reads only.

## Next step

When you have a mental model of your data, ask your agent to [suggest a playbook structure](suggest-playbook-structure.md) — entities, relationships, and which source each entity uses.
