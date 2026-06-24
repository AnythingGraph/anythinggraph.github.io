# Govern your agent with playbooks

A saved playbook turns open-ended data access into **scoped, auditable queries**. Agents ask business questions; Anything Graph enforces vocabulary, routing, and ReBAC — every time.

## What changes after you ship a playbook

| Before playbook | After playbook |
|-----------------|----------------|
| Schema stuffed into prompts | Stable entity names (`crm_user`, `owns_account`) |
| Agent invents SQL from chat | Governed `query_graph` against live sources |
| Access rules in prompt text | ReBAC enforced by the runtime |
| "Trust me" answers | Structured proof — playbook, plan, source rows |

## Prerequisites

- A saved playbook and bindings — [Create a playbook with your agent](create-playbook-with-agent.md)
- Agent connected to MCP — [Connect your own agent](connect-your-own-agent.md)

## Scope queries to one playbook

Instruct your agent to stay within a playbook id:

```
You are querying playbook crm-payroll-access only.
Use query_graph for all data questions — do not write raw SQL.
Before counting or listing, resolve the subject crm_user by name when needed.
```

Every `query_graph` call:

1. Loads playbook entities and relationships
2. Resolves the subject and applies `access` rules
3. Routes each entity to the correct binding and source
4. Returns a proof envelope

## User vs admin tokens

| Token role | Typical use |
|------------|-------------|
| **User** | Production agents — `query_graph`, `list_entity`, `sample_entity`, `list_allowed_rows` only |
| **Admin** | Authoring — introspect, propose/save playbooks and bindings |

For governed production use, give agents a **user** token. They can query within playbooks but cannot change policy or bindings.

Example Cursor config with user token:

```json
{
  "mcpServers": {
    "anythinggraph-cli": {
      "url": "http://127.0.0.1:3334/mcp",
      "headers": {
        "Authorization": "Bearer user-secret-change-me"
      }
    }
  }
}
```

## Everyday query tools

| Tool | Use when |
|------|----------|
| `list_playbooks` | Discover which playbooks are loaded |
| `get_playbook_context` | Show entities and relationships before querying |
| `query_graph` | Resolve subject, count or list via relationships |
| `list_allowed_rows` | Audit which row ids a subject may read under ReBAC |
| `list_entity` / `sample_entity` | Bounded browse within an entity (still playbook-scoped) |

## Example prompts

**Count:**

```
For playbook crm-payroll-access: how many accounts does Alex Anderson own?
```

**List:**

```
For playbook crm-payroll-access: list industries for accounts Alex owns.
```

**Cross-source:**

```
For playbook crm-payroll-access: how many payroll records does Alex Anderson have?
```

**Audit access:**

```
For playbook crm-payroll-access and subject Alex Anderson:
list_allowed_rows for crm_account and crm_payroll_record.
```

## Operating model

1. **IT / data** — Connect sources, maintain profiles and bindings
2. **Policy owners** — Define playbook entities, relationships, and ReBAC
3. **End users** — Ask natural-language questions through agents scoped to approved playbooks

When schema changes, update bindings — not every agent prompt. When policy changes, update the playbook `access` block and redeploy.

## Validate governance

Confirm your agent cannot bypass the playbook:

```
Try to query payroll for a user other than the resolved subject.
ReBAC should block or filter unauthorized rows.
```

Check the proof envelope on each `query_graph` response — it should cite the playbook id, resolved subject, and relationship used.

## Go further

- [Playbooks overview](../playbooks/index.md)
- [Role-Based Access Control](../playbooks/role-based-access-control.md)
- [Connectors](../connectors/index.md) — add more data sources as your ontology grows
