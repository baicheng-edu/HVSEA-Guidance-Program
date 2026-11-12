# 阶段任务提示词

把对应阶段的提示词整段粘给主会话即可。每段都假定主会话已读过 `AGENTS.md` 与 `docs/agents/orchestrator.md`。
日期参照 2026 年节奏，2027 年以官方公告为准。

---

## S0 启动（11-09 – 11-15）
```
现在进入 S0 启动阶段。目标：搭好仓库骨架，出 OpenAPI 契约 v0 和数据库 schema v0，准备 G1 评审。

请按顺序做：
1. 读 AGENTS.md、docs/plan.md、docs/entitlements.md、docs/open-questions.md、docs/adr/。
2. 派发 devops：搭 pnpm monorepo 骨架（apps/web、apps/api、packages/*、pipelines、contracts、db、tests、infra）和 docker-compose（PostgreSQL 16、Redis），补全 AGENTS.md 第 5 节的实际命令。只改 infra/、.github/ 和脚手架文件。
3. 同时派发 architect（路径不与 devops 重叠）：
   - contracts/openapi.yaml v0：覆盖模块 A–H、M、N 的主要接口；必须包含统一的 access 块（locked / lock_reason / unlock_options）、GET /me/entitlements、解锁与订单、AI 对话与额度接口；每条事实数据带 source_url、source_type、fetched_at、verified_status。
   - db/schema.md v0：实体见 docs/plan.md 第八节，含 UnlockGrant、Order/OrderItem、AiUsageDaily；不得出现 2026 年以前的录取字段或多年趋势。
   - 每个接口给出示例，供 Mock 使用。
4. 汇总两者的“待决策事项”，把阻塞项一次性问我。
5. 更新 docs/progress.md。
验收：契约与 schema 自洽、与 entitlements.md 一致；docker compose up 可起 PostgreSQL 与 Redis。到 G1 停下等我评审。
```

## S1 Demo + 数据（11-16 – 12-06）
```
G1 已通过，契约冻结为 v0.x。进入 S1：PC Demo + 数据管线首轮。

并行派发（可写范围不重叠）：
- frontend-dev：按 docs/plan.md 第五节做约 15 页 PC Demo（含 AI 助手页、科普页），packages/mock 用 MSW，VITE_USE_MOCK 切换；实现阶段切换器和身份切换器（游客/免费/单校解锁/会员）；所有历史数据页带 HistoryDataNotice；锁定态只依据契约 access 块；页面标注“演示数据”；约 10 所院校用真实公开信息做样例。
- data-collector：把 data/seed/ 里的 2024–2026 院校名单标准化入库；简章抽取先挑 5 份不同院校的做试跑（prompts/brochure-extraction.md），交 data-qa 核对、据此调整提示词后再全量抽取，输出 draft；采集 2026 录取结果，缺失标“未公布”。不得采集 2026 年以前录取数据。
- backend-dev：批次 1（认证、RBAC、建表迁移、后台骨架）。

每批数据入库后派发 data-qa 出抽检清单；到 G2 停下，把抽检清单和异常统计交给我。
```

## S2 验证（12-07 – 12-13）
```
Demo 已可演示。进入 S2：家长试用与契约冻结。
1. 让 frontend-dev 准备试用环境和观察要点：家长是否理解“单招无统一排名、要看今年报名人数”；是否理解哪些免费、哪些需解锁。
2. 我组织 5–10 位家长试用，把反馈发给你；你整理成修改清单并分派。
3. 修改清单关闭后，让 architect 出契约 v1，并列出 v0 → v1 的变更；到 G1（第二次）停下。
```

## S3 后端联调（12-14 – 01-10）
```
契约 v1 已冻结。进入 S3：批次 2–6。
逐模块推进，每个模块走完整流程：契约确认 → 建表迁移 → 接口与单测 → test-engineer 契约测试 → code-reviewer 审查 → 前端关闭该模块 Mock 开关联调。
范围：院校专业、招生数据（2026 计划与录取、简章）、筛选对比、资讯、用户与权益框架（Entitlement、兑换码、解锁数据模型）、简章监控与订阅。
特别要求：
- Entitlement 的判定矩阵、解锁、过期、阶段未开放、AI 额度跨日与并发扣次必须有单测。
- 简章监控赶在 1 月 25 日（参照）前能跑通，并有人工兜底入口。
- 微信支付商户号若已就绪且有余量，可提前评估接入，但不要挤占上线范围；先问我。
到权益框架完成时进入 G3 材料准备。
```

## S4 V1.0 上线（01-11 – 01-17）
```
进入 S4：上线 V1.0（PC Web，含简章监控与订阅，不含付费功能）。
1. data-qa 对全量数据出核对报告；test-engineer 跑全量回归与压测；code-reviewer 做安全与合规检查；devops 做灰度、备份、监控、回退开关。
2. 按 docs/agents/orchestrator.md 的 G4 清单逐项检查并汇总给我，包括 ICP 备案号展示、隐私政策与用户协议、公安联网备案计划。
3. 没有我的明确同意，不要做生产部署。到 G4 停下。
```

## S5 增强（01-18 – 02-21）
```
进入 S5：增强阶段，2 月 26 日（参照）前全部就绪。
按顺序推进：
1. 批次 7：志愿推荐与方案（以今年报名竞争度为主依据，免费体验 1 次并可保存）。
2. 批次 8：报名热度、单校解锁与会员支付、预警。用 2026 年数据（如有，见 Q17）或构造数据做回放演练，模拟 T+1 日更与高峰流量。
3. 批次 9：AI 助手。先确认 Q11（模型与备案路径）已有结论且我已同意；否则只交付工具层、额度与 Mock，功能开关保持关闭。
4. 微信 H5 适配（依赖 Q13 服务号情况；未就绪则先做移动 Web）。
到 G3 停下，提交权益与支付逻辑材料。
```

## S6 报名期运营（02-26 – 03-04，参照）
```
进入 S6：报名期运营。以官方 2027 公告为准校准所有日期配置。
每天：在配置的公布时间窗内轮询采集；落库快照；data-qa 抽检；发布；触发每日数据更新提醒；监控 AI 用量、接口延迟、权益判定错误。
采集失败：自动重试并告警；超过阈值切人工录入兜底，并在页面标注更新时间。
每天晚上向我发不超过 10 行的日报。
```

## S7 收尾（03 月起）
```
进入 S7：补录 2027 年录取结果与专业计划（官方口径，缺失标“未公布”）；启动小程序开发（uni-app，复用 packages/）；复盘转化、AI 用量、数据时效，更新 plan.md 与 decision-log.md。
```
