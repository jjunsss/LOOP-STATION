# Agent Roster

| agent | model/version | instruction profile | role | required | session | status | notes |
|---|---|---|---|---:|---:|---|---|
| `<agent>` | `<model/version or unknown>` | `<executor|supervisor|scientific reviewer|visual auditor|code auditor>` | `SUPERVISOR` | yes | 001 | `READY` |  |

## Flag Format

```text
{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}
```

## Signal Discovery

Canonical flag names are preferred, not the only discoverable signals. If names
vary, match by session, agent/role intent, status semantics, timestamp/order, and
linked artifact references. Record the matched source and why it was accepted.
Do not count ambiguous or artifactless signals as progress.

## Allowed Roles

- `SUPERVISOR`
- `EXECUTOR`
- `REVIEWER`
- `TESTER`

## Allowed Statuses

- `READY`
- `STANDBY`
- `RUNNING`
- `HEARTBEAT`
- `DONE`
- `BLOCKED`
- `ABSTAIN`
- `TIMEOUT`
- `VIOLATION`
- `REPAIRED`
