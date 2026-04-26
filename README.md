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
   Codex writes decision.md with the integrated judgment and updates summaries.

9. Compact checkpoint
   Codex or Claude may compact only after durable artifacts and summaries exist.

10. Prepare next session
   Codex creates the next session slate only after SUPERVISOR-DONE.

11. Next session starts
   The next session begins with the next EXECUTOR-RUNNING flag.
```

Agents can review more than metrics. A reviewer can inspect images, logs, code
variants, manifests, configs, generated scripts, and artifact completeness. When
useful, split that review across sub-agents: one for visual outputs, one for
metrics/logs, one for code/config, and one for artifact completeness. Experiments
should run through loop-owned variants or copies by default, so the original
project state is not mutated unless the user explicitly approves it.

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
Memory: reviewer summarizes prior trends/logs for Codex in loop_station/summaries; compact only at clean boundaries.
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

Work-unit scope:
Focus on subject 200014. Compare each variant against current best, relevant
anchors, and prior loop artifacts using both metrics and visual evidence.

Collaboration:
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

User intervention:
Code changes and hyperparameter changes are allowed when scientifically justified.
Prefer diverse, informative experiments over tiny one-parameter nudges. Ask before
directly patching maintained source if an isolated variant is not enough. If a
direct maintained-source edit is approved, create exact backups and a restore
manifest before editing.
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
write decision.md and update rolling summaries
SUPERVISOR-DONE
compact checkpoint when useful
prepare next session
next session EXECUTOR-RUNNING
```

Each loop keeps a shared folder inside the loop output root:

```text
loop_station/
  FRAME.md
  contract.json
  agent_roster.md
  summaries/
    ROLLING_SUMMARY.md
    SESSION_LEDGER.md
    COMPACT_HANDOFF.md
    EXECUTOR_BRIEF.md
    LOG_TREND_SUMMARY.md
    REVIEWER_ROLLUP.md
    compact/
      session_001-CODEX5.5.md
      session_001-CLAUDE.md
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

## Rolling Summaries And Compact

Session folders remain the source of truth. The `loop_station/summaries/` folder is the compact-ready layer that every agent should read first after context is reset.

Codex updates these after every completed session decision and before `SUPERVISOR-DONE`:

- `summaries/ROLLING_SUMMARY.md`: cumulative experiment direction, best candidate, retired paths, risks, and next plan.
- `summaries/SESSION_LEDGER.md`: one row per session with metrics, visual judgment, reviewer status, decision, and artifact links.
- `summaries/COMPACT_HANDOFF.md`: short operational handoff for the next agent after compact.

Claude or another reviewer updates these after `REVIEWER-DONE` when file edits are available:

- `summaries/REVIEWER_ROLLUP.md`: cumulative reviewer conclusions and recurring concerns.
- `summaries/LOG_TREND_SUMMARY.md`: reviewer-maintained summary of prior logs, failures, metric shifts, and recurring patterns.
- `summaries/EXECUTOR_BRIEF.md`: short next-session brief so Codex can design without rereading every historical log.
- `summaries/compact/session_{NNN}-{AGENT_NAME}.md`: compact note for that review/session.

The reviewer should spend the tokens needed to inspect previous summaries and relevant historical logs, then write the trend/log digest. Codex should not reread all old logs by default when a new session starts. It should first read `EXECUTOR_BRIEF.md`, `LOG_TREND_SUMMARY.md`, `ROLLING_SUMMARY.md`, `SESSION_LEDGER.md`, and the latest decision/review artifacts. Codex opens raw historical logs only when a summary is inconsistent, incomplete, or a specific old detail affects the next design.

Codex and Claude should compact, or recommend compaction, only at a clean
boundary: after the current agent has finished its turn and summaries are
updated. They should not compact during execution, review, or supervisor
synthesis. After compact, resume by reading `contract.json`, `FRAME.md`,
`summaries/COMPACT_HANDOFF.md`, `summaries/EXECUTOR_BRIEF.md`,
`summaries/LOG_TREND_SUMMARY.md`, `summaries/ROLLING_SUMMARY.md`,
`summaries/SESSION_LEDGER.md`, and then the latest session artifacts.

## Operational Scale

LOOP-STATION is intended for long-running agent work, not a single short prompt. In a real full-use run, token use can climb sharply over days as the loop accumulates experiments, reviews, images, logs, and summaries. Claude Code can stay alive through monitor/background watcher tooling without continuously consuming active execution time, while Codex may run long executor/supervisor sessions that continue for many hours. In one heavy run, Codex continued for more than 10 hours across handoffs, while Claude held through monitor tooling and woke when review-ready flags appeared.

<p align="center">
  <img src="./assets/loop-station-runtime-example.svg" alt="LOOP-STATION long-running runtime example" width="88%">
</p>

<p align="center">
  <img src="./assets/loop-station-token-usage-example.svg" alt="LOOP-STATION token usage example" width="88%">
</p>

This is why reviewer-maintained summaries are part of the protocol: the reviewer absorbs the cost of reading historical logs and trends, then leaves compact executor briefs so the next Codex session can plan from durable summaries instead of replaying the whole history.

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

The practical rule is simple: reviewers start after Codex has prepared the
session for review, and Codex starts the next session only after it has consumed
the review and updated the summaries.

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

If Claude is invoked before Codex finishes the current session, Claude should hold
and watch rather than writing a review immediately. It should poll the relevant
`flags/session_{NNN}/` folder and wait until Codex has written `EXECUTOR-DONE`
plus `SUPERVISOR-READY`. That pair means Codex has finished the run, reviewed its
own results, and prepared the artifacts for external review. Claude should not
modify `FRAME.md`, `contract.json`, code, configs, or session artifacts while
waiting.

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
- Use this identity: external scientific reviewer and auditor. Do not act as
  executor or supervisor unless explicitly reassigned.
- If FRAME.md and contract.json exist, reuse them instead of asking setup questions.
- If Codex has not finished the current session, hold and keep watching.
- For continuous holding/watching, start the available Monitor/background watcher immediately.
- Review only after Codex has marked the session finished and review-ready
  (`EXECUTOR-DONE` + `SUPERVISOR-READY`) and linked artifacts are readable.
- Read executor_report.md, executor_proposal.md, supervisor_analysis.md,
  reviewer_requests.md, metrics, logs, diffs, generated code, result images,
  and artifacts.
- Perform visual image checks when result images exist.
- Audit code/config changes, manifests, scripts, and logs when they are part of
  the session.
- Use sub-agents for focused visual, metric, code, or log/artifact review when
  available, then integrate their findings into one reviewer report.
- Do not execute experiments, modify code, edit frame files, or prepare the next session.
- Write a concise scientific review: trends, likely causes, risks, and next
  experiment suggestions.
- Update reviewer rollup and compact note if file writes are available.
- Compact only after the review artifact, reviewer timing signal, and summary updates are durable.
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
