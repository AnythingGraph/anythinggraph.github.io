# Suggest a playbook structure

Turn raw schema into a **business vocabulary**: entity names, relationships, and routing across your connected sources. Your agent can draft this structure from introspection results — you review and refine before saving anything.

## Prerequisites

- [Explore your connected data](explore-connected-data.md) — know your tables, collections, or files
- Admin MCP token connected

## The goal

A playbook structure answers:

1. **Entities** — What are the business objects? (`crm_user`, `crm_account`, `crm_payroll_record`)
2. **Relationships** — How do they connect? (`owns_account`, `user_has_payroll`)
3. **Sources** — Where does each entity live? (`postgres`, `csv`, `salesforce`, …)

Bindings come later. At this stage, focus on **names and meaning**, not column mappings.

## Prompt your agent

```
I've connected warehouse_pg (Postgres) and payroll_csv (CSV).

Using anythinggraph-cli MCP:
1. introspect_source for both sources
2. Propose a playbook structure for sales users who own CRM accounts and have payroll history
3. Output compact playbook JSON with:
   - entities (identifier + attributes)
   - relationships (from / to)
   - sources map (which entity uses postgres vs csv)
4. Do NOT save yet — show me the draft for review
```

Adapt the domain in the prompt to your data — healthcare patients, support tickets, inventory SKUs, or whatever you connected.

## Use suggest_bindings

After you have a draft playbook id and entities, the agent can call **`suggest_bindings`** to heuristically map entities to tables or files:

```
For playbook_id my-crm-access:
- get_playbook_context
- suggest_bindings for source warehouse_pg
```

Review suggestions against what you saw during exploration. The agent should correct mappings where column names differ from playbook field names.

## Federated vs single-source

| Pattern | When to use |
|---------|-------------|
| **Single source** | All entities in one database — one source key (e.g. `postgres`) |
| **Federated** | Entities span systems — multiple source keys, multiple binding files |

Example federated `sources` map:

```json
"sources": {
  "crm_user": "postgres",
  "crm_account": "postgres",
  "crm_payroll_record": "csv"
}
```

See [Example 2: CRM + payroll](../playbooks/example-2.md) for a full federated walkthrough.

## Review checklist

Before moving on, confirm:

- [ ] Entity names are stable business terms, not raw table names
- [ ] Each entity has a clear `identifier` field
- [ ] Relationships are directional (`from` subject → `to` object)
- [ ] Every entity has a `sources` entry
- [ ] Scope is intentional — one playbook per access domain, not one giant graph

## Next step

Decide **who may see what** — [define access rules (ReBAC)](define-access-rules.md) before saving the playbook.
