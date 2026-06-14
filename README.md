# xz（獬豸）问题排查插件

基于 MCP 编排的生产/测试环境问题诊断技能。

## 概述

xz 是一个强 MCP 编排型排障技能，集成了五类证据源：

| MCP | 职责 |
|-----|------|
| `elk` | 日志查询 |
| `sk` | 链路监控 / SkyWalking / Trace 查询 |
| `sql` | 只读 SQL 查询 |
| `demo-mcp` | 代码上下文查询（OpenViki/OpenViking） |
| `arthas` | JVM 诊断（仅 Java/JVM 项目） |

## 安装

### 方式一：Claude Code 插件安装（推荐）

**从 GitHub 仓库安装：**
```bash
# 在 Claude Code 中执行
/plugin install xz@github:Linklmm/xz
```

**从本地路径安装：**
```bash
# 1. 克隆仓库到本地
git clone https://github.com/Linklmm/xz.git ~/.claude/plugins/xz

# 2. 在 Claude Code 中添加插件路径
/plugin install ~/.claude/plugins/xz
```

**通过 Marketplace 安装：**
```bash
# 添加 marketplace
/plugin marketplace add https://github.com/Linklmm/xz

# 安装插件
/plugin install xz
```

### 方式二：Codex CLI 插件安装

**从 GitHub 仓库安装：**
```bash
# 在 Codex CLI 中执行
/plugins install xz@github:Linklmm/xz
```

**从本地路径安装：**
```bash
# 1. 克隆仓库到本地
git clone https://github.com/Linklmm/xz.git ~/.codex/plugins/xz

# 2. 在 Codex 中安装
/plugins install ~/.codex/plugins/xz
```

### 方式三：手动安装

**Claude Code：**
```bash
# 克隆到 Claude Code 技能目录
git clone https://github.com/Linklmm/xz.git ~/.claude/skills/xz

# 或在项目根目录创建软链接
ln -s /path/to/xz/skills/xz .claude/skills/xz
```

**Codex CLI：**
```bash
# 克隆到 Codex 插件目录
git clone https://github.com/Linknmm/xz.git ~/.codex/skills/xz
```

### 验证安装

安装完成后，可以通过以下方式验证：

- Claude Code：输入 `/plugin` 查看已安装插件列表
- Codex：输入 `/plugins` 查看已安装插件列表
- 或直接输入 `/xz` 或"獬豸"触发技能

## 项目结构

```
xz/
├── .claude-plugin/
│   └── plugin.json          # Claude Code 插件清单
├── .codex-plugin/
│   └── plugin.json          # Codex CLI 插件清单
├── skills/
│   └── xz/
│       └── SKILL.md         # 技能定义（核心文件）
├── scripts/
│   └── README.md            # 脚本说明
├── CLAUDE.md                # 项目级 Claude 指令
├── package.json
├── LICENSE
└── README.md
```

## MCP 配置

使用 xz 技能前，需要在环境中配置以下 MCP：

```json
{
  "mcpServers": {
    "elk": { "...": "..." },
    "sk": { "...": "..." },
    "sql": { "...": "..." },
    "demo-mcp": { "...": "..." },
    "arthas": {
      "type": "streamableHttp",
      "url": "http://<host>:8563/mcp"
    }
  }
}
```

注意：`arthas` 仅适用于 Java/JVM 项目。其他 MCP 如果未配置，xz 会自动降级为手动查询建议。

## 触发方式

- 用户输入 `/xz`
- 用户提到"獬豸"
- 用户提供 traceId、报错文本、异常堆栈等排查线索
- 用户提到 CPU、内存、GC、方法慢等 JVM 相关问题（Java 项目）

## License

MIT
