# Agent Roster

| agent | role | required | session | status | notes |
|---|---|---:|---:|---|---|
| `<agent>` | `SUPERVISOR` | yes | 001 | `READY` |  |

## Flag Format

```text
{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}
```

## Allowed Roles

- `SUPERVISOR`
- `EXECUTOR`
- `ANALYST`
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
