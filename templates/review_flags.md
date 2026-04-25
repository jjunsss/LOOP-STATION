# Review Flags

| timestamp | flag | meaning | action |
|---|---|---|---|

## Session Flag Checklist

- [ ] `EXECUTOR-RUNNING` flag written before execution
- [ ] `EXECUTOR-DONE`, `EXECUTOR-BLOCKED`, or `EXECUTOR-ABSTAIN` terminal flag written after execution
- [ ] executor terminal flag points to readable result artifacts
- [ ] `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, or `SUPERVISOR-ABSTAIN` terminal flag written after `decision.md`
- [ ] supervisor terminal flag points to `decision.md` and the next action

## Sequential Gate Checklist

- [ ] `reviewer_requests.md` updated
- [ ] `SUPERVISOR-READY` flag written
- [ ] expected `flags/session_{NNN}/` path checked
- [ ] expected `reviews/session_{NNN}/` path checked
- [ ] `RUNNING`, `DONE`, `BLOCKED`, or `ABSTAIN` observed before start timeout, or timeout recorded
- [ ] terminal reviewer flag observed, or timeout policy applied
- [ ] `DONE` review artifact exists and is readable before counting it
- [ ] consumed reviewer artifacts recorded before `decision.md`

## Reviewer Standby Checklist

- [ ] active `loop_station/` folder resolved
- [ ] target session resolved
- [ ] persistent monitor/background watcher started when continuous waiting was requested and the environment supports it
- [ ] reviewer did not modify `FRAME.md`, `contract.json`, `agent_roster.md`, or executor-owned artifacts
- [ ] `EXECUTOR-DONE`, `EXECUTOR-BLOCKED`, or `EXECUTOR-ABSTAIN` observed before review
- [ ] `SUPERVISOR-READY` observed before normal review
- [ ] `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, or `SUPERVISOR-ABSTAIN` observed only for explicit post-decision audit
- [ ] linked artifacts verified readable before review
- [ ] standby interruption recorded as `REVIEWER-BLOCKED` instead of a premature review

## Timeout Records

- `REVIEW_START_TIMEOUT`:
- `REVIEW_HEARTBEAT_TIMEOUT`:
- `REVIEW_DONE_TIMEOUT`:
- `REVIEW_DONE_WITHOUT_ARTIFACT`:
