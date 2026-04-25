---
name: loop-station
description: Use for bounded goal-directed loops that must first lock a goal, budget, work-unit scope, collaboration protocol, and user-intervention boundaries, then run adaptive sessions that inspect results, adjust strategy, isolate implementation variants, and stop or escalate without turning into open-ended retry. If invoked as a reviewer before executor/supervisor terminal flags exist, enter standby and use any available monitor/background watcher tool to poll loop_station flags instead of editing files or writing a premature review.
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
6. After execution finishes, write an executor terminal flag only after the result artifacts exist. Use `EXECUTOR-DONE` when `executor_report.md`, `executor_proposal.md`, metrics/log references, and produced artifacts are complete; otherwise use `EXECUTOR-BLOCKED` or `EXECUTOR-ABSTAIN` with the reason.
7. Add reviewer requests at any time when a new review lens would improve the next decision.
8. After requesting any reviewer, tester, or cooperating agent, enter the Sequential Collaboration Gate before writing `decision.md`, the next session slate, a final review summary, or any claim that review is complete.
9. Wait for required reviewer flags according to `review_wait_policy`, not indefinitely.
10. If reviewer timeout rules allow skipping, record the timeout and proceed.
11. Read available reviewer notes before creating the next session slate.

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

### Sequential Collaboration Gate

When the executor asks another agent to review, test, or collaborate, it must not immediately continue as if the review has happened. It must consume the collaboration flags in order.

The gate is mandatory for every requested reviewer or tester marked `required`, and recommended for optional reviewers when their result is expected to influence the next decision.

Required sequence:

1. Write or update `agent_roster.md` and `sessions/session_{NNN}/reviewer_requests.md`.
2. Write `{EXECUTOR_NAME}-SESSION{NNN}-SUPERVISOR-READY.flag` pointing to the request file, expected artifacts, and expected reviewer output path.
3. Check the exact `flags/session_{NNN}/` directory and expected `reviews/session_{NNN}/` path before proceeding.
4. Wait for a requested reviewer to write one of `RUNNING`, `DONE`, `BLOCKED`, or `ABSTAIN` within `start_timeout_seconds`.
5. If the reviewer writes `RUNNING`, keep waiting for a terminal flag: `DONE`, `BLOCKED`, or `ABSTAIN`. A `HEARTBEAT` only proves the reviewer is still active; it is not terminal.
6. If the reviewer writes `DONE`, verify that the expected review artifact exists and is readable. A `DONE` flag without the review artifact is incomplete and must be recorded in `review_flags.md`.
7. Read terminal review artifacts in flag timestamp order and record what was consumed in `review_flags.md`.
8. Only after the required done count is met, or after the locked timeout policy explicitly allows skipping, write `decision.md`, generate the next session slate, or summarize the review outcome.

If the executor cannot find the expected `loop_station/` folder, flag directory, or review artifact path, it must search the project for the active `loop_station/` folder and record the resolved path before continuing. If the path is still ambiguous, stop with `ask_user` instead of fabricating a review result.

### Reviewer / Project Review Mode

Use this mode when another agent, including Claude Code, is asked to review the loop.

If the shared frame is locked, the reviewer must not ask the user for the frame again and must not rewrite `FRAME.md`, `contract.json`, `agent_roster.md`, or executor-owned session artifacts. The reviewer may read them and may write only reviewer-owned flags and review artifacts unless explicitly assigned `EXECUTOR` or `SUPERVISOR`.

The reviewer reads:

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

### Reviewer Standby Mode

Use this mode when a reviewer is invoked before the executor has finished the current session.

The reviewer must not infer the frame and start editing loop files. If `FRAME.md` and `contract.json` already exist, treat them as Codex/executor-owned and read-only. If they do not exist, do not create them unless the user explicitly asks this reviewer to become the supervisor.

If the user asks for continuous waiting, real-time waiting, ongoing review, or "keep watching", the reviewer should immediately use the environment's persistent monitor, background watcher, automation, or equivalent long-running polling tool when available. Do this as part of entering standby, not only after the user repeats the request.

