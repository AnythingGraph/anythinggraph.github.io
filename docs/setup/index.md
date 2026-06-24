# MCP Setup

Advanced MCP configuration: auth tokens, environment variables, and role-based tool access.

For step-by-step agent connection in Cursor or Claude, start with [Connect your own agent](../getting-started/connect-your-own-agent.md).

## Configuration file

Create a config file at `~/.anythinggraph/config.yml`:

```yaml
api_url: https://api.anythinggraph.io
default_project: my-project
```

## Environment variables

| Variable | Description |
|----------|-------------|
| `ANYTHINGGRAPH_API_KEY` | Your API key for authentication |
| `ANYTHINGGRAPH_PROJECT` | Default project ID |
| `AG_MCP_PORT` | MCP HTTP port (default `3334`) |
| `AG_MCP_AUTH_TOKEN` | Default token for stdio MCP (optional) |
| `AG_REASONING_URL` | MCP → reasoning API (default `http://127.0.0.1:8787`) |

## Verify setup

```bash
anythinggraph auth status
```

If authentication succeeds, you are ready to [connect your agent](../getting-started/connect-your-own-agent.md).
