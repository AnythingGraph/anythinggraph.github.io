# Role-Based Access Control

Anything Graph uses **ReBAC** (relationship-based access control): access is granted based on how a **subject** relates to **resources**, not static role labels in a prompt.

Rules live in the playbook `access` block. The runtime enforces them on every query — agents cannot bypass them by rephrasing the question.

## Step 1 — Choose the subject entity

The subject is *who is asking*. Usually a person or service account modeled as an entity.

```json
"access": {
  "subject": "crm_user",
  "subject_id": "user_id"
}
```

When a query runs, the runtime resolves the subject to a concrete id (e.g. Alex Anderson → `user_id: 42`). All access checks use that id.

## Step 2 — Declare allowed relationships

Each `allow` entry grants read access to a **resource entity** when the subject is linked through a named **relationship**.

```json
"allow": [
  { "relationship": "owns_account", "resource": "crm_account" }
]
```

| Field | Meaning |
|-------|---------|
| `relationship` | Must match a key in `relationships` |
| `resource` | The `to` entity users may read when the relationship holds |

**Rule in plain language:** A `crm_user` may read `crm_account` rows where `owns_account` links that user to the account.

## Step 3 — Walk through an example

**Playbook excerpt** (`simple-crm-access`):

```json
{
  "relationships": {
    "owns_account": { "from": "crm_user", "to": "crm_account" }
  },
  "access": {
    "summary": "CRM users read accounts they own.",
    "subject": "crm_user",
    "subject_id": "user_id",
    "allow": [
      { "relationship": "owns_account", "resource": "crm_account" }
    ]
  }
}
```

**Binding link** (`accounts.owner_user_id` → `users.user_id`):

```yaml
relationships:
  owns_account:
    object: crm_account
    link_column: owner_user_id
```

**What happens when Alex asks *"List my accounts"*:**
1. Resolve Alex → `crm_user` with `user_id = 42`.
2. Check `allow`: subject `crm_user` may read `crm_account` via `owns_account`.
3. Query only accounts where `owner_user_id = 42`.
4. Return those rows with proof metadata.

Alex cannot see accounts owned by other users — the filter is structural, not prompt-based.

## Step 4 — Multiple resources

Grant access to more than one resource type by adding `allow` entries:

```json
"allow": [
  { "relationship": "owns_account", "resource": "crm_account" },
  { "relationship": "user_has_payroll", "resource": "crm_payroll_record" }
]
```

From `crm-payroll-access`:

- Sales users see **accounts they own** (Postgres).
- The same users see **their own payroll rows** (CSV).
- They do **not** see other users' payroll — `user_has_payroll` links only their rows.

```json
"access": {
  "summary": "CRM users read accounts they own and their own payroll records.",
  "subject": "crm_user",
  "subject_id": "user_id",
  "allow": [
    { "relationship": "owns_account", "resource": "crm_account" },
    { "relationship": "user_has_payroll", "resource": "crm_payroll_record" }
  ]
}
```

## Step 5 — ReBAC vs prompt instructions

| Prompt-only access control | Playbook ReBAC |
|----------------------------|----------------|
| "Only show the user's own data" in system prompt | Enforced by runtime on every query |
| Easy to bypass with creative phrasing | Filter applied before results return |
| Hard to audit | Proof shows which relationship authorized access |
| Breaks when team changes prompts | One playbook, consistent across agents |

## Step 6 — Design tips

1. **Model real relationships** — ownership, assignment, team membership map cleanly to `relationships` + `allow`.
2. **One subject per playbook** — if multiple actor types need different rules, consider separate playbooks or entities.
3. **Name relationships clearly** — `assigned_to`, `owns_account`, and `user_has_payroll` read better in audits than `rel_1`.
4. **Use `summary`** — a short human-readable description helps reviewers understand intent.

## See also

- [Entity mapping and relationships](entity-mapping.md)
- [Example 1: Simple CRM](example-1.md) — single allow rule
- [Example 2: CRM + payroll](example-2.md) — multiple resources across sources
