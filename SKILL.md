---
name: loop-station
description: Use for live multi-agent feedback loops where executors and reviewers stay in standby, monitor loop_station flags, exchange evidence-backed feedback, maintain compact-ready rolling summaries, and move to the next session only after required flags and artifacts exist. First lock what to improve, run limits, session target/scope, agent roles, and what to ask before doing; then run bounded adaptive sessions without open-ended retry. If invoked as a reviewer before EXECUTOR-DONE and SUPERVISOR-READY exist, enter standby and use any available monitor/background watcher tool instead of editing files or writing a premature review.
---

# Loop Station

## Language Rule

The skill body, durable loop contracts, manifests, flags, and session decision notes should be written in English by default so they stay portable across agents and repositories.

The user may speak Korean, English, or both. Match the user's language in chat, but keep the core loop artifacts and stable protocol wording in English unless the user explicitly asks otherwise.

## Hard Rule

Do not start from variants. Start from the decision frame.

Before launching any loop, obtain or infer the core user brief. Accept plain
language. The user does not need to use internal protocol field names.

- goal: what the loop must improve or decide
- budget: max sessions, resource pool, wall-time, cost, or other hard limit
- evidence: metrics, logs, images, tests, checks, or other signals that should guide decisions

Optional/custom fields can include session target/scope, reviewer roles,
artifact paths, tools, permissions, ask-before rules, summary requirements, or
anything else the user wants to control.

When writing `contract.json`, store the core brief as `goal`, `budget`, and
`evidence`. Store optional fields only when provided or safely inferred, such as
`work_unit_scope`, `collaboration_mode`, and `user_intervention_points`.

Ask only for missing core fields when they cannot be safely inferred. Do not
block on optional fields by default.

When fields are missing:

1. Infer what is safe from the current request, repo context, previous artifacts, and explicit user constraints.
2. Present the inferred frame and mark unknowns as `missing`.
3. Ask one bundled clarification covering only the missing core fields.
4. After the user answers, re-check the frame.
5. Repeat the bundled clarification loop until every core field is filled, or abstain if the user refuses or the answer would still be unsafe.

Do not ask the user to define detailed retire thresholds, parameter grids, or failure taxonomies by default. The supervisor derives those from current artifacts, metrics, visuals or review artifacts, prior sessions, and reviewer notes.

If the frame is still too ambiguous to protect budget or artifacts, abstain instead of launching.

The supervisor must explicitly state `FRAME LOCKED` before the first session plan or execution step. If `FRAME LOCKED` has not been stated, the loop has not started.

## Budget And Pivot Rule

Treat `max_sessions` or a requested session count as usable loop budget, not as a
reason to close early. A session budget may end early only when the user gave an
explicit stop condition, the budget is actually exhausted, the user asks to stop,
or every viable next axis is blocked and the supervisor writes `ask_user` or
`ABSTAIN` with evidence.

Repeated bad trends, a zero-promotion streak, or falsification of the current
direction means the current axis is exhausted. It does not mean the loop is
complete while budget remains.

Likewise, an early candidate that beats baseline is a good signal, not a
finish line. Finding improvement is what the loop is for, not a reason to
stop. After a promotion, the loop must continue: validate the candidate on
the rest of the work-unit scope, probe robustness on perturbations (subjects,
views, seeds, hyperparameters), run ablations isolating which mechanism
carries the gain, test alternative axes that the contract or prior summaries
flagged as deferred, and look for failure modes that a single-session result
cannot reveal. A premature stop on the first apparent winner usually produces
a result that does not generalize — exactly the failure pattern the budget
exists to prevent. If a candidate looks decisive enough that continuing seems
wasteful, ask the user before stopping; do not stop and then ask. "It got
better" is a reason to keep investigating, not a reason to close.

When an axis becomes unpromising:

1. Identify the best known candidate/checkpoint so far and the evidence that made
   it best.
2. Mark the harmful or exhausted axis as retired, superseded, or
   `do_not_continue`; do not stack more changes on top of a bad branch.
3. Return to or branch from the best known candidate using loop-owned variants,
   backups, or restore manifests.
4. Pick a different axis from open questions, reviewer notes, confounds, visual
   failures, code/config audit findings, or untested mechanisms.
5. Record the pivot rationale, retired direction, selected restart point, and
   remaining session budget in `decision.md`, `ROLLING_SUMMARY.md`,
   `SESSION_LEDGER.md`, and `COMPACT_HANDOFF.md`.
6. Start the next session from that pivot after `SUPERVISOR-DONE` or a repaired
   continuation point such as `SUPERVISOR-REPAIRED`.

Do not write a final close/phase-complete decision simply because the current
axis failed. Use `stop` only for true stop conditions. Use `pivot_axis`,
`retire_axis`, `return_to_best`, or `needs_new_axis` when budget remains.

