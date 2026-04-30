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

- Review-ready signal:
- Signal discovery: exact canonical flags checked first; equivalent ready/terminal signals checked next and recorded with source path.
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
  - [ ] executor terminal signal observed
  - [ ] review-ready signal observed
  - [ ] `supervisor_analysis.md` is readable
  - [ ] linked executor artifacts are readable
  - [ ] linked decision artifacts are readable only for explicit post-decision audit
- If required signals or linked artifacts are missing, stale, unreadable, or contradictory, keep waiting within policy. When wait policy ends, write `REVIEWER-BLOCKED` with last observed signals and missing artifacts. Do not infer completion from expectation alone.
- Poll paths:
  - flags:
  - reviews:
  - session artifacts:
- Summary updates before terminal signal: done|skipped with reason
- Next-session monitor audit: reused|duplicates recorded|not needed
- If interrupted, write `REVIEWER-BLOCKED` with the last observed signals and do not write a review.
