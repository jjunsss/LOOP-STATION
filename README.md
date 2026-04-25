<p align="center">
  <img src="./assets/LOOP-STATION.png" alt="LOOP-STATION" width="88%">
</p>

# LOOP-STATION

**A live multi-agent loop where executors and reviewers keep watching, exchange feedback, and move only on verified flags.**

LOOP-STATION helps Codex, Claude Code, and other agents stay active around the same experiment: Codex runs bounded sessions, reviewers wait for terminal flags, feedback is written to shared artifacts, and the next session starts only after the loop has evidence.

## Core Idea

LOOP-STATION is for live feedback loops, not one-off prompts.

```text
Codex runs -> Codex self-reviews -> Claude reviews -> Codex decides -> next session
```

Agents should stay in standby, monitor `loop_station/flags/`, and act only when the required `DONE`, `BLOCKED`, or `ABSTAIN` flags and linked artifacts exist.

## Install

### Codex

In Codex, ask:

```text
Install the Codex skill from https://github.com/jjunsss/LOOP-STATION as loop-station.
```

Codex will use its built-in skill installer and place the skill under your Codex skills directory. Restart Codex after installation so the new skill is loaded.

CLI fallback:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py --url https://github.com/jjunsss/LOOP-STATION --path . --name loop-station
```

### Claude Code

Claude Code can use the same skill files. Clone or copy this repository into a Claude Code skill folder.

```bash
git clone https://github.com/jjunsss/LOOP-STATION.git ~/.claude/skills/loop-station
```

For a project-local skill, place it here instead:

```text
<project>/
  .claude/
    skills/
      loop-station/
        SKILL.md
        templates/
```

Restart Claude Code after adding or updating the skill so it reloads the skill list.

### Manual Fallback

Use this only when an installer is not available.

Codex project-local fallback:

```text
<project>/
  skills/
    loop-station/
      SKILL.md
      agents/openai.yaml
      templates/
```

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
나는 이전에 feet loop를 만들어서 돌린 게 있어. 일단 이걸 확인해보고.

Goal:
200014의 사람 자체의 퀄리티를 높여야 할 것 같아. PSNR / LPIPS 같은
수치를 활용할 수 있을 것 같고, foot floater 등도 애초에 사람 퀄리티가
좋지 않아서 발생하는 것 같아. 가능하다면 매 session마다 수치적 + 시각적으로
결과를 확인해줘.

Budget:
session 40. GPU 4개 모두 사용해도 되고, 1개의 session은 모든 GPU에서
다양한 variants를 적용해서 돌리고 얻은 결과들을 모아서 분석하는 걸 기준으로 해.

Work-unit scope:
subject 200014를 중심으로 수치 지표와 렌더링 결과 이미지를 함께 비교해줘.

Collaboration:
GPT-5.5 xhigh를 활용해서 실험을 진행해도 좋고, 분석 시에는 sub-agent를
다양한 목적으로 활용하면서 생성된 코드와 결과 이미지를 분석해서 정리해도 좋아.
협력자로 CLAUDE를 활용할 건데, CLAUDE는 생성된 코드와 분석 설명을 확인해서
자신의 견해와 개선 방향을 review하는 형태로 사용할 거야.

User intervention:
코드를 수정해도 좋고, 보다 scientific하게 성능을 개선할 수 있어도 좋아.
하이퍼파라미터들을 변경해도 좋아. 다만 수치를 아주 조금씩 바꾸는 것보다,
다양한 실험을 통해 개선을 보이는 것이 중요해. 유지 중인 원본 코드를 직접
패치해야 한다면 먼저 확인해줘.
```

## How It Runs

The loop follows this rhythm:

```text
lock the frame
run a bounded session
write executor evidence and proposal
write Codex supervisor analysis
request named review when useful
wait within the review policy
consume review and write a decision
adapt, stop, or ask the user
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
나는 현재 Codex로 전신 실험을 돌리는 LOOP-STATION 실험을 하고 있어.
Codex가 어느 정도 session을 실행해서 중간 결과, metrics, 이미지, 코드 변경,
executor_report, executor_proposal을 남겨둔 상태야.

Loop/output root:
<loop_output_root>/loop_station/

Experiment context:
전신 품질 개선 실험이고, 중간 결과들의 경향성을 파악해서 의미 있는 개선 방향을
제안해주면 좋겠어. 결과가 충분히 쌓인 뒤에 review를 적어주면 되고,
잡다한 말보다 scientific하고 의미 있는 부분에 집중해줘.

Reviewer instructions:
- 먼저 loop/output root와 최근 session artifacts를 찾아.
- FRAME.md와 contract.json이 있으면 goal, budget, scope를 다시 묻지 말고 그대로 사용해.
- Codex가 아직 현재 session을 끝내지 않았다면 결과가 나올 때까지 standby로 대기해.
- 계속 대기하라는 요청이면 사용 가능한 Monitor/background watcher를 즉시 띄워서 session ready flag를 폴링해.
- `EXECUTOR-DONE`과 `SUPERVISOR-READY` flag가 생기고, 연결된 report/proposal/supervisor_analysis/metrics/images를 읽을 수 있을 때만 review를 작성해.
- review가 끝나면 Codex가 그 review를 소비해서 decision.md와 `SUPERVISOR-DONE`을 작성하는 순서야.
- 최신 executor_report.md, executor_proposal.md, supervisor_analysis.md, reviewer_requests.md, metrics,
  logs, diffs, generated code, result images, and artifacts를 읽어.
- 실험을 직접 실행하지 말고 reviewer로만 행동해.
- 코드 수정이나 다음 session 실행은 하지 마.
- 단순 요약보다 경향성, 실패 원인, 개선 가능성이 큰 방향, 다음 실험 제안을 중심으로 써.

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