If an agent already wrote a premature close marker or decision, preserve it as
provenance. Do not delete it. Write a repair/superseded note, update the contract
or summary state to reopened/continuing, and resume from the best known candidate
with the next session number.

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
  summaries/
    ROLLING_SUMMARY.md
    SESSION_LEDGER.md
    COMPACT_HANDOFF.md
    EXECUTOR_BRIEF.md
    LOG_TREND_SUMMARY.md
    REVIEWER_ROLLUP.md
    compact/
      session_{NNN}-{AGENT_NAME}.md
  sessions/
    session_{NNN}/
      executor_report.md
      executor_proposal.md
      supervisor_analysis.md
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
- `summary_policy`
- `compaction_policy`

If `contract.json` exists and `frame_locked` is true, any later agent must reuse it. Reviewer agents must not ask the user to restate the goal, budget, session scope, roles, or ask-before rules unless the contract is internally contradictory or explicitly marked stale.

Bundled templates use lowercase filenames. When used, copy or render them to the
runtime artifact names:

```text
templates/FRAME.md -> loop_station/FRAME.md
templates/contract.json -> loop_station/contract.json
templates/agent_roster.md -> loop_station/agent_roster.md
templates/rolling_summary.md -> loop_station/summaries/ROLLING_SUMMARY.md
templates/session_ledger.md -> loop_station/summaries/SESSION_LEDGER.md
templates/compact_handoff.md -> loop_station/summaries/COMPACT_HANDOFF.md
templates/executor_brief.md -> loop_station/summaries/EXECUTOR_BRIEF.md
templates/log_trend_summary.md -> loop_station/summaries/LOG_TREND_SUMMARY.md
templates/reviewer_rollup.md -> loop_station/summaries/REVIEWER_ROLLUP.md
templates/executor_report.md -> loop_station/sessions/session_{NNN}/executor_report.md
templates/executor_proposal.md -> loop_station/sessions/session_{NNN}/executor_proposal.md
templates/supervisor_analysis.md -> loop_station/sessions/session_{NNN}/supervisor_analysis.md
templates/reviewer_requests.md -> loop_station/sessions/session_{NNN}/reviewer_requests.md
templates/review_flags.md -> loop_station/sessions/session_{NNN}/review_flags.md
templates/reviewer_review.md -> loop_station/reviews/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-DONE.md
```

## Rolling Summary And Compact Rule

Session-wise artifacts are the source of truth. The shared `summaries/` folder is the compact-ready index that lets Codex, Claude Code, and later agents resume quickly after context compaction.

Maintain these files throughout the loop:

```text
loop_station/summaries/ROLLING_SUMMARY.md
loop_station/summaries/SESSION_LEDGER.md
loop_station/summaries/COMPACT_HANDOFF.md
loop_station/summaries/EXECUTOR_BRIEF.md
loop_station/summaries/LOG_TREND_SUMMARY.md
loop_station/summaries/REVIEWER_ROLLUP.md
loop_station/summaries/compact/session_{NNN}-{AGENT_NAME}.md
```

Before any agent intentionally compacts context, ends a long standby period, or asks the environment to compact, it must update the relevant summary files first.

