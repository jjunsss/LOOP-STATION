# Review Flags

| timestamp | flag | meaning | action |
|---|---|---|---|

## Session Flag Checklist

- Canonical lifecycle:
  1. `EXECUTOR-RUNNING`
  2. `EXECUTOR-DONE`
  3. internal validation / sub-agent checks when available
  4. `SUPERVISOR-READY` reviewer request
  5. optional `REVIEWER-STANDBY` while waiting before review-ready
  6. `REVIEWER-RUNNING`
  7. `REVIEWER-DONE`
  8. Codex checks flags and consumes review
  9. `SUPERVISOR-DONE`
  10. compact checkpoint when useful
  11. prepare next session
  12. next session `EXECUTOR-RUNNING`

  `SUPERVISOR-RUNNING` is the flag form of step 3.

- [ ] `EXECUTOR-RUNNING` flag written before execution
- [ ] `EXECUTOR-DONE`, `EXECUTOR-BLOCKED`, or `EXECUTOR-ABSTAIN` terminal flag written after execution
- [ ] executor terminal flag points to readable result artifacts
- [ ] `SUPERVISOR-RUNNING` flag written before Codex self-review/verification
- [ ] `supervisor_analysis.md` written after Codex self-review/verification
- [ ] `SUPERVISOR-READY` flag written after supervisor analysis and before external review
- [ ] `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, or `SUPERVISOR-ABSTAIN` terminal flag written after `decision.md` and summary updates
- [ ] supervisor terminal flag points to `decision.md` and the next action

## Sequential Gate Checklist

- [ ] `reviewer_requests.md` updated
- [ ] reviewer identity records agent, model/version or unknown, instruction profile, and `REVIEWER` role
- [ ] `SUPERVISOR-READY` flag written
- [ ] expected `flags/session_{NNN}/` path checked
- [ ] expected `reviews/session_{NNN}/` path checked
- [ ] `RUNNING`, `DONE`, `BLOCKED`, or `ABSTAIN` observed before start timeout, or timeout recorded per required reviewer
- [ ] pre-ready `STANDBY` was not counted as active review
- [ ] terminal reviewer flag observed, or timeout policy applied
- [ ] `DONE` review artifact exists and is readable before counting it
- [ ] reviewer summary updates completed or skipped with reason before `REVIEWER-DONE`
- [ ] required `REVIEWER-DONE` timestamp is earlier than `SUPERVISOR-DONE`
- [ ] consumed reviewer artifacts recorded before `decision.md`
- [ ] `summaries/ROLLING_SUMMARY.md` updated before supervisor terminal flag
- [ ] `summaries/SESSION_LEDGER.md` updated before supervisor terminal flag
- [ ] `summaries/COMPACT_HANDOFF.md` updated before supervisor terminal flag
- [ ] reviewer updated `summaries/LOG_TREND_SUMMARY.md` when reviewer could write files
- [ ] reviewer updated `summaries/EXECUTOR_BRIEF.md` when reviewer could write files
- [ ] next session not prepared before `SUPERVISOR-DONE`
- [ ] no next session slate or next `EXECUTOR-RUNNING` exists before reviewer consumption

## Compact Checklist

- [ ] compact happens only after terminal flags and summary updates
- [ ] `COMPACT_HANDOFF.md` lists current best, risks, budget, and exact next action
- [ ] `EXECUTOR_BRIEF.md` is sufficient for the executor to plan without rereading all raw logs
- [ ] `LOG_TREND_SUMMARY.md` lists which sessions/logs are already summarized
- [ ] reviewer compact notes updated when reviewer can write files
- [ ] resume read order recorded

## Protocol Violations

- `SUPERVISOR-DONE_BEFORE_REVIEWER-DONE`:
- `NEXT_SESSION_PREPARED_BEFORE_REVIEW_CONSUMED`:
- `SUPERVISOR-VIOLATION` flag:
- consumed late reviewer artifact:
- repair action:
- `SUPERVISOR-REPAIRED` flag:

## Reviewer Standby Checklist

- [ ] active `loop_station/` folder resolved
- [ ] target session resolved
- [ ] existing persistent monitor/background watcher checked before starting a new one
- [ ] persistent monitor/background watcher reused when possible, otherwise exactly one new watcher started
- [ ] duplicate monitors stopped, disabled, or recorded
- [ ] reviewer did not modify `FRAME.md`, `contract.json`, `agent_roster.md`, or executor-owned artifacts
- [ ] `EXECUTOR-DONE`, `EXECUTOR-BLOCKED`, or `EXECUTOR-ABSTAIN` observed before review
- [ ] `SUPERVISOR-READY` observed before normal review
- [ ] `supervisor_analysis.md` verified readable before normal review
- [ ] `REVIEWER-RUNNING` written only after review-ready flags and linked artifacts were readable
- [ ] `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, or `SUPERVISOR-ABSTAIN` observed only for explicit post-decision audit
- [ ] linked artifacts verified readable before review
- [ ] standby interruption recorded as `REVIEWER-BLOCKED` instead of a premature review
- [ ] next-session monitor setup checked before `REVIEWER-DONE`

## Timeout Records

- `REVIEW_START_TIMEOUT:{reviewer}`:
- `REVIEW_HEARTBEAT_TIMEOUT:{reviewer}`:
- `REVIEW_DONE_TIMEOUT:{reviewer}`:
- `REVIEW_DONE_WITHOUT_ARTIFACT:{reviewer}`:
