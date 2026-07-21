# P6 — 微服务拆分与治理（Month 18–21）

## 目标

在领域边界已稳定、对账与分片已跑通的前提下拆服务，形成可独立发布、限流与故障隔离的体系。强调 **演进过程可讲**，而非服务数量。

## 拆分顺序（强制）

1. Product  
2. Inventory  
3. User（含鉴权协作）  
4. Trade  
5. Promotion  
6. Payment  
7. AfterSale  
8. Recommend  
9. Risk  
10. AI  

每拆一个：独立仓库或独立 module 部署单元 + **独立数据库** + Flyway + Outbox + Dockerfile + 契约测试。

## Month 18 — 底座 + 前三个服务

- [ ] Nacos 注册/配置
- [ ] Spring Cloud Gateway 路由与鉴权插件
- [ ] 拆 Product / Inventory / User
- [ ] 统一鉴权：Gateway 验 JWT，下游传用户头（防伪造用网关签名或内网）
- [ ] OpenFeign + 超时重试（只对幂等读重试）
- [ ] 本地 compose 编排多服务

## Month 19 — 交易闭环服务化

- [ ] 拆 Trade / Promotion / Payment / AfterSale
- [ ] 下单跨服务：Trade → Inventory/Promotion（同步）+ Outbox 事件
- [ ] **Seata AT 试点**（见 ADR-0007）：仅「普通下单 + 库存占用」开 `@GlobalTransactional`；部署 Seata Server + `undo_log`
- [ ] 对照实验：同一接口关闭 Seata、改走 Outbox 补偿版，记录延迟/失败率差异（写进压测或 ADR 附录）
- [ ] **禁止** 秒杀/拼团/支付回调/搜推路径使用 Seata
- [ ] 秒杀/拼团全回归
- [ ] 分布式追踪跨服务验证
- [ ] 面试故事：AT 全局锁冲突或回滚一例 + 为何大促不用 Seata

## Month 20 — 增长与 AI 服务化 + 治理

- [ ] 拆 Recommend / Risk / AI
- [ ] Sentinel：资源名按接口；规则持久化 Nacos
- [ ] 隔离演练：停止 AI、Recommend，断言下单 200
- [ ] 灰度：Gateway 按 `X-Gray` 或 userId hash 百分比
- [ ] 限流被拒的用户体验文案

## Month 21 — Kubernetes 收尾（必做，见 ADR-0008）

- [ ] kind/k3d 或云 K8s 可用；`deploy/k8s/base` + `overlays/staging`
- [ ] 每个核心服务：Deployment / Service / 资源 requests·limits
- [ ] liveness + readiness；Ingress → Gateway
- [ ] ConfigMap + Secret（支付/JWT/LLM 不进镜像）
- [ ] HPA 或手动扩缩演示（AI 或秒杀 Consumer）
- [ ] CI：build → push → apply staging → smoke test
- [ ] 滚动发布与 `kubectl rollout undo` 录屏
- [ ] 服务依赖图、同步调用红线清单
- [ ] 拆分前后对比报告（发布频率、故障域、延迟）
- [ ] ADR：最终一致性 vs 强一致；部署相关笔记（探针/回滚）

## 同步调用红线（示例）

允许同步：Trade → Inventory 占用（短超时）。  
必须异步：行为、通知、embedding、推荐计算、部分搜索索引。  

## DoD

- 核心链路在 **K8s staging** 下通过  
- 故障隔离演示录屏（含杀 Pod）  
- 契约测试进 CI；镜像构建进 CI  
- 面试可画 10 分钟拆分演进图 + 部署拓扑图
