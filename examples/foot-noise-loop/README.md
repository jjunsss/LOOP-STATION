# Example: Foot Noise Loop

This example mirrors the kind of loop LOOP-STATION is designed to coordinate:

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
Reduce subject 200014 foot outside-alpha noise without damaging foot fidelity.
```

Example frame:

```text
Goal: lower outside_alpha_area_mean
Budget: 50 sessions, 4 GPUs
Work-unit scope: single subject 200014
Collaboration: executor + reviewer flag after each session
Intervention: code changes only through isolated implementation variants
```

Example flag:

```text
CODEX5.5-SESSION050-EXECUTOR-DONE
CLAUDE-SESSION050-REVIEWER-DONE
```
