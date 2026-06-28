# Getting Started

Anything Graph is a data governance layer between your AI agents and data sources. It applies an ontology on top of your existing schema—without moving or copying your data.

## Key features

- **Policy as code** — define and version access rules in code, so governance stays consistent across environments and teams.
- **Entity and relationship mapping** — model your data as entities and relationships, so agents and applications work with a clear graph of how your data connects.
- **ReBAC (relationship-based access control)** — grant access based on how users relate to resources (for example, team membership or ownership), not just static roles.

## Installation

The recommended path is the **npm CLI** — it clones [anything-cli](https://github.com/AnythingGraph/AnythingGraph) to `~/.anythinggraph/source`, builds the Rust reasoning service, and starts MCP on port `3334`. You do **not** need to manually run `cargo build` for a normal install.

### Option A — npm CLI (recommended)

**Requirements:** Node.js 18+, `git`, and Rust `cargo` (used once during onboard to compile reasoning-service).

```bash
npm install -g @anythinggraph/cli@latest
anythinggraph onboard --install-daemon
anythinggraph start
```

What this does:

- `onboard` — clones or updates the repo under `~/.anythinggraph/source` and builds binaries into `~/.anythinggraph/bin/`
- `--install-daemon` — optional macOS launchd / Linux systemd user service so services restart on login
- `start` — runs Anthing Graph locally

the same stack as `./start-all.sh` (reasoning API + MCP HTTP)

Next - See [Connect your data](connect-your-data.md) to configure credentials and environment variables.

**Useful CLI commands:**

| Command | Purpose |
|---------|---------|
| `anythinggraph status` | Show service URLs and health |
| `anythinggraph doctor` | Check prerequisites and service health |
| `anythinggraph stop` | Stop supervised services |
| `anythinggraph mcp print-config` | Print Cursor MCP JSON snippet |
| `anythinggraph mcp print-config --target claude` | Claude Desktop `mcp-remote` config |

Bootstrap without npm (alternative):

```bash
curl -fsSL https://raw.githubusercontent.com/AnythingGraph/AnythingGraph/main/Sources/OSS/cli/install.sh | bash
```

### Option B — clone from source (contributors)

For local development or contributing to the repo:

```bash
git clone https://github.com/AnythingGraph/AnythingGraph.git
cd anything-cli

cp .env.example .env
# edit connection strings — see Connect your data for details

chmod +x start-all.sh
./start-all.sh
```

`start-all.sh` stops any existing processes on ports `8787` and `3334`, loads `.env` when present, sets `AG_AUTH_DISABLED=1` for local dev, and starts both services. Press **Ctrl+C** to stop.

Manual build (only if you are hacking on Rust without the npm CLI):

```bash
cargo build --release
cd mcp && npm install && cd ..
```

### Services

| Service | URL |
|---------|-----|
| Reasoning API | `http://127.0.0.1:8787` |
| MCP (Cursor / Claude) | `http://127.0.0.1:3334/mcp` |

!!! note "Not included"
    Anything CLI does not install or start the legacy OSS dashboard, RDF cache, or `mcp-service` on port `3333`. New integrations should use **anythinggraph-cli** MCP on port `3334`.

## Next in this section

- [Connect your data](connect-your-data.md) — configure credentials and data sources
- [Connect your own agent](connect-your-own-agent.md) — connect Cursor or Claude via MCP

## Next steps

- [MCP Setup](../setup/index.md) — configure MCP for your agent
