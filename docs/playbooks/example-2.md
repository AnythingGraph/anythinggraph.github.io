# Example 2: CRM + payroll access

A federated playbook: CRM data in **Postgres**, payroll history in a **CSV** file. One business vocabulary, two bindings, cross-source access rules.

**Use case:** A sales manager asks *"Show payroll for users who own enterprise accounts"* — the agent traverses Postgres and CSV through one playbook.

## Files

| File | Role |
|------|------|
| `playbooks/crm-payroll-access.json` | Three entities, two relationships, two source keys |
| `bindings/crm-payroll-access.postgres.yaml` | `crm_user`, `crm_account` → Postgres |
| `bindings/crm-payroll-access.csv.yaml` | `crm_payroll_record` → CSV |
| `profiles/local.yaml` | `warehouse_pg` + `payroll_csv` |

## Step 1 — Connect both sources

```bash
anythinggraph source add   # PostgreSQL → warehouse_pg
anythinggraph source add   # CSV → payroll_csv
```

Profile excerpt:

```yaml
sources:
  warehouse_pg:
    adapter: sql
    dsn: env:AG_SQL_DSN
  payroll_csv:
    adapter: csv
    file_path: env:AG_PAYROLL_CSV_PATH
```

## Step 2 — Write the playbook

```json
{
  "id": "crm-payroll-access",
  "name": "CRM + payroll access",
  "description": "Users and accounts in Postgres; payroll in CSV.",
  "entities": {
    "crm_user": {
      "identifier": "user_id",
      "attributes": ["full_name"]
    },
    "crm_account": {
      "identifier": "account_name",
      "attributes": ["industry"]
    },
    "crm_payroll_record": {
      "identifier": "payroll_id",
      "attributes": ["user_id", "pay_period", "gross_pay", "currency", "pay_date"]
    }
  },
  "relationships": {
    "owns_account": { "from": "crm_user", "to": "crm_account" },
    "user_has_payroll": { "from": "crm_user", "to": "crm_payroll_record" }
  },
  "sources": {
    "crm_user": "postgres",
    "crm_account": "postgres",
    "crm_payroll_record": "csv"
  },
  "access": {
    "summary": "CRM users read accounts they own and their own payroll records.",
    "subject": "crm_user",
    "subject_id": "user_id",
    "allow": [
      { "relationship": "owns_account", "resource": "crm_account" },
      { "relationship": "user_has_payroll", "resource": "crm_payroll_record" }
    ]
  }
}
```

Notice `sources`: two entities share `postgres`, one uses `csv` → **two binding files**.

## Step 3 — Postgres binding

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

Save to `bindings/crm-payroll-access.postgres.yaml`.

## Step 4 — CSV binding

The same CSV file can back multiple entities. Map column names where they differ from playbook fields:

```yaml
source_id: payroll_csv

entities:
  crm_user:
    from: payroll.csv
    id: user_id
    fields:
      user_id: user
      full_name: full_name

  crm_payroll_record:
    from: payroll.csv
    id: payroll_id
    fields:
      user_id: user

relationships:
  user_has_payroll:
    object: crm_payroll_record
    link_column: user
```

Save to `bindings/crm-payroll-access.csv.yaml`.

Including `crm_user` in the CSV binding lets the runtime resolve subjects from payroll data when needed for federated queries.

## Step 5 — How routing works

| Entity | Source key | Binding file |
|--------|------------|--------------|
| `crm_user` | `postgres` | `crm-payroll-access.postgres.yaml` |
| `crm_account` | `postgres` | `crm-payroll-access.postgres.yaml` |
| `crm_payroll_record` | `csv` | `crm-payroll-access.csv.yaml` |

The runtime picks the binding from the entity's `sources` entry — no need to pass `binding_name` on queries.

## Step 6 — Test queries

**Count owned accounts (Postgres):**

```bash
curl -s http://127.0.0.1:8787/query \
  -H 'Content-Type: application/json' \
  -d '{
    "playbook_id": "crm-payroll-access",
    "resolve": { "entity": "crm_user", "by_name": "Alex Anderson" },
    "count": { "relationship": "owns_account", "object_entity": "crm_account" }
  }'
```

**Count payroll records (CSV):**

```bash
curl -s http://127.0.0.1:8787/query \
  -H 'Content-Type: application/json' \
  -d '{
    "playbook_id": "crm-payroll-access",
    "resolve": { "entity": "crm_user", "by_name": "Alex Anderson" },
    "count": { "relationship": "user_has_payroll", "object_entity": "crm_payroll_record" }
  }'
```

## Step 7 — Ask via MCP

Example prompts:

- *How many accounts does Alex Anderson own?*
- *How many payroll records does Alex Anderson have?*
- *What is Alex's total gross pay this year?*

Access rules ensure Alex only sees their own payroll rows and accounts they own.

## What you learned

- One playbook can federate multiple sources with the `sources` map.
- One binding file per **source key**, not per entity.
- ReBAC can span sources — relationships authorize access regardless of where data lives.

## Next

- [Example 1: Simple CRM](example-1.md) — start smaller if this feels complex
- [Connectors](../connectors/index.md) — Postgres and CSV setup details
