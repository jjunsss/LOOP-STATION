# Agent Roster

| agent | model/version | instruction profile | role | required | session | status | notes |
|---|---|---|---|---:|---:|---|---|
| `<agent>` | `<model/version or unknown>` | `<executor|supervisor|scientific reviewer|visual auditor|code auditor>` | `SUPERVISOR` | yes | 001 | `READY` |  |

## Flag Format

```text
{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}
```

## Allowed Roles

- `SUPERVISOR`
- `EXECUTOR`
- `REVIEWER`
- `TESTER`

## Allowed Statuses

- `READY`
- `RUNNING`
- `HEARTBEAT`
- `DONE`
- `BLOCKED`
- `ABSTAIN`
- `TIMEOUT`
- `VIOLATION`
- `REPAIRED`
