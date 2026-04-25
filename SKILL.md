---
name: loop-station
description: Use for bounded goal-directed loops that must first lock a goal, budget, work-unit scope, collaboration protocol, and user-intervention boundaries, then run adaptive sessions that inspect results, adjust strategy, isolate implementation variants, and stop or escalate without turning into open-ended retry.
---

# Loop Station

## Language Rule

The skill body, durable loop contracts, manifests, flags, and session decision notes should be written in English by default so they stay portable across agents and repositories.

The user may speak Korean, English, or both. Match the user's language in chat, but keep the core loop artifacts and stable protocol wording in English unless the user explicitly asks otherwise.

## Hard Rule

Do not start from variants. Start from the decision frame.

Before launching any loop, obtain or infer the required user frame:

- goal: what the loop must improve or decide
- budget: max sessions, resource pool, wall-time, cost, or other hard limit
- work-unit scope: one sample/item/case, a fixed set, or a robustness/generalization set
- collaboration mode: supervisor only, executor/reviewer split, or external review flag
- user intervention points: changes that require explicit user approval

Ask only for missing user-owned frame fields, but keep asking until no required field is missing. Do not proceed to planning, session creation, execution, or code-variant creation until the frame is complete.

When fields are missing:

1. Infer what is safe from the current request, repo context, previous artifacts, and explicit user constraints.
2. Present the inferred frame and mark unknowns as `missing`.
3. Ask one bundled clarification covering only the missing required fields.
4. After the user answers, re-check the frame.
5. Repeat the bundled clarification loop until every required field is filled, or abstain if the user refuses or the answer would still be unsafe.

Do not ask the user to define detailed retire thresholds, parameter grids, or failure taxonomies by default. The supervisor derives those from current artifacts, metrics, visuals or review artifacts, prior sessions, and reviewer notes.

If the frame is still too ambiguous to protect budget or artifacts, abstain instead of launching.

The supervisor must explicitly state `FRAME LOCKED` before the first session plan or execution step. If `FRAME LOCKED` has not been stated, the loop has not started.

## Shared Loop Station Folder

When a frame is locked, persist it in a shared folder so executor and reviewer agents can continue without asking the same questions again.

Preferred location:

```text
{loop_output_root}/loop_station/
```

Required shared files:

```text
loop_station/
  FRAME.md
  contract.json
  agent_roster.md
  sessions/
    session_{NNN}/
      executor_report.md
      executor_proposal.md
      decision.md
      reviewer_requests.md
      review_flags.md
  reviews/
    session_{NNN}/
      {AGENT_NAME}-SESSION{NNN}-REVIEWER-DONE.md
  flags/
    session_{NNN}/
      {AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}.flag
```

`contract.json` must include at least:

- `frame_locked: true`
- `goal`
- `budget`
- `work_unit_scope`
- `collaboration_mode`
- `user_intervention_points`
- `loop_output_root`
- `review_flag_format`
- `review_wait_policy`
- `current_session`

If `contract.json` exists and `frame_locked` is true, any later agent must reuse it. Reviewer agents must not ask the user to restate the goal, budget, work-unit scope, collaboration mode, or intervention boundaries unless the contract is internally contradictory or explicitly marked stale.

## Agent Modes

Loop Station supports two common modes.

### Supervisor / Executor Mode

Use this mode when the agent is allowed to run sessions or prepare the next session.

The executor must:

1. Read `loop_station/contract.json`, prior `decision.md`, prior reviewer notes, metrics, logs, and review artifacts.
2. Run or supervise the session within the locked frame.
3. Write `executor_report.md` with results, changed values, changed implementation variants, commands, metrics, artifacts, and failures.
4. Write `executor_proposal.md` with sharp analysis and the next proposed intervention.
5. Write an executor start flag before work, such as `CODEX5.5-SESSION050-EXECUTOR-RUNNING`.
6. Write an executor done flag after artifacts are complete, such as `CODEX5.5-SESSION050-EXECUTOR-DONE`.
7. Add reviewer requests at any time when a new review lens would improve the next decision.
8. Wait for required reviewer flags according to `review_wait_policy`, not indefinitely.
9. If reviewer timeout rules allow skipping, record the timeout and proceed.
10. Read available reviewer notes before creating the next session slate.

The executor's own report must include analysis, not only a metric dump. It should identify surprising results, harmful directions, promising directions, and why the next intervention is justified.

### Dynamic Reviewer Requests

