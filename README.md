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

## 实战案例

### 场景：订单支付接口超时

#### 用户输入

```
/xz 长乐生产环境订单支付接口超时，traceId: abc123def456，用户反馈支付失败但扣款成功
```

#### xz 诊断流程

**Phase 0: 信息补全**
- 识别 traceId: `abc123def456`
- 识别问题类型: 支付超时 + 数据不一致
- 识别环境: 长乐生产
- 识别应用: order-service, payment-service, bank-gateway

**Phase 1: 链路分析（sk MCP）**
```
调用 sk.queryTrace(
  traceId="abc123def456",
  env="长乐生产"
)

返回结果:
- 服务 A (order-service): 1200ms ✅
- 服务 B (payment-service): 5200ms ⚠️ 超时
- 服务 C (bank-gateway): 超时未响应 ❌
```

**Phase 2: 日志分析（elk MCP）**
```
调用 elk.searchLogs(
  traceId="abc123def456",
  env="长乐生产",
  timeRange="last 1h"
)

关键日志:
[14:23:15] [长乐生产] payment-service - 调用 bank-gateway 超时
[14:23:15] [长乐生产] payment-service - 重试次数: 0
[14:23:16] [长乐生产] payment-service - 抛出 TimeoutException
```

**Phase 3: 数据验证（sql MCP）**
```
调用 sql.query("
  SELECT order_id, status, amount, updated_at 
  FROM orders 
  WHERE order_id = 'ORD20240101001'
")

结果:
order_id          | status | amount | updated_at
ORD20240101001    | PAID   | 100.00 | 2024-01-01 14:23:16

结论: 订单状态已更新为 PAID，但用户侧显示未支付
```

**Phase 4: JVM 诊断（arthas MCP，Java 项目）**
```
调用 arthas.threadAnalysis()

发现:
- DubboServerHandler-thread-12: WAITING (parking)
  等待 bank-gateway 响应，已等待 5.2s
- 线程池活跃线程: 48/50，接近饱和
```

**Phase 5: 代码分析**
```
定位代码:
- 入口: PaymentController.pay()
- 调用: PaymentService.processPayment()
- 问题点: BankGatewayClient.call() 无重试机制

代码片段:
```java
// BankGatewayClient.java:45
public BankResponse call(BankRequest req) {
    // 超时配置: connectTimeout=3s, readTimeout=5s
    // 无重试机制
    return httpClient.post(url, req);
}
```
```

**Phase 6: 生成诊断报告**

#### 输出报告

```markdown
# 诊断报告

## 问题概述
- **环境**: 长乐生产
- **时间**: 2024-01-01 14:23:15
- **问题**: 订单支付接口超时，用户支付失败但扣款成功，属于数据不一致问题

## 根因分析
**直接原因**: bank-gateway 服务响应超时（5.2s），payment-service 未实现重试机制
**根本原因**: 
1. bank-gateway 服务负载过高，响应时间不稳定
2. payment-service 对第三方调用缺少重试和降级策略
3. 前端轮询机制不完善，未及时同步后端状态

## 证据链
| 证据 | 来源 | 可信度 | 说明 |
|------|------|--------|------|
| traceId 链路超时 | sk MCP | 已验证 | payment-service 耗时 5.2s |
| TimeoutException 日志 | elk MCP | 已验证 | 确认超时异常 |
| 订单状态 PAID | sql MCP | 已验证 | 后端数据已更新 |
| 线程池饱和 | arthas MCP | 已验证 | 活跃线程 48/50 |
| 无重试机制 | 代码分析 | 高概率 | BankGatewayClient 未实现重试 |

## 修复建议
### 紧急修复（P0）
1. **payment-service 添加重试机制**
   - 对 bank-gateway 调用实现 3 次重试
   - 重试间隔: 1s, 2s, 4s（指数退避）

2. **前端状态同步优化**
   - 支付后主动查询订单状态
   - 增加轮询机制，间隔 2s，最多 5 次

### 长期优化（P1）
3. **bank-gateway 限流降级**
   - 添加 Hystrix 熔断器
   - 配置降级策略，返回友好提示

4. **线程池扩容**
   - 当前: 50 线程
   - 建议: 100 线程
   - 添加监控告警

## 验证方案
1. 本地测试: 模拟 bank-gateway 超时，验证重试机制
2. 预发验证: 压测 payment-service，观察线程池表现
3. 监控告警: 配置超时告警阈值 > 3s

## 手动查询补充
- ELK: `traceId:abc123def456 AND level:ERROR`
- SQL: `SELECT * FROM payment_logs WHERE order_id='ORD20240101001'`
- Arthas: `trace PaymentService processPayment '#cost > 3000'`
```

---

### 场景：JVM CPU 飙高排查

#### 用户输入

```
/xz 马尾测试环境 order-service 的 CPU 使用率持续 90% 以上，机器 IP: 192.168.1.100
```

#### xz 诊断流程

**Phase 0: 信息补全**
- 识别环境: 马尾测试
- 识别应用: order-service
- 识别机器: 192.168.1.100
- 识别问题类型: JVM CPU 飙高

**Phase 1: JVM 诊断（arthas MCP）**
```
# 根据环境"马尾测试"和机器 IP 定位 arthas 实例
调用 arthas.threadAnalysis(
  env="马尾测试",
  host="192.168.1.100",
  app="order-service"
)

发现:
- "DubboServerHandler-thread-23" CPU: 45%
  调用栈: OrderService.calculateDiscount() -> DiscountRule.apply()
- "GC task thread" CPU: 35%
  频繁 Full GC，每 30s 一次
```

**Phase 2: 内存分析**
```
调用 arthas.heapDump(
  env="马尾测试",
  host="192.168.1.100"
)

发现:
- 老年代使用率: 92%
- 大对象: List<Order> (100,000 条记录，占用 500MB)
```

**Phase 3: 代码定位**
```
问题代码:
```java
// OrderService.java:128
public void calculateDiscount() {
    // 一次性加载所有订单到内存
    List<Order> allOrders = orderMapper.selectAll();  // ❌ 无分页
    
    for (Order order : allOrders) {
        // 复杂计算逻辑
        applyDiscountRules(order);
    }
}
```

**Phase 4: 生成报告**
```markdown
# CPU 飙高诊断报告

## 问题概述
- **环境**: 马尾测试
- **机器**: 192.168.1.100
- **应用**: order-service
- **问题**: CPU 使用率持续 90% 以上

## 根因
1. **内存泄漏**: OrderService 一次性加载 10 万条订单到内存
2. **频繁 GC**: 老年代使用率 92%，触发 Full GC
3. **CPU 消耗**: GC 线程占用 35% CPU

## 修复建议
1. 添加分页查询，每次处理 1000 条
2. 使用流式处理替代全量加载
3. 优化折扣计算算法，减少内存占用
```

## License

MIT