Codex or the active supervisor must update `ROLLING_SUMMARY.md`, `SESSION_LEDGER.md`, and `COMPACT_HANDOFF.md` after every completed session decision and before writing `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, `SUPERVISOR-ABSTAIN`, or `SUPERVISOR-REPAIRED`.

Reviewer agents, including Claude Code, must update `REVIEWER_ROLLUP.md`, `LOG_TREND_SUMMARY.md`, `EXECUTOR_BRIEF.md`, and a per-compact note under `summaries/compact/` before writing `REVIEWER-DONE`, `REVIEWER-BLOCKED`, or `REVIEWER-ABSTAIN` when they have enough context and file-write access to summarize. These files are reviewer-writable handoff artifacts. Reviewers must not edit executor-owned session artifacts while doing this.

The reviewer-owned summaries are a token-saving handoff for the executor. The executor should not reread all historical logs by default when starting a new session. It should first read `EXECUTOR_BRIEF.md`, `LOG_TREND_SUMMARY.md`, `ROLLING_SUMMARY.md`, `SESSION_LEDGER.md`, and the latest decision/review artifacts. It should open older raw logs only when the summaries conflict, a claim needs verification, or the next experiment depends on a specific historical detail.

After a clean session boundary, an agent should compact or recommend compaction when one of these is true:

- the current conversation has accumulated enough context that future reasoning may degrade
- at least `compact_every_sessions` sessions have completed since the last compact handoff
- a long reviewer standby or multi-session review has completed
- the next session can be planned entirely from `FRAME.md`, `contract.json`, `summaries/`, and the latest session artifacts

Do not compact in the middle of execution, review, or supervisor synthesis. Compact only after durable artifacts and summaries are written.

`COMPACT_HANDOFF.md` must be short and operational. It should state:

- current goal, scope, budget remaining, and resource assumptions
- current best candidate and why
- rejected or retired directions
- unresolved risks, confounds, and missing validation
- latest reviewer conclusions
- exact next session recommendation
- paths that the next agent should read first

When resuming after compaction, read in this order:

1. `loop_station/contract.json`
2. `loop_station/FRAME.md`
3. `loop_station/summaries/COMPACT_HANDOFF.md`
4. `loop_station/summaries/EXECUTOR_BRIEF.md`
5. `loop_station/summaries/LOG_TREND_SUMMARY.md`
6. `loop_station/summaries/ROLLING_SUMMARY.md`
7. `loop_station/summaries/SESSION_LEDGER.md`
8. latest `loop_station/sessions/session_{NNN}/decision.md`
9. latest relevant reviewer artifact

If summaries disagree with session artifacts, trust the session artifacts and repair the summaries before continuing.

## Agent Modes

Loop Station supports two common modes.

### Supervisor / Executor Mode

Use this mode when the agent is allowed to run sessions or prepare the next session.

The executor must:

1. Read `loop_station/contract.json`, `loop_station/summaries/COMPACT_HANDOFF.md`, `loop_station/summaries/EXECUTOR_BRIEF.md`, `loop_station/summaries/LOG_TREND_SUMMARY.md`, prior `decision.md`, prior reviewer notes, metrics summaries, and review artifacts. Do not reread all historical raw logs by default.
2. Write an executor start flag before work, such as `CODEX5.5-SESSION050-EXECUTOR-RUNNING`.
3. Run or supervise the session within the locked frame.
4. Write `executor_report.md` with results, changed values, changed implementation variants, commands, metrics, artifacts, and failures.
5. Write `executor_proposal.md` with sharp analysis and the next proposed intervention.
6. After execution finishes, write an executor terminal flag only after the result artifacts exist. Use `EXECUTOR-DONE` when `executor_report.md`, `executor_proposal.md`, metrics/log references, and produced artifacts are complete; otherwise use `EXECUTOR-BLOCKED` or `EXECUTOR-ABSTAIN` with the reason.
7. Hand off to the supervisor self-review phase before external reviewers are allowed to act.
8. After requesting any reviewer, tester, or cooperating agent, enter the Sequential Collaboration Gate before writing `decision.md`, the next session slate, a final review summary, or any claim that review is complete.
9. Wait for required reviewer flags according to `review_wait_policy`, not indefinitely.
10. If reviewer timeout rules allow skipping, record the timeout and proceed.
11. Read available reviewer notes before creating the next session slate.

The executor's own report must include analysis, not only a metric dump. It should identify surprising results, harmful directions, promising directions, and why the next intervention is justified.

The executor should not spend execution budget browsing papers, searching prior art, or doing literature research unless the user explicitly assigns that to the executor. For normal loops, put literature/method questions into `reviewer_requests.md` and let the external reviewer handle them.

### Supervisor Self-Review Phase

After `EXECUTOR-DONE` and before `SUPERVISOR-READY`, Codex or the active supervisor must review its own session output. This is not the external reviewer step.

The supervisor must:

1. Write `SUPERVISOR-RUNNING` when it starts self-review.
2. Read `executor_report.md`, `executor_proposal.md`, metrics, logs, result images, code diffs, manifests, and prior decisions.
3. Use sub-agents or helper reviewers when available to verify generated code, metrics, visual artifacts, logs, and variant summaries.
4. Write `supervisor_analysis.md` with the supervisor's own explanation, trend interpretation, verification notes, risks, and recommended reviewer questions.
5. Update `reviewer_requests.md` with exact artifacts the external reviewer should inspect, including any literature/method questions the reviewer should investigate.
6. Only after `supervisor_analysis.md`, `executor_report.md`, `executor_proposal.md`, and referenced artifacts are readable, write `SUPERVISOR-READY`.

`SUPERVISOR-READY` means Codex has already executed the session and completed its own analysis/verification. It does not mean the session is finally decided. The final session decision comes later, after reviewer feedback is consumed.

### Dynamic Reviewer Requests

The supervisor may draft additional reviewer requests after executor artifacts exist and Codex self-review has started. A draft request does not activate the reviewer. External reviewer handoff is activated only after `supervisor_analysis.md` exists, `reviewer_requests.md` is updated, and `SUPERVISOR-READY` is written.

When adding a reviewer, the supervisor must update:

```text
loop_station/agent_roster.md
loop_station/sessions/session_{NNN}/reviewer_requests.md
```

Each activated reviewer request must include:

- requested reviewer agent name or reviewer class
- requested reviewer model/version when known
- reviewer instruction profile, such as `scientific reviewer`, `visual auditor`, `code auditor`, or `metric/log analyst`
- requested role: `REVIEWER` or `TESTER`
- session id
- reason this reviewer is useful
- artifacts to read
- expected output path
- timeout policy
- whether the reviewer is required or optional for the next session

The supervisor must also write a request flag after self-review is complete:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-READY.flag
```

