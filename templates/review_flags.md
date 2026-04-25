# Review Flags

| timestamp | flag | meaning | action |
|---|---|---|---|

## Sequential Gate Checklist

- [ ] `reviewer_requests.md` updated
- [ ] `SUPERVISOR-READY` flag written
- [ ] expected `flags/session_{NNN}/` path checked
- [ ] expected `reviews/session_{NNN}/` path checked
- [ ] `RUNNING`, `DONE`, `BLOCKED`, or `ABSTAIN` observed before start timeout, or timeout recorded
- [ ] terminal reviewer flag observed, or timeout policy applied
- [ ] `DONE` review artifact exists and is readable before counting it
- [ ] consumed reviewer artifacts recorded before `decision.md`

## Timeout Records

- `REVIEW_START_TIMEOUT`:
- `REVIEW_HEARTBEAT_TIMEOUT`:
- `REVIEW_DONE_TIMEOUT`:
- `REVIEW_DONE_WITHOUT_ARTIFACT`:
