# Salesforce

Connect Anything Graph to Salesforce objects via the REST API using the `soql` adapter. Bindings reference Salesforce object API names; SOQL is compiled at runtime.

## Quick connect

```bash
anythinggraph source add
```

Choose **Salesforce (`soql`)**, pick a profile name (for example `salesforce_main`), then enter:

- **Instance URL:** `https://your-instance.my.salesforce.com`
- **Access token:** OAuth bearer token with API access

To obtain a token for local development, use Salesforce Connected App OAuth or a session from your org's API tools. Store tokens only in `.env`, never in playbooks or bindings.

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  salesforce_main:
    adapter: soql
    instance_url: env:AG_SALESFORCE_MAIN_INSTANCE_URL
    auth: env:AG_SALESFORCE_MAIN_ACCESS_TOKEN
```

**Secrets** (`.env`):

```bash
AG_SF_INSTANCE_URL=https://your-instance.my.salesforce.com
AG_SF_ACCESS_TOKEN=your_oauth_access_token
```

The demo profile uses `AG_SF_INSTANCE_URL` and `AG_SF_ACCESS_TOKEN` for `salesforce_main`.

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key |
| `entities.*.from` | Salesforce **object API name** (e.g. `Account`, `Lead`) |
| `entities.*.fields` | Playbook field → Salesforce field API name |
| `relationships.*.link_column` | Lookup or reference field on the object |

Binding file suffix: `.salesforce.yaml` (for adapter `soql`)

Example:

```yaml
source_id: salesforce_main

entities:
  crm_user:
    from: User
    id: Id
    fields:
      user_id: Id
      full_name: Name

  crm_lead:
    from: Lead
    id: Id
    fields:
      lead_id: Id
      lead_name: Name

relationships:
  assigned_to:
    object: crm_lead
    link_column: OwnerId
```

**Do not put** instance URL, access token, or raw SOQL in binding files.

## Introspect (MCP)

```json
{ "source_id": "salesforce_main", "schema_name": "Account" }
```

`schema_name` optionally filters to a single Salesforce object API name. Omit it to describe all accessible objects.

## See also

- [Connectors overview](index.md)
- [Connect your data](../getting-started/connect-your-data.md)
