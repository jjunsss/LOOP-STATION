# Supervisor Analysis

- agent:
- session:
- status:
- started_at:
- finished_at:

## Inputs Reviewed

- executor_report:
- executor_proposal:
- metrics:
- logs:
- result images:
- code diffs / manifests:
- prior decisions:

## Internal Verification

- sub-agents or helper reviewers used:
- metric checks:
- visual checks:
- code/config checks:
- artifact completeness:

## Supervisor Interpretation

- trend:
- likely mechanism:
- tradeoffs:
- confounds:
- risks:

## Reviewer Handoff

- artifacts reviewer must inspect:
- historical logs or prior sessions reviewer should summarize for executor:
- visual image checks requested:
- code/config audit requested:
- reviewer literature/method check requested:
- original protection / backup audit requested:
- metric/log audit requested:
- executor brief update requested:
- suggested sub-agent split:
- specific questions for reviewer:
- what would change the next decision:

## Ready Status

- ready_for_external_review: yes|no
- ready signal to write:
- artifact links verified readable: yes|no
- missing/stale/ambiguous artifacts:
- if not ready, terminal status should be `SUPERVISOR-BLOCKED`, not `SUPERVISOR-READY`
- blocking reason if not ready:
