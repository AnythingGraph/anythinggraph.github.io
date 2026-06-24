# REST / HTTP API

Connect Anything Graph to HTTP JSON APIs using the `rest` adapter. Bindings reference resource paths; the runtime compiles `GET` request templates. OpenAPI-based discovery is supported for catalog introspection.

## Quick connect

```bash
anythinggraph source add
```

Choose **REST / HTTP API (`rest`)**, pick a profile name (for example `api_main`), then enter:

- **Base URL:** `https://api.example.com`
- **Bearer token:** optional — press Enter to skip for public APIs

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  api_main:
    adapter: rest
    base_url: env:AG_API_MAIN_BASE_URL
    auth: env:AG_API_MAIN_TOKEN
```

Omit `auth` if the API does not require a bearer token.

**Secrets** (`.env`):

```bash
AG_REST_BASE_URL=https://api.example.com
AG_REST_TOKEN=your_bearer_token
```

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key |
| `entities.*.from` | REST **resource path** (e.g. `/users`, `/accounts`) |
| `entities.*.fields` | Playbook field → JSON response field name |
| `relationships.*.link_column` | Query parameter or JSON field linking subject to object |

Binding file suffix: `.rest.yaml`

Example:

```yaml
source_id: api_main

entities:
  crm_user:
    from: /users
    id: id
    fields:
      user_id: id
      full_name: name

  crm_account:
    from: /accounts
    id: id
    fields:
      account_name: name

relationships:
  owns_account:
    object: crm_account
    link_column: owner_id
```

**Do not put** `base_url` or hand-written GET strings in binding YAML — those come from the profile and compiler.

## Introspect (MCP)

```json
{ "source_id": "api_main", "schema_name": "/users" }
```

`schema_name` optionally hints at a resource path prefix for catalog discovery.

## See also

- [Connectors overview](index.md)
- [Connect your data](../getting-started/connect-your-data.md)
