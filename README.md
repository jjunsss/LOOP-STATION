# LOOP-STATION

**A control-room skill for bounded, goal-directed loops.**

LOOP-STATION helps agents run iterative work without drifting into open-ended retry. It first locks the goal frame, then coordinates executor, reviewer, analyst, resource, artifact, and timeout protocols across sessions.

It is useful when the work should improve through repeated cycles:

- model or experiment optimization
- data cleanup
- code-quality refinement
- prompt or workflow iteration
- visual/metric review loops
- multi-agent execution with reviewer handoff

The core idea is simple:

```text
lock the goal
run a bounded session
write evidence and proposal
review with named agents
adapt the next session
stop when the budget or decision says stop
```

## Why This Exists

Long adaptive runs often fail in predictable ways:

- the goal changes mid-loop
- reviewers ask the same setup questions again
- executor decisions are lost in logs
- code edits happen in-place and are hard to revert
- agents wait forever for review
- a visually better result quietly violates protection metrics

LOOP-STATION turns those risks into a shared folder protocol.

## The Shared Folder

Each loop keeps a `loop_station/` folder inside the loop output root:

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

If `contract.json` has `frame_locked: true`, later agents reuse the frame instead of asking the user to restate the goal, budget, scope, collaboration mode, or intervention boundaries.

## Agent Roles

LOOP-STATION supports three common modes:

- **Supervisor / Executor**: runs or prepares sessions, writes results and the next proposal.
- **Reviewer**: reads the locked frame and executor artifacts, then writes high-value review only.
- **Analyst**: adds deeper side analysis without approving or blocking execution.

Every flag names the agent, session, role, and status:

```text
{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}
```

Examples:

```text
CODEX5.5-SESSION050-EXECUTOR-RUNNING
CODEX5.5-SESSION050-EXECUTOR-DONE
CLAUDE-SESSION050-REVIEWER-RUNNING
CLAUDE-SESSION050-REVIEWER-DONE
CODEX5.5-SESSION050-SUPERVISOR-TIMEOUT
```

## Bounded Review Waits

Reviewer waits are bounded by `review_wait_policy`:

```json
{
  "required_done_count": 1,
  "optional_done_count": 0,
  "start_timeout_seconds": 300,
  "done_timeout_seconds": 1800,
  "heartbeat_timeout_seconds": 600,
  "allow_skip_on_timeout": true
}
```

If reviewers do not start or finish in time, the executor records the timeout and proceeds when the contract allows it. Reviewer timeouts are coordination failures, not evidence failures.

## Safe Implementation Variants

Loop-driven code changes should not patch maintained source in place. Create an isolated implementation variant instead:

```text
{loop_output_root}/code_variants/session_003/outside_alpha_schedule/
  manifest.json
  decision.md
  src_or_scripts_to_run/
```

The manifest records the copied source files, source hashes, reason for the variant, and command/config that uses it.

## Install

Copy this folder into a Codex skill path or keep it in a project-local `skills/` directory:

```text
skills/loop-station/
  SKILL.md
  agents/openai.yaml
  templates/
```

Then invoke:

```text
Use $loop-station to lock or reuse the loop frame, then act as supervisor, executor, or reviewer for the next bounded adaptive session.
```

## Claude Code Review Prompt

Use this when a Claude Code instance should review a running loop:

```text
Use LOOP-STATION in Reviewer / Project Review Mode.

Loop root:
<loop_output_root>/loop_station/

Agent name:
CLAUDE

If contract.json has frame_locked=true, do not ask the user to restate the goal.
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

## Repository About Text

Suggested GitHub description:

```text
A control-room skill for bounded goal loops, multi-agent review, adaptive sessions, and safe implementation variants.
```

Suggested topics:

```text
codex-skill, agent-workflow, multi-agent, experiment-loop, adaptive-loop, review-protocol, ai-agents, automation
```

## License

Add a license before publishing for external reuse.
