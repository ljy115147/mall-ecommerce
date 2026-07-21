# 开发计划总入口（2 年详细版）

本目录是项目的 **唯一执行手册**。目标：做出可上线、接近真实线上复杂度、在拼多多社招面试中能连续深挖 45–60 分钟的「迷你电商中台」。

## 你怎么用这套文档

1. 打开 [04-roadmap-24-months.md](./04-roadmap-24-months.md)，按 **月份** 推进（主日历）。
2. 当月对应阶段，打开 `phases/Px-*.md`，按周完成 Checklist。
3. 每做一个技术决策，在 `adr/` 追加一条 ADR（面试时这就是「为什么」）。
4. 每完成一个里程碑，更新 `checklists/` 并往 `interview/` 补 1 个可讲故事。
5. 深挖某个子系统时读 `deep-dives/`，不要只记名词。

## 文档地图

| 路径 | 内容 |
|------|------|
| [01-vision-and-goals.md](./01-vision-and-goals.md) | 产品定位、面试叙事、成功标准 |
| [02-tech-stack.md](./02-tech-stack.md) | 技术栈与版本建议、中间件清单 |
| [03-architecture.md](./03-architecture.md) | 逻辑/物理架构、演进三阶段 |
| [04-roadmap-24-months.md](./04-roadmap-24-months.md) | **24 个月逐月排期** |
| [05-domain-models.md](./05-domain-models.md) | 领域、聚合、状态机、核心表 |
| [06-non-functional.md](./06-non-functional.md) | 性能、可用性、安全、合规指标 |
| [07-repo-and-engineering.md](./07-repo-and-engineering.md) | 仓库结构、规范、CI/CD |
| [08-advanced-stretch.md](./08-advanced-stretch.md) | 2 年充裕时的加码项（Flink/多模态/单元化等） |
| `phases/` | P0–P8 阶段执行手册（含周任务） |
| `deep-dives/` | Kafka/库存秒杀/拼团/检索/推荐/RAG/分片对账/微服务/SRE/**Docker·K8s 部署** |
| `interview/` | 简历话术、难点题、故事模板 |
| `adr/` | 架构决策记录（含 Kafka/Outbox/**Seata 局部试点** 等） |
| `checklists/` | 里程碑验收清单 |

## 阶段总览（复杂度拉满版）

| 阶段 | 主题 | 约略月份 |
|------|------|----------|
| P0 | 中台脚手架、Kafka、观测、AI 骨架 | M1 |
| P1 | 多商户交易 MVP 上线 | M2–M4 |
| P2 | 库存中心、拼团、秒杀、风控、压测 | M5–M8 |
| P3 | Canal/ES、Milvus 混合检索、算价、售后 | M9–M11 |
| P4 | 多路召回推荐、实时特征、RAG 客服 | M12–M14 |
| P5 | 分库分表、读写分离、对账、数仓雏形 | M15–M17 |
| P6 | 微服务拆分、治理、灰度、多活叙事 | M18–M21 |
| P7 | 稳定性工程、混沌、全链路压测平台化 | M21–M22 |
| P8 | 面试材料打磨、开源文档、演示环境硬化 | M23–M24 |

> P7/P8 在路线图里单独拆出，把「可讲的线上感」做到极致。

## 执行纪律（2 年项目仍要遵守）

- **先可演示，再加深**：每个季度至少 1 次可对外演示的版本。
- **每个组件写 ADR**：解决什么问题、代价、如何验证。
- **每个核心模块准备 1 个故障复盘**：按影响面 → 根因 → 止血 → 改进。
- **指标说话**：QPS、P99、超卖=0、Lag、CTR、对账差异率。
- **禁止空转造轮子**：复杂度必须服务于可讲述的业务问题。

## 建议的周节奏

| 日 | 活动 |
|----|------|
| 周一 | 对齐本周 Checklist，开/关任务 |
| 周二–周五 | 实现 + 单测 + 文档 |
| 周六 | 压测/联调/画图/补 ADR |
| 周日 | 写面试故事草稿 or 休息 |

开始执行：从 [phases/P0-scaffold.md](./phases/P0-scaffold.md) 与路线图 **Month 1** 同步开始。
