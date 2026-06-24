# CSV

Connect Anything Graph to local CSV or flat files using the `csv` adapter. Column headers are read from the file at introspection time — no database server required.

## Quick connect

```bash
anythinggraph source add
```

Choose **CSV file (`csv`)**, pick a profile name (for example `payroll_csv`), and enter the absolute or relative path to your file:

```
./data/payroll.csv
```

Use an absolute path if the reasoning service runs from a different working directory.

## Manual setup

**Profile** (`profiles/local.yaml`):

```yaml
sources:
  payroll_csv:
    adapter: csv
    file_path: env:AG_PAYROLL_CSV_PATH
```

**Secrets** (`.env`):

```bash
AG_PAYROLL_CSV_PATH=./data/payroll.csv
```

`start-all.sh` defaults to `./data/payroll.csv` when this variable is unset.

## Binding YAML

| Field | Meaning |
|-------|---------|
| `source_id` | Profile key, e.g. `payroll_csv` |
| `entities.*.from` | CSV **filename** (e.g. `payroll.csv`) |
| `entities.*.fields` | Playbook field → CSV column when names differ |

Binding file suffix: `.csv.yaml`

Example (federated subject + payroll rows from one file):

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

**Do not put** the file path in binding YAML — the path belongs in the profile only.

## Introspect (MCP)

```json
{ "source_id": "payroll_csv" }
```

No `schema_name` is required. Introspection returns column headers from the file.

## See also

- [Connectors overview](index.md)
- [Connect your data](../getting-started/connect-your-data.md)
