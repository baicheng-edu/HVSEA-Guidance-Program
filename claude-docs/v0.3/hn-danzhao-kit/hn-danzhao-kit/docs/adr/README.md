# 架构决策记录（ADR）

每个重要且难以回退的技术决策一份文件，编号递增。状态：proposed（提议）/ accepted（已采纳）/ superseded（被取代）。
被取代的 ADR 保留原文，只在头部标注“被 ADR-xxxx 取代”。

| 编号 | 标题 | 状态 |
|---|---|---|
| 0001 | 架构与技术栈 | accepted |
| 0002 | 权益判定与单校解锁 | accepted（价格与细则见 Q1、Q2） |
| 0003 | 数据范围与来源 | accepted |
| 0004 | AI 助手：工具化、额度与模型抽象 | proposed（模型选型待用户同意，Q11） |

## 模板
```
# ADR-NNNN 标题
状态：proposed | accepted | superseded by ADR-xxxx　｜　日期：YYYY-MM-DD　｜　决策人：
## 背景
## 决策
## 备选方案与取舍
## 后果（含需要同步修改的文档与代码）
```
