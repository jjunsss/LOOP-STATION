<p align="center">
  <img src="./assets/LOOP-STATION.png" alt="LOOP-STATION" width="88%">
</p>

# LOOP-STATION

**A live multi-agent loop where executors and reviewers take turns, exchange evidence, and use shared flags to coordinate timing.**

LOOP-STATION helps Codex, Claude Code, and other agents stay active around the same experiment. Codex runs bounded sessions, reviewers hold until the session is ready, feedback is written to shared artifacts, and the next session starts only after the prior work has been reviewed and summarized.

⭐ It is built for work where agents need to keep analyzing results and improve
performance over many sessions. Traditional ML automation loops often stay
inside a predeclared sweep space, such as fixed Hydra configs or parameter
grids. LOOP-STATION lets agents inspect the actual code changes, logs, metrics,
images, and trends, then plan the next intervention from what happened.

## Install

### Codex

Ask Codex:

```text
Install the Codex skill from https://github.com/jjunsss/LOOP-STATION as loop-station.
```

Restart Codex after installation.

### Claude Code

Ask Claude code:

```text
Install LOOP-STATION from https://github.com/jjunsss/LOOP-STATION.
```

Restart Claude Code after installation.

### Permission Note

For long-running LOOP-STATION work, using **Full Access** can reduce repeated
permission prompts and save tokens because the executor does not need to pause
and explain every file or shell action. This helps the agent keep momentum while
sessions, monitors, reviews, and summary updates run for a long time.

## Core Idea

LOOP-STATION is for live feedback loops, not one-off prompts.

The goal is not just to run many variants. The goal is to make agents understand
what changed, why the result moved, what failed, and what should be tried next.
That makes LOOP-STATION useful for ongoing optimization work where progress
depends on interpreting evidence, not only enumerating preset values.

```text
Codex runs -> Codex self-reviews -> Claude reviews -> Codex decides -> next session
```

Agents do not all run at once. Each agent works when it has a job, holds when it is waiting, and uses small flag files in `loop_station/flags/` to tell the other agents what happened: running, ready, done, blocked, or abstained. Those flags are the timing signals that keep Codex, Claude, and other reviewers from writing too early or moving to the next session before the right artifacts exist.

## How It Runs

Agents use flag files as timing signals. They work when their turn opens, hold
while waiting, and move the loop forward only after the expected result files
exist.

```mermaid
flowchart LR
  A["Codex runs<br/>EXECUTOR-RUNNING"] --> B["Codex finishes results<br/>EXECUTOR-DONE"]
  B --> C["Codex checks its own work<br/>sub-agents if useful"]
  C --> D["Review opens<br/>SUPERVISOR-READY"]
  D --> E["Reviewer waits, reads, reviews<br/>REVIEWER-DONE"]
  E --> F["Codex consumes review<br/>decision + summaries"]
  F --> G["Session closes<br/>SUPERVISOR-DONE"]
  G --> H["Next session starts"]
```

Simple rule: reviewers start after `EXECUTOR-DONE` + `SUPERVISOR-READY`, and
Codex starts the next session only after it has consumed `REVIEWER-DONE`, written
the decision, updated summaries, and closed the session with `SUPERVISOR-DONE`.

Reviews can include visual image checks, metric/log analysis, code/config audit,
and artifact-completeness checks. Experiments should use loop-owned variants or
copies by default so the original project state is not changed unless the user
approves it.

## How to Command It

Write the command like a short experiment brief. You do not need special protocol
words. Tell LOOP-STATION what you want, what evidence matters, how far it may
run, and who should do which part.

### Minimal Live Loop

The command can be short. It works better when the goal, metrics, visual checks,
budget, and agent roles are clear.

```text
$loop-station
Improve subject 200014 full-body quality.
Use PSNR, LPIPS, SSIM, and rendered-image checks each session.
Run up to 40 sessions and use up to 4 GPUs.
Codex should run experiments and write its own analysis.
Claude should wait until Codex marks the session review-ready, then write a scientific review.
Ask me before editing original project files, deleting outputs, or changing the goal.
Keep rolling summaries in loop_station/summaries so agents can compact and continue.
```

Start the reviewer after Codex has begun producing loop artifacts:

```text
/loop-station
Watch the running Codex LOOP-STATION experiment.
Hold and keep watching until Codex says the session is finished and ready for external review (`EXECUTOR-DONE` + `SUPERVISOR-READY`).
When a session is ready, read the report, proposal, supervisor analysis, metrics, logs, and images.
Perform visual image checks and code/config audit when relevant. Use sub-agents for separate audit axes if available.
Summarize prior experiment trends and relevant logs so Codex can plan without rereading all historical raw logs.
Write a concise scientific review. Codex will consume it, write decision.md, then mark SUPERVISOR-DONE.
Update reviewer rollup/compact notes if file writes are available, then compact only after the review is durable.
Keep watching for the next session.
```

Useful details to include:

```text
Target: the subject, item, model, dataset, bug, or decision
Context: prior runs, current artifacts, important paths, or known failures
Evidence: metrics, rendered images, logs, visual checks, or regression guards
Limit: max sessions, wall time, GPUs, cost, or stop condition
Roles: who runs experiments, who reviews, and when the reviewer should start
Ask before: source edits, deletion, goal changes, extra resources, or risky operations
Memory: what summaries to keep so agents can compact and continue
```

