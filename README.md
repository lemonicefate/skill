# Skill

Reusable Codex Agent Skills.

- [coding-lead](coding-lead/SKILL.md) — 用 Herdr Core `hc` 執行 Coding Lead 多代理工程統籌：Preflight、Issue DAG 派工、TDD、worktree 整合、獨立審查與子代理清理。

## Install on Windows (global Codex)

Clone this repository, then point the global Codex skill folder to the checked-out skill (PowerShell):

```powershell
New-Item -ItemType Junction -Path "$HOME\.codex\skills\coding-lead" -Target "<clone-path>\coding-lead"
```

The repo copy is the source of truth; update it with Git rather than maintaining two independent copies.