The request flag should point to `reviewer_requests.md`, `supervisor_analysis.md`, `executor_report.md`, `executor_proposal.md`, and the artifacts the reviewer must inspect.

### Sequential Collaboration Gate

When the supervisor asks another agent to review, test, or collaborate, it must not immediately continue as if the review has happened. It must consume the collaboration flags in order.

The gate is mandatory for every requested reviewer or tester marked `required`, and recommended for optional reviewers when their result is expected to influence the next decision.

Strict ordering invariant:

- Do not write `SUPERVISOR-DONE`, prepare the next session slate, or start the
  next `EXECUTOR-RUNNING` until every required reviewer has a terminal flag
  (`REVIEWER-DONE`, `REVIEWER-BLOCKED`, `REVIEWER-ABSTAIN`) or the locked timeout
  policy has been recorded, and the rolling summaries have been updated.
- For a `REVIEWER-DONE` terminal flag, the matching review artifact must exist,
  be readable, be read by Codex, and be recorded in `review_flags.md` before
  `decision.md` and `SUPERVISOR-DONE`.
- If an existing `SUPERVISOR-DONE` timestamp is earlier than a required
  `REVIEWER-DONE` timestamp for the same session, treat the session as
  out-of-order. Do not overwrite the early flag. Write `SUPERVISOR-VIOLATION`,
  consume the review, update `decision.md` or write a repair decision, record the
  repair in `review_flags.md`, update summaries, then write `SUPERVISOR-REPAIRED`
  before any next session preparation.

Required sequence:

1. Write or update `agent_roster.md` and `sessions/session_{NNN}/reviewer_requests.md`.
2. Write `{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-READY.flag` only after the Supervisor Self-Review Phase is complete, pointing to the request file, `supervisor_analysis.md`, expected artifacts, and expected reviewer output path.
3. Check the exact `flags/session_{NNN}/` directory and expected `reviews/session_{NNN}/` path before proceeding.
4. Wait for a requested reviewer to write one of `RUNNING`, `DONE`, `BLOCKED`, or `ABSTAIN` within `start_timeout_seconds`.
5. If the reviewer writes `RUNNING`, keep waiting for a terminal flag: `DONE`, `BLOCKED`, or `ABSTAIN`. A `HEARTBEAT` only proves the reviewer is still active; it is not terminal.
6. If the reviewer writes `DONE`, verify that the expected review artifact exists and is readable. A `DONE` flag without the review artifact is incomplete and must be recorded in `review_flags.md`.
7. Read terminal review artifacts in flag timestamp order and record what was consumed in `review_flags.md`.
8. Only after the required done count is met, or after the locked timeout policy explicitly allows skipping, and after the strict ordering invariant is satisfied, write `decision.md`, generate the next session slate, or summarize the review outcome.

If the executor cannot find the expected `loop_station/` folder, flag directory, or review artifact path, it must search the project for the active `loop_station/` folder and record the resolved path before continuing. If the path is still ambiguous, stop with `ask_user` instead of fabricating a review result.

### Reviewer / Project Review Mode

Use this mode when another agent, including Claude Code, is asked to review the loop.

If the shared frame is locked, the reviewer must not ask the user for the frame again and must not rewrite `FRAME.md`, `contract.json`, `agent_roster.md`, or executor-owned session artifacts. The reviewer may read them and may write only reviewer-owned flags, review artifacts, reviewer-owned summaries (`REVIEWER_ROLLUP.md`, `LOG_TREND_SUMMARY.md`, `EXECUTOR_BRIEF.md`, and compact notes), unless explicitly assigned `EXECUTOR` or `SUPERVISOR`.

The review request should identify the reviewer as a separate reviewer identity: a different model/version, a different agent, or an explicit instruction profile. Examples: `Claude Code scientific reviewer`, `GPT-5.5 xhigh reviewer`, `vision-focused auditor`, `code/config auditor`, or `metric/log analyst`. If the model/version is unknown, the request must still state the reviewer instruction profile and that the agent is acting as `REVIEWER`.

The reviewer reads:

- `loop_station/FRAME.md`
- `loop_station/contract.json`
- the latest `executor_report.md`
- the latest `executor_proposal.md`
- the latest `supervisor_analysis.md`
- changed values, code-variant manifests, diffs, metrics, logs, and review artifacts referenced by the executor
- prior reviewer notes for the session when present

