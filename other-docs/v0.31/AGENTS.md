# AGENTS.md — 湖南高职单招志愿填报辅导平台（hn-danzhao）

本文件是所有 AI 代理（Claude Code 主会话与各子代理）共同遵守的工作约定。开工前必读；与用户指令冲突时，以用户为准。
版本 V1.2（2026-11-10）。变更历史不写在本文件，统一记在 `docs/decision-log.md`。

## 1. 项目是什么
面向 2027 年湖南高职单招考生与家长的“信息 + 决策”平台：院校与专业库、荣誉与王牌专业筛选、院校对比、去年招生计划与录取情况、2027 简章首发推送与解读、报名期每日报名人数与竞争度、志愿推荐与第一/第二志愿方案、AI 对话志愿指导、会员制（历史数据免费，2027 实时报名数据付费）。
完整规划见 `docs/plan.md`；权益规则唯一依据见 `docs/entitlements.md`。

**核心判断（所有功能与文案都必须服从）**：湖南单招由各校单独命题、单独组织考试，不是全省统考，没有统一排名。能不能录取，主要看“今年各校各专业的报名人数 ÷ 招生计划数”，与去年的分数、位次关系有限。历年数据只是辅助，必须教育用户这一点。平台的差异化是 T+1 的一手报名数据，帮助用户每天根据最新数据调整志愿。

## 2. 目标与时间线
- 团队：1 名产品负责人 + AI。人工只在“关口”介入（见第 8 节）。
- V1.0（PC Web，含简章监控与订阅、权益框架）：2027-01-17 前上线。
- 增强（志愿推荐、报名热度、单校解锁与会员支付、AI 助手、微信 H5）：2027-02-26 前就绪。
- 日期参照 2026 年官方节奏（章程 1 月 25 日前统一公布；志愿填报 2 月 26 日至 3 月 4 日）。2027 年以官方公告为准，日期放配置，不写死。
- 终端顺序（已确认）：PC Web → 微信 H5 → 小程序。
- 资质（产品负责人已确认）：域名 ICP 备案已完成；微信支付与小程序主体开通中，12 月内完成。收费相关的其他资质见 `docs/open-questions.md`。

