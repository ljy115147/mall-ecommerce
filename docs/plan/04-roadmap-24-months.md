# 24 个月逐月执行路线图

> **用法**：每月初打开本月小节，勾选任务；月末做「演示 + ADR + 面试故事」三件套。  
> 详细步骤以 `phases/` 为准；本表负责任务拆解与节奏。

---

## Year 1 — 可上线交易中台 + 大促 + 搜推 AI 基础

### Month 1 — P0 脚手架与工程基座

**目标**：空业务也能体现「中台级横切能力」。

- [ ] 初始化 mono 或多仓结构（推荐多仓：frontend/admin/backend/ai/deploy/docs）
- [ ] Spring Boot 多模块骨架 + 统一响应/异常/校验
- [ ] Outbox 表 + Relay 投递 Kafka 示例
- [ ] 幂等注解 + Redis/DB 去重
- [ ] Caffeine + Redis 缓存组件骨架
- [ ] docker-compose：MySQL、Redis、Kafka(KRaft)、MinIO、Prometheus、Grafana、SkyWalking、Kafka UI
- [ ] 后端/AI 的 `Dockerfile` 骨架（多阶段，可先跑通 hello）
- [ ] XXL-JOB 接入空任务
- [ ] `mall-ai` FastAPI：`/health`、`/embeddings`（可先 mock 向量）
- [ ] 前端两个 Vite 工程空壳 + 路由鉴权骨架
- [ ] 编写 ADR：Kafka、Outbox、**部署演进 Compose→K8s（0008）**
- [ ] CI 雏形：PR 触发编译/单测（镜像推送可下月）
- [ ] 里程碑演示：发一条 Outbox 消息，Consumer 打印，Grafana 有 JVM/Kafka 指标

**验收**：见 [checklists/M01.md](./checklists/M01.md)

---

### Month 2 — P1 用户/商家/商品

- [ ] 用户注册登录、JWT、地址 CRUD
- [ ] 商家入驻（简化审核）、店铺模型
- [ ] 类目树、SPU/SKU、图片上传 MinIO
- [ ] 上下架；商品详情多级缓存
- [ ] 管理端：平台商品审核/商家商品管理基础页
- [ ] Flyway 初始 Schema；种子数据
- [ ] 越权测试：商家 A 不能改商家 B 商品

**面试点**：缓存一致性策略、商家数据隔离。

---

### Month 3 — P1 购物车/订单/库存占用

- [ ] 购物车（用户维度）
- [ ] 下单：校验、库存占用（预扣）、订单+明细
- [ ] 订单状态机 + 流转日志
- [ ] 取消订单释放占用
- [ ] Outbox 发 `mall.order.events`
- [ ] C 端订单列表/详情；管理端订单查询
- [ ] 单测：状态机非法流转、库存不足

**面试点**：预扣 vs 支付扣减；状态机为何显式建模。

---

### Month 4 — P1 支付/关单/MVP 上线

- [ ] 支付单、回调流水、验签结构
- [ ] 模拟支付 + 微信/支付宝沙箱（至少一种真沙箱或完整 Mock 可切换）
- [ ] 支付成功推进订单；幂等回调
- [ ] `pay_deadline` + XXL-JOB 扫关单 → Kafka → 释放库存
- [ ] 全应用多阶段 Dockerfile；Compose 一键起「中间件+后端+前端+AI」
- [ ] 镜像打 tag（gitSha）；推送到选定 Registry（或本地 kind 加载）
- [ ] Nginx 反代配置；staging 演示环境
- [ ] CI：build 镜像并推送（至少 backend）
- [ ] **MVP 对外演示**（录屏）
- [ ] 面试故事 #1：支付回调重复投递

**验收**：买家下单→支付→发货→完成；超时未支付自动关单。

---

### Month 5 — P2 库存中心重构

- [ ] 独立库存模型：可售/占用/已售 + 流水
- [ ] 从「商品表库存字段」迁移到库存中心
- [ ] 库存对账 Job（订单占用 vs 库存占用）
- [ ] 管理端库存流水查询
- [ ] ADR：库存中心边界

**面试点**：库存流水如何追溯一次超卖排查。

---

### Month 6 — P2 拼团完整链路

- [ ] 团/成员表；开团/参团/成团/失败状态机
- [ ] 参团幂等；成团事件 Kafka
- [ ] 超时关团 Job + 自动退款单
- [ ] C 端拼团页；分享参团（链接带 groupId）
- [ ] 失败路径压测：并发参团最后一名

**面试点**：成团临界并发；失败退款与库存回滚顺序。

---

