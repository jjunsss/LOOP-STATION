<p align="center">
  <img src="./assets/LOOP-STATION.png" alt="LOOP-STATION" width="88%">
</p>

# LOOP-STATION

**A live multi-agent loop where executors and reviewers keep watching, exchange feedback, and move only on verified flags.**

LOOP-STATION helps Codex, Claude Code, and other agents stay active around the same experiment: Codex runs bounded sessions, reviewers wait for terminal flags, feedback is written to shared artifacts, and the next session starts only after the loop has evidence.

## Install

### Codex

Ask Codex:

```text
Install the Codex skill from https://github.com/jjunsss/LOOP-STATION as loop-station.
```

Restart Codex after installation.

### Claude Code

```text
Install LOOP-STATION from https://github.com/jjunsss/LOOP-STATION.
```

Restart Claude Code after installation.

## Core Idea

LOOP-STATION is for live feedback loops, not one-off prompts.

```text
Codex runs -> Codex self-reviews -> Claude reviews -> Codex decides -> next session
```

Agents should stay in standby, monitor `loop_station/flags/`, and act only when the required `DONE`, `BLOCKED`, or `ABSTAIN` flags and linked artifacts exist.

## Canonical Session Lifecycle

Every reviewed session follows this order:

```text
1. EXECUTOR-RUNNING
   Codex starts the session.

2. EXECUTOR-DONE
   Codex has finished the run and produced executor_report.md, executor_proposal.md,
   metrics, logs, images, and other result artifacts.

3. Internal validation
   Codex reviews its own results, writes supervisor_analysis.md, and uses
   sub-agents when available to inspect code, metrics, images, logs, and variants.

4. REVIEWER request
   Codex writes reviewer_requests.md and SUPERVISOR-READY.
   Claude/reviewer may start only after EXECUTOR-DONE + SUPERVISOR-READY.

5. REVIEWER-RUNNING
   Claude/reviewer starts external review.

6. REVIEWER-DONE
   Claude/reviewer writes the review artifact and done flag.

7. Codex consumes review
   Codex checks flags, reads the reviewer artifact, and records what was consumed.

8. SUPERVISOR-DONE
   Codex writes decision.md with the integrated judgment.

9. Prepare next session
   Codex creates the next session slate only after SUPERVISOR-DONE.

10. Next session starts
   The next session begins with the next EXECUTOR-RUNNING flag.
```

In the concrete flag trail, `SUPERVISOR-RUNNING` is the flag form of step 3:
Codex internal validation and optional sub-agent checks.

## What the Skill Does

LOOP-STATION gives an agent a concrete operating protocol:

- **Locks the frame** before work starts: goal, budget, work-unit scope, collaboration mode, and user-intervention boundaries.
- **Runs bounded sessions** instead of open-ended retries.
- **Writes durable evidence** after each session: reports, changed values, artifacts, failures, and next proposals.
- **Coordinates reviewer handoff** so Codex, Claude Code, or another agent can review without asking the user to restate the goal.
- **Uses named flags** to show which agent is running, done, waiting for review, or finished.
- **Keeps code variants isolated** so loop-driven changes do not patch maintained source until the user explicitly promotes them.
- **Forces a decision point** at each session boundary: promote, keep, retire, continue, stop, or ask the user.

Use it when the work needs adaptive loops rather than one-shot execution:

- model or pipeline optimization
- data cleanup and quality repair
- code-quality or workflow refinement
- prompt and agent-behavior iteration
- visual and metric review loops
- multi-agent execution with reviewer handoff

## How to Command It

Invoke LOOP-STATION with a required frame. The frame can be short, but it can also include project context, prior loop references, metrics, compute budget, collaboration rules, and experiment preferences.

### Minimal Live Loop

The minimal frame can be short, but better goals produce better loops. Include concrete context, target subjects/items, important metrics, visual checks, compute budget, and who should execute or review whenever you know them.

```text
$loop-station
Goal: improve subject 200014 full-body quality.
Budget: 40 sessions, use up to 4 GPUs.
Work-unit scope: compare metrics and rendered images each session.
Collaboration: Codex runs experiments; Claude waits and reviews after each Codex session is done.
User intervention: ask before direct maintained-source edits.
```

Start the reviewer after Codex has begun producing loop artifacts:

