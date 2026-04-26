# Example: Quality Improvement Loop

This example is for long-running quality-improvement work where an executor
runs experiments, a reviewer waits for finished sessions, and both agents use
evidence to decide the next direction.

Use this shape for problems like:

- improving rendered image quality
- reducing visible artifacts while protecting good regions
- comparing metrics and visual results across many sessions
- testing code/config variants without damaging the original project

## What To Customize

```text
Goal: what should improve or be decided
Evidence: metrics, images, logs, or checks that matter
Budget: sessions, time, resources, or stop condition
Reviewer: who reviews, when to start, and whether to include literature/method checks
Ask before: source edits, deletion, goal changes, budget changes
```

The important split is:

```text
loop_station/  # frame, flags, reports, reviews, decisions, summaries
artifacts/     # images, metrics, logs, generated outputs
```

## Codex Executor / Supervisor Command

Run this first in Codex. Replace the target, metrics, budget, and paths with
your actual experiment.

```text
$loop-station
I previously ran a feet-focused loop. First, inspect that prior loop and its artifacts.

Handoff:
CODEX is the executor and supervisor. CLAUDE, or another specified model/version,
is the external reviewer only. External review starts after CODEX marks the
session finished and review-ready (`EXECUTOR-DONE` + `SUPERVISOR-READY`).

Goal:
Improve the full-body human quality for subject 200014. Use quantitative metrics
such as PSNR, LPIPS, and SSIM where useful, and inspect rendered images every
session. Foot floaters may be a symptom of poor whole-body quality, so do not
optimize only local foot artifacts if the full-body result regresses.

Budget:
Use a 40-session budget. You may use all 4 GPUs. Treat one session as a batch of
variants across GPUs, then aggregate metrics, images, logs, and failures before
choosing the next session. Use the session budget for exploration unless a real
stop condition is reached.

Session focus:
Focus on subject 200014. Compare each variant against current best, relevant
anchors, and prior loop artifacts using both metrics and visual evidence.

Roles:
CODEX should run the experiments and perform its own supervisor analysis before
external review. It may use sub-agents to inspect generated code, result images,
metrics, logs, and variant summaries. CLAUDE should act only as reviewer: read
CODEX's report, proposal, supervisor analysis, artifacts, and code changes, then
write an independent review with improvement directions. Literature and prior-method
research belongs to CLAUDE review unless you explicitly assign it to CODEX.

Reviewer expectations:
CLAUDE may perform visual image checks, code/config audit, metric/log analysis,
artifact-completeness checks, and scientific/literature-backed challenge. When
sub-agents are available, CLAUDE may split the review across visual, metric,
code, log/artifact, and literature/method axes, then write one integrated review.
The review request should specify the reviewer identity, such as "Claude Code",
"GPT-5.5 xhigh reviewer", "a vision-focused reviewer instruction", or "a code
audit reviewer instruction", so the reviewer is intentionally different from
the executor path.

Memory:
Both agents should keep `loop_station/summaries/` current so context compaction
does not lose the experiment direction. Compact only after the current agent has
finished its turn, written the required flag, and updated summaries. CLAUDE
should maintain `LOG_TREND_SUMMARY.md` and `EXECUTOR_BRIEF.md` from prior
sessions and logs so CODEX can use those summaries first instead of spending
tokens rereading all historical logs.

Ask before:
Code changes and hyperparameter changes are allowed when scientifically justified.
Prefer diverse, informative experiments over tiny one-parameter nudges. Ask before
directly patching maintained source if an isolated variant is not enough. If a
direct maintained-source edit is approved, create exact backups and a restore
manifest before editing.
```

## Claude Reviewer Command

Start Claude after Codex has created loop artifacts, or ask Claude to hold and
monitor until the session is review-ready.

```text
/loop-station
I am reviewing a running LOOP-STATION experiment that Codex is executing.
Codex is the executor and supervisor. Claude Code is the external reviewer only.

Loop/output root:
<loop_output_root>/loop_station/

Experiment context:
Codex is improving subject 200014 full-body quality. It may already have prior
feet-loop artifacts, current session outputs, metrics, rendered images, logs,
code variants, and supervisor analysis. Do not rerun the experiment. Review the
evidence and help Codex choose the next session direction.

Reviewer identity:
Act as a scientific reviewer and auditor, not as executor or supervisor. If
sub-agents are available, use them for separate visual, metric/log, code/config,
artifact-completeness, and literature/method checks, then write one integrated
review.

Hold / monitor rule:
If Codex has not finished the current session, hold and keep watching. Start the
available Monitor/background watcher immediately when continuous waiting is
possible. Review only after Codex has marked the session finished and review-ready
(`EXECUTOR-DONE` + `SUPERVISOR-READY`) and the linked artifacts are readable.

Review focus:
- visual image checks: rendered images, crops, current-best comparison, visible
  regressions that metrics may hide
- metric/log analysis: PSNR, LPIPS, SSIM, failures, seeds, command logs, resource
  use, and recurring trends
- scientific/literature check: explain why the attempted improvement may have
  failed, cite relevant papers or prior methods when useful, and propose a
  falsifiable next mechanism
- code/config audit: generated scripts, config changes, manifests, code variants,
  and whether the implementation matches the stated experiment
- historical trend digest: update LOG_TREND_SUMMARY.md and EXECUTOR_BRIEF.md so
  Codex can plan the next session without rereading all historical raw logs

Write outputs:
<loop_output_root>/loop_station/reviews/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.md
<loop_output_root>/loop_station/flags/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.flag

After review:
Update reviewer rollup / compact notes if file writes are available before
writing REVIEWER-DONE. Codex will consume the review, write decision.md, update
summaries, and then prepare the next session.
```

## Minimal Flag Trail

```text
EXECUTOR-RUNNING
EXECUTOR-DONE
SUPERVISOR-READY
REVIEWER-RUNNING
REVIEWER-DONE
SUPERVISOR-DONE
next EXECUTOR-RUNNING
```
