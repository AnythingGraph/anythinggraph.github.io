# Connect your own agent

Connect Cursor, Claude Desktop, or another MCP client to Anything Graph so your agent can query your data through the reasoning layer — not with raw database access.

The MCP server at `http://127.0.0.1:3334/mcp` is a thin bridge to the Rust reasoning service at `http://127.0.0.1:8787`. Agents call MCP tools; MCP forwards requests to the reasoning API.

## Before you connect

Make sure Anything Graph is running:

```bash
anythinggraph start
```

Check that both services are up:

```bash
anythinggraph status
```

| Service | URL |
|---------|-----|
| Reasoning API | `http://127.0.0.1:8787` |
| MCP (Cursor / Claude) | `http://127.0.0.1:3334/mcp` |

If you have not connected data sources yet, complete [Connect your data](connect-your-data.md) first.

## Connect in Cursor

1. Run `anythinggraph start`.
2. Open **Cursor → Settings → MCP**.
3. Add the MCP server URL: `http://127.0.0.1:3334/mcp`.
4. Or run `anythinggraph mcp print-config` and paste the generated JSON snippet.

Example Cursor MCP config (local dev, no auth):

```json
{
  "mcpServers": {
    "anythinggraph-cli": {
      "url": "http://127.0.0.1:3334/mcp"
    }
  }
}
```

When auth is enabled, add an `Authorization` header (admin for authoring, user for query-only):

```json
{
  "mcpServers": {
    "anythinggraph-cli": {
      "url": "http://127.0.0.1:3334/mcp",
      "headers": {
        "Authorization": "Bearer admin-secret-change-me"
      }
    }
  }
}
```

Print a ready-to-paste config from the CLI:

```bash
anythinggraph mcp print-config
```

## Connect in Claude Desktop

Use the Claude-specific config snippet:

```bash
anythinggraph mcp print-config --target claude
```

Paste the output into your Claude Desktop MCP configuration.

## Verify the connection

In your agent, ask a simple health or discovery question:

```
Using anythinggraph-cli MCP: list available playbooks and run a health check.
```

Typical query flow for everyday use:

1. `list_playbooks` — see loaded playbook ids
2. `get_playbook_context` — read entities and relationships for a playbook
3. `query_graph` — ask a business question (plan + execute + proof)

Example prompt (replace the playbook id and names with yours):

```
For playbook crm-payroll-access: how many accounts does Alex Anderson own?
```

!!! note "Read-only by design"
    Live data sources accept SELECT-style reads only. MCP has no insert, update, or delete tools.

## MCP tool reference

The **anythinggraph-cli** MCP server (`http://127.0.0.1:3334/mcp`) exposes the tools below. Which tools your agent can call depends on the bearer token role (**user** or **admin**).

### User tools

| Tool | Purpose |
|------|---------|
| `health_check` | Ping the Rust reasoning service |
| `list_playbooks` | List playbook ids loaded from `playbooks/` |
| `get_playbook_context` | Load playbook schema summary (entities and relationships) |
| `list_entity` | List rows for a playbook entity (bounded browse; default limit 1000) |
| `sample_entity` | Return a small sample of rows for a playbook entity (default limit 5) |
| `plan_query` | Compile a structured federated query into plan IR |
| `execute_plan` | Execute a compiled plan IR via read-only adapters |
| `query_graph` | Compile and execute a federated read query in one step (proof envelope) |
| `list_allowed_rows` | List row identifiers a subject may read under enforced ReBAC rules |

### Admin tools

Admin tokens include all user tools **plus**:

| Tool | Purpose |
|------|---------|
| `list_sources` | List configured data sources from `profiles/local.yaml` (no secrets) |
| `get_adapter_guide` | Per-adapter binding authoring guide for a `source_id` — call before `propose_binding` |
| `list_bindings` | List loaded binding file stems in `bindings/` |
| `get_binding` | Load one saved binding YAML by stem |
| `introspect_source` | Read source schema for agent mapping (tables/columns — read-only) |
| `sample_source` | Read a few raw rows from a source table/collection/object (no playbook required) |
| `suggest_bindings` | Suggest playbook entity-to-table mappings from introspected schema |
| `propose_binding` | Validate declarative binding YAML (no SQL) |
| `test_binding` | Compile a sample query against a proposed or saved binding; optionally execute read-only |
| `save_binding` | Save declarative binding YAML to `bindings/{playbook_id}.{adapter_suffix}.yaml` |
| `propose_playbook` | Validate compact playbook JSON |
| `save_playbook` | Save compact playbook JSON to `playbooks/{playbook_id}.json` |

!!! note "Profiles are manual"
    Profiles are never written via MCP — edit `profiles/local.yaml` yourself. Admin agents use `list_sources` → `get_adapter_guide` → `introspect_source` before authoring bindings.

## Next steps

Follow the playbook authoring path:

1. [Explore your connected data](explore-connected-data.md)
2. [Suggest a playbook structure](suggest-playbook-structure.md)
3. [Define access rules (ReBAC)](define-access-rules.md)
4. [Create a playbook with your agent](create-playbook-with-agent.md)
5. [Govern your agent with playbooks](govern-agent-with-playbooks.md)
