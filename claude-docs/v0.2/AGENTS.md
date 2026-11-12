# AGENTS.md — 湖南高职单招志愿填报辅导平台（hn-danzhao）

本文件是所有 AI 代理（Claude Code 主会话与各子代理）共同遵守的工作约定。开工前必读；与用户指令冲突时，以用户为准。

## 1. 项目是什么
面向 2027 年湖南高职单招考生与家长的“信息 + 决策”平台：院校与专业库、荣誉与王牌专业筛选、院校对比、去年招生计划与录取情况、2027 简章首发推送与解读、报名期每日报名人数与竞争度、志愿推荐与第一/第二志愿方案、会员制（免费用户部分功能受限，付费解锁）。
完整规划见 `docs/plan.md`。

## 2. 目标与时间线
- 团队：1 名产品负责人 + AI。人工只在“关口”介入（见第 8 节）。
- V1.0（PC Web，含简章监控与订阅、会员权益）：2027-01-17 前上线。
- 增强（志愿推荐、报名热度、微信支付、微信 H5）：2027-02-26 前就绪。
- 日期参照 2026 年官方节奏（章程 1 月 25 日前统一公布；志愿填报 2 月 26 日至 3 月 4 日）。2027 年以官方公告为准，日期放配置，不写死。
- 终端顺序：PC Web → 微信 H5 → 小程序。

## 3. 技术栈（已定，不得擅自更换）
- 前端：Vue 3 + TypeScript(strict) + Vite + Element Plus（PC），ECharts；pnpm monorepo。后续 H5 用 Vant，小程序用 uni-app，复用 `packages/` 下的共享代码。
- 后端：Python 3.12 + FastAPI + Pydantic v2 + SQLAlchemy 2 + Alembic；PostgreSQL 16；Redis。模块化单体。
- 数据管线：Python（Scrapy / Playwright、pdfplumber / PyMuPDF），大模型抽取走 Claude API。
- 契约：OpenAPI 3.1，`contracts/openapi.yaml` 是前后端唯一依据，前端类型由它生成。
- 测试：pytest、Vitest、Playwright；契约测试对照 openapi.yaml。
- 部署：Docker Compose + GitHub Actions；国内云服务器（待定）。

## 4. 仓库结构
```
apps/web        PC Web（Vue 3）
apps/api        FastAPI 后端（app/modules/<module>/）
apps/mobile     H5 / 小程序（uni-app，后期）
packages/       api-client（由契约生成）、shared（类型与业务逻辑）、mock（MSW）
pipelines/      采集、PDF 抽取、报名人数采集
contracts/      openapi.yaml
db/             schema.md（表设计说明），迁移文件在 apps/api/migrations
tests/          契约测试、E2E、压测
infra/          Docker、部署脚本
docs/           plan.md、progress.md、open-questions.md、adr/、agents/
prompts/        运行时提示词（生产使用，如简章抽取）
reports/        data-qa 输出
content/        资讯文案草稿
```

## 5. 常用命令（脚手架完成后以实际为准，变更时同步更新本节）
- `pnpm install`；`pnpm --filter web dev`；`pnpm --filter web test`；`pnpm gen:api`（由契约生成类型）
- `cd apps/api && uv sync`；`uv run uvicorn app.main:app --reload`；`uv run pytest`；`uv run alembic upgrade head`
- `docker compose up -d`（PostgreSQL、Redis）

## 6. 领域规则（必须遵守）
- **只用官方来源**：省教育考试院、各校官网与招办公告、报名系统公布的聚合人数。不使用第三方数据，不抓登录后内容，不采集考生个人信息。
- 每条事实数据必须带 `source_url`、`source_type`、`fetched_at`、`verified_status`（draft / checked / published）。缺失就标“未公布”，不得编造。
- 年份约定：“去年”= 2026 年数据，“今年”= 2027 招生季。
- 招生类别为 A–E。志愿结构参照 2026 年：第一志愿 1 所 + 第二志愿 1 所（规则放配置，以 2027 官方公告为准）。
- 报名人数口径是“第一志愿第一专业”填报人数，按公布日快照（T+1，非实时）。采集时间窗可配置（2026 年为每天 16:00 前公布，2027 预期次日 15:00，待确认）。
- 竞争度 = 填报人数 ÷ 参考计划数。参考计划数的来源必须标注（章程 / 去年实际 / 正式计划），展示时必须带口径说明；分母缺失返回 null。
- 志愿在考试前填报，没有单招成绩：推荐只给“定位 + 依据”，不承诺录取；页面必须有“仅供参考，以官方发布为准”。
- 志愿方案页必须提醒：被高职单招录取后，不能再参加本年度统一高考和对口招生考试。
- 会员权益由后端 Entitlement 统一判定；未解锁内容返回脱敏数据 + `locked: true`，前端不得只靠隐藏。
- 大模型只用于“抽取 / 摘要 / 解读”。事实数字必须来自结构化库，不得让模型补数字，模型输出不得直接入库为事实。

## 7. 开发规范
- 契约先行：先改 `contracts/openapi.yaml` 并过评审，再写代码；前后端不得私自加字段。
- Mock：前端用 `packages/mock`（MSW），环境变量 `VITE_USE_MOCK` 切换；Mock 数据与契约示例一致，页面标注“演示数据”。
- 代码：TypeScript strict；Python 全量类型标注，ruff + mypy；提交前 lint 与测试通过。
- 分支与提交：`feat/<module>-<desc>`，Conventional Commits，小步提交。
- 测试：每个接口至少有成功与失败用例；权益、日期、竞争度计算必须有单测。
- 安全：不提交密钥，`.env` 不入库；输入校验；ORM 参数化；日志不写个人信息。
- 不确定就写进 `docs/open-questions.md`，不要自行编造数据或规则。

## 8. 子代理与工作流
定义在 `.claude/agents/`。主会话是总调度（见 `docs/agents/orchestrator.md`）：子代理不能再派生子代理，也不能中途向用户提问，所以派发由主会话完成，待决策事项由子代理写在输出末尾、主会话汇总后一次性问用户。

| 子代理 | 负责 | 可写范围 |
|---|---|---|
| architect | 契约、schema、ADR | docs/、contracts/、db/ |
| frontend-dev | PC Web、Mock、共享包 | apps/web、packages/ |
| backend-dev | FastAPI 模块、迁移、单测 | apps/api |
| data-collector | 采集、PDF 抽取、报名人数任务 | pipelines/、data/ |
| data-qa | 数据核对与抽检清单 | reports/（其余只读） |
| test-engineer | 契约测试、E2E、压测 | tests/ |
| code-reviewer | 合并前审查 | 只读 |
| devops | Docker、CI/CD、部署、监控 | infra/、.github/ |
| content-writer | 资讯与简章解读文案 | content/ |

**必须停下等用户的关口**
- G1 契约冻结　- G2 入库数据抽检　- G3 会员权益与支付逻辑　- G4 上线前检查
- 另外：生产部署、删除数据、更换技术栈，必须得到用户明确同意。

## 9. 完成的定义
功能分支合并前：测试与 lint 通过；与契约一致；code-reviewer 无阻塞项；涉及数据的批次有 data-qa 报告；`docs/progress.md` 已更新。

## 10. 禁止事项
- 编造院校、专业、分数、计划数、报名人数。
- 使用第三方汇总数据，或抓取需要登录的内容。
- 在前端自行判断会员身份来决定数据可见性。
- 未经用户同意做不可逆操作（删库、强推、生产部署）。
- 在同一时间让多个子代理修改同一文件。