### Month 7 — P2 秒杀全链路

- [ ] 活动配置、预热到 Redis、分桶库存
- [ ] 秒杀 API：限流 → 风控 → DECR → 排队
- [ ] Kafka `mall.seckill.commands` 异步下单消费者
- [ ] 结果查询接口；售罄处理
- [ ] 热点 key、本地缓存挡查询
- [ ] 一键预热/清理脚本

**面试点**：为何异步；如何保证不超卖；分桶原理。

---

### Month 8 — P2 风控加强 + 大促压测复盘

- [ ] 规则：频控、黑名单、SKU 限购、异常地址（简化）
- [ ] Sentinel 或自研令牌桶在网关/应用双层
- [ ] JMeter/k6 压测脚本入库
- [ ] 故障注入：Kafka 重放、消费者重复、Redis 抖动
- [ ] 输出压测报告 + 故障复盘 #2、#3
- [ ] **季度演示：大促专场**

**验收**：超卖=0；有优化前后对比数据。

---

### Month 9 — P3 Canal + ES 搜索

- [ ] Canal 订阅商品 binlog → ES
- [ ] 搜索 API：关键词、类目过滤、排序
- [ ] 下架/删除近实时从 ES 移除
- [ ] 管理端搜推配置入口（同步状态）
- [ ] 补偿：全量重建索引 Job

**面试点**：双写 vs Canal；最终一致与可见性。

---

### Month 10 — P3 Embedding 管道 + Milvus + 混合检索

- [ ] `mall-ai` 接入真实 Embedding（BGE）
- [ ] 消费 `mall.product.changed` → 写 Milvus
- [ ] 混合检索融合策略（权重可配置）
- [ ] 降级开关：向量挂了走纯 ES
- [ ] 评估集：20 条语义 Query 人工标注

**面试点**：冷启动商品向量；维度与相似度度量。

---

### Month 11 — P3 算价引擎 + 售后逆向 + 观测硬化

- [ ] 促销：满减、优惠券、拼团价；算价流水
- [ ] 下单费用明细快照
- [ ] 售后状态机：仅退款优先路径打通
- [ ] 退款单与支付单关联；库存回补策略
- [ ] Grafana 业务大盘：下单/支付/关单/Lag/搜索耗时
- [ ] **Year1 Q3 演示**

**面试点**：为何快照；券核销幂等；售后与库存。

---

### Month 12 — P4 推荐 v1（行为 + ItemCF + 热门）

- [ ] 行为埋点 SDK（前端）+ `mall.behavior.events`
- [ ] 行为仓储；小时级 ItemCF Job
- [ ] 首页猜你喜欢、商详相关推荐
- [ ] Redis 缓存推荐列表；强制兜底
- [ ] CTR 粗统计看板

**面试点**：ItemCF 公式与稀疏性；与交易解耦。

---

## Year 2 — AI 深化、容量、微服务、稳定性、面试硬化

### Month 13 — P4 向量召回 + 多路融合

- [ ] 基于近行为的向量召回通路
- [ ] 多路召回融合（热门/类目/ItemCF/向量）
- [ ] 运营置顶/黑名单
- [ ] 一次推荐故障降级演练并复盘

---

### Month 14 — P4 RAG 客服 + 商家 AI 辅助

- [ ] 知识库切片入库 Milvus `kb_emb`
- [ ] RAG Pipeline：检索 → Prompt → LLM → citations
- [ ] 管理端知识库 CRUD 与重建向量
- [ ] 商家标题/卖点生成（人工确认后保存）
- [ ] Prompt 注入与拒答策略
- [ ] **半年演示：搜推+客服 AI 专场**

**面试点**：幻觉治理；知识更新；成本与超时降级。

---

### Month 15 — P5 订单分库分表

- [ ] ShardingSphere 接入；分片键设计 ADR
- [ ] 订单号生成（雪花/号段）与分片路由
- [ ] 历史数据迁移方案（双写或停机窗口演练）
- [ ] 跨片查询限制与运营侧方案（ES 订单宽表）
- [ ] 回归：下单/支付/关单/售后全量

**面试点**：分片键选择失误会怎样；非分片键查询。

---

### Month 16 — P5 读写分离 + 三方对账

- [ ] MySQL 主从；读写分离路由
- [ ] 写后读强制主库场景清单
- [ ] 对账：渠道账单 vs 支付单；订单 vs 支付；库存 vs 订单
- [ ] 差异单工作流（管理端）
- [ ] 日终对账报告

**面试点**：账实一致；长尾差异怎么处理。

---

