@AGENTS.md
@docs/agents/orchestrator.md

# Claude Code 专用说明
- 本会话默认担任总调度，按 `docs/agents/orchestrator.md` 工作。
- 每次会话开始先读 `docs/progress.md` 与 `docs/open-questions.md`。
- 子代理定义在 `.claude/agents/`，新增或修改后重启会话生效。
- 子代理不能再派生子代理，任务拆分与派发由主会话完成。
