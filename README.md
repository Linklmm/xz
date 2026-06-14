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

### Claude Code

```bash
/plugin install xz@<your-marketplace>
```

### 本地开发

将本项目放置到 Claude Code 可识别的插件目录中。

## 项目结构

```
xz/
├── .claude-plugin/
│   └── plugin.json          # 插件清单
├── skills/
│   └── xz/
│       └── SKILL.md         # 技能定义（核心文件）
├── scripts/
│   └── README.md            # 脚本说明
├── docs/
│   └── superpowers/
│       ├── specs/           # 设计文档
│       └── plans/           # 实现计划
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