```text
/loop-station
Watch the running Codex LOOP-STATION experiment.
Stay in standby until each session has EXECUTOR-DONE and SUPERVISOR-READY flags.
When a session is ready, read the report, proposal, supervisor analysis, metrics, logs, and images.
Write a concise scientific review. Codex will consume it, write decision.md, then mark SUPERVISOR-DONE.
Keep watching for the next session.
```

Frame fields:

```text
Goal: what should improve or be decided
Budget: max sessions, wall time, resource pool, cost, or stop limit
Work-unit scope: one item, a fixed set, or a robustness set
Collaboration: supervisor only, executor + reviewer, or external review
User intervention: changes that require explicit approval
```

If any required field is missing, LOOP-STATION should ask only for the missing parts and keep asking until the frame is complete. Once the frame is locked, later agents reuse the same shared state instead of asking the setup questions again.

### Realistic Experiment Command

```text
$loop-station
I previously ran a feet-focused loop. First, inspect that prior loop and its artifacts.

Roles:
CODEX: executor and supervisor. Run experiments, use sub-agents when useful, write
executor_report.md, executor_proposal.md, supervisor_analysis.md, flags, and the
final decision.md after reviewer feedback.
CLAUDE: external reviewer. Stay in standby until CODEX writes EXECUTOR-DONE and
SUPERVISOR-READY. Then review the artifacts scientifically and write reviewer output.

Goal:
Improve the full-body human quality for subject 200014. Use quantitative metrics
such as PSNR, LPIPS, and SSIM where useful, and inspect rendered images every
session. Foot floaters may be a symptom of poor whole-body quality, so do not
optimize only local foot artifacts if the full-body result regresses.

Budget:
Run up to 40 sessions. You may use all 4 GPUs. Treat one session as a batch of
variants across GPUs, then aggregate metrics, images, logs, and failures before
choosing the next session.

Work-unit scope:
Focus on subject 200014. Compare each variant against current best, relevant
anchors, and prior loop artifacts using both metrics and visual evidence.

Collaboration:
CODEX should run the experiments and perform its own supervisor analysis before
external review. It may use sub-agents to inspect generated code, result images,
metrics, logs, and variant summaries. CLAUDE should act only as reviewer: read
CODEX's report, proposal, supervisor analysis, artifacts, and code changes, then
write an independent review with improvement directions.

User intervention:
Code changes and hyperparameter changes are allowed when scientifically justified.
Prefer diverse, informative experiments over tiny one-parameter nudges. Ask before
directly patching maintained source if an isolated variant is not enough.
```

## How It Runs

The loop follows this rhythm:

```text
lock the frame
EXECUTOR-RUNNING
EXECUTOR-DONE
internal validation / sub-agent checks when available
SUPERVISOR-READY reviewer request
REVIEWER-RUNNING
REVIEWER-DONE
Codex checks flags and consumes review
SUPERVISOR-DONE
prepare next session
next session EXECUTOR-RUNNING
```

Each loop keeps a shared folder inside the loop output root:

```text
loop_station/
  FRAME.md
  contract.json
  agent_roster.md
  sessions/
    session_001/
      executor_report.md
      executor_proposal.md
      supervisor_analysis.md
      decision.md
      reviewer_requests.md
      review_flags.md
  reviews/
    session_001/
      CLAUDE-SESSION001-REVIEWER-DONE.md
  flags/
    session_001/
      CODEX5.5-SESSION001-EXECUTOR-RUNNING.flag
      CODEX5.5-SESSION001-EXECUTOR-DONE.flag
      CODEX5.5-SESSION001-SUPERVISOR-RUNNING.flag
      CODEX5.5-SESSION001-SUPERVISOR-READY.flag
      CLAUDE-SESSION001-REVIEWER-RUNNING.flag
      CLAUDE-SESSION001-REVIEWER-DONE.flag
      CODEX5.5-SESSION001-SUPERVISOR-DONE.flag
```

If `contract.json` has `frame_locked: true`, later agents reuse the frame instead of asking for the goal, budget, scope, collaboration mode, or intervention boundaries again.

Flags use this naming pattern:

```text
{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}
```

Examples:

```text
CODEX5.5-SESSION050-EXECUTOR-RUNNING
CODEX5.5-SESSION050-EXECUTOR-DONE
CODEX5.5-SESSION050-SUPERVISOR-RUNNING
CODEX5.5-SESSION050-SUPERVISOR-READY
CLAUDE-SESSION050-REVIEWER-RUNNING
CLAUDE-SESSION050-REVIEWER-DONE
CODEX5.5-SESSION050-SUPERVISOR-DONE
```

Reviewer waits are controlled by `review_wait_policy` in `contract.json`. The executor records review start, heartbeat, and completion timeouts, then proceeds only when the locked policy allows it.

Do not move `REVIEWER-RUNNING` before `SUPERVISOR-READY`. `SUPERVISOR-READY` is the reviewer request signal: Codex has already completed its internal validation, written `supervisor_analysis.md`, and prepared the artifacts the external reviewer should inspect.

## Claude Code Reviewer Mode

Use Claude after Codex has already started the LOOP-STATION run and produced at least some session artifacts. The normal flow is:

```text
Codex starts and runs LOOP-STATION
Codex writes session reports, proposals, metrics, images, logs, and flags
Claude Code is invoked as a reviewer with brief experiment context
Claude reads the loop artifacts and writes a focused review
Codex uses the review when deciding the next session
```

The `loop_station/` folder is created by the running loop, but Claude may not know where it is or what the experiment is about. Give Claude enough context to find the run and understand what kind of review you want.

If Claude is invoked before Codex finishes the current session, Claude should stay in reviewer standby. It should poll the relevant `flags/session_{NNN}/` folder and wait until Codex has written `EXECUTOR-DONE` plus `SUPERVISOR-READY`. `SUPERVISOR-READY` means Codex has already reviewed its own results, written its analysis/explanation, and prepared the artifacts for external review. Claude should not modify `FRAME.md`, `contract.json`, code, configs, or session artifacts while waiting.

If the request asks Claude to keep waiting continuously, Claude should use its available Monitor/background watcher tool immediately. The watcher should poll for review-ready flags and linked artifacts, then trigger one review per ready session.

Example Claude reviewer prompt:

```text
/loop-station
I am running a full-body quality experiment with Codex through LOOP-STATION.
Codex is the executor and supervisor. Claude is the external reviewer.

Loop/output root:
<loop_output_root>/loop_station/

Experiment context:
Codex is improving subject 200014. Focus on scientific trends across sessions:
PSNR, LPIPS, SSIM, visual quality, failure modes, floaters, and regressions.
Do not rewrite the experiment. Wait for Codex results, then review them.

Reviewer instructions:
- Find the active loop/output root and latest session artifacts.
- If FRAME.md and contract.json exist, reuse them instead of asking setup questions.
- If Codex has not finished the current session, stay in standby.
- For continuous standby, start the available Monitor/background watcher immediately.
- Review only after `EXECUTOR-DONE` and `SUPERVISOR-READY` exist and linked
  artifacts are readable.
- Read executor_report.md, executor_proposal.md, supervisor_analysis.md,
  reviewer_requests.md, metrics, logs, diffs, generated code, result images,
  and artifacts.
- Do not execute experiments, modify code, edit frame files, or prepare the next session.
- Write a concise scientific review: trends, likely causes, risks, and next
  experiment suggestions.
- After the review, Codex will consume it, write decision.md, mark
  `SUPERVISOR-DONE`, then prepare the next session.

Write the review to:
<loop_output_root>/loop_station/reviews/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.md
<loop_output_root>/loop_station/flags/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.flag
```

If you do not know the exact loop root, provide the project directory, subject or experiment name, and what Codex has been running. Claude should search for the active `loop_station/` folder, then use the locked frame if it exists.

## Example

See [`examples/quality-improvement-loop/`](examples/quality-improvement-loop/) for a generic quality-improvement loop frame and artifact layout.

## Notes

- LOOP-STATION is a protocol skill, not an optimizer by itself. The executor still needs project-specific commands, metrics, artifacts, and resource checks.
- Do not use it as an unbounded retry loop. A loop must have a budget and a stop or escalation path.
- Do not treat reviewer timeout as evidence that a result is good or bad. It is only a coordination event.
- Keep durable artifacts in the loop output root and keep temporary logs or generated data out of source control unless intentionally promoted.
- Add a license before publishing for external reuse.
