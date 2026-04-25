# Example: Quality Improvement Loop

This example shows the kind of bounded loop LOOP-STATION is meant to coordinate:

- one locked goal
- bounded sessions
- executor reports
- reviewer handoff
- visual and metric artifacts
- safe implementation variants
- timeout-tolerant reviewer waits

The important split is:

```text
loop_station/  # contracts, decisions, flags, review
artifacts/     # images, metrics, logs, generated outputs
```

Example goal:

```text
Reduce a visible artifact in a target item while protecting the parts that are already correct.
```

Example frame:

```text
Goal: reduce the target artifact without damaging protected quality regions
Budget: 20 sessions, 2 GPUs, stop early if no improvement appears for 5 sessions
Work-unit scope: one target item first, then a fixed validation set before promotion
Collaboration: executor writes a session report, reviewer checks evidence before the next slate
User intervention: direct maintained-source edits, new paid resources, or scope expansion require approval
```

Example flag:

```text
CODEX5.5-SESSION020-EXECUTOR-DONE
CLAUDE-SESSION020-REVIEWER-DONE
```
