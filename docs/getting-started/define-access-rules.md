# Define access rules (ReBAC)

**ReBAC** (relationship-based access control) defines your AI policy in the playbook itself: which resources a subject may read, based on declared relationships — not instructions buried in a system prompt.

## Why policy belongs in the playbook

| Prompt-only policy | Playbook ReBAC |
|--------------------|----------------|
| "Only show the user's own data" | Enforced on every `query_graph` call |
| Easy to bypass with rephrasing | Runtime filters before results return |
| Inconsistent across agents | Same rules for Cursor, Claude, and API clients |
| Hard to audit | Proof envelope shows authorized paths |

## Prerequisites

- A draft playbook structure with entities and relationships — see [Suggest a playbook structure](suggest-playbook-structure.md)

## The access block

Add an `access` section to your playbook JSON:

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

| Field | Meaning |
|-------|---------|
| `subject` | Entity representing the person or actor asking |
| `subject_id` | Field used to resolve the subject (e.g. after lookup by name) |
| `allow[]` | Each entry grants read access to a `resource` via a `relationship` |
| `summary` | Human-readable description for reviewers |

## Step by step

### 1. Pick the subject

Who is asking? Usually one entity type:

```json
"subject": "crm_user",
"subject_id": "user_id"
```

When Alex asks a question, the runtime resolves Alex → `user_id: 42`. All access checks use that id.

### 2. Model real relationships

Each `allow` rule ties a relationship to a resource the subject may read:

```json
{ "relationship": "owns_account", "resource": "crm_account" }
```

**Plain language:** A `crm_user` may read `crm_account` rows linked through `owns_account`.

The binding supplies the physical foreign key (`owner_user_id` on `accounts`). The playbook supplies the policy.

### 3. Add rules per resource type

Grant payroll access separately:

```json
{ "relationship": "user_has_payroll", "resource": "crm_payroll_record" }
```

Alex sees their payroll rows and their accounts — not other users' data.

### 4. Validate with your agent

```
Review the access block for playbook crm-payroll-access:
- Can a user read another user's payroll? (should be no)
- Can a user read accounts they do not own? (should be no)
- List which relationships authorize which resources
```

## Prompt examples by domain

**Sales CRM:**

```
Subject is crm_user. Allow read on crm_account via owns_account.
Allow read on crm_lead via assigned_to.
```

**Support:**

```
Subject is support_agent. Allow read on support_ticket via assigned_to.
Allow read on customer via handles_customer.
```

Start from how access works in your org today — ownership, assignment, team membership — and encode it as relationships.

## Deeper reference

- [Role-Based Access Control](../playbooks/role-based-access-control.md) — full ReBAC guide with examples
- [Example 1: Simple CRM](../playbooks/example-1.md) — single allow rule

## Next step

With structure and policy defined, [create the playbook with your agent](create-playbook-with-agent.md) — validate, test bindings, and save.
