# Reviewer Requests

| reviewer | role | required | reason | artifacts to read | expected review output | expected terminal flag | timeout |
|---|---|---:|---|---|---|---|---|

## Request Notes

- 

## Sequential Gate

- Supervisor ready flag:
- Start timeout:
- Done timeout:
- Heartbeat timeout:
- Required terminal flags before decision:
- Optional terminal flags to consume if available:

## Reviewer Standby Instructions

- Active session:
- Continuous wait requested: yes|no
- Persistent monitor/background watcher:
  - tool:
  - poll interval:
  - ready signal:
- Review starts only after:
  - [ ] executor terminal flag observed
  - [ ] `SUPERVISOR-READY` observed
  - [ ] `supervisor_analysis.md` is readable
  - [ ] linked executor artifacts are readable
  - [ ] linked decision artifacts are readable only for explicit post-decision audit
- Poll paths:
  - flags:
  - reviews:
  - session artifacts:
- If interrupted, write `REVIEWER-BLOCKED` with the last observed flags and do not write a review.