The reviewer must not run the next session or modify code unless explicitly assigned `EXECUTOR`.

The reviewer is an active critic, not a passive summarizer. After the session is
review-ready, it should decide which checks are useful for the project and use
available tools to challenge the executor's interpretation. For research,
scientific, method-dependent, or paper-backed projects, online search and local
paper/prior-method checks are recommended when they can improve the quality of
the per-session review. The reviewer should decide this automatically after
reading the session result, without waiting for the executor to request each
source. Use outside evidence only when it can change the next decision, cite the
sources in the review artifact, and turn the finding into a concrete mechanism,
objection, or next-session proposal.

Reviewer and audit work may include multiple evidence axes. Use the axes that fit the goal and available artifacts:

- visual image checks: compare rendered frames, panels, crops, masks, failure examples, and current-best outputs
- metric audit: compare target metrics against prior sessions, anchors, variance, and known confounds
- code/config audit: inspect generated code, scripts, config changes, code variants, manifests, and diffs
- log/artifact audit: verify commands, seeds, resource use, failed runs, missing files, stale outputs, and readable artifact paths
- scientific analysis: challenge weak explanations, explain plausible mechanisms,
  tradeoffs, hidden regressions, and why the attempted improvement may not have
  worked
- literature/method check: for research or method-dependent work, online search
  or local papers are recommended when useful for review quality; summarize only
  the parts that change the next decision and cite sources in the reviewer
  artifact

When sub-agents or helper reviewers are available, the reviewer may split the audit by axis, for example one sub-agent for code/config, one for visual outputs, one for metrics/logs, and one for artifact completeness. The final reviewer artifact must integrate those findings into one coherent recommendation.

The reviewer should write only high-value review:

- whether the executor's interpretation is supported by evidence
- visual observations that support or contradict metric changes
- code/config concerns that could invalidate the result
- whether the changed values or implementation variant match the goal
- a scientific explanation for why the current intervention did or did not work,
  including literature-backed hypotheses when useful
- what direction should be promoted, kept, or retired
- what risk, confound, or missing validation could invalidate the next session
- one concise recommended next action or `ABSTAIN`

The reviewer output path should be:

```text
{loop_output_root}/loop_station/reviews/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-DONE.md
```

Before writing a terminal reviewer flag, the reviewer must update reviewer-owned
summaries when file writes are available, or state in the terminal flag why
summary updates were skipped.

Before writing `REVIEWER-DONE`, a long-running reviewer must also check monitor
state for the next session. If continuous watching should continue, reuse or
retarget the existing monitor/background watcher for the same `loop_station/`
root instead of starting another one. If duplicate monitors already exist, stop
or disable duplicates when the environment supports it; otherwise record the
duplicate monitor ids/names in the review artifact and terminal flag so the user
or next reviewer can clean them up.

