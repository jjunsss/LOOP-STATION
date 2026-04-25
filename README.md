# LOOP-STATION

**A control-room skill for bounded, goal-directed loops.**

LOOP-STATION helps agents improve a target through repeated sessions without losing the goal, budget, evidence, or reviewer handoff. It locks a small decision frame first, then keeps every session traceable through shared files, named flags, executor reports, reviewer notes, and safe implementation variants.

Use it when the work needs adaptive loops rather than one-shot execution:

- model or pipeline optimization
- data cleanup and quality repair
- code-quality or workflow refinement
- prompt and agent-behavior iteration
- visual and metric review loops
- multi-agent execution with reviewer handoff

The user-facing frame is intentionally small:

```text
Goal: what should improve or be decided
Budget: max sessions, wall time, resource pool, cost, or stop limit
Work-unit scope: one item, a fixed set, or a robustness set
Collaboration: supervisor only, executor + reviewer, or external review
User intervention: changes that require explicit approval
```

Once that frame is locked, agents can run, review, and continue from the same shared state instead of asking the setup questions again.

## Features

- Locks the goal frame before any loop starts.
- Keeps a persistent `loop_station/` folder for contracts, decisions, reports, reviews, and flags.
- Lets Codex, Claude Code, or another agent join as a reviewer without re-asking the user for the goal.
- Requires executor reports with evidence, changed values, artifacts, failures, and next proposals.
- Supports bounded reviewer waits so a loop can continue when a reviewer does not start or finish in time.
- Keeps loop-driven code changes in isolated variant folders instead of patching maintained source in place.
- Records enough evidence to promote, keep, retire, stop, or ask the user at each session boundary.

## Install For Codex

Clone or copy this repository into a Codex skill location.

Project-local example:

```text
<project>/
  skills/
    loop-station/
      SKILL.md
      agents/openai.yaml
      templates/
```

Then invoke it from Codex:

```text
Use $loop-station for this goal-directed loop.
Goal: ...
Budget: ...
Work-unit scope: ...
Collaboration: ...
User intervention: ...
```

If any required frame field is missing, LOOP-STATION should ask for only the missing parts and keep asking until the frame is complete.

## Install For Claude Code

Claude Code discovers skills from filesystem skill folders.

Personal skill:

```text
~/.claude/skills/loop-station/
  SKILL.md
  templates/
```

Project skill:

```text
<project>/
  .claude/
    skills/
      loop-station/
        SKILL.md
        templates/
```

Restart Claude Code after adding or updating the skill so it reloads the skill list.

You can ask Claude Code to use the skill directly:

```text
Use LOOP-STATION in Reviewer / Project Review Mode.

Loop root:
<loop_output_root>/loop_station/

Agent name:
CLAUDE

If contract.json has frame_locked=true, do not ask me to restate the goal.
Read FRAME.md, contract.json, the latest executor_report.md, executor_proposal.md,
reviewer_requests.md, changed values, manifests, diffs, metrics, logs, and artifacts.

Do not run the next session.
Do not modify code.

Write only high-value review:
- whether the executor interpretation is supported by evidence
- whether changed values or implementation variants match the goal
- what to promote, keep, or retire
- risks or missing validation
- one recommended next action or ABSTAIN

Write:
<loop_output_root>/loop_station/reviews/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.md
<loop_output_root>/loop_station/flags/session_{NNN}/CLAUDE-SESSION{NNN}-REVIEWER-DONE.flag
```

## How It Works

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
      CLAUDE-SESSION001-REVIEWER-RUNNING.flag
      CLAUDE-SESSION001-REVIEWER-DONE.flag
```

The loop moves through this rhythm:

```text
lock the frame
run a bounded session
write evidence and proposal
request named review when useful
wait within the review policy
write a decision
adapt, stop, or ask the user
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
CLAUDE-SESSION050-REVIEWER-RUNNING
CLAUDE-SESSION050-REVIEWER-DONE
```

Reviewer waits are controlled by `review_wait_policy` in `contract.json`. The executor records review start, heartbeat, and completion timeouts, then proceeds only when the locked policy allows it.

## Safe Code Variants

Loop-driven implementation changes should not patch maintained source in place. Create an isolated implementation variant instead:

```text
{loop_output_root}/code_variants/session_003/quality_gate_adjustment/
  manifest.json
  decision.md
  src_or_scripts_to_run/
```

The manifest records copied source files, source hashes, changed files, the reason config-only changes were insufficient, and the command or config that runs the variant.

Direct edits to maintained source should require explicit user approval and a restore path.

## Example

See [`examples/quality-improvement-loop/`](examples/quality-improvement-loop/) for a generic quality-improvement loop frame and artifact layout.

## Notes

- LOOP-STATION is a protocol skill, not an optimizer by itself. The executor still needs project-specific commands, metrics, artifacts, and resource checks.
- Do not use it as an unbounded retry loop. A loop must have a budget and a stop or escalation path.
- Do not treat reviewer timeout as evidence that a result is good or bad. It is only a coordination event.
- Keep durable artifacts in the loop output root and keep temporary logs or generated data out of source control unless intentionally promoted.
- Add a license before publishing for external reuse.
