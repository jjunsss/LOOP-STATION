# Example: Long-Running Agent Loop

This is a compact example for running LOOP-STATION with Codex as executor and
Claude Code as reviewer. Replace the target, evidence, budget, and paths for
your own project.

## Minimal Shape

```text
Goal: what should improve or be decided
Budget: sessions, time, resources, or stop condition
Evidence: metrics, logs, images, tests, or checks that should guide decisions
Optional: reviewer role, paths, tools, ask-before rules, summaries
```

Goal, Budget, and Evidence are the useful core. Everything else is optional and
project-specific.

## Codex Executor / Supervisor

Run this first in Codex.

```text
$loop-station
Goal: subject 200014의 full-body quality를 개선하고 싶어.
Budget: 40 sessions. GPU 4개까지 사용 가능.
Evidence: PSNR / LPIPS / SSIM, rendered images, logs, failure cases를 같이 보고 판단해줘.

Context:
이전에 feet-focused loop를 돌린 적이 있으니 먼저 관련 artifacts를 확인해줘.
foot floater는 local foot 문제일 수도 있지만, full-body 품질 저하의 증상일 수 있으니 전체 품질 기준으로 판단해줘.

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

```text
/loop-station
나는 Codex가 실행 중인 LOOP-STATION 실험을 reviewer로 볼거야.
Codex는 executor/supervisor이고, Claude Code는 external reviewer야.

Loop/output root:
<loop_output_root>/loop_station/

Review rule:
Codex session이 완료되고 review-ready signal과 linked artifacts가 확인될 때까지 대기해.
정확한 flag 이름이 다르더라도 active loop의 signal pattern을 찾아서 판단해.
완료 신호만 보고 리뷰하지 말고, report / proposal / supervisor analysis / metrics / images / logs가 읽히는지 확인해.

Review focus:
- metric/log trend
- visual image check
- code/config audit when relevant
- literature or online search only when it improves review quality
- concise next-session recommendation

After review:
review artifact와 terminal reviewer signal을 남겨줘.
LOG_TREND_SUMMARY.md와 EXECUTOR_BRIEF.md를 업데이트해서 Codex가 과거 raw logs를 다시 다 읽지 않게 해줘.
다음 session monitor가 중복으로 늘어나지 않는지도 확인해줘.
```

## Notes

- Use the canonical flag names when possible, but follow equivalent signals when
  the active loop uses a different naming pattern.
- Do not start review from partial output unless the user explicitly asks for
  live partial review.
- If required artifacts are missing or unreadable, keep waiting within policy or
  write `REVIEWER-BLOCKED` with the missing pieces.
