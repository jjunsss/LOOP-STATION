<p align="center">
  <img src="./assets/LOOP-STATION.png" alt="LOOP-STATION" width="88%">
</p>

# LOOP-STATION

**A live multi-agent loop where executors and reviewers take turns, exchange evidence, and use shared flags to coordinate timing.**

LOOP-STATION is a protocol skill, not an optimizer by itself. It keeps Codex,
Claude Code, and other agents coordinated around long experiment loops where
each session leaves reports, reviews, decisions, flags, and summaries under
`loop_station/`.

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

## How The Loop Works

LOOP-STATION is for live feedback loops, not one-off prompts.

The goal is not just to run many variants. The goal is to make agents understand
what changed, why the result moved, what failed, and what should be tried next.
That makes LOOP-STATION useful for ongoing optimization work where progress
depends on interpreting evidence, not only enumerating preset values.

```text
Codex runs -> Codex self-reviews -> Claude reviews -> Codex decides -> next session
```

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
  F --> G["Session ends<br/>SUPERVISOR-DONE"]
  G --> H["Next session starts"]
```

Simple rule: reviewers start after `EXECUTOR-DONE` + `SUPERVISOR-READY`, and
Codex starts the next session only after it has consumed `REVIEWER-DONE`, written
the decision, updated summaries, and ended the session with `SUPERVISOR-DONE`.

Tell LOOP-STATION what tools, checks, and review style you want. For example,
you can ask the reviewer for visual checks, metric checks, code audit,
literature/method checks, online search, or a stricter scientific review. A
reviewer should be active: when a session result needs research context, online
search and paper/prior-method checks are recommended for a higher-quality
review. The reviewer can use them to challenge weak explanations and propose
better next directions. Experiments should use loop-owned variants or copies by
default so the original project state is not changed unless the user approves
it.

## How to Command It

Start from Codex with one `$loop-station` command. Write it like a short
experiment brief; you do not need special protocol words.

```text
$loop-station
Goal: what you want to improve or decide
Budget: sessions, time, GPUs/resources, or stop condition
Evidence: metrics, logs, images, tests, or checks that should guide decisions
Optional/custom: reviewer role, paths, tools, ask-before rules, summaries
```

This is an example shape, not a required checklist. The important part is to
brief LOOP-STATION clearly for your own project. Goal, Budget, and Evidence
(especially metrics) are usually worth including; customize the rest freely.
More detail gives the agents tighter control.

Below is one real example command I used, with matching Codex and Claude
prompts:
[`examples/quality-improvement-loop/`](examples/quality-improvement-loop/).

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

## Notes

- LOOP-STATION is a protocol skill, not an optimizer by itself. The executor still needs project-specific commands, metrics, artifacts, and resource checks.
- Consider token and runtime cost before starting a long loop. Give a session budget, resource budget, and stop condition.
- Understand the permission level you give the agent. Full Access can reduce repeated prompts and save tokens in long trusted runs, but it also gives broad file and command execution.
- Give clear code-change rules: whether source edits are allowed, whether variants/backups are required, and what the agent must ask before changing.
- Check intermediate trends when you can. The user can intervene mid-loop, redirect the next axis, or stop a weak direction before the budget is spent.
- Keep durable artifacts in the loop output root and keep temporary logs or generated data out of source control unless intentionally promoted.