The executor may request additional reviewers at any time during a session.

When adding a reviewer, the executor must update:

```text
loop_station/agent_roster.md
loop_station/sessions/session_{NNN}/reviewer_requests.md
```

Each reviewer request must include:

- requested reviewer agent name or reviewer class
- requested role: `REVIEWER` or `TESTER`
- session id
- reason this reviewer is useful
- artifacts to read
- expected output path
- timeout policy
- whether the reviewer is required or optional for the next session

The executor must also write a request flag:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{EXECUTOR_NAME}-SESSION{NNN}-SUPERVISOR-READY.flag
```

The request flag should point to `reviewer_requests.md`.

### Reviewer / Project Review Mode

Use this mode when another agent, including Claude Code, is asked to review the loop.

If the shared frame is locked, the reviewer must not ask the user for the frame again. The reviewer reads:

- `loop_station/FRAME.md`
- `loop_station/contract.json`
- the latest `executor_report.md`
- the latest `executor_proposal.md`
- changed values, code-variant manifests, diffs, metrics, logs, and review artifacts referenced by the executor
- prior reviewer notes for the session when present

The reviewer must not run the next session or modify code unless explicitly assigned `EXECUTOR`.

The reviewer should write only high-value review:

- whether the executor's interpretation is supported by evidence
- whether the changed values or implementation variant match the goal
- what direction should be promoted, kept, or retired
- what risk, confound, or missing validation could invalidate the next session
- one concise recommended next action or `ABSTAIN`

The reviewer output path should be:

```text
{loop_output_root}/loop_station/reviews/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-DONE.md
```

The reviewer must also write a flag:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-DONE.flag
```

Before doing review work, the reviewer should write a start flag:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-RUNNING.flag
```

If the reviewer cannot complete the review, it should write `BLOCKED` or `ABSTAIN` with a concise reason.

## Review Wait Policy

Reviewer waits must be bounded.

`contract.json` should define `review_wait_policy`, for example:

```json
{
  "review_wait_policy": {
    "required_done_count": 1,
    "optional_done_count": 0,
    "start_timeout_seconds": 300,
    "done_timeout_seconds": 1800,
    "heartbeat_timeout_seconds": 600,
    "allow_skip_on_timeout": true
  }
}
```

The executor must not wait forever for reviewers.

Use these rules:

1. After requesting review, wait up to `start_timeout_seconds` for at least one requested reviewer to write `RUNNING`, `DONE`, `BLOCKED`, or `ABSTAIN`.
2. If no reviewer writes any flag or review artifact before `start_timeout_seconds`, record `REVIEW_START_TIMEOUT` in `review_flags.md`.
3. After a reviewer writes `RUNNING`, wait up to `done_timeout_seconds` for `DONE`, `BLOCKED`, or `ABSTAIN`.
4. If a `RUNNING` reviewer writes no review artifact and no fresh `HEARTBEAT` flag within `heartbeat_timeout_seconds`, record `REVIEW_HEARTBEAT_TIMEOUT`.
5. If the required done count is met, proceed.
6. If timeout occurs and `allow_skip_on_timeout` is true, proceed with available evidence and write why the reviewer wait was skipped.
7. If timeout occurs and `allow_skip_on_timeout` is false, stop with `ask_user` or `ABSTAIN`.

Timeouts are reviewer coordination failures, not proof that the session result is invalid. The executor should continue when the locked budget and wait policy allow it, but must preserve the timeout record.

## Session Contract

Each session must produce a candidate slate, run or intentionally skip it, and end with an intervention decision.

At session end, the supervisor must:

1. Read metrics, logs, review artifacts, run configs, and reviewer notes.
2. Compare against baseline, current best, previous session, and user goal.
3. Classify directions as `promote`, `keep`, `retire`, `needs_more_evidence`, `needs_code_variant`, `stop`, or `ask_user`.
4. Decide the next action:
   - change important parameters
   - continue a promising axis
   - reduce or reverse a harmful axis
   - add an isolated implementation variant
   - expand or reduce work-unit scope
   - wait for review
   - stop or abstain
5. Write a session decision note with rationale.
6. Generate the next session slate only after the decision note exists.

## Implementation Variant Rule

Loop-driven implementation changes must not patch maintained source in place.

When a session needs code changes or script/workflow changes, create a separate loop-owned variant folder and run through that variant.

Preferred layouts:

```text
{project_loop_root}/code_variants/session_{NNN}/{change_slug}/
{loop_output_root}/code_variants/session_{NNN}/{change_slug}/
```

The variant folder must contain:

- copied or newly authored code/scripts/config adapters needed for the variant
- `manifest.json`
- `README.md` or `decision.md` explaining why config-only changes were insufficient
- a runnable entrypoint or config pointer

The `manifest.json` must record:

- source files copied from maintained code
- source file sha256 values before copy
- variant file paths
- session id
- change slug
- reason for the implementation variant
- command or config that uses the variant

Do not overwrite maintained runtime files during the loop. If direct source modification is unavoidable, stop and ask the user first. If the user explicitly approves direct source modification, create exact pre-edit backups and a restore manifest before editing.

## Agent Collaboration Flags

When collaborating with other agents, every flag must name the producing agent, session, role, and status.

Use this normalized format:

```text
{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}
```

Examples:

```text
CODEX5.5-SESSION050-SUPERVISOR-DONE
CODEX5.5-SESSION050-EXECUTOR-DONE
CLAUDE-SESSION050-REVIEWER-RUNNING
CLAUDE-SESSION050-REVIEWER-HEARTBEAT
REVIEWER_B-SESSION050-REVIEWER-BLOCKED
TESTER_A-SESSION050-TESTER-DONE
CODEX5.5-SESSION050-SUPERVISOR-TIMEOUT
```

Allowed roles:

- `SUPERVISOR`
- `EXECUTOR`
- `REVIEWER`
- `TESTER`

Allowed statuses:

- `READY`
- `RUNNING`
- `HEARTBEAT`
- `DONE`
- `BLOCKED`
- `ABSTAIN`
- `TIMEOUT`

Agents should write a `RUNNING` flag when they start work, then replace nothing. Long-running reviewers may add `HEARTBEAT` flags. They should add a separate terminal flag (`DONE`, `BLOCKED`, or `ABSTAIN`) when finished. The supervisor/executor may add `TIMEOUT` flags when bounded waits expire. Flags are append-only provenance.

Recommended flag locations:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-{ROLE}-{STATUS}.flag
{loop_output_root}/loop_station/sessions/session_{NNN}/review_flags.md
```

