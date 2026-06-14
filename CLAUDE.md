# xz 项目 Claude 指令

## 项目概述

这是一个 Claude Code 技能插件项目，包含"獬豸(xz)"问题排查技能。

## 关键文件

- `skills/xz/SKILL.md` — 技能核心定义文件，所有排障逻辑、MCP 编排、Phase 流程都在这里
- `.claude-plugin/plugin.json` — 插件清单
- `scripts/README.md` — 脚本说明

## 修改技能时的注意事项

1. SKILL.md 中的 Phase 0-6 是有严格顺序的排障流程，修改时注意保持流程完整性
2. MCP 安全边界（SQL 只读、Arthas 命令白名单）是关键约束，不可随意放松
3. 新增 MCP 能力时，需要同步更新：MCP 列表、命名规范表、能力期望、Phase 流程、安全边界
4. arthas 仅适用于 Java/JVM 项目，非 Java 项目不应触发 arthas 相关内容

## 代码规范

- 代码写完不要 commit，由用户审查后自行 commit
- 项目启动命令请告知用户，不要自行启动
- 调试验证完成后，自行关闭启动的项目