The reviewer must then write a terminal flag:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-DONE.flag
```

The reviewer should read prior summaries and the relevant historical logs/artifacts needed to detect trends, then summarize those trends for the executor. This lets the executor design the next session from the brief instead of spending tokens rereading every prior log.

Before doing review work, the reviewer should write a start flag only after the
session is review-ready and linked artifacts are readable:

```text
{loop_output_root}/loop_station/flags/session_{NNN}/{AGENT_NAME}-SESSION{NNN}-REVIEWER-RUNNING.flag
```

If the reviewer cannot complete the review, it should write `BLOCKED` or `ABSTAIN` with a concise reason.

### Reviewer Standby Mode

Use this mode when a reviewer is invoked before the executor has finished the current session.

The reviewer must not infer the frame and start editing loop files. If `FRAME.md` and `contract.json` already exist, treat them as Codex/executor-owned and read-only. If they do not exist, do not create them unless the user explicitly asks this reviewer to become the supervisor.

If the user asks for continuous waiting, real-time waiting, ongoing review, or "keep watching", the reviewer should immediately use the environment's persistent monitor, background watcher, automation, or equivalent long-running polling tool when available. Before creating one, check whether a monitor already exists for the same `loop_station/` root, session pattern, or reviewer identity; reuse it when possible and do not create parallel duplicate watchers. Do this as part of entering standby, not only after the user repeats the request.

The watcher should poll for ready signals, not modify experiment files. It should emit or act only when a session becomes review-ready: `EXECUTOR-DONE` present, `SUPERVISOR-READY` present, and linked executor artifacts readable. Wait for `SUPERVISOR-DONE` only for explicit post-decision audit requests.

Standby sequence:

1. Locate the active `loop_station/` folder from the user's provided project, subject, experiment name, or output root.
2. Read `FRAME.md`, `contract.json`, `agent_roster.md`, `reviewer_requests.md`, and existing flags if they exist.
3. If continuous waiting is requested and a persistent monitor/background watcher is available, reuse an existing watcher for the same loop when one exists; otherwise start exactly one before writing any review.
4. If a visible waiting signal is useful, write `REVIEWER-STANDBY`; do not write
   `REVIEWER-RUNNING` yet. `STANDBY` does not start the reviewer done timeout.
5. Do not write `REVIEWER-RUNNING`, `REVIEWER-DONE`, a review markdown file, a
   decision, a proposal, or any code/config changes until the executor terminal
   flag and `SUPERVISOR-READY` for the target session exist.
6. Poll `flags/session_{NNN}/` for `EXECUTOR-DONE`, `EXECUTOR-BLOCKED`, or `EXECUTOR-ABSTAIN`, then wait for `SUPERVISOR-READY` when the request asks for Claude/reviewer input before Codex writes the final decision. Wait for `SUPERVISOR-DONE`, `SUPERVISOR-BLOCKED`, or `SUPERVISOR-ABSTAIN` only when the user explicitly asks for a post-decision audit.
7. A plain `EXECUTOR-RUNNING`, partial report file, or `SUPERVISOR-READY` without `EXECUTOR-DONE` is not enough to start review unless the user explicitly asks for live partial review.
8. When the needed ready flags appear, verify that the linked `executor_report.md`, `executor_proposal.md`, `supervisor_analysis.md`, metrics/logs, and result images are readable, then write `REVIEWER-RUNNING` before writing the review. Require `decision.md` only for explicit post-decision audit.
9. If no terminal flag appears yet, continue standby according to the user's wait instruction. Write at most short status updates; do not fabricate a review.

For continuous waiting, repeat the flag check in this order each poll:

```text
1. resolve active session number
2. check executor terminal flag
3. check `SUPERVISOR-READY` for normal review, or supervisor terminal flag only for post-decision audit
4. verify linked artifacts are readable
5. emit one ready signal or write review only after the required flags and artifacts exist
```

At the end of each reviewer turn, audit monitor state again before continuing to
the next session. There should normally be one active continuous watcher per
reviewer and loop root. Duplicates should be closed, disabled, or recorded with
their ids and reason.

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
      "ready_signal_once_per_session": true,
      "dedupe_by_loop_root_and_reviewer": true
    },
    "allow_skip_on_timeout": true
  }
}
```

The executor must not wait forever for reviewers.

Use these rules:

1. After activating review with `SUPERVISOR-READY`, track start/done/heartbeat
   timeout separately for each requested required reviewer or tester.
2. For each required reviewer, wait up to `start_timeout_seconds` for `RUNNING`,
   `DONE`, `BLOCKED`, or `ABSTAIN`. A pre-ready `STANDBY` or `HEARTBEAT` does
   not count as `RUNNING` and does not start `done_timeout_seconds`.
3. If a required reviewer writes no start or terminal flag before
   `start_timeout_seconds`, record `REVIEW_START_TIMEOUT:{reviewer}` in
   `review_flags.md`.
4. After a reviewer writes `RUNNING`, wait up to `done_timeout_seconds` for
   `DONE`, `BLOCKED`, or `ABSTAIN`.
5. If a `RUNNING` reviewer writes no review artifact and no fresh `HEARTBEAT`
   flag within `heartbeat_timeout_seconds`, record
   `REVIEW_HEARTBEAT_TIMEOUT:{reviewer}`.
6. If a reviewer writes `DONE`, verify that the expected review artifact exists
   and that the flag states reviewer summary updates are complete or skipped with
   a reason before counting it toward `required_done_count`.
7. If `DONE` exists without a readable review artifact, record
   `REVIEW_DONE_WITHOUT_ARTIFACT:{reviewer}` and continue waiting or apply the
   timeout policy.
8. Proceed only after every required reviewer has either a valid terminal result
   or a recorded timeout allowed by policy, and the required done count is met or
   explicitly skipped.
9. If timeout occurs and `allow_skip_on_timeout` is true, proceed with available
   evidence and write why the reviewer wait was skipped.
10. If timeout occurs and `allow_skip_on_timeout` is false, stop with `ask_user`
   or `ABSTAIN`.

Preserve timeout records in `review_flags.md`. Continue only when the locked
budget and wait policy allow it.

## Session Contract

Each session must produce a candidate slate, run or intentionally skip it, and end with an intervention decision.

Before external reviewer handoff, the supervisor must:

1. Read metrics, logs, result artifacts, run configs, executor reports, executor proposals, and prior reviewer notes.
2. Compare against baseline, current best, previous session, and user goal.
3. Classify directions as `promote`, `keep`, `retire`, `pivot_axis`,
   `return_to_best`, `needs_more_evidence`, `needs_code_variant`, `stop`, or
   `ask_user`.
4. Draft a provisional next direction for reviewer inspection:
   - change important parameters
   - continue a promising axis
   - reduce or reverse a harmful axis
   - return to the best known candidate and pivot to a new axis
   - add an isolated implementation variant
   - expand or reduce work-unit scope
   - wait for review
   - stop or abstain only when a true stop condition is met
5. Write `supervisor_analysis.md` with rationale, verification notes, and the review questions.
6. Write `SUPERVISOR-READY` for external review.

After external reviewer feedback is available or skipped by policy, the supervisor must:

1. Complete the Sequential Collaboration Gate for any required reviewer/tester/cooperating-agent request.
2. Read consumed review artifacts and record them in `review_flags.md`.
3. Write `decision.md` with the integrated judgment, remaining budget, best known
   candidate, retired axes, pivot decision when relevant, and next action.
4. Update `summaries/ROLLING_SUMMARY.md`, `summaries/SESSION_LEDGER.md`, and `summaries/COMPACT_HANDOFF.md`.
5. Write a supervisor terminal flag for the session, such as `CODEX5.5-SESSION050-SUPERVISOR-DONE`, after `decision.md` exists, summaries are updated, and the flag points to the executor report, proposal, supervisor analysis, consumed review artifacts, summaries, and next action. If the supervisor cannot make a decision, write `SUPERVISOR-BLOCKED` or `SUPERVISOR-ABSTAIN` with the reason.
6. Generate the next session slate only after both `decision.md` and the supervisor terminal flag exist.

## Implementation Variant Rule

Loop-driven implementation changes must not patch maintained source in place.

When a session needs code changes or script/workflow changes, create a separate loop-owned variant folder and run through that variant. Treat original source files, original configs, and original data as protected inputs.

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
- whether each source file was copied, left untouched, or directly modified with approval
- variant file paths
- session id
- change slug
- reason for the implementation variant
- command or config that uses the variant

Do not overwrite maintained runtime files during the loop. If direct source modification is unavoidable, stop and ask the user first. Before any maintained-source edit, the approval reference must be recorded in `contract.json`, `decision.md`, or `reviewer_requests.md`; exact pre-edit backups must exist; and a restore manifest must be created at `loop_station/sessions/session_{NNN}/restore_manifest.json`. The restore manifest must list every edited file, backup path, pre-edit sha256, post-edit sha256 when available, restore command or procedure, approval reference, and session id.

Reviewers should audit implementation sessions for original protection:

- variant path exists and is outside maintained source
- source hashes were recorded before copy
- direct edits have explicit user approval
- pre-edit backups exist for every directly edited maintained file
- restore manifest is readable and points to restorable files

## Agent Collaboration Flags

When collaborating with other agents, every flag must name the producing agent, session, role, and status.

The canonical reviewed-session lifecycle is mandatory:

```text
1. EXECUTOR-RUNNING
   The executor starts the session.

2. EXECUTOR-DONE
   The executor has produced result artifacts, metrics/log references,
   executor_report.md, and executor_proposal.md.

3. Internal validation / sub-agent checks
   Codex or the supervisor reviews the result, uses sub-agents when available,
   verifies code/metrics/images/logs/variants, and writes supervisor_analysis.md.

4. REVIEWER request
   The supervisor writes reviewer_requests.md and SUPERVISOR-READY.
   This is the handoff point to Claude or another external reviewer.

5. REVIEWER-RUNNING
   The reviewer starts external review.

6. REVIEWER-DONE
   The reviewer writes the review artifact and done flag.

7. Codex consumes review
   Codex checks the reviewer flags, reads the review artifact, and records
   consumed reviewer artifacts in review_flags.md.

8. SUPERVISOR-DONE
   Codex writes decision.md, updates summaries, then writes the terminal flag.

9. Compact checkpoint
   Codex, Claude, or another agent may compact only after durable artifacts and
   summaries exist.

10. Prepare next session
   Codex prepares the next session slate only after SUPERVISOR-DONE.

11. Next session starts
   The next session begins with the next EXECUTOR-RUNNING flag.
```

In the concrete flag trail, `SUPERVISOR-RUNNING` is the flag form of step 3:
Codex internal validation and optional sub-agent checks.

Do not reorder these phases. In particular, `SUPERVISOR-READY` is the reviewer request signal and is written only after Codex has completed its own self-review/verification and `supervisor_analysis.md` is readable. `SUPERVISOR-DONE` is written only after reviewer output is consumed, `decision.md` exists, and rolling summaries are updated. The next session must not start before `SUPERVISOR-DONE`.

If `SUPERVISOR-DONE` appears before the required reviewer terminal flag for the
same session, the session is invalid until repaired. Codex must not use that
decision to prepare the next session. The repair path is
`SUPERVISOR-VIOLATION -> consume review -> update decision/repair artifact ->
SUPERVISOR-REPAIRED`.