### Month 17 — P5 数仓雏形 + Kafka 治理 + Flink 可选

- [ ] 行为/订单同步到 ClickHouse 或 Doris（选一）
- [ ] Kafka：DLQ 管理台、重放工具、Lag 告警分级
- [ ] （可选加强）Flink 实时热度/窗口特征写入 Redis
- [ ] 容量规划文档：分区、副本、消费者水平扩展
- [ ] **Year2 中期演示：数据与资金安全**

---

### Month 18 — P6 拆分 Product / Inventory / User

- [ ] Nacos + Gateway 落地
- [ ] 拆 Product、Inventory、User 独立服务+独立库
- [ ] Feign/API 契约与错误码统一
- [ ] 契约测试；本地 docker 编排多服务

---

### Month 19 — P6 拆分 Trade / Promotion / Payment / AfterSale

- [ ] 交易相关服务拆分
- [ ] Outbox 在各服务落地；禁止跨库直连
- [ ] 分布式事务取舍 ADR（默认最终一致）
- [ ] 拼团/秒杀回归

---

### Month 20 — P6 拆分 Recommend / Risk / AI + 治理

- [ ] Recommend、Risk、AI 独立部署
- [ ] Sentinel 规则：秒杀、搜索、推荐、LLM 分开限流
- [ ] 故障隔离演练：杀 AI Pod，下单仍可用
- [ ] 配置灰度（Gateway 按 header/百分比）

---

### Month 21 — P6 收尾 + Kubernetes（必做）

- [ ] 本地/云上 K8s（kind/k3d/云厂商任选）：核心微服务 Deployment/Service
- [ ] Ingress 暴露 Gateway；ConfigMap + Secret；liveness/readiness
- [ ] HPA 或手动扩缩演示（AI / 秒杀 Consumer 至少一个）
- [ ] Kustomize overlays（sit/staging）
- [ ] CI：构建镜像 → 部署 staging → smoke test
- [ ] 滚动发布与 `rollout undo` 回滚演练录屏
- [ ] 服务依赖图与 SLO 初稿
- [ ] 拆分前后对比文档（延迟、故障域、发布节奏）
- [ ] 面试故事：同步调用链改事件驱动；容器探针踩坑一例

---

### Month 22 — P7 稳定性工程与混沌

- [ ] 全链路压测脚本平台化（场景：浏览→搜→推→下单→秒杀）
- [ ] 混沌：杀 Pod、杀 Kafka 消费、Redis 延迟、DB 慢查询注入
- [ ] K8s 资源水位与重启原因排查演练（OOMKilled / Probe 失败）
- [ ] （加码）Helm 打包或 GitOps 雏形（Argo CD/Flux 二选一文档+最小落地）
- [ ] 降级预案手册（runbook）
- [ ] 核心链路 SLO/Error Budget 文档（即使是自建指标）
- [ ] 安全加固：镜像扫描 + 越权回归集

---

### Month 23 — P8 面试材料与文档硬化

- [ ] `docs/interview` 全部填满：题库、故事、简历句
- [ ] 架构图重绘（对外版/对内版）
- [ ] 演示脚本（15 分钟 / 45 分钟两版）
- [ ] 已知问题清单与演进 Roadmap（展示判断力）
- [ ] 代码关键路径注释与 README 达到「陌生人可启动」

---

### Month 24 — P8 模拟面试与收官发布

- [ ] 至少 3 次全长模拟面试（录音复盘）
- [ ] staging 环境固化；演示数据一键重置脚本
- [ ] 版本 Tag：`v1.0-interview`
- [ ] 最终压测报告与对账报告归档
- [ ] 个人总结：最深的 5 个技术决策与若重新来过会怎么改

---

## 季度里程碑一览

| 季度 | 里程碑 |
|------|--------|
| Y1Q1 | MVP 可上线演示 |
| Y1Q2 | 拼团+秒杀压测零超卖 |
| Y1Q3 | 混合检索+算价售后 |
| Y1Q4 | 推荐 v1 |
| Y2Q1 | RAG+向量召回 |
| Y2Q2 | 分片+对账+数仓雏形 |
| Y2Q3 | 微服务拆分与隔离 |
| Y2Q4 | 稳定性+面试收官 |

## 缓冲与风险月

若某月严重延期：**优先保交易一致性与秒杀/对账**；部署侧至少保留「Compose 可演示 + 核心服务有 Dockerfile」。K8s 完整能力可压缩 Helm/GitOps，但 **M21 的 Deployment/Ingress/探针/回滚** 不建议砍。缓冲优先吞并 Flink、多模态、商家 AI 文案等。
