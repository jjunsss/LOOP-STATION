# Example: Long-Running Agent Loop

This is a compact example for running LOOP-STATION with Codex as executor and
Claude Code as reviewer. Replace the target, evidence, budget, and paths for
your own project.

## Minimal Shape

```text
Goal: what should improve or be decided
Budget: sessions, time, resources, or stop condition
Evidence: metrics, logs, images, tests, or checks that should guide decisions
Optional: reviewer role, base/output paths, tools, ask-before rules, summaries
```

Goal, Budget, and Evidence are the useful core. Everything else is optional and
project-specific.

## Codex Executor / Supervisor

Run this first in Codex.

English:

```text
$loop-station
Goal:
Improve full-body quality for subject 200014.

Budget:
Use a 40-session budget. Up to 4 GPUs are available.

Evidence:
Use PSNR / LPIPS / SSIM, rendered images, logs, and failure cases.

Context:
First inspect the prior feet-focused loop artifacts.
Foot floaters may be a local issue, but they may also be a symptom of weaker full-body quality.

Base/output path:
Create the LOOP-STATION output under this experiment path so I can inspect and control the run:
<base_experiment_path>/loop_station/

Roles:
Codex is the executor/supervisor. It should run sessions and summarize results, decisions, and next directions.
Claude Code is the external reviewer. It should review only after a session is complete.

Rules:
Protect the original project. Prefer loop-owned variants or backups.
Update summaries after each session so the loop can continue after compact/resume.
```

Korean example:

```text
$loop-station
Goal:
subject 200014의 full-body quality를 개선하고 싶어.

Budget:
40 sessions 기준으로 진행하고, GPU 4개까지 사용 가능해.

Evidence:
PSNR / LPIPS / SSIM, 결과 이미지, 로그, 실패 케이스를 같이 보고 판단해줘.

Context:
이전에 feet-focused loop를 돌린 적이 있으니 먼저 관련 artifacts를 확인해줘.
foot floater는 local foot 문제일 수도 있지만, full-body 품질 저하의 증상일 수 있으니 전체 품질 기준으로 판단해줘.

Base/output path:
내가 직접 확인하고 통제할 수 있도록 아래 실험 경로에 LOOP-STATION output을 만들어줘.
<base_experiment_path>/loop_station/

Roles:
Codex는 executor/supervisor로 실험을 실행하고, 결과와 판단을 정리해줘.
Claude Code는 external reviewer로 두고, session 결과가 완료된 뒤에만 리뷰하게 해줘.

Rules:
원본 프로젝트는 보호하고, 가능한 loop-owned variants/backups에서 실험해줘.
매 session 이후 summaries를 업데이트해서 compact 후에도 이어갈 수 있게 해줘.
```

## Claude Reviewer

Start Claude after Codex has created loop artifacts, or ask Claude to hold and
monitor until the session is ready.

This is where you define the reviewer conditions: visual checks, metric checks,
code/config audit, literature or method checks, online search, or a stricter
scientific review when the project needs it.

English:

```text
/loop-station
Reviewer role:
You are the external reviewer for a running LOOP-STATION experiment.
Codex is the executor/supervisor. You are Claude Code, the external reviewer.

Loop/output root:
<loop_output_root>/loop_station/

Review rule:
Wait until the Codex session is complete and review-ready signals plus linked artifacts are confirmed.
Use the files under `loop_station/flags/` to understand whose turn it is.
Do not review from a signal alone. Confirm that report / proposal / supervisor analysis / metrics / images / logs are readable.

Review focus:
- metric/log trend
- visual image check
- code/config audit when relevant
- literature or online search only when it improves review quality
- concise next-session recommendation

After review:
Write the review artifact and terminal reviewer signal.
Update LOG_TREND_SUMMARY.md and EXECUTOR_BRIEF.md so Codex does not need to reread old raw logs.
Check that next-session monitors are not duplicated.
```

Korean example:

```text
/loop-station
Reviewer role:
너는 Codex가 실행 중인 LOOP-STATION 실험의 external reviewer야.
Codex는 executor/supervisor이고, 너는 Claude Code reviewer로 행동하면 돼.

Loop/output root:
<loop_output_root>/loop_station/

Review rule:
Codex session이 완료되고 review-ready signal과 linked artifacts가 확인될 때까지 대기해.
`loop_station/flags/` 아래 파일을 보고 지금 누구 차례인지 판단해.
완료 신호만 보고 리뷰하지 말고, report / proposal / supervisor analysis / metrics / images / logs가 읽히는지 확인해.

Review focus:
- metric/log trend
- visual image check
- 필요시 code/config audit
- 리뷰 품질에 도움이 될 때만 논문이나 online search 활용
- 다음 session 방향을 간결하게 제안

After review:
review artifact와 terminal reviewer signal을 남겨줘.
LOG_TREND_SUMMARY.md와 EXECUTOR_BRIEF.md를 업데이트해서 Codex가 과거 raw logs를 다시 다 읽지 않게 해줘.
다음 session monitor가 중복으로 늘어나지 않는지도 확인해줘.
```
