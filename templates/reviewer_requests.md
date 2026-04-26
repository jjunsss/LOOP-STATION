# Reviewer Requests

| reviewer | model/version | instruction profile | role | required | reason | artifacts to read | expected review output | expected terminal flag | timeout |
|---|---|---|---|---:|---|---|---|---|---|

## Request Notes

- 

## Reviewer Identity

- reviewer agent:
- model/version:
- instruction profile:
- must act as: `REVIEWER`
- must not act as: `EXECUTOR` or `SUPERVISOR` unless explicitly reassigned

## Suggested Audit Axes

- historical trend/log digest:
  - prior sessions to summarize:
  - logs to inspect:
  - summaries to update:
    - `summaries/LOG_TREND_SUMMARY.md`
    - `summaries/EXECUTOR_BRIEF.md`
  - raw logs the executor should not need to reread:
- visual image check:
  - result images / panels:
  - crops or regions:
  - current-best comparison:
- metric audit:
  - metrics:
  - anchors:
  - confounds:
- scientific / literature check:
  - failure explanation to challenge:
  - reviewer may decide online search is needed: yes|no
  - papers, methods, docs, or prior art to search or discover:
  - sources that would change the next decision:
  - falsifiable mechanism to test next:
- code/config audit:
  - generated code:
  - configs:
  - manifests / diffs:
  - original protection / backups / restore manifest:
- log/artifact audit:
  - commands:
  - resource usage:
  - missing or stale artifacts:
- sub-agent decomposition:
  - visual reviewer:
  - metric/log reviewer:
  - code/config reviewer:
  - artifact completeness reviewer:

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
  - existing monitor checked before starting a new one: yes|no
  - dedupe key, such as loop root + reviewer:
  - reused monitor id/name:
  - duplicate monitors found:
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
- Before terminal review completion, update token-saving summaries when file writes are available:
  - [ ] `summaries/LOG_TREND_SUMMARY.md`
  - [ ] `summaries/EXECUTOR_BRIEF.md`
  - [ ] `summaries/REVIEWER_ROLLUP.md`
- Before terminal review completion, audit next-session monitor setup:
  - [ ] existing monitor reused or retargeted when possible
  - [ ] duplicate monitors stopped, disabled, or recorded
- If interrupted, write `REVIEWER-BLOCKED` with the last observed flags and do not write a review.
