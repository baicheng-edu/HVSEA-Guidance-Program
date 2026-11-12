# ADR-0001 架构与技术栈
状态：accepted　｜　日期：2026-11-10（沿用项目起点既有决策）　｜　决策人：产品负责人

## 背景
团队为 1 名产品负责人 + AI，需要在约 9 周内上线 V1.0，并在 2 月 26 日前完成增强功能。数据采集、PDF 解析、大模型抽取都以 Python 为主。

## 决策
- 前端：Vue 3 + TypeScript(strict) + Vite + Element Plus，ECharts；pnpm monorepo；后续 Vant（H5）、uni-app（小程序），共享 `packages/`。
- 后端：Python 3.12 + FastAPI + Pydantic v2 + SQLAlchemy 2 + Alembic；PostgreSQL 16；Redis；模块化单体。
- 数据管线：Python（Scrapy / Playwright、pdfplumber / PyMuPDF）。
- 契约：OpenAPI 3.1，`contracts/openapi.yaml` 为前后端唯一依据；前端类型由它生成；前端用 MSW Mock，`VITE_USE_MOCK` 切换。
- 全文检索用 PostgreSQL，暂不引入 Elasticsearch。
- 部署：Docker Compose + GitHub Actions；国内云服务器（待选，Q18）。

## 备选方案与取舍
- 前后端同为 TypeScript：与数据管线分属两种语言，维护成本高，不选。
- 微服务：1 人团队运维成本过高，先模块化单体，压力出现再拆。

## 后果
- 全项目只有 Python 与 TypeScript 两种语言。
- 更换技术栈必须得到用户明确同意（AGENTS.md 第 8 节）。