## 3. 技术栈（已定，不得擅自更换）
- 前端：Vue 3 + TypeScript(strict) + Vite + Element Plus（PC），ECharts；pnpm monorepo。后续 H5 用 Vant，小程序用 uni-app，复用 `packages/` 下的共享代码。
- 后端：Python 3.12 + FastAPI + Pydantic v2 + SQLAlchemy 2 + Alembic；PostgreSQL 16；Redis。模块化单体。
- 数据管线：Python（Scrapy / Playwright、pdfplumber / PyMuPDF）；后台抽取走 Claude API。
- 面向用户的 AI 对话：所用模型尚未确定（见 open-questions Q11、ADR-0004）。业务代码一律经 `LLMProvider` 抽象接口调用，不得写死某一厂商。
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
docs/           plan.md、entitlements.md、progress.md、open-questions.md、decision-log.md、adr/、agents/
prompts/        运行时提示词（生产使用：简章抽取、AI 助手、推荐解读）
reports/        data-qa 输出
content/        资讯、简章解读、科普文案草稿
data/           种子数据与采集产物（seed/ 放产品负责人提供的院校名单）
.claude/agents/ 子代理定义
```

## 5. 常用命令（脚手架完成后以实际为准，变更时同步更新本节）
- `pnpm install`；`pnpm --filter web dev`；`pnpm --filter web test`；`pnpm gen:api`（由契约生成类型）
- `cd apps/api && uv sync`；`uv run uvicorn app.main:app --reload`；`uv run pytest`；`uv run alembic upgrade head`
- `docker compose up -d`（PostgreSQL、Redis）

## 6. 领域规则（必须遵守）

### 6.1 数据
- **只用官方来源**：省教育考试院、各校官网与招办公告、报名系统公布的聚合人数。不使用第三方数据，不抓登录后内容，不采集考生个人信息。
- 每条事实数据必须带 `source_url`、`source_type`、`fetched_at`、`verified_status`（draft / checked / published）。缺失就标“未公布”，不得编造。
- **数据范围（已确认）**：院校名单使用 2024–2026 年的名单（手里已有，2027 年名单尚未公布，公布后以官方为准校准）；录取数据只有 2026 年，**不再补 2026 年以前的录取数据**（没有历史参考价值）；简章已采集 2024–2026。不得为“多年趋势”之类功能去找或编造更早数据。
- 年份约定：“去年”= 2026 年数据，“今年”= 2027 招生季。

### 6.2 报名人数与竞争度
- 招生类别为 A–E。志愿结构参照 2026 年：第一志愿 1 所 + 第二志愿 1 所（规则放配置，以 2027 官方公告为准）。
- 报名人数口径是“第一志愿第一专业”填报人数，按公布日快照（T+1，非实时）。采集时间窗可配置（2026 年为每天 16:00 前公布，2027 预期次日 15:00，待确认）。
- 竞争度 = 填报人数 ÷ 参考计划数。参考计划数的来源必须标注（章程 / 去年实际 / 正式计划），展示时必须带口径说明；分母缺失返回 null。分级阈值放配置（见 open-questions Q9）。
- 文案不得断言“一定能录取 / 一定考不上”，用“录取机会较大 / 极低”等表述，并带“仅供参考，以官方发布为准”。

### 6.3 权益（详见 docs/entitlements.md）
- **历史免费，今年付费**：不涉及 2027 招生季数据的功能全部免费，用来吸引注册。付费价值 = 2027 实时报名数据（报名人数、竞争度、趋势、预警）及基于它的推荐、方案与 AI 实时指导。
- 会员：可看全部学校的报名竞争度，志愿推荐与方案不限次数。
- 非会员单校解锁：68 元 / 一所学校的一个专业；288 元 / 一所学校的全部专业。价格、有效期放配置，不得硬编码。会员价格尚未确定（Q1）。
- 志愿方案：免费用户赠送 1 次体验，且允许保存（具体含义见 Q4）。
- AI 对话：付费用户每天 100 次，免费用户每天 10 次。免费用户在 2027 志愿填报阶段不提供 AI 实时志愿指导；实时热门数据不提供给免费用户的 AI。
- 所有权益由后端 Entitlement 统一判定；未解锁内容返回脱敏数据 + `locked: true`，前端不得只靠隐藏，AI 工具调用同样走 Entitlement。

### 6.4 教育用户（产品要求）
- 所有展示历史数据（去年计划、去年录取、历年简章）的页面和组件，必须带统一的“历史参考有限”提示（组件 `HistoryDataNotice`），说明：各校单独命题、无统一排名，去年数据只能辅助，要看今年报名情况。
- 志愿方案页、AI 回复在合适时机提醒“请在每日报名人数公布后复核并调整志愿”。
- 举例说明（A 校 X 专业计划 10 人、报名 1200 人；B 校 Y 专业计划 50 人、报名 49 人）只能作为“举例”标注，不得冒充真实数据。

### 6.5 志愿与推荐
- 志愿在考试前填报，没有单招成绩：推荐只给“定位 + 依据”，不承诺录取；页面必须有“仅供参考，以官方发布为准”。
- 志愿方案页必须提醒：被高职单招录取后，不能再参加本年度统一高考和对口招生考试。

### 6.6 大模型与 AI 助手
- 大模型用于“抽取 / 摘要 / 解读”和 AI 对话。事实数字（计划数、报名人数、竞争度、日期、学费）必须来自结构化库的工具返回，不得让模型补数字；模型输出不得直接入库为事实。
- AI 助手只能通过后端工具读取数据，工具内部走 Entitlement；工具返回 `locked` 时，AI 只能说明需要解锁，不得猜测或估算数值。
- AI 额度、阶段限制（填报阶段免费用户不提供实时指导）由后端判定，不能靠提示词约束。
- 大模型暂不设预算（产品负责人确认），但仍须记录调用量，且额度限制必须在后端生效。

## 7. 开发规范
- 契约先行：先改 `contracts/openapi.yaml` 并过评审，再写代码；前后端不得私自加字段。
- Mock：前端用 `packages/mock`（MSW），环境变量 `VITE_USE_MOCK` 切换；Mock 数据与契约示例一致，页面标注“演示数据”。
- 代码：TypeScript strict；Python 全量类型标注，ruff + mypy；提交前 lint 与测试通过。
- 分支与提交：`feat/<module>-<desc>`，Conventional Commits，小步提交。
- 测试：每个接口至少有成功与失败用例；权益、解锁、AI 额度、日期、竞争度计算必须有单测。
- 安全：不提交密钥，`.env` 不入库；输入校验；ORM 参数化；日志不写个人信息（AI 对话日志同样适用）。
- 不确定就写进 `docs/open-questions.md`，不要自行编造数据或规则。
- 文档只保留最新权威版本；决策或约束变化时，更新原文并在 `docs/decision-log.md` 追加一条，不在正文里堆历史。

## 8. 子代理与工作流
定义在 `.claude/agents/`。主会话是总调度（见 `docs/agents/orchestrator.md`，各阶段任务提示词见 `docs/agents/stage-prompts.md`）：子代理不能再派生子代理，也不能中途向用户提问，所以派发由主会话完成，待决策事项由子代理写在输出末尾、主会话汇总后一次性问用户。

| 子代理 | 负责 | 可写范围 |
|---|---|---|
| architect | 契约、schema、ADR、权益规则文档 | docs/、contracts/、db/ |
| frontend-dev | PC Web、Mock、共享包 | apps/web、packages/ |
| backend-dev | FastAPI 模块、迁移、单测、AI 助手后端 | apps/api、prompts/ai-advisor-system.md、prompts/recommend-explain.md |
| data-collector | 采集、PDF 抽取、报名人数任务 | pipelines/、data/、prompts/brochure-extraction.md |
| data-qa | 数据核对与抽检清单 | reports/（其余只读） |
| test-engineer | 契约测试、E2E、压测 | tests/ |
| code-reviewer | 合并前审查 | 只读 |
| devops | Docker、CI/CD、部署、监控 | infra/、.github/ |
| content-writer | 资讯、简章解读、科普文案 | content/ |

**必须停下等用户的关口**
- G1 契约冻结　- G2 入库数据抽检　- G3 会员权益与支付逻辑　- G4 上线前检查
- 另外：生产部署、删除数据、更换技术栈、选定面向用户的 AI 模型，必须得到用户明确同意。

## 9. 完成的定义
功能分支合并前：测试与 lint 通过；与契约一致；code-reviewer 无阻塞项；涉及数据的批次有 data-qa 报告；`docs/progress.md` 已更新。

## 10. 禁止事项
- 编造院校、专业、分数、计划数、报名人数。
- 使用第三方汇总数据，或抓取需要登录的内容。
- 补录或展示 2026 年以前的录取数据。
- 在前端自行判断会员身份来决定数据可见性；在提示词里“请模型别说”来代替权益判定。
- 让 AI 在没有工具数据时给出具体人数、竞争度、计划数。
- 未经用户同意做不可逆操作（删库、强推、生产部署）。
- 在同一时间让多个子代理修改同一文件。

## 11. 文档地图（以此为准，避免读到过期文档）
| 文档 | 内容 | 维护者 |
|---|---|---|
| docs/plan.md | 项目规划（最新版 V0.3） | architect |
| docs/entitlements.md | 权益、解锁、AI 额度规则（权益唯一依据） | architect |
| docs/open-questions.md | 尚未决定的问题与默认假设 | 全员追加，主会话汇总 |
| docs/decision-log.md | 决策与约束变更记录 | 主会话 |
| docs/progress.md | 每日进度与阻塞 | 主会话 |
| docs/adr/ | 架构决策记录 | architect |
| docs/agents/ | 总调度流程、阶段任务提示词 | 主会话 |
| prompts/ | 运行时提示词 | 见第 8 节 |