If something essential is missing, LOOP-STATION should ask only for that missing
detail. Once the run is started, later agents reuse the shared state instead of
asking the same setup questions again.

<details>
<summary><strong>Realistic Experiment Command</strong></summary>

Use the Codex command first. Start Claude Code after Codex has created loop
artifacts, or ask Claude to hold and monitor until the session is review-ready.

#### Codex executor / supervisor command

```text
$loop-station
I previously ran a feet-focused loop. First, inspect that prior loop and its artifacts.

Roles:
CODEX: executor and supervisor. Run experiments, use sub-agents when useful, write
executor_report.md, executor_proposal.md, supervisor_analysis.md, flags, and the
final decision.md after reviewer feedback.
CLAUDE or another specified reviewer model/version: external reviewer only.
Use a reviewer instruction profile: "act as a scientific reviewer and auditor,
not as executor." Hold until CODEX marks the session as finished and review-ready
(`EXECUTOR-DONE` + `SUPERVISOR-READY`). Then review the artifacts scientifically
and write reviewer output.

Goal:
Improve the full-body human quality for subject 200014. Use quantitative metrics
such as PSNR, LPIPS, and SSIM where useful, and inspect rendered images every
session. Foot floaters may be a symptom of poor whole-body quality, so do not
optimize only local foot artifacts if the full-body result regresses.

Budget:
Run up to 40 sessions. You may use all 4 GPUs. Treat one session as a batch of
variants across GPUs, then aggregate metrics, images, logs, and failures before
choosing the next session.

Session focus:
Focus on subject 200014. Compare each variant against current best, relevant
anchors, and prior loop artifacts using both metrics and visual evidence.

Roles:
CODEX should run the experiments and perform its own supervisor analysis before
external review. It may use sub-agents to inspect generated code, result images,
metrics, logs, and variant summaries. CLAUDE should act only as reviewer: read
CODEX's report, proposal, supervisor analysis, artifacts, and code changes, then
write an independent review with improvement directions. CLAUDE may perform
visual image checks, code/config audit, metric/log analysis, and artifact
completeness checks. When sub-agents are available, CLAUDE may split the review
across visual, metric, code, and log/artifact audit axes, then write one
integrated reviewer report.
The review request should specify the reviewer identity, such as "Claude Code",
"GPT-5.5 xhigh reviewer", "a vision-focused reviewer instruction", or "a code
audit reviewer instruction", so the reviewer is intentionally different from
the executor path.
Both agents should keep `loop_station/summaries/` current so context compaction
does not lose the experiment direction. Compact only after the current agent has
finished its turn, written the required flag, and updated summaries.
CLAUDE should maintain `LOG_TREND_SUMMARY.md` and `EXECUTOR_BRIEF.md` from prior
sessions and logs so CODEX can use those summaries first instead of spending
tokens rereading all historical logs.

Ask before:
Code changes and hyperparameter changes are allowed when scientifically justified.
Prefer diverse, informative experiments over tiny one-parameter nudges. Ask before
directly patching maintained source if an isolated variant is not enough. If a
direct maintained-source edit is approved, create exact backups and a restore
manifest before editing.
```

#### Claude Code reviewer command

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
and artifact-completeness checks, then write one integrated review.

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
- code/config audit: generated scripts, config changes, manifests, code variants,
  and whether the implementation matches the stated experiment
- historical trend digest: update LOG_TREND_SUMMARY.md and EXECUTOR_BRIEF.md so
  Codex can plan the next session without rereading all historical raw logs

Write outputs:
<loop_output_root>/loop_station/reviews/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.md
<loop_output_root>/loop_station/flags/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.flag

After review:
Update reviewer rollup / compact notes if file writes are available. Codex will
consume the review, write decision.md, update summaries, and then prepare the
next session.
```

</details>

## Operational Scale

Yes, it can actually run overnight. In one real run, Codex kept going for 10+
hours across handoffs, Claude stayed alive through Monitor, and the loop hit the
usage limit while the user was asleep. That is the point: LOOP-STATION is
for work that should keep producing results, reviews, summaries, and next-session
plans after a normal chat would have stopped.

<p align="center">
  <img src="./assets/loop-station-runtime-example.svg" alt="LOOP-STATION long-running runtime example" width="88%">
</p>

<p align="center">
  <img src="./assets/loop-station-token-usage-example.svg" alt="LOOP-STATION token usage example" width="88%">
</p>

Agents leave compact briefs in `loop_station/summaries/` so they can wake back
up from summaries instead of replaying every old log.

## Example

See [`examples/quality-improvement-loop/`](examples/quality-improvement-loop/) for a generic quality-improvement loop frame and artifact layout.

## Notes

- LOOP-STATION is a protocol skill, not an optimizer by itself. The executor still needs project-specific commands, metrics, artifacts, and resource checks.
- Do not use it as an unbounded retry loop. A loop must have a budget and a stop or escalation path.
- Do not treat reviewer timeout as evidence that a result is good or bad. It is only a coordination event.
- Keep durable artifacts in the loop output root and keep temporary logs or generated data out of source control unless intentionally promoted.
- Add a license before publishing for external reuse.
