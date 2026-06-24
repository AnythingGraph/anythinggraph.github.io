# Connect your data

After installation, connect your data sources with the CLI. We recommend `anythinggraph source add` for an interactive setup. You can also configure sources manually by editing `profiles/local.yaml` and `.env`.

## Use the CLI

Make sure Anything Graph is installed and running:

```bash
npm install -g @anythinggraph/cli@latest
anythinggraph onboard --install-daemon
anythinggraph start
```

### Add a source

Run the following command to connect a data source:

```bash
anythinggraph source add
```

The CLI lists all supported adapter types:

```
  1) PostgreSQL  (sql)
  2) MySQL / MariaDB  (mysql)
  3) SQL Server  (mssql)
  4) MongoDB  (mongodb)
  5) Salesforce  (soql)
  6) CSV file  (csv)
  7) REST / HTTP API  (rest)
```

Follow the prompts and enter your connection details. For PostgreSQL, a connection string looks like:

`postgres://username:password@localhost:5432/postgres`

After validation succeeds, you should see a message like:

```
✓ Found 12 table(s).
```

### List sources

To show all connected sources:

```bash
anythinggraph sources
```

Example output:

```
sources:
Profile: .anythinggraph/source/profiles/local.yaml

source_id   adapter  status
pg          sql      ok — Found 12 table(s).
payroll_csv csv      ok — Found 4 CSV column(s).
```

### Remove a source

To remove a connected source:

```bash
anythinggraph source remove
```

## Manual configuration

Register sources in `profiles/local.yaml` and set credentials in `.env`. Credentials should never go in playbooks or bindings. See [Connectors](../connectors/index.md) for supported adapter types and binding patterns.
