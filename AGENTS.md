# Glaux planning assistant instructions

This is the planning repository, `DGIWG-P507/glaux`. Glaux Server code and its implementation issues belong to `DGIWG-P507/glaux-server`.

For Glaux Server work:

- Read the [Current state section of the follow-up action list](Docs/Plans/glaux-server/Review/action-list.md#current-state) for standing decisions and the next step, and the [Roadmap](Docs/Plans/glaux-server/glaux-server-roadmap.md) for the authorised task and controlling sources. The action list's older sections and the completed review are historical evidence, not a new work queue.
- Record each implementation issue's delivery once: in its server PR and the issue's execution record, which names the next ready issue. Do not append per-issue handoffs to the action list, rewrite Roadmap status after each issue, or open a per-issue planning PR. Change planning documents only for a project-lead decision, an approved planning change, or completion of a Roadmap capability group or phase (project-lead decision, 27 September 2026).
- Read the server's current [CONTRIBUTING.md](https://github.com/DGIWG-P507/glaux-server/blob/main/CONTRIBUTING.md) and [AGENTS.md](https://github.com/DGIWG-P507/glaux-server/blob/main/AGENTS.md) before implementation, even when the session starts in this planning checkout. Follow their separate-review and merge procedure; do not rely on conversation memory to trigger review.
- For authorised server-planning changes in this repository, also launch a separate reviewer agent/session to inspect the actual diff, relevant sources and validation evidence before publication. Record the reviewed commit and outcome in the delivery record. Changes after review need further coverage; unavailable, incomplete or blocking review prevents publication.
- **Before starting any server task, check the open [`review-gate` issues](https://github.com/DGIWG-P507/glaux-server/issues?q=is%3Aopen+label%3Areview-gate).** All fifteen gates are open until the project lead closes them.
  - Do not start a task inside an open gate's blocked range, as defined in [Roadmap §5.4](Docs/Plans/glaux-server/glaux-server-roadmap.md#54-review-gates). Skip those tasks when naming the next ready issue.
  - If no unblocked task is ready, stop and tell the project lead that the lowest-numbered open gate is next.
  - If you cannot check the gates, do not start a task.
  - Only the project lead closes a gate.
- One `proceed` authorises the stated bounded next step, not the entire queue. A documentation or setup iteration does not itself authorise the first implementation issue. Stop after the handoff.
- Preserve unrelated work and the completed review's archived evidence. Do not install software, expand scope, or silently turn recommendations into requirements.

These are persistent session instructions, not a GitHub-triggered reviewer or a mechanical merge gate. Keep decisions and delivery status in the existing documents; do not create a parallel governance system.
