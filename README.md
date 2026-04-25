<p align="center">
  <img src="./assets/LOOP-STATION.png" alt="LOOP-STATION" width="88%">
</p>

# LOOP-STATION

**A reusable agent skill for running bounded improvement loops with evidence, review, and handoff.**

LOOP-STATION helps Codex, Claude Code, and other agents improve a target over multiple sessions without drifting from the original goal. It is not an optimizer by itself. It is the control layer around the work: it locks the frame, limits the loop, records evidence, coordinates review, and keeps implementation variants isolated until they are ready to promote.

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

```text
$loop-station
Goal: ...
Budget: ...
Work-unit scope: ...
Collaboration: ...
User intervention: ...
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
write evidence and proposal
request named review when useful
wait within the review policy
write a decision
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

## Claude Code Reviewer Mode

For reviewer-only use, give Claude Code this prompt:

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

## Example

See [`examples/quality-improvement-loop/`](examples/quality-improvement-loop/) for a generic quality-improvement loop frame and artifact layout.

## Notes

- LOOP-STATION is a protocol skill, not an optimizer by itself. The executor still needs project-specific commands, metrics, artifacts, and resource checks.
- Do not use it as an unbounded retry loop. A loop must have a budget and a stop or escalation path.
- Do not treat reviewer timeout as evidence that a result is good or bad. It is only a coordination event.
- Keep durable artifacts in the loop output root and keep temporary logs or generated data out of source control unless intentionally promoted.
- Add a license before publishing for external reuse.