Use this normalized format:

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
CLAUDE-SESSION050-REVIEWER-HEARTBEAT
CLAUDE-SESSION050-REVIEWER-DONE
CODEX5.5-SESSION050-SUPERVISOR-DONE
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
- `STANDBY`
- `RUNNING`
- `HEARTBEAT`
- `DONE`
- `BLOCKED`
- `ABSTAIN`
- `TIMEOUT`
- `VIOLATION`
- `REPAIRED`

Agents should write a `RUNNING` flag when they start active work, then replace nothing. Reviewers invoked before review-ready may write `STANDBY`; `STANDBY` means holding/watching and must not be counted as active review. Long-running reviewers may add `HEARTBEAT` flags. They should add a separate terminal flag (`DONE`, `BLOCKED`, or `ABSTAIN`) when finished. The supervisor/executor may add `TIMEOUT` flags when bounded waits expire. `VIOLATION` and `REPAIRED` are reserved for protocol-order repair. Flags are append-only provenance.

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

For an unreviewed supervisor-only session, Codex or any other executor must leave this minimum flag trail:

```text
{EXECUTOR_NAME}-SESSION{NNN}-EXECUTOR-RUNNING.flag
{EXECUTOR_NAME}-SESSION{NNN}-EXECUTOR-DONE.flag
{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-RUNNING.flag
{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-DONE.flag
```

For a reviewed session, the normal flag trail is:

```text
{EXECUTOR_NAME}-SESSION{NNN}-EXECUTOR-RUNNING.flag
{EXECUTOR_NAME}-SESSION{NNN}-EXECUTOR-DONE.flag
{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-RUNNING.flag
{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-READY.flag
{REVIEWER_NAME}-SESSION{NNN}-REVIEWER-RUNNING.flag
{REVIEWER_NAME}-SESSION{NNN}-REVIEWER-DONE.flag
{SUPERVISOR_NAME}-SESSION{NNN}-SUPERVISOR-DONE.flag
```

Use `BLOCKED` or `ABSTAIN` instead of `DONE` when the expected result artifacts or decision artifacts are not complete. Do not write a `DONE` flag early. A `DONE` flag is valid only when the matching artifact listed inside the flag exists and is readable.

## Resource Pool Rule

Before resource-heavy work, check the relevant resource pool and record the choice in the session artifact.

For GPU work, check current GPU idleness before launch. Use all available GPUs only when the variants can be partitioned cleanly. If resources are busy, wait, reduce the pool, or ask the user when budget assumptions change.

## Review Artifact Rule

Every meaningful session must leave a review artifact appropriate to the goal. Use visual panels for visual failure modes, metric tables for quantitative loops, diffs for code loops, transcripts for language loops, or other concrete artifacts that let a reviewer judge progress.

The supervisor must state which artifacts to inspect and what decision each artifact supports. If the session produced images, visual comparisons, or rendered outputs, the review request should explicitly ask for visual image checks. If the session changed code, scripts, configs, or variants, the review request should explicitly ask for code/config audit. If the review is broad enough to benefit from decomposition, the request may ask the reviewer to use sub-agents for separate visual, metric, code, and log/artifact analysis before writing the integrated review.

## Output Contract

Maintain these mandatory protocol artifacts while a loop is active:

```text
loop_station/FRAME.md
loop_station/contract.json
loop_station/agent_roster.md
loop_station/summaries/ROLLING_SUMMARY.md
loop_station/summaries/SESSION_LEDGER.md
loop_station/summaries/COMPACT_HANDOFF.md
loop_station/summaries/EXECUTOR_BRIEF.md
loop_station/summaries/LOG_TREND_SUMMARY.md
loop_station/summaries/REVIEWER_ROLLUP.md
loop_station/summaries/compact/
loop_station/sessions/session_{NNN}/executor_report.md
loop_station/sessions/session_{NNN}/executor_proposal.md
loop_station/sessions/session_{NNN}/supervisor_analysis.md
loop_station/sessions/session_{NNN}/decision.md
loop_station/sessions/session_{NNN}/reviewer_requests.md
loop_station/sessions/session_{NNN}/review_flags.md
loop_station/reviews/session_{NNN}/
loop_station/flags/session_{NNN}/
```

Optional project-level artifacts may include:

```text
loop_state.json
leaderboard.csv
interventions.md
sessions/session_{NNN}/config.json
sessions/session_{NNN}/decision.md
visualizations/
FINAL_SUMMARY.md
```

If a project also has top-level `sessions/`, treat it as project output, not as
the shared protocol state. The canonical protocol decision path is
`loop_station/sessions/session_{NNN}/decision.md`.

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
- Do not close the loop just because one direction was falsified or produced a
  long bad trend while session budget remains. Retire that direction, return to
  the best known candidate, and pivot to another axis.