The watcher should poll for ready signals, not modify experiment files. It should emit or act only when a session becomes review-ready: executor terminal flag present, supervisor terminal flag present when required, and linked artifacts readable.

Standby sequence:

1. Locate the active `loop_station/` folder from the user's provided project, subject, experiment name, or output root.
2. Read `FRAME.md`, `contract.json`, `agent_roster.md`, `reviewer_requests.md`, and existing flags if they exist.
3. If continuous waiting is requested and a persistent monitor/background watcher is available, start it before writing any review.
4. Write a reviewer `RUNNING` flag only after the reviewer has resolved the target session and output paths.
5. Do not write `REVIEWER-DONE`, a review markdown file, a decision, a proposal, or any code/config changes until the executor terminal flag for the target session exists.
6. Poll `flags/session_{NNN}/` for `EXECUTOR-DONE`, `EXECUTOR-BLOCKED`, or `EXECUTOR-ABSTAIN`, and then wait for `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, or `SUPERVISOR-ABSTAIN` when the request asks for review after Codex analysis is complete.
7. A plain `EXECUTOR-RUNNING`, `SUPERVISOR-READY`, or partial report file is not enough to start review unless the user explicitly asks for live partial review.
8. When the needed terminal flags appear, verify that the linked `executor_report.md`, `executor_proposal.md`, metrics/logs, result images, and `decision.md` are readable before writing the review.
9. If no terminal flag appears yet, continue standby according to the user's wait instruction. Write at most short status updates; do not fabricate a review.

For continuous waiting, repeat the flag check in this order each poll:

```text
1. resolve active session number
2. check executor terminal flag
3. check supervisor terminal flag when required
4. verify linked artifacts are readable
5. emit one ready signal or write review only after the required terminal flags and artifacts exist
```

If the environment cannot keep a long-running wait alive, write a `REVIEWER-BLOCKED` flag explaining that the wait was interrupted and list the exact flag paths the user or executor should ping on. Do not convert an interrupted wait into a review.

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
    "sequential_flag_gate": true,
    "require_artifact_for_done": true,
    "continuous_monitor": {
      "enabled": false,
      "poll_interval_seconds": 60,
      "ready_signal_once_per_session": true
    },
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
5. If a reviewer writes `DONE`, verify that the expected review artifact exists before counting it toward `required_done_count`.
6. If `DONE` exists without a readable review artifact, record `REVIEW_DONE_WITHOUT_ARTIFACT` and continue waiting or apply the timeout policy.
7. If the required done count is met, proceed.
8. If timeout occurs and `allow_skip_on_timeout` is true, proceed with available evidence and write why the reviewer wait was skipped.
9. If timeout occurs and `allow_skip_on_timeout` is false, stop with `ask_user` or `ABSTAIN`.

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
5. If any required reviewer/tester/cooperating-agent request exists for the session, complete the Sequential Collaboration Gate before writing the session decision.
6. Write a session decision note with rationale.
7. Write a supervisor terminal flag for the session, such as `CODEX5.5-SESSION050-SUPERVISOR-DONE`, after `decision.md` exists and points to the executor report, proposal, consumed review artifacts, and next action. If the supervisor cannot make a decision, write `SUPERVISOR-BLOCKED` or `SUPERVISOR-ABSTAIN` with the reason.
8. Generate the next session slate only after both `decision.md` and the supervisor terminal flag exist.

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

For each executor-run session, Codex or any other executor must leave this minimum flag trail:

```text
{EXECUTOR_NAME}-SESSION{NNN}-EXECUTOR-RUNNING.flag
{EXECUTOR_NAME}-SESSION{NNN}-EXECUTOR-DONE.flag
{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-DONE.flag
```

Use `BLOCKED` or `ABSTAIN` instead of `DONE` when the expected result artifacts or decision artifacts are not complete. Do not write a `DONE` flag early. A `DONE` flag is valid only when the matching artifact listed inside the flag exists and is readable.

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
