## Handoff: team-plan → team-exec
- **Decided**: 先并行排查 3 条线：旧项目目录创建链路、新项目参考实现、旧项目提取码/任务上下文链路；待根因收敛后集中修改旧项目。
- **Rejected**: 未直接盲改 `storage.py`，避免只修复表象并引入规则/任务上下文回归。
- **Risks**: 旧项目可能同时存在路径规范化问题和任务上下文串用问题；若逻辑分散在调度器与转存函数之间，修复会涉及多文件联动。
- **Files**: `storage.py`, `scheduler.py`, `utils.py`, `web_app.py`, 新项目 `D:/work/kokojacket/panbox-autosave` 下的百度转存相关文件。
- **Remaining**: 收集 3 个排查结论，随后在旧项目实现修复并验证关键流程。
