# Example 1: Simple CRM access

A minimal playbook: one Postgres source, two entities, one relationship, and one access rule.

**Use case:** A sales rep asks their agent *"How many accounts do I own?"* and only sees accounts linked to their `user_id`.

## Files

| File | Role |
|------|------|
| `playbooks/simple-crm-access.json` | Entities, relationship, routing, access |
| `bindings/simple-crm-access.postgres.yaml` | Maps to `users` and `accounts` tables |
| `profiles/local.yaml` | Postgres credentials (`warehouse_pg`) |

## Step 1 — Connect Postgres

```bash
anythinggraph source add
```

Choose PostgreSQL and register `warehouse_pg`, or set manually:

```yaml
# profiles/local.yaml
sources:
  warehouse_pg:
    adapter: sql
    dsn: env:AG_SQL_DSN
```

## Step 2 — Write the playbook

```json
{
  "id": "simple-crm-access",
  "name": "Simple CRM access",
  "description": "Sales rep sees accounts they own in Postgres.",
  "entities": {
    "crm_user": {
      "identifier": "user_id",
      "attributes": ["full_name"]
    },
    "crm_account": {
      "identifier": "account_name",
      "attributes": ["industry"]
    }
  },
  "relationships": {
    "owns_account": { "from": "crm_user", "to": "crm_account" }
  },
  "sources": {
    "crm_user": "postgres",
    "crm_account": "postgres"
  },
  "access": {
    "summary": "CRM users read accounts they own.",
    "subject": "crm_user",
    "subject_id": "user_id",
    "allow": [
      { "relationship": "owns_account", "resource": "crm_account" }
    ]
  }
}
```

Save to `playbooks/simple-crm-access.json`.

## Step 3 — Write the binding

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

Save to `bindings/simple-crm-access.postgres.yaml`.

`owner_user_id` on `accounts` is the foreign key that implements `owns_account`.

## Step 4 — Test the query

```bash
curl -s http://127.0.0.1:8787/query \
  -H 'Content-Type: application/json' \
  -d '{
    "playbook_id": "simple-crm-access",
    "resolve": { "entity": "crm_user", "by_name": "Alex Anderson" },
    "count": { "relationship": "owns_account", "object_entity": "crm_account" }
  }'
```

Expected behavior: returns a count of accounts where `owner_user_id` matches Alex's `user_id`, with structured proof.

## Step 5 — Ask via MCP

With Cursor or Claude connected to Anything Graph MCP, try:

- *Who is Alex Anderson?*
- *How many accounts does Alex Anderson own?*
- *List industries for accounts Alex owns.*

The agent uses playbook vocabulary — not ad-hoc SQL.

## What you learned

- One source key (`postgres`) → one binding file.
- Entities define **what**; bindings define **where**.
- ReBAC limits results to rows linked through `owns_account`.

## Next

- [Example 2: CRM + payroll](example-2.md) — add a second source (CSV)
- [Entity mapping and relationships](entity-mapping.md)
