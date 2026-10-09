# MCP 集成说明 (MCP_INTEGRATION.md)

## 1. 实际使用的麦当劳 MCP Server

| 项 | 内容 |
|----|------|
| 名称 | `mcd-mcp`（麦当劳中国官方 MCP） |
| 协议 / 地址 | `streamablehttp`，`https://mcp.mcd.cn` |
| 鉴权 | 请求头 `Authorization: Bearer <MCP_TOKEN>`（Token 来自麦当劳 MCP 控制台；本仓库配置文件仅用占位符，不落真实凭证） |
| 官方指南 | https://github.com/M-China/mcd-mcp-server |

> 接入方式见官方 README「WorkBuddy 开发指南」：`type=streamablehttp`、`url=https://mcp.mcd.cn`、`headers.Authorization=Bearer YOUR_MCP_TOKEN`。

## 2. 实际使用的 Tool

| Tool | 用途 |
|------|------|
| `query-lottery-info` | 拉取当前积分抽奖实时信息，解析 `drawDecision`（`resourceEligible` / `nextConsumption` / `fallbackConsumption` / `availableTimes` / `drawTypeText`） |
| `draw-lottery` | 在资源满足且用户明确确认后执行**一次**抽奖，返回 `status` / `win` / `prizes` |
| `query-my-prizes` | 查询用户历史中奖记录，用于沉淀「锦鲤档案」 |
| `query-my-account` | 查询积分余额与即将过期积分，用于次数 / 积分调度建议 |

> 边界：本 Skill **仅**使用上述抽奖相关 Tool，不越界调用无关能力。

## 3. 调用流程

```
用户意图
  │
  ├─ 仅查信息 / 看中奖记录 / 要策略建议 → 直接调用对应 Tool（不抽奖）
  │
  └─ 明确要抽奖：
       1) query-lottery-info     → 解析 drawDecision
       2) resourceEligible==false → 告知资源不足，结束（不抽）
       3) 展示 nextConsumption（及 fallbackConsumption 提示）→ 等待用户明确确认
       4) 用户确认 → draw-lottery（一次，禁连抽 / 试抽）
       5) 抽完 → 重新 query-lottery-info 展示更新后的剩余资源
       6) query-my-prizes        → 生成锦鲤档案
       7) query-my-account       → 次数 / 积分调度建议
       8) 渲染为「今日锦鲤运势」HTML 卡片输出（show_widget）
```

## 4. 业务价值

- **业务价值**：把抽奖工具自带的「展示消耗 → 确认 → 才抽」强约束固化为铁律，杜绝误扣与连抽；输出统一为 HTML 卡片（确认卡 / 战报卡 / 档案卡 / 调度卡），并叠加锦鲤档案与积分调度建议，提升积分抽奖的透明度与体验。
- **合规体验**：将抽奖接口强约束（展示消耗 → 确认 → 才抽）固化为 Skill 铁律，避免误抽 / 连抽，保护用户积分与次数资产。
- **资产优化**：结合积分账户与即将过期积分，给出「次数先用完、快过期积分先抽掉」的理性调度，提升积分利用率。
- **趣味留存**：以「锦鲤档案 / 欧气值 / 今日运势」等轻量趣味包装，提升使用频次与口碑传播。
