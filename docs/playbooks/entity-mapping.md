# Entity mapping and relationships

Entities and relationships live in the **playbook**. Physical tables, columns, and foreign keys live in **bindings**. This page walks through both, step by step.

## Step 1 — Name your business objects (entities)

Start from how people talk about the domain, not from raw schema names.

| Business concept | Playbook entity | Identifier field | Attributes |
|------------------|-----------------|------------------|------------|
| Sales rep | `crm_user` | `user_id` | `full_name` |
| Customer account | `crm_account` | `account_name` | `industry` |

In playbook JSON:

```json
"entities": {
  "crm_user": {
    "identifier": "user_id",
    "attributes": ["full_name"]
  },
  "crm_account": {
    "identifier": "account_name",
    "attributes": ["industry"]
  }
}
```

- **`identifier`** — the stable id agents use to resolve a subject or join rows.
- **`attributes`** — fields agents may read or display. Names here are playbook vocabulary, not necessarily database column names.

## Step 2 — Define how entities connect (relationships)

A relationship is a **named edge** from one entity to another.

| Relationship | From | To | Meaning |
|--------------|------|-----|---------|
| `owns_account` | `crm_user` | `crm_account` | A user owns one or more accounts |

In playbook JSON:

```json
"relationships": {
  "owns_account": { "from": "crm_user", "to": "crm_account" }
}
```

Agents query relationships in plain language: *"How many accounts does Alex own?"* maps to counting `owns_account` where the subject is Alex's `user_id`.

Relationships are **directional**. `from` is usually the subject side; `to` is the object being accessed or counted.

## Step 3 — Route entities to data sources

The `sources` map tells the runtime which binding file to use for each entity.

```json
"sources": {
  "crm_user": "postgres",
  "crm_account": "postgres"
}
```

Both entities use the `postgres` source key → one binding file: `bindings/simple-crm-access.postgres.yaml`.

If payroll lives in a CSV while CRM stays in Postgres:

```json
"sources": {
  "crm_user": "postgres",
  "crm_account": "postgres",
  "crm_payroll_record": "csv"
}
```

That playbook needs **two** binding files: `.postgres.yaml` and `.csv.yaml`.

## Step 4 — Map entities to physical schema (binding)

Bindings translate playbook names to real tables and columns.

**Postgres binding** (`bindings/simple-crm-access.postgres.yaml`):

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

| Binding field | Meaning |
|---------------|---------|
| `source_id` | Profile key in `profiles/local.yaml` (credentials) |
| `entities.<name>.from` | Table, collection, file, or API path |
| `entities.<name>.id` | Physical id column (maps to playbook `identifier`) |
| `entities.<name>.fields` | Playbook attribute → physical column (omit when names match) |
| `relationships.<name>.object` | The `to` entity in playbook terms |
| `relationships.<name>.link_column` | Foreign key on the object side linking back to the subject |

The runtime compiles lookups and counts from this YAML — **do not** write raw SQL or SOQL in bindings.

## Step 5 — Map columns when names differ

CSV columns often do not match playbook field names. Use `fields` to map left (playbook) → right (physical):

```yaml
entities:
  crm_user:
    from: payroll.csv
    id: user_id
    fields:
      user_id: user
      full_name: full_name
```

Here the CSV column `user` stores the value for playbook field `user_id`.

## Step 6 — Validate the mapping

1. **Introspect** the source via MCP to confirm table and column names.
2. **Propose** binding YAML with `propose_binding` before saving.
3. **Test** with `test_binding` or a `query_graph` call:

```bash
curl -s http://127.0.0.1:8787/query \
  -H 'Content-Type: application/json' \
  -d '{
    "playbook_id": "simple-crm-access",
    "resolve": { "entity": "crm_user", "by_name": "Alex Anderson" },
    "count": { "relationship": "owns_account", "object_entity": "crm_account" }
  }'
```

A successful response proves entity mapping and the `owns_account` link column are correct.

## Mental model

```
Playbook                          Binding
────────                          ───────
crm_user          ──────────────► users table, user_id column
crm_account       ──────────────► accounts table, account_name column
owns_account      ──────────────► accounts.owner_user_id → users.user_id
```

Keep business language in the playbook. Keep schema details in bindings. When IT renames `owner_user_id`, update one binding file — not every agent prompt.

## See also

- [Playbook overview](index.md)
- [Role-Based Access Control](role-based-access-control.md)
- [Example 1: Simple CRM](example-1.md)
