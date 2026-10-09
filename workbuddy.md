# WorkBuddy 开发对话上下文 (workbuddy.md)

> 本文件记录使用 WorkBuddy 开发「麦麦锦鲤 · 积分抽奖策略官」Skill 时的对话上下文，
> 用于核验是否符合麦当劳 × WorkBuddy 联动活动奖励条件。

## 参赛与选题

本次目标是在麦当劳程序员节创意开发大赛中，基于已连接的 `mcd-mcp` 连接器，
用 WorkBuddy 的 Skill 能力做出一个可实际运行的创意作品。

`mcd-mcp` 提供了积分抽奖、点餐、优惠券、派对、营养等多个子系统。在梳理官方
README 与已有 issue 的主题分布后，最终选择聚焦「积分抽奖」子系统，原因是它的三个
工具自带一条很适合做成"守规矩助手"的强约束：必须先展示本次消耗、等用户确认、才执行
抽奖。把这条约束固化成 Skill 的铁律，既能做出有差异的体验，又天然合规。

## 创意落地过程

整个创意在 WorkBuddy 里一步步落地，主要过程如下：

1. **调研能力**：先用 ToolSearch 浏览 `mcd-mcp` 的工具清单，确认积分抽奖子系统的
   三个工具：`query-lottery-info`（查活动与消耗规则）、`draw-lottery`（执行一次抽奖）、
   `query-my-prizes`（历史中奖），并补充 `query-my-account`（积分余额与过期积分）。
2. **明确约束**：读取 `draw-lottery` 的工具说明，确认其硬约束——只能在前端向用户展示
   `query-lottery-info` 返回的 `drawDecision.nextConsumption`、且用户明确确认后才能调用，
   严禁试抽、自动抽、连续抽。
3. **编写 Skill**：按 WorkBuddy 技能规范写出 `SKILL.md`，frontmatter 含 `name` /
   `description`（含触发词与适用场景），正文为 Markdown 指令：把"先查后抽、展示消耗→
   确认才抽、资源不足不抽、失败原样展示、资源以实时返回为准"写成铁律，并设计了锦鲤
   档案、欧气值、积分调度建议与"今日锦鲤运势"卡片的呈现流程。
4. **安装技能**：按约定安装到 `~/.agents/skills/mcd-lucky-koi/`，并在
   `~/.workbuddy/skills/mcd-lucky-koi` 建软链，使技能面板可见、可被调用。
5. **补齐参赛仓库**：对照官方 README「开发 Skill」条款，补齐一套可直接提交的仓库结构
   —— `README.md`（介绍/安装/示例/目标用户）、`CONTEST_DECLARATION.md`（官方原文、
   不可改）、`MCP_INTEGRATION.md`（Server/Tool/调用流程/业务价值）、源代码
   `skills/mcd-lucky-koi/SKILL.md`，并提交到 git 保持可推送状态。

## Skill 核心设计

- **铁律优先**：任何抽奖动作前必调 `query-lottery-info`，以 `drawDecision.resourceEligible`
  判断是否可抽、以 `nextConsumption` 说明本次实际扣减；绝不在用户未确认时调用 `draw-lottery`。
- **锦鲤档案**：抽后用 `query-my-prizes` 沉淀累计中奖、最近中奖、最值钱奖品，并结合可统计
  数据算出个人"欧气值"（中奖率）。
- **调度建议**：用 `query-my-account` 拿积分余额与即将过期积分，给出"次数先用完、快过期的
  积分先抽掉"的轻量建议，不做任何中奖承诺。
- **趣味包装**：把结果整理成"今日锦鲤运势"卡片，消耗与规则信息保持准确、不误导。

## 使用的 WorkBuddy 能力

- **技能系统（Skill）**：创建并安装 Skill，写入铁律级指令。
- **连接器（Connector）**：基于已连接的 `mcd-mcp`（`https://mcp.mcd.cn`，streamablehttp）
  规划工具调用顺序与约束。
- **联网检索**：抓取官方仓库 README 与 Issues，确认参赛条款与提交要求。

## 合规性自检

- 不含真实 Token / 密钥；MCP 配置仅用 `YOUR_MCP_TOKEN` 占位符。
- 不越界调用无关工具；不保证中奖；抽奖结果以官方实时数据为准。
- `CONTEST_DECLARATION.md` 内容与官方完全一致，未改动。