Each flag must include:

- producer agent name
- role
- session id
- timestamp
- artifact paths produced or reviewed
- next agent expected to consume it
- expected timeout or wait policy when relevant
- blocking reason if status is `BLOCKED` or `ABSTAIN`

No agent may overwrite another agent's flag. The supervisor may consume flags, but should preserve them as provenance.

## Resource Pool Rule

Before resource-heavy work, check the relevant resource pool and record the choice in the session artifact.

For GPU work, check current GPU idleness before launch. Use all available GPUs only when the variants can be partitioned cleanly. If resources are busy, wait, reduce the pool, or ask the user when budget assumptions change.

## Review Artifact Rule

Every meaningful session must leave a review artifact appropriate to the goal. Use visual panels for visual failure modes, metric tables for quantitative loops, diffs for code loops, transcripts for language loops, or other concrete artifacts that let a reviewer judge progress.

The supervisor must state which artifacts to inspect and what decision each artifact supports.

## Output Contract

Maintain these artifacts when feasible:

```text
loop_station/FRAME.md
loop_station/contract.json
loop_station/agent_roster.md
loop_station/sessions/session_{NNN}/executor_report.md
loop_station/sessions/session_{NNN}/executor_proposal.md
loop_station/sessions/session_{NNN}/decision.md
loop_station/sessions/session_{NNN}/reviewer_requests.md
loop_station/sessions/session_{NNN}/review_flags.md
loop_station/reviews/session_{NNN}/
loop_station/flags/session_{NNN}/
loop_state.json
leaderboard.csv
interventions.md
sessions/session_{NNN}/config.json
sessions/session_{NNN}/decision.md
visualizations/
FINAL_SUMMARY.md
```

Final summaries must separate:

- strict goal winner
- review-artifact winner
- rejected but informative candidates
- retired directions
- unvalidated generalization risks

## Failure Modes To Avoid

- Do not run adaptive retry until success.
- Do not ask the user for low-level grids when the agent can infer them.
- Do not hide code changes inside maintained source during a loop.
- Do not proceed after code-variant creation unless the manifest points back to the source state.
- Do not accept anonymous review flags.
- Do not treat one artifact's improvement as success when protection criteria fail, unless the user goal explicitly allows that tradeoff.
